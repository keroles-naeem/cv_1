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

Below is a complete solution to automate **provider artifact versioning, naming, and upload** for a Terraform provider in your GitLab CI/CD, integrated with your `Makefile` and pipeline stages. This ensures every run produces the correct SemVer-formatted ZIPs (e.g. `terraform-provider-oase_3924.0.0_linux_amd64.zip`) and reliably uploads them for registry use.

***

## 1. Add Provider Build & Release Steps to GitLab CI

Add these jobs after your existing build jobs in your `.gitlab-ci.yml`:

```yaml
build-provider:
  stage: build
  image: golang:1.22
  before_script:
    - apt-get update && apt-get install -y zip
  script:
    - export PROVIDER_VERSION="${CI_PIPELINE_ID}.0.0"
    - echo "Building provider version $PROVIDER_VERSION"
    - mkdir -p bin/release/$PROVIDER_VERSION
    # Build for each target platform
    - |
      for osarch in \
        "linux amd64" "linux 386" "linux arm" \
        "darwin amd64" "windows amd64" "windows 386"
      do
        set -- $osarch
        OS=$1 ; ARCH=$2
        BIN="terraform-provider-oase_${PROVIDER_VERSION}_${OS}_${ARCH}"
        OUTPATH="bin/release/$PROVIDER_VERSION/$BIN"
        GOOS=$OS GOARCH=$ARCH CGO_ENABLED=0 go build -o "$OUTPATH"
        cd "bin/release/$PROVIDER_VERSION"
        zip "${BIN}.zip" "$BIN"
        sha256sum "${BIN}.zip" > "${BIN}.SHA256SUMS"
        rm "$BIN"
        cd -
      done
  artifacts:
    paths:
      - bin/release/
    expire_in: 1 hour
  tags:
    - golang
    - env:dev

upload-provider:
  stage: deploy
  image: curlimages/curl:latest
  script:
    - export PROVIDER_VERSION="${CI_PIPELINE_ID}.0.0"
    - cd bin/release/$PROVIDER_VERSION
    - ls -lh
    # Upload all ZIPs and SHA256SUMS for each OS/ARCH
    - |
      for f in terraform-provider-oase_${PROVIDER_VERSION}_*.zip; do
        base="${f%.zip}"
        curl -k --location 'https://tfp.entw.oase.itz.itzbund.net/upload' \
          -F "files=@$f" \
          -F "files=@${base}.SHA256SUMS"
      done
  dependencies:
    - build-provider
  tags:
    - bash
    - env:dev
```

***

## 2. Update Your Makefile (if needed)

If you want to call builds locally with the same version scheme, update/add:

```makefile
# Add to top if not already present
PROVIDER_VERSION ?= $(shell echo $${CI_PIPELINE_ID}.0.0)

build-provider:
	mkdir -p bin/release/$(PROVIDER_VERSION)
	@for osarch in "linux amd64" "linux 386" "linux arm" "darwin amd64" "windows amd64" "windows 386"; do \
	  set -- $$osarch; \
	  OS=$$1 ; ARCH=$$2 ; \
	  BIN="terraform-provider-oase_$(PROVIDER_VERSION)_$${OS}_$${ARCH}"; \
	  OUTPATH="bin/release/$(PROVIDER_VERSION)/$$BIN"; \
	  GOOS=$$OS GOARCH=$$ARCH CGO_ENABLED=0 go build -o "$$OUTPATH"; \
	  cd "bin/release/$(PROVIDER_VERSION)"; \
	  zip "$$BIN.zip" "$$BIN"; \
	  sha256sum "$$BIN.zip" > "$$BIN.SHA256SUMS"; \
	  rm "$$BIN"; \
	  cd -; \
	done
```
Call with `make build-provider` with the appropriate `PROVIDER_VERSION`.

***

## 3. Pipeline Flow

- If you run the pipeline with `CI_PIPELINE_ID=3924`, you will get:
  - `terraform-provider-oase_3924.0.0_linux_amd64.zip`
  - And associated SHA256SUMS, for each platform/arch.
- Each file is named **precisely** as Terraform/Tofu expects.
- All are uploaded automatically right after building.
- If your registry exposes `/v1/providers/nearcloud/oase/versions`, it will now show and resolve the new version properly, and Terraform will download the correct ZIP.

***

**You can now remove any other custom or manual artifact build/upload logic from your pipeline, as this covers all use cases for provider artifact distribution!**

If you need the upload to also sign SHA256SUMS, that can be automated by extending the `build-provider` script (let me know if needed).

Let me know if you want a full .gitlab-ci.yml example or have special security/upload requirements!

