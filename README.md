# Difference between different GitLab CI Merge Request rules

The `merge_request_event-rule`

Using this rule:

```
if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
```

 > will create a pipeline each time a Merge-Request is created/updated, but there will also be a pipeline created for the branch if you do not have another prevention mechanism (rule). As the variable naming also points out, this is about the source of the pipeline trigger, other sources could be schedule, push, trigger etc.


The `CI_OPEN_MERGE_REQUESTS` variable:

Using a rule like:

```
if: '$CI_OPEN_MERGE_REQUESTS'
```
> GitLab will create new pipelines if there is an open Merge-Request for this branch. Pipelines because there will be a Merge-Request pipeline (denoted with the detached flag) and a branch pipeline for the branch you pushed changes.

```
if: '$CI_COMMIT_BRANCH && $CI_OPEN_MERGE_REQUESTS'
```
> This rule above will create a pipeline for your branch when, and only when there is an open MR on that branch.

```
if: '$CI_COMMIT_BRANCH && $CI_OPEN_MERGE_REQUESTS'
when: never
```

> When using the above combination no pipeline will be created if there are open Merge-Requests on that branch, which might also be undesirable since the CI should run tests for branches and/or Merge-Requests.


# Gitlab CI AND operator in rules


You should just combine both conditions into one mapping. I.e., remove extra dash before changes:
```
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event" && $CI_MERGE_REQUEST_TARGET_BRANCH_NAME == "main"'
      changes:
        - filder1/*.xml
```
But please also take into account that default action is on_success, so you should add another mapping with never to prevent the job from adding:
```
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event" && $CI_MERGE_REQUEST_TARGET_BRANCH_NAME == "main"'
      changes:
        - filder1/*.xml
      when: on_success
    - when: never
```

# Duplicate pipeline
when you using `CI_COMMIT_REF_NAME` and have merge request it will create twice pipeline
```
- if: $CI_COMMIT_REF_NAME == "development"
```
Instead, using `CI_COMMIT_BRANCH` or `CI_COMMIT_TAG`


# Build manual on specific branch
```
.test_rules:
    rules:
        - if: $CI_COMMIT_BRANCH == "dev"
          variables:
              ENV: dev
        - if: $CI_COMMIT_BRANCH == "prod"
          variables:
              ENV: prod
          when: manual
build:
    stage: build
    extends: .test_rules
    script:
      - echo $ENV
stages:
    - build
```
# Schedule pipelines
Example:
```
.my-job-template:
  script:
    - echo "Hello from template"

Manual MyJob:
  extends: .my-job-template
  when: manual
  except:
    - schedules

Scheduled MyJob:
  extends: .my-job-template
  only:
    - schedules
```

# Run jobs only when files in a specific directory have changed
example:
```
docker build:
  script: docker build -t my-image:$CI_COMMIT_REF_SLUG .
  only:
    changes:
      - Dockerfile
      - docker/scripts/*
      - dockerfiles/**/*
      - more_scripts/*.{rb,py,sh}
```
or
```
rules:
    - changes:
      - package.json
      - yarn.lock
    - exists: 
      - node_modules
      when: never
    - when: always
```