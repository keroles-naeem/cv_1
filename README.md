CV

const PATTERN = `^([A-Za-z0-9]+\-[A-Za-z0-9]+\-[A-Za-z0-9]+)\_([A-Za-z0-9]+)\_([A-Za-z0-9]+)\_([A-Za-z0-9]+)\.([A-Za-z0-9]+)\.?([A-Za-z0-9]*)`
$ curl -k --location 'https://tfp.entw.oase.itz.itzbund.net/upload' -F "files=@bin/release/0_0_${CI_PIPELINE_ID}/terraform-provider-oase_0_0_${CI_PIPELINE_ID}_linux_amd64.SHA256SUMS" -F "files=@bin/release/0_0_${CI_PIPELINE_ID}/terraform-provider-oase_0_0_${CI_PIPELINE_ID}_linux_amd64.SHA256SUMS.sig" -F "files=@bin/release/0_0_${CI_PIPELINE_ID}/terraform-provider-oase_0_0_${CI_PIPELINE_ID}_linux_amd64.zip" -F "gpg-key=@bin/release/0_0_${CI_PIPELINE_ID}/key.gpg"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 5456k  100    76  100 5456k    783  54.9M --:--:-- --:--:-- --:--:-- 55.5M
Invalid file format: terraform-provider-oase_0_0_3955_linux_amd64.SHA256SUMS
Cleaning up project directory and file based variables
