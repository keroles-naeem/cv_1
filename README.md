
Your GitLab CI snippet and Makefile mostly look correct but have a few important issues to fix for proper functionality:

***

### Issues to fix

1. **Duplicate `script` keys in `push-to-terraform-registry`:**  
Your `push-to-terraform-registry` job has *two* `script:` blocks, which is invalid. Merge them into a single script block.

2. **Inconsistent use of `VERSION` vs `VERSION_PIP_ID`:**  
- You export `VERSION="${VERSION}"` but then call `make release VERSION=${VERSION_PIP_ID}` — this overrides your intended `VERSION`.  
- Use either one consistently; better to export and use `VERSION` throughout.

3. **No explicit export of `VERSION_PIP_ID`:**  
Make sure your pipeline declares `VERSION_PIP_ID` as an environment variable at pipeline or job level.

***

### Corrected GitLab CI snippet example:

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
    - make release VERSION=${VERSION}    # Use VERSION consistently here
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
```

***

### Makefile snippet (top lines)

```makefile
VERSION ?= v0_0_3997
```

Make sure when you call `make release` you use:

```bash
make release VERSION=v0_0_3924
```

And your build/release rules already take `$(VERSION)`.

***

### Summary

- Use **one single version variable (`VERSION`)** everywhere to avoid confusion.
- Fix multiple `script:` blocks in GitLab CI job.
- Make sure `VERSION` is properly set and passed to `make`.
- Keep your version format `v0_0_xxx` consistent across CI and Makefile.

***

If you want, I can prepare a full corrected `.gitlab-ci.yml` and `Makefile` example for you to copy-paste.

