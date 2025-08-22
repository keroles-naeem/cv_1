show me the gitlab and the Makefile

Here is a full example of a **GitLab CI pipeline** and a **Makefile** set up to build, package, and upload Terraform provider artifacts using the underscore version format like `0_0_3997`, with the version sourced dynamically from `CI_PIPELINE_ID`:

***

## `.gitlab-ci.yml`

```yaml
stages:
  - check
  - test
  - install
  - build
  - distribute
  - e2e-test

variables:
  VERSION_PIP_ID: "0_0_${CI_PIPELINE_ID}"
  VERSION: "${VERSION_PIP_ID}"

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
  image: registry.gitlab.itsm-suite.service.itzbund.net/oase/container-deployment/1.22-bullseye:latest
  script:
    - make testacc
  tags:
    - golang
    - env:dev

install:
  stage: install
  tags:
    - golang
    - env:dev
  script:
    - apt-get update
    - apt-get install -y zip
    - echo "Building version ${VERSION}"
    - make release VERSION=${VERSION}
    - cp sign_files.sh bin/release/${VERSION}/
    - gpg --import $key
    - gpg --list-secret-keys
    - cd bin/release/${VERSION}/ && gpg --armor --export ci/cd > key.gpg
    - sh sign_files.sh && rm sign_files.sh
  artifacts:
    paths:
      - bin/release
    expire_in: 1 hour

push-to-terraform-registry:
  stage: distribute
  dependencies:
    - install
  tags:
    - bash
    - env:dev
  script:
    - echo "Uploading version ${VERSION}"
    - curl -k --location 'https://tfp.entw.oase.itz.itzbund.net/upload' \
        -F "files=@bin/release/${VERSION}/terraform-provider-oase_${VERSION}_linux_amd64.SHA256SUMS" \
        -F "files=@bin/release/${VERSION}/terraform-provider-oase_${VERSION}_linux_amd64.SHA256SUMS.sig" \
        -F "files=@bin/release/${VERSION}/terraform-provider-oase_${VERSION}_linux_amd64.zip" \
        -F "gpg-key=@bin/release/${VERSION}/key.gpg"

e2e-test:
  stage: e2e-test
  dependencies:
    - install
  variables:
    TF_LOG: DEBUG
  tags:
    - bash
    - env:dev
  script:
    - cd test_e2e
    - tofu --version
    - tofu init
    - tofu apply --auto-approve
    - tofu destroy --auto-approve
```

***

## `Makefile`

```makefile
VERSION ?= 0_0_3997

default: testacc

.PHONY: format testacc
testacc:
	TF_ACC=1 go test ./... -v $(TESTARGS) -timeout 120m

fmt:
	@gofmt -w -s .

check:
	@revive -formatter unix ./...

install:
	@go install .

clean:
	rm -fr bin/ .tmp/

build: bin/current_system/terraform-provider-oase_$(VERSION)

run: build
	bin/current_system/terraform-provider-oase

bin/current_system/terraform-provider-oase_%: GOARGS =
bin/darwin_amd64/terraform-provider-oase_%: GOARGS = GOOS=darwin GOARCH=amd64
bin/linux_amd64/terraform-provider-oase_%: GOARGS = GOOS=linux GOARCH=amd64
bin/linux_386/terraform-provider-oase_%: GOARGS = GOOS=linux GOARCH=386
bin/linux_arm/terraform-provider-oase_%: GOARGS = GOOS=linux GOARCH=arm
bin/windows_amd64/terraform-provider-oase_%: GOARGS = GOOS=windows GOARCH=amd64
bin/windows_386/terraform-provider-oase_%: GOARGS = GOOS=windows GOARCH=386

bin/%/terraform-provider-oase_$(VERSION): clean
	$(GOARGS) CGO_ENABLED=0 go build -o $@ -ldflags="-s -w" .

bin/release/terraform-provider-oase_%.zip: NAME=terraform-provider-oase_$(VERSION)_$*
bin/release/terraform-provider-oase_%.zip: DEST=bin/release/$(VERSION)/$(NAME)
bin/release/terraform-provider-oase_%.zip: bin/%/terraform-provider-oase_$(VERSION)
	mkdir -p $(DEST)
	cp bin/$*/terraform-provider-oase_$(VERSION) $(DEST)
	cd $(DEST) && zip -r ../$(NAME).zip . && cd .. && sha256sum $(NAME).zip > $(NAME).SHA256SUMS && rm -rf $(NAME)

release: \
	bin/release/terraform-provider-oase_darwin_amd64.zip \
	bin/release/terraform-provider-oase_linux_amd64.zip \
	bin/release/terraform-provider-oase_linux_386.zip \
	bin/release/terraform-provider-oase_linux_arm.zip \
	bin/release/terraform-provider-oase_windows_amd64.zip \
	bin/release/terraform-provider-oase_windows_386.zip
```

***

### Notes:

- The version is dynamically set via `VERSION_PIP_ID` in GitLab CI based on `CI_PIPELINE_ID`, formatted as `0_0_<id>`.
- The Makefile uses `$(VERSION)` for naming binaries and release zips consistently.
- Artifacts are uploaded in the versioned folder `bin/release/0_0_3997/` etc.
- Checksums and signatures are generated and uploaded alongside the zip.
- The folder structure and naming match Terraform provider registry expectations, using your underscore version scheme.

***

If you want, I can help with additional scripts for uploading or verification!

Citations:
[1] Using Terraform with GitLab https://scalr.com/learning-center/using-terraform-with-gitlab/
[2] How to run Terraform in your Gitlab CI/CD pipeline? https://blog.digger.dev/how-to-run-terraform-in-your-gitlab-ci-cd-pipeline/
[3] How to Implement GitLab CI/CD Pipeline with Terraform https://spacelift.io/blog/gitlab-terraform
[4] GitLab CI/CD examples https://docs.gitlab.com/ci/examples/
[5] Infrastructure as Code with OpenTofu and GitLab https://docs.gitlab.com/user/infrastructure/iac/
[6] Example GitLab CI Pipeline using Terraform, etc. https://gist.github.com/mrlesmithjr/72e23d0a0cceefef553b83b4fce5d06f
[7] Speed Up Terraform in CI/CD Pipelines https://infinitelambda.com/speed-up-terraform-cicd-pipeline/
[8] Terraform Module Registry https://docs.gitlab.com/user/packages/terraform_module_registry/
