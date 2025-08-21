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
