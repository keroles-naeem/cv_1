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

