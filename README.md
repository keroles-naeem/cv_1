mkdir -p bin/release/0_0_3987/terraform-provider-oase_0_0_3987_darwin_amd64
cp bin/darwin_amd64/terraform-provider-oase_0_0_3987 bin/release/0_0_3987/terraform-provider-oase_0_0_3987_darwin_amd64
cd bin/release/0_0_3987/terraform-provider-oase_0_0_3987_darwin_amd64 && zip -r ../terraform-provider-oase_0_0_3987_darwin_amd64.zip . && cd .. && sha256sum terraform-provider-oase_0_0_3987_darwin_amd64.zip > terraform-provider-oase_0_0_3987_darwin_amd64.SHA256SUMS && rm -rf terraform-provider-oase_0_0_3987_darwin_amd64
  adding: terraform-provider-oase_0_0_3987 (deflated 64%)
GOOS=linux GOARCH=amd64 CGO_ENABLED=0 go build -o bin/linux_amd64/terraform-provider-oase_0_0_3987 -ldflags="-s -w" .
mkdir -p bin/release/0_0_3987/terraform-provider-oase_0_0_3987_linux_amd64
cp bin/linux_amd64/terraform-provider-oase_0_0_3987 bin/release/0_0_3987/terraform-provider-oase_0_0_3987_linux_amd64
cd bin/release/0_0_3987/terraform-provider-oase_0_0_3987_linux_amd64 && zip -r ../terraform-provider-oase_0_0_3987_linux_amd64.zip . && cd .. && sha256sum terraform-provider-oase_0_0_3987_linux_amd64.zip > terraform-provider-oase_0_0_3987_linux_amd64.SHA256SUMS && rm -rf terraform-provider-oase_0_0_3987_linux_amd64
  adding: terraform-provider-oase_0_0_3987 (deflated 64%)
GOOS=linux GOARCH=386 CGO_ENABLED=0 go build -o bin/linux_386/terraform-provider-oase_0_0_3987 -ldflags="-s -w" .
mkdir -p bin/release/0_0_3987/terraform-provider-oase_0_0_3987_linux_386
cp bin/linux_386/terraform-provider-oase_0_0_3987 bin/release/0_0_3987/terraform-provider-oase_0_0_3987_linux_386
cd bin/release/0_0_3987/terraform-provider-oase_0_0_3987_linux_386 && zip -r ../terraform-provider-oase_0_0_3987_linux_386.zip . && cd .. && sha256sum terraform-provider-oase_0_0_3987_linux_386.zip > terraform-provider-oase_0_0_3987_linux_386.SHA256SUMS && rm -rf terraform-provider-oase_0_0_3987_linux_386
  adding: terraform-provider-oase_0_0_3987 (deflated 64%)
GOOS=linux GOARCH=arm CGO_ENABLED=0 go build -o bin/linux_arm/terraform-provider-oase_0_0_3987 -ldflags="-s -w" .
mkdir -p bin/release/0_0_3987/terraform-provider-oase_0_0_3987_linux_arm
cp bin/linux_arm/terraform-provider-oase_0_0_3987 bin/release/0_0_3987/terraform-provider-oase_0_0_3987_linux_arm
cd bin/release/0_0_3987/terraform-provider-oase_0_0_3987_linux_arm && zip -r ../terraform-provider-oase_0_0_3987_linux_arm.zip . && cd .. && sha256sum terraform-provider-oase_0_0_3987_linux_arm.zip > terraform-provider-oase_0_0_3987_linux_arm.SHA256SUMS && rm -rf terraform-provider-oase_0_0_3987_linux_arm
  adding: terraform-provider-oase_0_0_3987 (deflated 65%)
GOOS=windows GOARCH=amd64 CGO_ENABLED=0 go build -o bin/windows_amd64/terraform-provider-oase_0_0_3987 -ldflags="-s -w" .
mkdir -p bin/release/0_0_3987/terraform-provider-oase_0_0_3987_windows_amd64
cp bin/windows_amd64/terraform-provider-oase_0_0_3987 bin/release/0_0_3987/terraform-provider-oase_0_0_3987_windows_amd64
cd bin/release/0_0_3987/terraform-provider-oase_0_0_3987_windows_amd64 && zip -r ../terraform-provider-oase_0_0_3987_windows_amd64.zip . && cd .. && sha256sum terraform-provider-oase_0_0_3987_windows_amd64.zip > terraform-provider-oase_0_0_3987_windows_amd64.SHA256SUMS && rm -rf terraform-provider-oase_0_0_3987_windows_amd64
  adding: terraform-provider-oase_0_0_3987 (deflated 64%)
GOOS=windows GOARCH=386 CGO_ENABLED=0 go build -o bin/windows_386/terraform-provider-oase_0_0_3987 -ldflags="-s -w" .
mkdir -p bin/release/0_0_3987/terraform-provider-oase_0_0_3987_windows_386
cp bin/windows_386/terraform-provider-oase_0_0_3987 bin/release/0_0_3987/terraform-provider-oase_0_0_3987_windows_386
cd bin/release/0_0_3987/terraform-provider-oase_0_0_3987_windows_386 && zip -r ../terraform-provider-oase_0_0_3987_windows_386.zip . && cd .. && sha256sum terraform-provider-oase_0_0_3987_windows_386.zip > terraform-provider-oase_0_0_3987_windows_386.SHA256SUMS && rm -rf terraform-provider-oase_0_0_3987_windows_386
  adding: terraform-provider-oase_0_0_3987 (deflated 64%)
rm bin/linux_arm/terraform-provider-oase_0_0_3987 bin/darwin_amd64/terraform-provider-oase_0_0_3987 bin/windows_386/terraform-provider-oase_0_0_3987 bin/linux_amd64/terraform-provider-oase_0_0_3987 bin/windows_amd64/terraform-provider-oase_0_0_3987 bin/linux_386/terraform-provider-oase_0_0_3987
Downloading artifacts from coordinator... ok        host=gitlab.itsm-suite.service.itzbund.net id=14337 responseStatus=200 OK token=64_mHXmS9
WARNING: bin/release/: lchown bin/release/: operation not permitted (suppressing repeats) 
Executing "step_script" stage of the job script
00:01
$ echo  $VERSION_PIP_ID
0_0_3987
$ curl -k --location 'https://tfp.entw.oase.itz.itzbund.net/upload' -F "files=@bin/release/${VERSION_PIP_ID}/terraform-provider-oase_${VERSION_PIP_ID}_linux_amd64.SHA256SUMS" -F "files=@bin/release/${VERSION_PIP_ID}/terraform-provider-oase_${VERSION_PIP_ID}_linux_amd64.SHA256SUMS.sig" -F "files=@bin/release/${VERSION_PIP_ID}/terraform-provider-oase_${VERSION_PIP_ID}_linux_amd64.zip" -F "gpg-key=@bin/release/${VERSION_PIP_ID}/key.gpg"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 5456k  100    42  100 5456k    378  48.0M --:--:-- --:--:-- --:--:-- 48.0M
Uploaded successfully 3 files with fields.
Cleaning up project directory and file based variables
00:00
Job succeeded

$ tofu init
2025-08-21T12:39:46.904+0200 [INFO]  OpenTofu version: 1.9.1
2025-08-21T12:39:46.904+0200 [DEBUG] using github.com/hashicorp/go-tfe v1.36.0
2025-08-21T12:39:46.904+0200 [DEBUG] using github.com/opentofu/hcl/v2 v2.0.0-20240814143621-8048794c5c52
2025-08-21T12:39:46.904+0200 [DEBUG] using github.com/hashicorp/terraform-svchost v0.1.1
2025-08-21T12:39:46.904+0200 [DEBUG] using github.com/zclconf/go-cty v1.14.4
2025-08-21T12:39:46.904+0200 [INFO]  Go runtime version: go1.22.12
2025-08-21T12:39:46.904+0200 [INFO]  CLI args: []string{"tofu", "init"}
2025-08-21T12:39:46.905+0200 [DEBUG] Attempting to open CLI config file: /home/[MASKED]/.tofurc
2025-08-21T12:39:46.905+0200 [DEBUG] File doesn't exist, but doesn't need to. Ignoring.
2025-08-21T12:39:46.905+0200 [DEBUG] ignoring non-existing provider search directory terraform.d/plugins
2025-08-21T12:39:46.905+0200 [DEBUG] ignoring non-existing provider search directory /home/[MASKED]/.terraform.d/plugins
2025-08-21T12:39:46.905+0200 [DEBUG] ignoring non-existing provider search directory /home/[MASKED]/.local/share/terraform/plugins
2025-08-21T12:39:46.905+0200 [DEBUG] ignoring non-existing provider search directory /usr/local/share/terraform/plugins
2025-08-21T12:39:46.905+0200 [DEBUG] ignoring non-existing provider search directory /usr/share/terraform/plugins
2025-08-21T12:39:46.905+0200 [DEBUG] Found the config directory: /home/[MASKED]/.terraform.d
2025-08-21T12:39:46.907+0200 [INFO]  CLI command args: []string{"init"}
Initializing the backend...
2025-08-21T12:39:46.912+0200 [DEBUG] New state was assigned lineage "e8264f06-1de5-c203-188e-336c6f8e9966"
2025-08-21T12:39:46.912+0200 [DEBUG] checking for provisioner in "."
2025-08-21T12:39:46.914+0200 [DEBUG] checking for provisioner in "/usr/bin"
Initializing provider plugins...
2025-08-21T12:39:46.915+0200 [DEBUG] Service discovery for tfp.entw.oase.itz.itzbund.net at https://tfp.entw.oase.itz.itzbund.net/.well-known/terraform.json
- Finding latest version of tfp.entw.oase.itz.itzbund.net/nearcloud/oase...
2025-08-21T12:39:46.983+0200 [DEBUG] GET https://tfp.entw.oase.itz.itzbund.net/v1/providers/nearcloud/oase/versions
2025-08-21T12:39:47.009+0200 [DEBUG] GET https://tfp.entw.oase.itz.itzbund.net/v1/providers/nearcloud/oase/3924.0.0/download/linux/amd64
╷
│ Error: Incompatible provider version
│ 
│ Provider tfp.entw.oase.itz.itzbund.net/nearcloud/oase v3924.0.0 does not
│ have a package available for your current platform, linux_amd64.
│ 
│ Provider releases are separate from OpenTofu CLI releases, so not all
│ providers are available for all platforms. Other versions of this provider
│ may have different platforms supported.
╵
Cleaning up project directory and file based variables
00:00
ERROR: Job failed: exit status 1
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
    - export RELEASE_VERSION="${VERSION_PIP_ID}"
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
