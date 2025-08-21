package main

import (
	"fmt"
	"regexp"
)

func main() {
	// name_major_minor_build_os_arch.ext[.sig]
	pattern := `^([A-Za-z0-9\-]+)_([0-9]+)_([0-9]+)_([0-9]+)_([A-Za-z0-9]+)_([A-Za-z0-9]+)\.([A-Za-z0-9]+)(?:\.([A-Za-z0-9]+))?$`
	re := regexp.MustCompile(pattern)

	tests := []string{
		"terraform-provider-oase_0_0_3955_linux_amd64.SHA256SUMS",
		"terraform-provider-oase_0_0_3955_linux_amd64.SHA256SUMS.sig",
		"terraform-provider-oase_0_0_3955_linux_amd64.zip",
	}

	for _, t := range tests {
		if m := re.FindStringSubmatch(t); m != nil {
			fmt.Println("✅ Match:", t)
			fmt.Println("  name:   ", m[1])
			fmt.Println("  major:  ", m[2])
			fmt.Println("  minor:  ", m[3])
			fmt.Println("  build:  ", m[4])
			fmt.Println("  os:     ", m[5])
			fmt.Println("  arch:   ", m[6])
			fmt.Println("  ext:    ", m[7])
			if m[8] != "" {
				fmt.Println("  sig:    ", m[8])
			}
		} else {
			fmt.Println("❌ No Match:", t)
		}
	}
}