# Structure your pipeline and choose when to run jobs

To configure a job when structuring pipelines, you can use these keywords `include`, `extend`,`rules` and `trigger` where necessary. 

## Include
We can include other .gitlab-ci.yml files. This is useful because, as your project configuration grows, because you can split it into multiple files and include them strategically.

There are 4 ways to use include.
* local
* file
* remote
* template

### local:
you can include a file from the local repository where you will run the jobs, it must be on the same branch.

You can put the logic of the jobs in separate files of the same folder.
```
stages:
  - build
  - test
  - deploy

include:
  - local: configs/build.yaml
  - local: configs/test.yaml
  - local: configs/deploy.yaml
```

### file:
You can get access to private projects on the same gitlab instance 

A gitlab instance is the URL you receive after creating a new project (git@[instance URL]/project_name)

This is useful when the projects and setup are similar, hence we can use 1 template file for all projects.
```
include:
  - project: devops/rabbitMQ
  - file: tmplt/microservices.yaml
  - ref: master #from master branch
```
```
include:
  - project: devops/redis
  - file: tmplt/microservices.yaml
  - ref: master #from master branch
```

Note: because these keywords are evaluated during pipeline creation, if you make changes to the files you have to create a new pipeline to see the changes.

### remote
You can include a remote file that must be accessed by gitlab instance
```
include:
  - remote: http://website.com/file.yaml
```

### template
You can include templates from gitlab
```
include:
  - template: Jobs/Code-Quality.gitlab-ci.yml
```

## Extends
you can inherit actions from other jobs, especially hidden jobs indicated with `.example_job`
```
.deploy
  script: ./deploy_file.sh

deploy_development:
  extends: .deploy

deploy_production:
  extends: .deploy
  when: manual
```
we have 1 hidden job `.deploy` and 2 regular jobs `deploy_development` and `deploy_production`.

both regular jobs inherit their actions from the hidden job with an added action on the `deploy_production` job to run manually.

the logic will be similar to this:
```
deploy_development:
  extends: ../deploy_file.sh

deploy_production:
  extends: ./deploy_file.sh
  when: manual
```
Note: we can merge `hashes` but not `arrays` because hashes are represented with [key: value] pairs and arrays are only values.

Thus, arrays will be overwritten with the most recent scope.

Example of a hashed job:
```
job1:
  variables:
    MY_VAR: 'Hello'

job2:
  extends: job1
  variables:
    MY_SECOND_VAR: 'World'
```

the logic will be similar to this:
```
job1:
  variables:
    MY_VAR: 'Hello'

job2:
  variables:
    MY_VAR: 'Hello'
    MY_SECOND_VAR: 'World'
```

Example of an array job:
```
job1:
  script:
    - echo 'Hello'
    - echo 'World'

job2:
  extends: job1
  script:
    - echo 'This is the second job'
```

the logic will be similar to this:
```
job1:
  script:
    - echo 'Hello'
    - echo 'World'

job2:
  script:
    - echo 'This is the second job'
```

job2 was overwrote job1, a potential alternative would be to use a before script or after script for this.
```
job1:
  before_script:
    - echo 'Hello'
    - echo 'World'

job2:
  extends: job1
  script:
    - echo 'This is the second job'
```

the logic will be similar to this:
```
job1:
  before_script:
    - echo 'Hello'
    - echo 'World'

job2:
  before_script:
    - echo 'Hello'
    - echo 'World'
  script:
    - echo 'This is the second job'
```

## Rules
You can include or exclude jobs in pipelines,
it replaces `only/except` keywords.

`rules` and cannot be used together with `only/except` in the same job.

using rules, you can provide 2 attributes
* when
* allow_failure

rules are evaluated until the first match.

if matched, the job is included or excluded from the pipeline.
```
build:
  rules:
    - if: $CI_COMMIT_REF_NAME == 'master'
      when: always
    - when: manual
      allow_failure: true
```
on this job, we have 2 rules.
* if the pipeline is pushed on master branch, the first rule is evaluated as true and the job runs.
    * The keyword when: always indicates that we always run this job on master.
    * if we push on a develop branch, the first rule is evaluated as false and the is skipped to the next rule.
* if the job is manual, it should run.
    * if it fails, the pipeline should still continue.

Within the rules, we can use the clauses:
* if
* changes
* exists

### if
```
example_job:
  rules:
  - if: $CI_COMMIT_BRANCH == 'master'
    when: always
  - when: never
  script:
    - echo 'Hello'
```
the job evaluates the rules using if statement,
if the condition is not met, we indicated `when: never` thus, the job will never run.

### changes
```
example_job:
  rules:
  - changes:
    - application/*
  script:
    - echo 'Hi'
```
the job runs only if something has been changed in application folder.

### exists
```
example_job:
  script:
    - docker build -t my-image:$CI_COMMIT_REF_SLUG .
  rules:
  - exists:
    - Dockerfile
```
the job builds an image only if a Dockerfile exists

We can combine rule clauses as well.
```
example_job:
  rules:
  - if: $CI_PIPELINE_SOURCE == 'merge_request_event'
    changes:
      - application/*
  script:
    - echo 'Hello World'
```
the job evaluates if a pipeline is a merge request, if something has been changed in the application folder then run the job.

## Trigger
You can define a downstream pipeline trigger

A downstream pipeline is a pipeline that is triggered by another pipeline

There are 2 types of downstream pipelines:
* multi-project pipelines (pipeline in another project)
* child pipelines (sub-pipeline in the same project)

### multi-project pipeline
```
trigger-job:
  trigger:
  project: my_group/my_project  #path to the project
  branch: master (optional)
  strategy: depend (optional)
```
by default, the trigger-job does not wait for the its dependent pipeline to complete, if we want it to wait, we must specify `strategy: depend`

### child pipeline
```
trigger-job:
  trigger:
  include: my_group/my_project  #path to the project
  strategy: depend (optional)
```
we indicate the trigger job to use files from pipelines the same project or another project











