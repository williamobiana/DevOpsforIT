# Golang unit-test report with Gitlab CI

GitLab provides a report on the merge request, and you may configure your task to use unit test reports, making it simpler and faster to find the failure without having to look through the full log. 

## How to set it up
 
You must include artifacts:reports:junit and specify the paths to the generated test reports in.gitlab-ci.yml in order to allow Unit test reports in merge requests. If the reports are not.xml files, GitLab will return an error code 500. 

## Go unit-test report example
The job in the test stage of the following Go example runs, and GitLab collects the job's unit test report. The results are displayed in the merge request widget after the job has finished running, and the XML report is saved in GitLab as an artifact. 

```
golang:
  stage: test
  script:
    - go mod tidy
    - go get gotest.tools/gotestsum
    - gotestsum --junitfile report.xml --format testname
  artifacts:
    when: always
    reports:
      junit: report.xml
```

Include the output files from the Unit test reports with the artifacts:paths keyword as well, as seen in the example, to make them browsable. Use the artifacts to upload the report even if the job fails (for instance, if the tests are unsuccessful) use the keyword artifact:when:always. 

You cannot have multiple tests with the same name and class in your JUnit report format XML file.

## Troubleshooting

### No matching files. Ensure the artifact path is relative to the working directory

A test can show this error, but doesn't mean it job failed.

* If the job was successful, check if you are in the working directory where you dependencies are stored.

* If the job was unsuccessful, check if the you ran the right commands. You can test your flow on your local machine to see it results and replicate on CI if successful.
