var filePattern = regexp.MustCompile(
	`^([A-Za-z0-9\-]+)_([0-9]+)_([0-9]+)_([0-9]+)_([A-Za-z0-9]+)_([A-Za-z0-9]+)\.([A-Za-z0-9]+)(?:\.([A-Za-z0-9]+))?$`,
)
adjust the regx for variables like 
terraform-provider-oase_0_0_939_linux_am d64.zip

terraform-provider-oase_0_0_939_linux_am d64. SHA256SUMS

terraform-provider-oase_0_0_939_linux_am d64. SHA256SUMS. sig

Got it — your current regex only matches compact names with underscores and doesn’t account for things like spaces (`am d64`) or the additional suffixes like `.SHA256SUMS(.sig)`.

Let’s break down your new examples:

- `terraform-provider-oase_0_0_939_linux_am d64.zip`
- `terraform-provider-oase_0_0_939_linux_am d64.SHA256SUMS`
- `terraform-provider-oase_0_0_939_linux_am d64.SHA256SUMS.sig`

### Problems with your current regex
1. `[A-Za-z0-9\-]+` only matches alphanumerics and dash — won’t match `am d64` because of the space.
2. Your extension handling assumes a single file extension plus maybe a second, but in cases like SHA256SUMS + `.sig` there are more possibilities.
3. You need to allow for both `.zip` as well as `SHA256SUMS[.sig]`.

***

### Updated regex  
Here’s one that handles:
- Spaces in the last captured “platform/arch” field
- Either `.zip` or `.SHA256SUMS[.sig]`

```go
var filePattern = regexp.MustCompile(
	`^([A-Za-z0-9\-]+)_([0-9]+)_([0-9]+)_([0-9]+)_([A-Za-z0-9]+)_([A-Za-z0-9 ]+)\.(zip|SHA256SUMS(?:\.sig)?)$`,
)
```

### Explanation
- `([A-Za-z0-9\-]+)` → project name (`terraform-provider-oase`)
- `_([0-9]+)` x3 → semantic version parts (0,0,939)
- `_([A-Za-z0-9]+)` → os (`linux`)
- `_([A-Za-z0-9 ]+)` → architecture, now allows spaces (`am d64`)
- `\.(zip|SHA256SUMS(?:\.sig)?)` → matches:
  * `.zip`
  * `.SHA256SUMS`
  * `.SHA256SUMS.sig`

***

### Example Matches

- ✅ `terraform-provider-oase_0_0_939_linux_am d64.zip`
- ✅ `terraform-provider-oase_0_0_939_linux_am d64.SHA256SUMS`
- ✅ `terraform-provider-oase_0_0_939_linux_am d64.SHA256SUMS.sig`

***

👉 Do you also expect other extensions (like `.tar.gz`, `.tgz`), or is it strictly `.zip` and checksum files?

