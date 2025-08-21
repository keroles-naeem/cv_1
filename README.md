Thanks for sharing the test code 🙏.
From what you pasted, nothing in the TestUploadArchFiles test itself looks wrong — the problem is very likely with the regex that validates your uploaded filenames.

Your file:

terraform-provider-oase_0_0_939_linux_amd64.zip

Expected storage path from your test assertions:

upload_results/
  terraform-provider-oase/
    terraform-provider-oase.json
    0_0_939/
      key.gpg
      terraform-provider-oase_0_0_939_linux_amd64.SHA256SUMS
      terraform-provider-oase_0_0_939_linux_amd64.SHA256SUMS.sig
      terraform-provider-oase_0_0_939_linux_amd64.zip

So the backend must:

1. Parse the file name into → name (terraform-provider-oase), version (0_0_939), os (linux), arch (amd64), ext (zip), maybe .sig.


2. Use that parsed info to decide where to put the file.




---

Where it breaks

Your original regex only allowed 3 underscore groups after the name, but your file has 5:

terraform-provider-oase _ 0 _ 0 _ 939 _ linux _ amd64 .zip

That’s why the backend says invalid file format.


---

Correct Go Regex

For your filenames, the right regex should be:

var filePattern = regexp.MustCompile(
	`^([A-Za-z0-9\-]+)_([0-9]+)_([0-9]+)_([0-9]+)_([A-Za-z0-9]+)_([A-Za-z0-9]+)\.([A-Za-z0-9]+)(?:\.([A-Za-z0-9]+))?$`,
)

Breakdown:

([A-Za-z0-9\-]+) → provider name (terraform-provider-oase)

_([0-9]+) → major (0)

_([0-9]+) → minor (0)

_([0-9]+) → patch/build (939)

_([A-Za-z0-9]+) → os (linux)

_([A-Za-z0-9]+) → arch (amd64)

\.([A-Za-z0-9]+) → extension (zip, SHA256SUMS)

(?:\.([A-Za-z0-9]+))? → optional .sig



---

How to Use in Your Test Code

You likely have something like:

if !filePattern.MatchString(filename) {
    return fmt.Errorf("invalid file format: %s", filename)
}

Replace the old PATTERN with the one above, and your test will pass for:

terraform-provider-oase_0_0_939_linux_amd64.zip

terraform-provider-oase_0_0_939_linux_amd64.SHA256SUMS

terraform-provider-oase_0_0_939_linux_amd64.SHA256SUMS.sig



---

👉 Do you want me to also adjust the regex so it supports any provider name (like terraform-provider-aws, terraform-provider-azure, etc.), or should it stay locked to only terraform-provider-oase?

