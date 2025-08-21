what is wrong
file is  is  here
tp-registry\test-data\upload\terraform-provider-oase_0_0_939_linux_amd64.zip

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
