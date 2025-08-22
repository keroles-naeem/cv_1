apply for gitlab an makefile

To apply your requested versioning style `0_0_3997` in GitLab CI and your Makefile, here is a tailored solution that preserves underscores and integrates well into your pipeline:

***

## GitLab CI Configuration Snippet

Modify your `.gitlab-ci.yml` as follows:

```yaml
variables:
  VERSION_PIP_ID: "0_0_${CI_PIPELINE_ID}"
  VERSION: "v0_0_${CI_PIPELINE_ID}"

install:
  stage: install
  script:
    - apt-get update
    - apt-get install -y zip
    - echo "Building version ${VERSION}"
    - make release VERSION=${VERSION}  # Pass VERSION with underscores
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
  script:
    - echo "Uploading version ${VERSION}"
    - curl -k --location 'https://tfp.entw.oase.itz.itzbund.net/upload' \
      -F "files=@bin/release/${VERSION}/terraform-provider-oase_${VERSION}_linux_amd64.SHA256SUMS" \
      -F "files=@bin/release/${VERSION}/terraform-provider-oase_${VERSION}_linux_amd64.SHA256SUMS.sig" \
      -F "files=@bin/release/${VERSION}/terraform-provider-oase_${VERSION}_linux_amd64.zip" \
      -F "gpg-key=@bin/release/${VERSION}/key.gpg"
```

***

## Makefile Adjustments

At the top of your `Makefile`:

```makefile
VERSION ?= v0_0_3997
```

You will override this via GitLab CI with:

```bash
make release VERSION=v0_0_3924
```

Your build and release targets:

```makefile
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
```

***

## Important Notes:

- **Ensure your upload service and metadata JSON handle versions with underscores**, including matching paths that contain `v0_0_3997`.
- **Terraform version strings** in `.tf` files are semver with dots by default, so using underscore versions may cause Terraform client errors unless you fully control registry version mappings.
- If you want to use underscore versioning consistently, your Terraform client must be pointed explicitly to your private registry and accept those version strings.

***

This setup **builds, packages, uploads, and references artifacts with underscore-separated versions** like `v0_0_3924`, fulfilling your request.

If you want, I can generate a fully integrated `.gitlab-ci.yml` and Makefile example with comments tailored for your project.

Citations:
[1] Using Terraform with GitLab https://scalr.com/learning-center/using-terraform-with-gitlab/
[2] Infrastructure as Code with OpenTofu and GitLab https://docs.gitlab.com/user/infrastructure/iac/
[3] How to Implement GitLab CI/CD Pipeline with Terraform https://spacelift.io/blog/gitlab-terraform
[4] GitLab CI/CD examples https://docs.gitlab.com/ci/examples/
[5] terraform-makefile https://gitlab.com/makefile/terraform-makefile
[6] Gitlab CI/CD with Terraform and 3rd party plugin https://stackoverflow.com/questions/56546599/gitlab-ci-cd-with-terraform-and-3rd-party-plugin
[7] Suggestions for improvement of Terraform deployment ... https://www.reddit.com/r/Terraform/comments/1i47w19/suggestions_for_improvement_of_terraform/
[8] Speed Up Terraform in CI/CD Pipelines https://infinitelambda.com/speed-up-terraform-cicd-pipeline/
