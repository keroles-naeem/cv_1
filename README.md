
You’ve built, zipped, SHA256SUMed, and uploaded all the provider artifacts for multiple OS/arch/versions successfully—**but in the e2e-test step, OpenTofu complains:**
```
Provider tfp.entw.oase.itz.itzbund.net/nearcloud/oase v3924.0.0 does not have a package available for your current platform, linux_amd64.
...
│ Incompatible provider version
```
### What’s happening?

1. **Build and upload worked:**  
   Your `curl` command posted these files (for version `0_0_3987`) to your registry:
   - terraform-provider-oase_0_0_3987_linux_amd64.zip
   - terraform-provider-oase_0_0_3987_linux_amd64.SHA256SUMS
   - terraform-provider-oase_0_0_3987_linux_amd64.SHA256SUMS.sig
   - key.gpg

2. **But tofu tried to install v3924.0.0** (that’s the provider version OpenTofu is seeking). Your local build/upload is `0_0_3987`, not `3924.0.0`.

3. **No matching artifacts for v3924.0.0** in the registry for platform `linux_amd64`—hence, “no package available”.

***

## Diagnosis

### **Your build/upload version ≠ the version OpenTofu wants**

- **Build/uploaded version:**  
  `${VERSION_PIP_ID}` = `0_0_${CI_PIPELINE_ID}` = `0_0_3987` (latest run)

- **OpenTofu wants:**  
  `v3924.0.0` (this is not the same as any `${VERSION_PIP_ID}`!)

***

### Why is tofu requesting v3924.0.0?

- Your registry’s `/v1/providers/nearcloud/oase/versions` endpoint returns `["3924.0.0", ...]` (maybe from an earlier run).
- Tofu asks for `v3924.0.0`, not your new artifact (`0_0_3987`).

### But you are building and uploading `0_0_3987`, not `3924.0.0`.  
Terraform/OpenTofu doesn’t know about your `_` separated format—it expects [semantic versioning](https://semver.org/) like `1.2.3`.

***

## **How to Fix This**

### 1. **Align your build version with SemVer (and match what tofu expects):**

- Instead of exporting `VERSION_PIP_ID="0_0_${CI_PIPELINE_ID}"`,  
  **export** `VERSION_PIP_ID="${CI_PIPELINE_ID}.0.0"` **(or similar, based on what versioning your registry and tofu expects)**

  Example:
  - If `CI_PIPELINE_ID=3987`, use `3987.0.0`
  - Your artifacts would be:
    - `terraform-provider-oase_3987.0.0_linux_amd64.zip` etc.

- **Update build, package, upload, and registry logic accordingly.**

### 2. **Ensure registry index returns correct versions:**

- Your registry should publish exactly the SemVer-style versions tofu expects, e.g.,
  - `/v1/providers/nearcloud/oase/versions` must show recent versions like `"3987.0.0"`, not `"0_0_3987"`.

### 3. **Do not use underscores in version numbers for provider registries:**  
  - Use only dots and follow the [Terraform Provider Protocol](https://www.terraform.io/internals/provider-registry-protocol).

### 4. **Make artifact filenames match:**  
  - Must match:  
    `terraform-provider-<NAME>_<VERSION>_<OS>_<ARCH>.zip`

  Example:  
  `terraform-provider-oase_3987.0.0_linux_amd64.zip`

***

## **Summary Table**

| Component    | Current           | Should be      |
|--------------|-------------------|---------------|
| Build ver    | 0_0_3987          | 3987.0.0      |
| Artifact name| ..._0_0_3987_...  | ..._3987.0.0_... |
| Registry     | /v1/providers/.../versions includes 3924.0.0 | .../versions includes 3987.0.0 |
| tofu expects | v3924.0.0         | v3987.0.0     |

***

## **Action Plan (Do This!):**

1. **Switch to true SemVer in version and filenames:**
   - Set `VERSION_PIP_ID="${CI_PIPELINE_ID}.0.0"`
   - All build/upload logic uses this version.
2. **Clean up old registry artifacts if needed (to avoid bogus versions showing up in `/versions`).**
3. **Re-run your pipeline.**
4. **e2e-test will pass when tofu finds the right version for your platform.**

***

**Tip:**  
If you must automate this via CI, do something like:

```bash
export CI_MAJOR=${CI_PIPELINE_ID}
export CI_MINOR=0
export CI_PATCH=0
export VERSION_PIP_ID="${CI_MAJOR}.${CI_MINOR}.${CI_PATCH}"
```

And update all places in your build/upload to use this `${VERSION_PIP_ID}`.

***

Let me know if you want a code/Makefile or script snippet for this versioning fix!

