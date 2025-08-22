apply them.
stages:
  - check
  - test
  - install
  - build
  - distribute
  - e2e-test
variables:
  VERSION_PIP_ID: "0_0_${CI_PIPELINE_ID}"
workflow:
   rules:
     - if: $CI_MERGE_REQUEST_IID
     - if: $CI_COMMIT_TAG
     - if: $CI_PIPELINE_SOURCE == "schedule"
     - if: $CI_COMMIT_REF_PROTECTED == "true"

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
    name: registry.gitlab.itsm-suite.service.itzbund.net/oase/container-deployment/1.22-bullseye:latest
  script: make testacc
  tags:
    - golang
    - env:dev
       

install:
  stage: install
  tags:
    - golang
    - env:dev
  except:
    - schedules
  variables:
    HTTP_PROXY: http://10.130.165.20:3128
    HTTPS_PROXY: http://10.130.165.20:3128
  before_script:
    - echo "Acquire::http::Proxy \"http://10.130.165.20:3128/\";" > /etc/apt/apt.conf.d/99proxy
    - echo "Acquire::https::Proxy \"http://10.130.165.20:3128/\";" >> /etc/apt/apt.conf.d/99proxy
  script:
    - apt-get update 
    - apt-get install -y zip
    - export RELEASE_VERSION="${CI_PIPELINE_ID}"
    - echo "Building version ${VERSION_PIP_ID}"
    - make release VERSION=${VERSION_PIP_ID}
    - cp sign_files.sh bin/release/$VERSION_PIP_ID/
    - gpg --import $key
    - gpg --list-secret-keys
    - cd bin/release/$VERSION_PIP_ID/ && gpg --armor --export ci/cd > key.gpg
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
      - echo  $VERSION_PIP_ID
      - curl -k --location 'https://tfp.entw.oase.itz.itzbund.net/upload' -F "files=@bin/release/${VERSION_PIP_ID}/terraform-provider-oase_${VERSION_PIP_ID}_linux_amd64.SHA256SUMS" -F "files=@bin/release/${VERSION_PIP_ID}/terraform-provider-oase_${VERSION_PIP_ID}_linux_amd64.SHA256SUMS.sig" -F "files=@bin/release/${VERSION_PIP_ID}/terraform-provider-oase_${VERSION_PIP_ID}_linux_amd64.zip" -F "gpg-key=@bin/release/${VERSION_PIP_ID}/key.gpg"
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
      - cd ..
      - ls -a
      - cd test_e2e
      - pwd
      - ls -a
      - export TF_LOG=DEBUG
      - tofu --version
      - tofu init
      - tofu apply --auto-approve
      - tofu destroy --auto-approve


--------------------------------------------------------------------------------
makefile
VERSION:=v0.0.0-local
default: testacc

# Run acceptance tests
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

#? build: build binary for current system
build: bin/current_system/terraform-provider-oase_$(VERSION)

#? run: run plugin
run: build
	bin/current_system/terraform-provider-oase

bin/current_system/terraform-provider-oase_%:  GOARGS =
bin/darwin_amd64/terraform-provider-oase_%:  GOARGS = GOOS=darwin GOARCH=amd64
bin/linux_amd64/terraform-provider-oase_%:  GOARGS = GOOS=linux GOARCH=amd64
bin/linux_386/terraform-provider-oase_%:  GOARGS = GOOS=linux GOARCH=386
bin/linux_arm/terraform-provider-oase_%:  GOARGS = GOOS=linux GOARCH=arm
bin/windows_amd64/terraform-provider-oase_%:  GOARGS = GOOS=windows GOARCH=amd64
bin/windows_386/terraform-provider-oase_%:  GOARGS = GOOS=windows GOARCH=386

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
