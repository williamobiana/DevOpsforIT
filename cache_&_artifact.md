# Cache and Artifacts explained

Nobody wants to wait for the CI completion build to take a long time.
If it takes more than 10 mins, it is too long.

Cache and artifacts help reduce the time it takes to run a Pipeline.

The Pipeline is a set of stages and each stage can have one or more jobs.
Jobs work on runners.

When we start a Pipeline, a runner with free resources executes the needed job.
For simplicity, let's consider Docker as an executor for all runners.

Each job starts with a clean slate and doesn't know the results of the previous one.
If you don't use cache and artifacts, 
the runner will have to go find and download the necessary packages when installing project dependencies.


## What is cache?
A set of saved files that a job can download before running and upload after execution.
By default, the cache is stored in the same place where GitLab Runner is installed.

example: 
When you run a Pipeline for the first time with a local cache. 
The job will not find the cache but will upload one after the execution.

When we run a second job on the same runner, the job will find the cache and use it. 
After execution, it will upload the new cache.


## What are artifacts? 
Unlike cache which is saved on the runner,
Artifacts are files stored on the GitLab server after a job is executed

example:
a job creates an artifact and saves it on the Gitlab server.
When we run a second job (on the same or diffrent runner), 
the job downloads the artifact from the Gitlab server before running the commands


## Main diffrence between artifact and cache
To compare,
The artifact is created in the first job and is used in the following ones. 
The cache is created on each each job (each job creates a cache).

## Example of a cache and artifact in play
For the sake of this example, we will create a ci build for a node.js application that showcases how cache and artifact can be used. we will slipt it into 3 phases

### phase 1
```
image: node:latest # (1)

stage:
  - test 

cache:
  paths:
    - node_modules/ # (2) This folder is cached between builds

test_async:
  stage: test 
  script:
    - npm install # (3)
    - node ./specs/start.js ./specs/async.spec.js

test_db:
  stage: test 
  script:
    - npm install # (4)
    - node ./specs/start.js ./specs/db-postgres.spec.js
```

Always specifying the full version of the image (1), incase of updates (node:16.3.0)

The 'node_modules' (2) directory is specified for caching and contains package for npm.

The first job (3) will find nothing, but will upload a cache when complete, the second job (4) will find the cache of the first job in the 'node modules' directory and will use it.

Since no key is specified for the cache, the word default will be used as a key.

It means that the cache will be permanent, shared between all git branches.

### phase 2
```
image: node:16.3.0 # (1)

stage:
  - test 

variables:
  npm_config_cache: "$CI_PROJECT_DIR/.npm" (6)

cache:
  key:
    files:
      - package-lock.json # (5) This folder is cached between builds
  paths:
    - .npm # (2)
  policy: pull 

test_async:
  stage: test 
  script:
    - npm ci # (3)
    - node ./specs/start.js ./specs/async.spec.js

test_db:
  stage: test 
  script:
    - npm ci # (4)
    - node ./specs/start.js ./specs/db-postgres.spec.js
```

npm stores dependencies in two files 'package.json' and 'package-lock.json' the fixed dependencies are stored on 'package-lock.json'(5).

to install the fixed dependencies from 'package-lock.json', we need to use npm ci, and specfy the file to be cached.

using a policy: pull, we prevent and update to the package-lock.json file.

finally, npm ci will delete the 'node-module' in the .npm folder, so we will make it a variable (6) to prevent this. 

### phase 3
```
image: node: 16.3.0 

stages:
  - setup
  - test

variables:
  npm_config_cache: "$CI_PROJECT_DIR/.npm" 

# Define a hidden job to be used with extends
# Better than default to avoid activating cache for all jobs
.dependencies_cache:
  cache: # (1)
    key:
      files:
        - package-lock.json
    paths:
      - .npm
    policy: pull

setup:
  stage: setup
  script:
    - npm ci
  extends: .dependencies_cache
  cache:  #we tell the setup phase to pull and push updates
    policy: pull-push
  artifacts: # (2)
    expire_in: 1h
    paths:
      - node_modules

test_async:
  stage: test
  script:
    - node ./specs/start.js ./specs/async.spec.js

test_db:
  stage: test
  script:
    - node ./specs/start.js ./specs/db-postgres.spec.js
```

next we make the caching process a job, that can be used in any stage we want (1).

To prevent every job from installing the packages from the 'node-module' again, we can make it an artifiact (2).

we indicated thay the setup stage policy should always update, pull-push.

we also indicated that the artifacts should remain on the Gitlab server for only 1hr and be deleted,
since we already have our cache for the jobs on the runner. 

we can run the job using the artifact and delete the npm ci command.

## Summary:

* image is imported from docker 
* we define our stages (setup to install dependencies, test to run the jobs)
* create a variable to ensure our npm files and variables are intact
* create a job for importing dependencies files and creating a cache for it.(dependencies_cache)
* run the setup for installing the dependencies_cache, update the cache, create an artifact of the installed dependencies, and save in a created node_modules folder.
* run the test jobs, (it will use the already installed dependencies in the artifact in the node_modules folder)

