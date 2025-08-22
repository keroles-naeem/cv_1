$ tofu init
2025-08-22T09:44:04.752+0200 [INFO]  OpenTofu version: 1.9.1
2025-08-22T09:44:04.753+0200 [DEBUG] using github.com/hashicorp/go-tfe v1.36.0
2025-08-22T09:44:04.753+0200 [DEBUG] using github.com/opentofu/hcl/v2 v2.0.0-20240814143621-8048794c5c52
2025-08-22T09:44:04.753+0200 [DEBUG] using github.com/hashicorp/terraform-svchost v0.1.1
2025-08-22T09:44:04.753+0200 [DEBUG] using github.com/zclconf/go-cty v1.14.4
2025-08-22T09:44:04.753+0200 [INFO]  Go runtime version: go1.22.12
2025-08-22T09:44:04.753+0200 [INFO]  CLI args: []string{"tofu", "init"}
2025-08-22T09:44:04.754+0200 [DEBUG] Attempting to open CLI config file: /home/[MASKED]/.tofurc
2025-08-22T09:44:04.754+0200 [DEBUG] File doesn't exist, but doesn't need to. Ignoring.
2025-08-22T09:44:04.754+0200 [DEBUG] ignoring non-existing provider search directory terraform.d/plugins
2025-08-22T09:44:04.754+0200 [DEBUG] ignoring non-existing provider search directory /home/[MASKED]/.terraform.d/plugins
2025-08-22T09:44:04.754+0200 [DEBUG] ignoring non-existing provider search directory /home/[MASKED]/.local/share/terraform/plugins
2025-08-22T09:44:04.755+0200 [DEBUG] ignoring non-existing provider search directory /usr/local/share/terraform/plugins
2025-08-22T09:44:04.755+0200 [DEBUG] ignoring non-existing provider search directory /usr/share/terraform/plugins
2025-08-22T09:44:04.755+0200 [DEBUG] Found the config directory: /home/[MASKED]/.terraform.d
2025-08-22T09:44:04.756+0200 [INFO]  CLI command args: []string{"init"}
Initializing the backend...
2025-08-22T09:44:04.761+0200 [DEBUG] New state was assigned lineage "8559203d-7d53-a0c5-0835-5349ac2cff91"
2025-08-22T09:44:04.761+0200 [DEBUG] checking for provisioner in "."
2025-08-22T09:44:04.763+0200 [DEBUG] checking for provisioner in "/usr/bin"
2025-08-22T09:44:04.764+0200 [DEBUG] Service discovery for tfp.entw.oase.itz.itzbund.net at https://tfp.entw.oase.itz.itzbund.net/.well-known/terraform.json
Initializing provider plugins...
- Finding latest version of tfp.entw.oase.itz.itzbund.net/nearcloud/oase...
2025-08-22T09:44:04.834+0200 [DEBUG] GET https://tfp.entw.oase.itz.itzbund.net/v1/providers/nearcloud/oase/versions
2025-08-22T09:44:04.853+0200 [DEBUG] GET https://tfp.entw.oase.itz.itzbund.net/v1/providers/nearcloud/oase/3924.0.0/download/linux/amd64
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
00:01
ERROR: Job failed: exit status 1
-------------------------------------------------
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
    - curl -k --location 'https://tfp.entw.oase.itz.itzbund.net/upload' -F "files=@bin/release/${VERSION}/terraform-provider-oase_${VERSION}_linux_amd64.SHA256SUMS" -F "files=@bin/release/${VERSION}/terraform-provider-oase_${VERSION}_linux_amd64.SHA256SUMS.sig" -F "files=@bin/release/${VERSION}/terraform-provider-oase_${VERSION}_linux_amd64.zip" -F "gpg-key=@bin/release/${VERSION}/key.gpg"

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
    - cd test_e2e_2
    - tofu --version
    - tofu init
    - tofu apply --auto-approve
    - tofu destroy --auto-approve

----------------------
VERSION ?= 0_0_3997
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

The error message during `tofu init` indicates that Terraform/OpenTofu is requesting version `v3924.0.0`:

```
Provider tfp.entw.oase.itz.itzbund.net/nearcloud/oase v3924.0.0 does not have a package available for your current platform, linux_amd64.
```

However, your pipeline and Makefile produce versions like `0_0_3924` (underscore separated), *without* the `v` prefix and *with underscores*, e.g.,

```
terraform-provider-oase_0_0_3924_linux_amd64.zip
```

***

## Problem Analysis

- Your provider version as recorded and served by your registry is **`0_0_3924`** (underscore format), but OpenTofu/Terraform request **`3924.0.0`** (dot-separated semantic version).
- This creates a mismatch causing Terraform/OpenTofu to **not find a matching ZIP package** for that version and platform.
- The Terraform/OpenTofu CLI expects standard semver style versions like `X.Y.Z` (dots), *not* underscore-separated versions like `0_0_3924`.
- Your CI and Makefile produce artifacts and upload metadata based on the underscore version, but the client requests the dot version (with a `v` sometimes).

***

## How to fix it

### Option 1: Align your version format with Terraform expectations (recommended)

- Change your CI and Makefile to use **dot-separated semantic version strings**, e.g.:

```yaml
variables:
  VERSION_PIP_ID: "${CI_PIPELINE_ID}.0.0"
  VERSION: "${VERSION_PIP_ID}"
```

- Then build and upload files named:

```
terraform-provider-oase_3924.0.0_linux_amd64.zip
```

- This will match what Terraform/OpenTofu requests (note the exact version string with dots).

***

### Option 2: Map or alias versions in your registry (advanced)

- Keep uploading files with underscore versions, but **add registry metadata** or translation layer to map incoming client version requests like `3924.0.0` to your stored files named `0_0_3924`.
- This requires custom code in your registry server or proxy to remap versions and rewrite URLs.
- Consider renaming your artifacts on upload to dot-semver style for maximum compatibility.

***

### Option 3: Modify provider version references in your Terraform/OpenTofu config (not typical)

- If possible, force Terraform/OpenTofu to request provider versions with underscores, e.g., `0_0_3924`, but this is not standard and likely won’t work out of the box.

***

## What to do next?

1. **Change GitLab CI variables to:**

```yaml
variables:
  VERSION_PIP_ID: "${CI_PIPELINE_ID}.0.0"
  VERSION: "${VERSION_PIP_ID}"
```

2. **Update your Makefile VERSION default to:**

```makefile
VERSION ?= 0.0.0
```

and override with `make release VERSION=3924.0.0` from CI.

3. This will produce artifacts named with dots:

```
terraform-provider-oase_3924.0.0_linux_amd64.zip
```

which Terraform/OpenTofu will correctly find on init.

***

If you want, I can generate an updated `.gitlab-ci.yml` and Makefile with these changes applied to fix your version mismatch. This is the cleanest and most supported approach. Would you like me to do that?

