=== Failed
=== FAIL: internal/registry TestUploadArchFiles (1.02s)
2025/08/21 09:23:01 INFO Repository root path path=../../test-data/upload_results/
2025/08/21 09:23:01 INFO Saving file path=../../test-data/upload_results/terraform-provider-oase/0/terraform-provider-oase_0_0_939_linux_amd64.zip
2025/08/21 09:23:01 WARN Unexpected file type encountered filename=terraform-provider-oase_0_0_939_linux_amd64.zip
2025/08/21 09:23:01 INFO Saving file path=../../test-data/upload_results/terraform-provider-oase/0/terraform-provider-oase_0_0_939_linux_amd64.SHA256SUMS.sig
2025/08/21 09:23:01 WARN Unexpected file type encountered filename=terraform-provider-oase_0_0_939_linux_amd64.SHA256SUMS.sig
2025/08/21 09:23:01 INFO Saving file path=../../test-data/upload_results/terraform-provider-oase/0/terraform-provider-oase_0_0_939_linux_amd64.SHA256SUMS
2025/08/21 09:23:01 WARN Unexpected file type encountered filename=terraform-provider-oase_0_0_939_linux_amd64.SHA256SUMS
2025/08/21 09:23:01 INFO Saving GPG file path=../../test-data/upload_results/terraform-provider-oase/0/key.gpg
2025/08/21 09:23:01 INFO Cleaning up lower versions if necessary
2025/08/21 09:23:01 INFO Writing metadata JSON file path=../../test-data/upload_results/terraform-provider-oase/terraform-provider-oase.json
[GIN] 2025/08/21 - 09:23:01 | 200 |   213.11475ms |                 | POST     "/upload"
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
2025/08/21 09:23:01 INFO Repository root path path=../../test-data/upload_results/
2025/08/21 09:23:01 INFO Saving file path=../../test-data/upload_results/terraform-provider-oase/0/terraform-provider-oase_0_0_940_linux_amd64.zip
2025/08/21 09:23:01 WARN Unexpected file type encountered filename=terraform-provider-oase_0_0_940_linux_amd64.zip
2025/08/21 09:23:01 INFO Saving file path=../../test-data/upload_results/terraform-provider-oase/0/terraform-provider-oase_0_0_940_linux_amd64.SHA256SUMS.sig
2025/08/21 09:23:01 WARN Unexpected file type encountered filename=terraform-provider-oase_0_0_940_linux_amd64.SHA256SUMS.sig
2025/08/21 09:23:01 INFO Saving file path=../../test-data/upload_results/terraform-provider-oase/0/terraform-provider-oase_0_0_940_linux_amd64.SHA256SUMS
2025/08/21 09:23:01 WARN Unexpected file type encountered filename=terraform-provider-oase_0_0_940_linux_amd64.SHA256SUMS
2025/08/21 09:23:01 INFO Saving GPG file path=../../test-data/upload_results/terraform-provider-oase/0/key_0_0_940.gpg
2025/08/21 09:23:01 INFO found same stuff as followed->  version=0
2025/08/21 09:23:01 INFO Cleaning up lower versions if necessary
2025/08/21 09:23:01 INFO Writing metadata JSON file path=../../test-data/upload_results/terraform-provider-oase/terraform-provider-oase.json
[GIN] 2025/08/21 - 09:23:01 | 200 |   199.52301ms |                 | POST     "/upload"
DONE 36 tests, 1 failure in 2.059s
