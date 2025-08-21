FAIL internal/registry (coverage: 79.8% of statements)
=== Failed
=== FAIL: internal/registry TestUploadArchFiles (0.82s)
2025/08/21 09:13:24 INFO Repository root path path=../../test-data/upload_results/
2025/08/21 09:13:24 INFO Saving file path=../../test-data/upload_results/terraform-provider-oase/0/terraform-provider-oase_0_0_939_linux_amd64.zip
2025/08/21 09:13:24 WARN Unexpected file type encountered filename=terraform-provider-oase_0_0_939_linux_amd64.zip
2025/08/21 09:13:24 INFO Saving file path=../../test-data/upload_results/terraform-provider-oase/0/terraform-provider-oase_0_0_939_linux_amd64.SHA256SUMS.sig
2025/08/21 09:13:24 WARN Unexpected file type encountered filename=terraform-provider-oase_0_0_939_linux_amd64.SHA256SUMS.sig
2025/08/21 09:13:24 INFO Saving file path=../../test-data/upload_results/terraform-provider-oase/0/terraform-provider-oase_0_0_939_linux_amd64.SHA256SUMS
2025/08/21 09:13:24 WARN Unexpected file type encountered filename=terraform-provider-oase_0_0_939_linux_amd64.SHA256SUMS
2025/08/21 09:13:24 INFO Saving GPG file path=../../test-data/upload_results/terraform-provider-oase/0/key.gpg
2025/08/21 09:13:24 INFO Cleaning up lower versions if necessary
2025/08/21 09:13:24 INFO Writing metadata JSON file path=../../test-data/upload_results/terraform-provider-oase/terraform-provider-oase.json
[GIN] 2025/08/21 - 09:13:24 | 200 |  187.863391ms |                 | POST     "/upload"
    upload_service_test.go:80: 
        	Error Trace:	/builds/oase/tp-registry/internal/registry/upload_service_test.go:80
        	Error:      	Should be true
        	Test:       	TestUploadArchFiles
    upload_service_test.go:81: 
        	Error Trace:	/builds/oase/tp-registry/internal/registry/upload_service_test.go:81
        	Error:      	Should be true
        	Test:       	TestUploadArchFiles
    upload_service_test.go:82: 
        	Error Trace:	/builds/oase/tp-registry/internal/registry/upload_service_test.go:82
        	Error:      	Should be true
        	Test:       	TestUploadArchFiles
    upload_service_test.go:83: 
        	Error Trace:	/builds/oase/tp-registry/internal/registry/upload_service_test.go:83
        	Error:      	Should be true
        	Test:       	TestUploadArchFiles
2025/08/21 09:13:25 INFO Repository root path path=../../test-data/upload_results/
2025/08/21 09:13:25 INFO Saving file path=../../test-data/upload_results/terraform-provider-oase/0/terraform-provider-oase_0_0_940_linux_amd64.zip
2025/08/21 09:13:25 WARN Unexpected file type encountered filename=terraform-provider-oase_0_0_940_linux_amd64.zip
2025/08/21 09:13:25 INFO Saving file path=../../test-data/upload_results/terraform-provider-oase/0/terraform-provider-oase_0_0_940_linux_amd64.SHA256SUMS.sig
2025/08/21 09:13:25 WARN Unexpected file type encountered filename=terraform-provider-oase_0_0_940_linux_amd64.SHA256SUMS.sig
2025/08/21 09:13:25 INFO Saving file path=../../test-data/upload_results/terraform-provider-oase/0/terraform-provider-oase_0_0_940_linux_amd64.SHA256SUMS
2025/08/21 09:13:25 WARN Unexpected file type encountered filename=terraform-provider-oase_0_0_940_linux_amd64.SHA256SUMS
2025/08/21 09:13:25 INFO Saving GPG file path=../../test-data/upload_results/terraform-provider-oase/0/key_0_0_940.gpg
2025/08/21 09:13:25 INFO found same stuff as followed->  version=0
2025/08/21 09:13:25 INFO Cleaning up lower versions if necessary
2025/08/21 09:13:25 INFO Writing metadata JSON file path=../../test-data/upload_results/terraform-provider-oase/terraform-provider-oase.json
[GIN] 2025/08/21 - 09:13:25 | 200 |  248.568606ms |                 | POST     "/upload"
DONE 36 tests, 1 failure in 1.867s
Uploading artifacts for failed job

func TestUploadArchFiles(t *testing.T) {
	uploadPath := "../../test-data/upload_results/"

	os.RemoveAll(uploadPath)

	SetEnv("DOWNLOAD_URL", "")
	SetEnv("REPOSITORY_PATH", uploadPath)
	defer UnSetEnv("DOWNLOAD_URL")
	defer UnSetEnv("REPOSITORY_PATH")

	engine := SetupEngine()

	var files = map[string][]string{
		"files":   {"../../test-data/upload/terraform-provider-oase_0_0_939_linux_amd64.zip", "../../test-data/upload/terraform-provider-oase_0_0_939_linux_amd64.SHA256SUMS.sig", "../../test-data/upload/terraform-provider-oase_0_0_939_linux_amd64.SHA256SUMS"},
		"gpg-key": {"../../test-data/upload/key.gpg"},
	}

	request, err := newfileUploadRequest("/upload", files)
	if err != nil {
		log.Fatal(err)
	}

	w := httptest.NewRecorder()
	engine.ServeHTTP(w, request)

	assert.Equal(t, http.StatusOK, w.Code)

	assert.True(t, checkFileExist(uploadPath+"terraform-provider-oase/terraform-provider-oase.json"))
	assert.True(t, checkFileExist(uploadPath+"terraform-provider-oase/0_0_939/key.gpg"))
	assert.True(t, checkFileExist(uploadPath+"terraform-provider-oase/0_0_939/terraform-provider-oase_0_0_939_linux_amd64.SHA256SUMS"))
	assert.True(t, checkFileExist(uploadPath+"terraform-provider-oase/0_0_939/terraform-provider-oase_0_0_939_linux_amd64.SHA256SUMS.sig"))
	assert.True(t, checkFileExist(uploadPath+"terraform-provider-oase/0_0_939/terraform-provider-oase_0_0_939_linux_amd64.zip"))

	var filesWithNextVersion = map[string][]string{
		"files":   {"../../test-data/upload/terraform-provider-oase_0_0_940_linux_amd64.zip", "../../test-data/upload/terraform-provider-oase_0_0_940_linux_amd64.SHA256SUMS.sig", "../../test-data/upload/terraform-provider-oase_0_0_940_linux_amd64.SHA256SUMS"},
		"gpg-key": {"../../test-data/upload/key_0_0_940.gpg"},
	}

	request, err = newfileUploadRequest("/upload", filesWithNextVersion)
	if err != nil {
		log.Fatal(err)
	}

	w = httptest.NewRecorder()
	engine.ServeHTTP(w, request)

	assert.Equal(t, http.StatusOK, w.Code)

	os.RemoveAll(uploadPath)
}

Perfect, thanks for posting the error log + the test.  
The root symptom is visible here in your logs:

```
WARN Unexpected file type encountered filename=terraform-provider-oase_0_0_939_linux_amd64.zip
```

That means your **regex** (or file type detection function) did not recognize `.zip`, `.SHA256SUMS` or `.SHA256SUMS.sig` as valid artifacts.  
Because of that, the handler stored them into a wrong directory (`.../0/`) instead of per-version (`.../0_0_939/`), and the later `assert.True(checkFileExist(...))` failed.

***

### Why it fails

Looking back at your test:

```go
assert.True(t, checkFileExist(uploadPath+"terraform-provider-oase/terraform-provider-oase.json"))
assert.True(t, checkFileExist(uploadPath+"terraform-provider-oase/0_0_939/key.gpg"))
assert.True(t, checkFileExist(uploadPath+"terraform-provider-oase/0_0_939/terraform-provider-oase_0_0_939_linux_amd64.SHA256SUMS"))
assert.True(t, checkFileExist(uploadPath+"terraform-provider-oase/0_0_939/terraform-provider-oase_0_0_939_linux_amd64.SHA256SUMS.sig"))
assert.True(t, checkFileExist(uploadPath+"terraform-provider-oase/0_0_939/terraform-provider-oase_0_0_939_linux_amd64.zip"))
```

But your server log shows files were being written to:

```
../../test-data/upload_results/terraform-provider-oase/0/...
```

instead of:

```
../../test-data/upload_results/terraform-provider-oase/0_0_939/...
```

So:  
- The regex failed, couldn’t parse the version correctly, defaulted to `0`.
- That’s why you got `.../0/` instead of `.../0_0_939/`.
- The test checks `0_0_939/...`, file not found → `assert.True` failed.

***

### Fix

You need to update your **filename regex parser** to correctly handle the `.zip`, `.SHA256SUMS`, `.SHA256SUMS.sig` cases.  
Here’s an adjusted Go regex pattern:

```go
var filePattern = regexp.MustCompile(
	`^([A-Za-z0-9\-]+)_([0-9]+)_([0-9]+)_([0-9]+)_([A-Za-z0-9]+)_([A-Za-z0-9]+)\.(zip|SHA256SUMS(?:\.sig)?)$`,
)
```

- `[A-Za-z0-9\-]+` → matches provider name (`terraform-provider-oase`)
- `_([0-9]+)_([0-9]+)_([0-9]+)` → matches `0_0_939`
- `_([A-Za-z0-9]+)` → OS (`linux`)
- `_([A-Za-z0-9]+)` → arch (`amd64`, `arm64`, etc.)
- `\.(zip|SHA256SUMS(?:\.sig)?)` → matches `.zip`, `.SHA256SUMS` and `.SHA256SUMS.sig`

If you also need to allow spaces (like your earlier `am d64` typo), you can use:

```go
([A-Za-z0-9 ]+)
```

for the last group instead of `([A-Za-z0-9]+)`.

***

### Next Steps

1. Update the regex in your upload service to the one above.  
2. Re-run `TestUploadArchFiles`.  
   - Files should now be recognized, and placed under `terraform-provider-oase/0_0_939/...`.  
   - The `assert.True(...)` checks will pass.  

***

👉 So the failure isn’t in the **test** — it’s the regex being too strict and rejecting your valid artifact names.  

Do you want me to rewrite your **file parsing function** in full Go code (so you can just drop it in), instead of only giving the regex?

