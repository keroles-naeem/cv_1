no i want version according this shape 
0_0_3997

If you want your provider version format to remain exactly as `0_0_3997` (underscore-separated) and still work with Terraform/OpenTofu, you must ensure your entire system (build, upload, metadata, and Terraform usage) consistently supports this non-standard format.

***

### Important Considerations

- **Terraform and OpenTofu expect semantic versions with dots (e.g., `0.0.3997`).**  
  Using underscores (`0_0_3997`) is non-standard and may cause client errors like missing package for your platform/version unless you do custom handling.

- **Your registry metadata must list versions exactly as `0_0_3997`.**  
  The provider download URLs must correspond to your underscore version format.

- **Your Terraform configuration must also specify the provider version as `0_0_3997` if the client supports it.** If not, you must customize the registry or client behavior.

***

### How to implement underscore versioning fully

1. **Keep your GitLab CI variables:**

```yaml
variables:
  VERSION_PIP_ID: "0_0_${CI_PIPELINE_ID}"
  VERSION: "${VERSION_PIP_ID}"
```

2. **Makefile using `VERSION` without `v` prefix:**

```makefile
VERSION ?= 0_0_3997
```

3. **Ensure your provider artifact filenames are consistently named:**

```
terraform-provider-oase_0_0_3997_linux_amd64.zip
```

4. **Registry metadata must store and serve versions as `0_0_3997` (including the underscores).**

5. **Terraform/OpenTofu usage:**

- Your `required_providers` block in Terraform configuration must specify the exact version string with underscores:

```hcl
terraform {
  required_providers {
    oase = {
      source  = "tfp.entw.oase.itz.itzbund.net/nearcloud/oase"
      version = "0_0_3997"
    }
  }
}
```

*Note:* This is not standard and might not work with vanilla Terraform clients without registry or client customization (e.g., patches, proxy).

***

### Summary

- Your pipeline, build, upload, and metadata handle versions with underscores.
- You must confirm your Terraform/OpenTofu client supports or is configured to accept underscore version strings.
- If not, client or registry customization is needed.

***

If you want, I can help you generate:

- A GitLab CI/CD pipeline + Makefile setup for underscore versions.
- Guidance on how to patch or proxy Terraform/OpenTofu version checks for this version format.

Would you like me to provide detailed example files with this underscore versioning applied?



yes

Here is a detailed example of your **GitLab CI pipeline** and **Makefile** configured to consistently use the underscore version format like `0_0_3997`. This setup ensures your artifacts, upload, and version variable all align with your desired versioning scheme.

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

- Version strings are underscore-separated, e.g., `0_0_3997`.
- Provider binary and checksum files are named accordingly, e.g.,  
  `terraform-provider-oase_0_0_3997_linux_amd64.zip`.
- Metadata and upload expect the same version string format.
- Terraform/OpenTofu configuration must specify exact version strings with underscores.
- If using vanilla Terraform CLI, verify your registry & provider accept this format as it’s non-standard.

***

If you want, I can help with example Terraform provider configuration to use this versioning or assistance with registry adjustments to support underscore versions gracefully.

