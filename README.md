Got it 👍 The error comes from your regex pattern not matching your uploaded file names.
Let’s compare:

Your uploaded file name looks like this:

terraform-provider-oase_0_0_3955_linux_amd64.SHA256SUMS

But your regex:

^([A-Za-z0-9]+\-[A-Za-z0-9]+\-[A-Za-z0-9]+)\_([A-Za-z0-9]+)\_([A-Za-z0-9]+)\_([A-Za-z0-9]+)\.([A-Za-z0-9]+)\.?([A-Za-z0-9]*)

expects:

[some-part]-[some-part]-[some-part]_[part]_[part]_[part].[part].[maybe more]

Issue

Your file starts with terraform-provider-oase (3 hyphen-separated words), but your regex only allows exactly 3 groups of alphanum separated by -. That part is fine.

Then it expects exactly 3 underscore-separated fields, but your file has 4:

terraform-provider-oase _ 0 _ 0 _ 3955 _ linux _ amd64 .SHA256SUMS


So your regex is too strict.


---

Fixed Pattern

Try this one:

const PATTERN = /^([A-Za-z0-9\-]+)(?:_([A-Za-z0-9]+))+\.([A-Za-z0-9]+)(?:\.([A-Za-z0-9]+))?$/;

✔ Matches:

terraform-provider-oase_0_0_3955_linux_amd64.SHA256SUMS

terraform-provider-oase_0_0_3955_linux_amd64.SHA256SUMS.sig

terraform-provider-oase_0_0_3955_linux_amd64.zip



---

Explanation

^([A-Za-z0-9\-]+) → terraform-provider-oase

(?:_([A-Za-z0-9]+))+ → multiple _fields (0, 0, 3955, linux, amd64)

\.([A-Za-z0-9]+) → file extension (SHA256SUMS, zip)

(?:\.([A-Za-z0-9]+))? → optional second extension (sig)

$ → end of string



---

👉 Do you want me to keep strict positions (like always version_major_minor_build_os_arch.ext), or just make it flexible enough to accept any number of underscores?

