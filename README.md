apply them.
gitlab-ci
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
    name: registry.gitlab.itsm-suite.service.itzbund.net/oase/container-deployment/golang:1.24.2-bullseye
  script: echo "Image Done."
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
    - go version  
    - go install
    - go install gotest.tools/gotestsum@v1.12.2
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

build-podman-image:
  stage: container
  variables:
    REGISTRY: "registry.gitlab.itsm-suite.service.itzbund.net:443/oase/tp-registry"
  script:
    - echo "Building podman image for version ${CI_PIPELINE_ID}"
    - echo "$CI_REGISTRY_PASSWORD" | podman login --verbose "$CI_REGISTRY" -u "$CI_REGISTRY_USER" --password-stdin
    - podman build --dns=7.7.7.7 --dns=7.7.7.8 --build-arg HTTP_PROXY="http://10.130.165.20:3128" --build-arg HTTPS_PROXY="http://10.130.165.20:3128" -t "${REGISTRY}:${CI_PIPELINE_ID}" -t "${REGISTRY}:latest" .
    - podman push "${REGISTRY}:${CI_PIPELINE_ID}"
    - podman push "${REGISTRY}:latest"
  needs:
    - build-job
  tags:
    - bash
    - env:dev
development:
  stage: deploy
  tags:
    - bash
    - env:dev
  script:
    - systemctl --user daemon-reload
    - systemctl --user restart container-tp-registry
    # - curl --request POST --form ref=dev --form token=$CI_JOB_TOKEN --form variables[COMPONENT_VERSION]=$CI_PIPELINE_ID \
    #         --form variables[COMPONENT_NAME]=$CI_PROJECT_NAME https://gitlab.itsm-suite.service.itzbund.net/api/v4/projects/186/trigger/pipeline


--------------------------------------------------------------------------------
makefile
 PROJECT_NAME := "tp-registry"

.PHONY: all dep check build test race 

all: check test race build

check:
	@revive -formatter unix ./...

test:
	@go tool cover --func=cover.out|grep total                           
	@go test -v -coverprofile=cover.out --coverpkg=$(go list .)/... ./...

race: dep
	CGO_ENABLED=1 go test -race -short ./...

dep: 
	@go get -v -d ./...

build:
	CGO_ENABLED=1 go build -o build/ ./...
	
clean:
	@rm -fr build
