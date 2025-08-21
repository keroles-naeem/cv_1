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

Your current Makefile uses a fixed `VERSION:=v0.0.0-local`, which does not match the dynamic semantic versioning expected by your Terraform/OpenTofu provider registry. To fix the incompatible provider version error, you need to:

### 1. Change versioning from fixed `v0.0.0-local` to dynamic semantic version:

For example, derive the version from the environment or CI variables in a SemVer format with dots, such as:

```makefile
VERSION ?= $(CI_PIPELINE_ID).0.0
VERSION := v$(VERSION)
```

This sets `VERSION` to something like `v3987.0.0` if `CI_PIPELINE_ID=3987`.

### 2. Update your `release` targets and artifacts to use this dynamic `VERSION`:

Your bin and release naming rules should adopt this `${VERSION}` everywhere, for example:

```makefile
bin/%/terraform-provider-oase_$(VERSION): clean
	$(GOARGS) CGO_ENABLED=0 go build -o $@ -ldflags="-s -w" .

bin/release/terraform-provider-oase_%.zip: NAME=terraform-provider-oase_$(VERSION)_$*
bin/release/terraform-provider-oase_%.zip: DEST=bin/release/$(VERSION)/$(NAME)
bin/release/terraform-provider-oase_%.zip: bin/%/terraform-provider-oase_$(VERSION)
	mkdir -p $(DEST)
	cp bin/$*/terraform-provider-oase_$(VERSION) $(DEST)
	cd $(DEST) && zip -r ../$(NAME).zip . && cd .. && sha256sum $(NAME).zip > $(NAME).SHA256SUMS && rm -rf $(NAME)
```

### 3. How to use in CI pipeline

Make sure your pipeline exports `CI_PIPELINE_ID` or a proper semver version string before running `make release`. Example export in CI job:

```bash
export CI_PIPELINE_ID=3987  # or get from your CI environment variables
make release
```

### 4. Example final snippet

```makefile
VERSION ?= $(CI_PIPELINE_ID).0.0
VERSION := v$(VERSION)

.PHONY: release
release: \
	bin/release/terraform-provider-oase_darwin_amd64.zip \
	bin/release/terraform-provider-oase_linux_amd64.zip \
	bin/release/terraform-provider-oase_linux_386.zip \
	bin/release/terraform-provider-oase_linux_arm.zip \
	bin/release/terraform-provider-oase_windows_amd64.zip \
	bin/release/terraform-provider-oase_windows_386.zip
```

Run your build/upload with this updated semver, and the registry and OpenTofu should correctly recognize and find compatible provider versions.

***

If you want, I can help you generate a full adapted Makefile snippet or CI script to handle this versioning automatically.

