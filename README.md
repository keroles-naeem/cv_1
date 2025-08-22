func (uploadService UploadService) initMetaContainer(c *gin.Context, metaFileName, version, versionPath string) (VersionsContainer, error) {
	jsonFile, err := os.Open(metaFileName)
	if err != nil {
		if errors.Is(err, fs.ErrNotExist) {
			return VersionsContainer{}, nil
		}
		c.String(http.StatusBadRequest, "got meta file parse / open err: %s", err.Error())
		return VersionsContainer{}, err
	}

	defer jsonFile.Close()

	byteValue, err := io.ReadAll(jsonFile)
	if err != nil {
		return VersionsContainer{}, err
	}

	var versionsContainer VersionsContainer
	err = json.Unmarshal(byteValue, &versionsContainer)
	if err != nil {
		return VersionsContainer{}, err
	}

	vi := -1
	for i, item := range versionsContainer.Versions {
		if item.Version == version {
			vi = i
		}
	}

	if vi >= 0 {
		slog.Info("found same stuff as followed-> ", "version", versionsContainer.Versions[vi].Version)
		uploadService.cleanUpLowerVersion(versionPath, &versionsContainer.Versions)
	}

	uploadService.cleanUpLowerVersion(versionPath, &versionsContainer.Versions)

	return versionsContainer, nil
}

func (uploadService UploadService) cleanUpLowerVersion(versionPath string, versionContainers *[]VersionContainer) {
	// if len(*versionContainers) <= MAX_VERSIONS {
	// 	return
	// }
	// Sort versions in ascending order
	sort.Slice(*versionContainers, func(i, j int) bool {
		vi, errI := strconv.Atoi((*versionContainers)[i].Version)
		vj, errJ := strconv.Atoi((*versionContainers)[j].Version)
		if errI != nil || errJ != nil {
			return (*versionContainers)[i].Version < (*versionContainers)[j].Version
		}
		return vi < vj
	})
	*versionContainers = removeDuplicates(*versionContainers)

	// Remove older versions until we reach MAX_VERSIONS
	for len(*versionContainers) > MAX_VERSIONS {
		oldestVersion := (*versionContainers)[0]
		slog.Info("Deleting oldest version", "version", oldestVersion.Version)
		uploadService.cleanUpVersionAtPosition(0, versionPath, versionContainers)
	}
}

	t.Run("Mixed Numeric and Semantic Versions", func(t *testing.T) {
		versions := []VersionContainer{
			{Version: "0_0_1"},
			{Version: "0_0_2"},
			{Version: "0_0_2"}, // Duplicate
			{Version: "0_0_10"},
			{Version: "0_0_150"},
			{Version: "0_0_150"}, // Duplicate
		}
		uploadService.cleanUpLowerVersion(versionPath, &versions)
		if MAX_VERSIONS <= 4{

		assert.Len(t, versions, MAX_VERSIONS, "Should have MAX_VERSIONS versions after cleanup")
		}else{
			assert.Len(t, versions, 4, "Should have MAX_VERSIONS versions after cleanup")

		}
		// Check for uniqueness
		uniqueVersions := make(map[string]bool)
		for _, v := range versions {
			assert.False(t, uniqueVersions[v.Version], "Version %s should not be duplicate", v.Version)
			uniqueVersions[v.Version] = true
		}
		if MAX_VERSIONS <= 4{

			assert.Equal(t, "0_0_10", versions[MAX_VERSIONS-1].Version, "First version should be 0_0_10")
			assert.Equal(t, "0_0_150", versions[MAX_VERSIONS].Version, "Second version should be 0_0_150")
		}else{
			assert.Equal(t, "0_0_2", versions[2].Version, "First version should be 0_0_2")
			assert.Equal(t, "0_0_10", versions[3].Version, "Second version should be 0_0_10")
		}
	})

: internal/registry TestUploadService_CleanUpLowerVersion_DuplicateVersions/Mixed_Numeric_and_Semantic_Versions (0.00s)
2025/08/22 05:46:12 INFO Deleting oldest version version=0_0_1
    upload_service_test.go:371: 
        	Error Trace:	/builds/oase/tp-registry/internal/registry/upload_service_test.go:371
        	Error:      	Not equal: 
        	            	expected: "0_0_10"
        	            	actual  : "0_0_2"
        	            	
        	            	Diff:
        	            	--- Expected
        	            	+++ Actual
        	            	@@ -1 +1 @@
        	            	-0_0_10
        	            	+0_0_2
        	Test:       	TestUploadService_CleanUpLowerVersion_DuplicateVersions/Mixed_Numeric_and_Semantic_Versions
        	Messages:   	First version should be 0_0_10
=== FAIL: internal/registry TestUploadService_CleanUpLowerVersion_DuplicateVersions (0.00s)
panic: runtime error: index out of range [3] with length 3 [recovered]
	panic: runtime error: index out of range [3] with length 3
goroutine 96 [running]:
testing.tRunner.func1.2({0x9510c0, 0xc000388a98})
	/usr/local/go/src/testing/testing.go:1631 +0x3f7
testing.tRunner.func1()
	/usr/local/go/src/testing/testing.go:1634 +0x6b6
panic({0x9510c0?, 0xc000388a98?})
	/usr/local/go/src/runtime/panic.go:770 +0x132
tp-registry/internal/registry.TestUploadService_CleanUpLowerVersion_DuplicateVersions.func2(0xc0003324e0)
	/builds/oase/tp-registry/internal/registry/upload_service_test.go:372 +0x5f8
testing.tRunner(0xc0003324e0, 0xc000016b40)
	/usr/local/go/src/testing/testing.go:1689 +0x21f
created by testing.(*T).Run in goroutine 94
	/usr/local/go/src/testing/testing.go:1742 +0x826
