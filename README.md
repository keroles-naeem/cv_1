

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


yes

Here is an updated version of your `.gitlab-ci.yml` and `Makefile` configured to use **dot-separated semantic versioning** like `3924.0.0`, which matches Terraform/OpenTofu expectations and resolves your "incompatible provider version" error.

***

## Updated `.gitlab-ci.yml`

```yaml
stages:
  - check
  - test
  - install
  - build
  - distribute
  - e2e-test

variables:
  VERSION_PIP_ID: "${CI_PIPELINE_ID}.0.0"   # dot-separated semantic version
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

## Updated `Makefile`

```makefile
VERSION ?= 0.0.0

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

### What this does:

- Dynamically builds a version like `3924.0.0` from your pipeline ID.
- Uses dot-separated version strings compatible with Terraform provider versioning.
- Produces artifacts named precisely like:

```
terraform-provider-oase_3924.0.0_linux_amd64.zip
```

- Uploads with matching metadata so Terraform/OpenTofu CLI finds the correct packages during `init`.

***

If you run the pipeline with these changes, your `tofu init` will find a compatible provider package for your platform and version, resolving the error.

Let me know if you want me to generate any helper scripts or assist with registry metadata management!


