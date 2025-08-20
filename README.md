CV 

stages: 
  - check
  - test
  - build
  - container
  - deploy

variables:
  COMPONENT_NAME: "$CI_PROJECT_NAME"

workflow:
  rules:
    - if: $CI_MERGE_REQUEST_IID
    - if: $CI_COMMIT_TAG
    - if: $CI_PIPELINE_SOURCE == "schedule"
    - if: $CI_COMMIT_REF_PROTECTED == "true"
    - if: $CI_JOB_MANUAL
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'

.job_rules: &job_rules
  - if: $CI_COMMIT_BRANCH == "develop"
  - if: $CI_COMMIT_BRANCH == "main"
  - if: $CI_COMMIT_BRANCH =~ "/^release/"
  - if: $CI_COMMIT_BRANCH =~ "/^hotfix/"
  - if: $CI_COMMIT_BRANCH =~ "/^feature/"

check:
  stage: check 
  script:
    - go install github.com/mgechev/revive@v1.6.0
    - export PATH="${PATH}:$(go env GOPATH)/bin"
    - make check
  tags:
    - golang
    - env:dev
test:
  stage: test
  image: 
    name: <image?>
  script: make testacc
  tags:
    - golang
    - env:dev

build-job:
  stage: build
  tags:
    - golang
    - env:dev

  script:
    - apt-get update 
    - apt-get install -y zip
    - export RELEASE_VERSION="${CI_PIPELINE_ID}"
    - echo "Building version ${RELEASE_VERSION}"
    - make release VERSION=${RELEASE_VERSION}
    - cp sign_files.sh bin/release/${CI_PIPELINE_ID}/
    - gpg --import $key
    - gpg --list-secret-keys
    - cd bin/release/${CI_PIPELINE_ID}/ && gpg --armor --export ci/cd > key.gpg
    - sh sign_files.sh && rm sign_files.sh
    - go version  
    - go install
    - go install gotest.tools/gotestsum@latest  
    - go install github.com/t-yuki/gocover-cobertura@latest
    - gotestsum --junitfile report.xml --format testname -- -race -coverprofile=coverage.txt -covermode atomic ./...
    - gocover-cobertura < coverage.txt > coverage.xml
    - go tool cover -func coverage.txt
  coverage: '/total:.*\d+.\d+%/'
  artifacts:
    when: always
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage.xml
      junit: report.xml
  cache:
    key: ${CI_COMMIT_REF_SLUG}
    paths:
      - /go/pkg/mod/
      - /go/bin/


Running with gitlab-runner 18.0.1 (3e653c4e)
  on golang_runner t3_G9ZRQP, system ID: s_5cd514046c5e
  feature flags: FF_NETWORK_PER_BUILD:true
Preparing the "docker" executor
00:02
Using Docker executor with image  ...
Using effective pull policy of [always] for container 
Pulling docker image  ...
WARNING: Failed to pull image with policy "always": Error response from daemon: Head "https://registry.gitlab.itsm-suite.service.itzbund.net/v2/oase/container-deployment/golang/manifests/1.24.2-bullseye": denied: access forbidden (manager.go:238:0s)
ERROR: Job failed: failed to pull image "image" with specified policies [always]: Error response from daemon: Head "https://registry.gitlab.itsm-suite.service.itzbund.net/v2/oase/container-deployment/golang/manifests/1.24.2-bullseye": denied: access forbidden (manager.go:238:0s)
