---
layout: article
tags: gitlab
title: qe-pipeline
mathjax: true
key: Linux
---

## gitlab knowledge
```
	•	Control inheritance of default keywords in jobs with inherit:default.
	•	Global defaults are not passed to downstream pipelines, which run independently of the upstream pipeline that triggered the downstream pipeline.

The include files are:
	•	Merged with those in the .gitlab-ci.yml file.
	•	Always evaluated first and then merged with the content of the .gitlab-ci.yml file, regardless of the position of the include keyword.

	•	If a job does not specify a stage, the job is assigned the test stage.

You can use some predefined CI/CD variables in workflow configuration, but not variables that are only defined when jobs start.

include:
You cannot use variables defined in jobs, or in a global variables section which defines the default variables for all jobs. Includes are evaluated before jobs, so these variables cannot be used with include.
```


## qe-pipeline
```
definitions/plan-templates定义如何提交测试：
Example using wow plugin to generate beaker jobs:
.general_security:
  variables:
    TEST_JOB_NAME: general_security                                                         # copy your test plan name
    JOB_OWNER: "bgoncalv"                                                                   # bkr job submitted as this user
                                                                                            # also used for slack notification if JOB_NOTIFY is not set)
    TEST_PACKAGE_NAME_ARCHES: "kernel-aarch64 kernel-ppc64le kernel-s390x kernel-x86_64"    # kernel variant name and architecture(s) to be used
    WOW_WHITEBOARD: "Kernel Security Regression tests ${DW_CHECKOUT}"                       # beaker whiteboard
    WOW_TASKFILE: "${KERNEL_QE_CI_INTERNAL_URL}/general/security/tasks/SECURITY"            # wow taskfile


Example of storage test plan template using Testing Farm:
---
.storage_iscsi:
  variables:
    TEST_JOB_NAME: storage_iscsi                                                            # copy your test plan name
    TFT_PLAN_URL: "https://gitlab.com/redhat/centos-stream/tests/kernel/test-plans.git"     # repository with test plan
    TFT_PLAN: "iscsi/local"                                                                 # name of the test plan
    TFT_HARDWARE: ""                                                                        # (Optional) testing farm hardware requirements (semi-colon seperated).
    TFT_PARAMS: ""                                                                          # (Optional) testing farm additional parameters (semi-colon seperated).
    TEST_PACKAGE_NAME_ARCHES: "kernel-aarch64 kernel-x86_64"                                # kernel variant name and architecture(s) to be used
    JOB_NOTIFY: "bgoncalv"                                                                  # user to be notified on slack channel


definitions/rhel/xx/xx/plan-jobs add new job:
---
include:
  - definitions/plan-templates/rts/rt.yml                                       # include your team's test plan template

rt_tier2_ystream:                                                               # from your team's test template, select the test plan to run
  extends: [.9_5-bkr-job-submitter-runner, .rt_tier2_ystream, .kernel_matrix]   # extend your test plan, runner to submit jobs, and test matrix


---
include:
  - definitions/plan-templates/core/general.yml
  - definitions/rhel9/9.7/common.yml

general_security:
  extends: [.release-bkrwow-runner, .general_security, .kernel_matrix, .overrides]

general_perf:
  extends: [.release-bkr-job-submitter-runner, .general_perf, .kernel_matrix, .overrides]



.release-bkrwow-runner:
  extends: [.trigger-bkrwow-runner]
  variables:
    TEST_COMPOSE: "RHEL-9.6.0-%"
    TEST_COMPOSE_TAGS: "CTS_NIGHTLY CTS_integration CTS_beaker-available"


# runners/trigger-jobs.yml 
.trigger:
  trigger:
    include:
      - file: $QE_PIPELINE_DEFINITION_RUNNER_FILENAME
        project: redhat/centos-stream/tests/kernel/qe-pipeline-definition
        ref: $QE_PIPELINE_DEFINITION_REF
    strategy: depend
  rules:
    - if: $TEST_PACKAGE_NAME_ARCHES =~ $TEST_PACKAGE_NAME_ARCH

.trigger-tft-runner:
  extends: [.trigger]
  variables: {QE_PIPELINE_DEFINITION_RUNNER_FILENAME: runners/runner-tft.yml}

.trigger-bkrwow-runner:
  extends: [.trigger]
  variables: {QE_PIPELINE_DEFINITION_RUNNER_FILENAME: runners/runner-bkrwow.yml}



liali@liali-mac16 runners % cat runner-tft.yml 
---
include:
  - runners/common.yml

variables:
  TFT_API_URL: "https://api.dev.testing-farm.io/v0.1"

tft-submitter:
  extends: [.submitter]
  artifacts:
    reports:
      dotenv: test_variables.env
  before_script:
    - echo "${UMB_CERTIFICATE}" > /tmp/umb_certificate.pem
  script:
    - scripts/tft_submitter.sh
  needs:
    - {pipeline: $PARENT_PIPELINE_ID, job: git-repo}

liali@liali-mac16 qe-pipeline-definition % cat ./runners/templates-general.yml
---
.submitter:
  tags: [kernel-qe-pipeline-test-runner]
  image: quay.io/cki/kernel-qe-tools:production
  stage: submitter





1. qe-pipeline gitlab-ci.yml:
integration-test:
  stage: 🐛
  variables:
    QE_PIPELINE_DEFINITION_PIPELINE_FILENAME: definitions/integration-tests.yml
    QE_PIPELINE_DEFINITION_REF: $CI_COMMIT_SHA
  trigger:
    project: redhat/red-hat-ci-tools/kernel/qe-pipelines/kernel-ci-pipelines
    branch: production
    strategy: depend
  rules:
    - if: $CI_MERGE_REQUEST_ID

2. redhat/red-hat-ci-tools/kernel/qe-pipelines/kernel-ci-pipelines
---
include:
  - project: redhat/centos-stream/tests/kernel/qe-pipeline-definition
    ref: production
    file: internal/main.yml
    rules: [{if: $QE_PIPELINE_DEFINITION_REF == null}]
  - project: redhat/centos-stream/tests/kernel/qe-pipeline-definition
    ref: $QE_PIPELINE_DEFINITION_REF
    file: internal/main.yml
    rules: [{if: $QE_PIPELINE_DEFINITION_REF}]

3. qe-pipeline internal/main.yml
  # trigger job definitions
  - {local: runners/trigger-jobs.yml}
  # general templates
  - {local: runners/templates-general.yml}
  # main parent pipeline entrypoint
  - {local: $QE_PIPELINE_DEFINITION_PIPELINE_FILENAME, rules: [{if: $QE_PIPELINE_DEFINITION_PIPELINE_FILENAME}]}

3.1 runners/trigger-jobs.yml: 看起来都是和beaker job相关的工作
.trigger:
  trigger:
    include:
      - file: $QE_PIPELINE_DEFINITION_RUNNER_FILENAME
        project: redhat/centos-stream/tests/kernel/qe-pipeline-definition
        ref: $QE_PIPELINE_DEFINITION_REF
    strategy: depend
  rules:
    - if: $TEST_PACKAGE_NAME_ARCHES =~ $TEST_PACKAGE_NAME_ARCH

.trigger-bkrwow-runner:
  extends: [.trigger]
  variables: {QE_PIPELINE_DEFINITION_RUNNER_FILENAME: runners/runner-bkrwow.yml}

# runners/runner-bkrwow.yml
include:
  - runners/common.yml

bkrwow-submitter:
  extends: [.bkr-legacy-submitter]
  script:
    - scripts/bkrwow_submitter.sh
  needs:
    - {pipeline: $PARENT_PIPELINE_ID, job: git-repo}

# ./runners/templates-bkr.yml:.bkr-legacy-submitter:
.bkr-submitter:
  extends: [.submitter]
  artifacts:
    reports:
      dotenv: test_variables.env
    when: always
    paths:
      - beaker.xml
  before_script:
    - echo "${BEAKER_KEYTAB}" | base64 --decode > /etc/krb5.keytab
    - echo "${BEAKER_CLIENT_CONFIG}" > /etc/beaker/client.conf
    - echo "${UMB_CERTIFICATE}" > /tmp/umb_certificate.pem

.bkr-legacy-submitter:
  extends: [.bkr-submitter]
  image: quay.io/cki/kernel-qe-tools-legacy:production

# ./runners/templates-general.yml:.submitter:
.submitter:
  tags: [kernel-qe-pipeline-test-runner]
  image: quay.io/cki/kernel-qe-tools:production
  stage: submitter


3.2 runners/templates-general.yml

3.3 definitions/integration-tests.yml 这个是测试qe-pipeline自身的，不是测试kernel更新的
```

## temporary cfg
```
 38 integration-test:
 39   stage: 🐛
 40   variables:
 41     # QE_PIPELINE_DEFINITION_PIPELINE_FILENAME: definitions/integration-tests.yml
 42     QE_PIPELINE_DEFINITION_PIPELINE_FILENAME: definitions/rhel10/10.1/schedules/userspace-candidate.yml
 43     QE_PIPELINE_DEFINITION_REF: $CI_COMMIT_SHA
 44     _DRY_RUN: 1
 45     _TEST_SOURCE_PACKAGE_NVR: synce4l-1.1.0-5.el10
```
