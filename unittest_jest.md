# Unit Testing With Jest on Gitlab CI

Jest is a JavaScript testing framework designed to ensure correctness of codebase. It is frequently used with React, but it can also be used with other JavaScript frameworks like Typescript.

First, we must ensure that there is a test folder present in the project directory that has all the test.code.ts file present.

If this is not avaliable, then create one. You can follow this awesome article [here](https://medium.com/@dimi_2011/tdd-unit-testing-typescript-project-with-jest-2557e8b84414) for a step by step process.

Using Gitlab CI we can create a job run Jest tests on our typescript code as follows:
```
unittest:
  image: node:latest
  stage: test
  before_script:
    - 'yarn global add jest'
    - 'yarn add --dev jest-junit'
    - 'yarn add --dev ts-jest @types/jest typescript'
  script:
    - 'jest --ci --reporters=default --reporters=jest-junit'
  artifacts:
    when: always
    reports:
      junit:
        - junit.xml
```
we also added an artifact report to generate a report on our gitlab pipeline UI

Jest provides a complete testing solution for JavaScript. It is fast, reliable and easy to use. It has a large community and many plugins and integrations.
