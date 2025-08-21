var filePattern = regexp.MustCompile(
	`^([A-Za-z0-9\-]+)_([0-9]+_[0-9]+_[0-9]+)_([A-Za-z0-9]+)_([A-Za-z0-9]+)\.([A-Za-z0-9]+)(?:\.([A-Za-z0-9]+))?$`,
)






This failure makes sense now 🔎

From your log:

INFO Saving file path=.../terraform-provider-oase/0/terraform-provider-oase_0_0_939_linux_amd64.zip
WARN Unexpected file type encountered filename=terraform-provider-oase_0_0_939_linux_amd64.zip
...
INFO found same stuff as followed->  version=0

👉 The service is parsing the version as just 0, instead of the full 0_0_939.
That’s why it’s writing to .../0/ instead of .../0_0_939/, and your test assertions fail.


---

Where it goes wrong

Your regex (or filename parser) splits the version into parts (0, 0, 939)

But your service code is only using the first part (0) as the version string

So the test checks for 0_0_939/ but only finds 0/



---

Two Fix Options

✅ Option A (Regex Fix — simpler & matches your test expectations)

Change the regex so the whole version is captured as one group:

var filePattern = regexp.MustCompile(
	`^([A-Za-z0-9\-]+)_([0-9]+_[0-9]+_[0-9]+)_([A-Za-z0-9]+)_([A-Za-z0-9]+)\.([A-Za-z0-9]+)(?:\.([A-Za-z0-9]+))?$`,
)

Groups become:

1. provider → terraform-provider-oase


2. version  → 0_0_939


3. os       → linux


4. arch     → amd64


5. ext      → zip or SHA256SUMS


6. sig      → optional sig



This way the service will store into:

terraform-provider-oase/0_0_939/...

and your checkFileExist tests will pass.


---

✅ Option B (Code Fix — keep old regex)

If you want to keep version split into 3 parts (0, 0, 939), then you must update the service code to combine them:

version := fmt.Sprintf("%s_%s_%s", major, minor, patch)

Right now it looks like the code just takes major and ignores the rest.


---

Recommendation

Since your test is already written to expect 0_0_939/, the regex fix (Option A) is the cleanest solution.


---

Would you like me to patch the regex everywhere in your Go code so it captures the full version string (Option A), or would you prefer I show you where in the service code you can join the major, minor, patch parts (Option B)?

