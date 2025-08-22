yes

Here is a straightforward way to automate consistent provider versioning, artifact naming, and uploading in your CI/CD pipeline.

***

## 1. Set the Provider Version Dynamically

**Add this to the setup of your CI job** (e.g. `.gitlab-ci.yml` or your build script):

```bash
export PROVIDER_VERSION="${CI_PIPELINE_ID}.0.0"
export PROVIDER_VERSION_SHORT="${CI_PIPELINE_ID}"
```

***

## 2. Update Your Makefile or Build Script

Build and package the provider for each platform with exact Terraform conventions:

```makefile
# Assuming PROVIDER_VERSION is exported from CI/job
NAME=terraform-provider-oase
VERSION=$(PROVIDER_VERSION)

PLATFORMS = linux_amd64 linux_386 linux_arm darwin_amd64 windows_amd64 windows_386

release: $(PLATFORMS:%=bin/release/$(NAME)_$(VERSION)_%)
    @echo "All builds complete."

bin/release/$(NAME)_$(VERSION)_%: bin/%/$(NAME)_$(VERSION)
	mkdir -p bin/release/$(VERSION)
	cp bin/$*/$(NAME)_$(VERSION) bin/release/$(VERSION)/
	cd bin/release/$(VERSION) && zip -r $(NAME)_$(VERSION)_$*.zip $(NAME)_$(VERSION)
	cd bin/release/$(VERSION) && sha256sum $(NAME)_$(VERSION)_$*.zip > $(NAME)_$(VERSION)_$*.SHA256SUMS
	cd bin/release/$(VERSION) && rm $(NAME)_$(VERSION)
```

*Adjust as needed for your structure.*

***

## 3. Script for CI: Build, Upload, and Register

In your pipeline (shell or script step):

```bash
export PROVIDER_VERSION="${CI_PIPELINE_ID}.0.0"

make clean
make release PROVIDER_VERSION=$PROVIDER_VERSION

# Loop over platforms for upload (add/remove as needed)
for plat in linux_amd64 linux_386 linux_arm darwin_amd64 windows_amd64 windows_386; do
  ZIP="bin/release/${PROVIDER_VERSION}/terraform-provider-oase_${PROVIDER_VERSION}_${plat}.zip"
  SUM="bin/release/${PROVIDER_VERSION}/terraform-provider-oase_${PROVIDER_VERSION}_${plat}.SHA256SUMS"
  [ -f "$ZIP" ] && curl -k --location 'https://tfp.entw.oase.itz.itzbund.net/upload' \
    -F "files=@${ZIP}" -F "files=@${SUM}" -F "gpg-key=@bin/release/${PROVIDER_VERSION}/key.gpg"
done
```

***

## 4. Registry Index (Metadata/JSON)

Your registry’s index or metadata file (e.g. `terraform-provider-oase.json`) must list `PROVIDER_VERSION`, and each download endpoint for `/v1/providers/.../download/linux/amd64` must resolve to the corresponding ZIP for that version and platform.

***

## 5. Result: Each pipeline run will produce artifacts like:

```
terraform-provider-oase_3924.0.0_linux_amd64.zip
terraform-provider-oase_3924.0.0_linux_amd64.SHA256SUMS
```
…for each platform.

***

**With these conventions, Terraform or OpenTofu will always find your latest published version when you run `init` with a matching version.**

Let me know if you want more specifics for a GitLab, GitHub, or other CI system, or an end-to-end script!

