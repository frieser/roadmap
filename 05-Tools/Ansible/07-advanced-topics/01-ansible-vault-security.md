---
---

## Summary
Ansible Vault allows you to keep sensitive data (passwords, API keys, certificates) in encrypted files within your source control, rather than in plain text. It uses AES-256 encryption. At runtime, you provide a password (or password file) to decrypt the secrets.

## Detailed Explanation

### Basic Commands
1.  **Create Encrypted File**: `ansible-vault create secret.yml`
2.  **Edit**: `ansible-vault edit secret.yml` (Opens editor, handles decrypt/encrypt).
3.  **View**: `ansible-vault view secret.yml`
4.  **Encrypt Existing**: `ansible-vault encrypt plain.yml`
5.  **Decrypt**: `ansible-vault decrypt secret.yml`

### Usage in Playbooks
Usually, you encrypt a variables file (e.g., `group_vars/all/vault.yml`).
To run the playbook:
```bash
ansible-playbook site.yml --ask-vault-pass
# OR
ansible-playbook site.yml --vault-password-file ~/.vault_pass.txt
```

### Encrypting Strings
Instead of encrypting the whole file, you can encrypt just a specific value.
`ansible-vault encrypt_string 'mypassword' --name 'db_password'`
This outputs a YAML block you can paste into a regular playbook.

## Go-Specific Context/Examples

If your Go application needs to read Ansible Vault files (e.g., a custom deployment tool written in Go), you need a way to decrypt them.

### Example: Decrypting Vault in Go
You can wrap the `ansible-vault` command or use a library like `github.com/sosedoff/ansible-vault-go`.

```go
package main

import (
	"fmt"
	"log"

	"github.com/sosedoff/ansible-vault-go"
)

func main() {
	password := "my_secret_password"
	
	// Read encrypted file content usually done here
	// For demo, assume we have the string
	
	err := vault.DecryptFile("encrypted_secrets.yml", "decrypted.yml", password)
	if err != nil {
		log.Fatal(err)
	}
	
	fmt.Println("File decrypted successfully!")
}
```

## Interview Questions

**Q: Can you have multiple vault passwords?**
**A:** Yes. Ansible supports **Vault IDs**. You can have one password for "dev" secrets and another for "prod" secrets.
`ansible-playbook site.yml --vault-id dev@prompt --vault-id prod@.pass_file`

**Q: Is it safe to commit vault files to Git?**
**A:** Yes, assuming your Vault password is strong. AES-256 is industry standard. However, you must ensure the Vault Password itself is **never** committed to Git.

**Q: How do you handle Vault passwords in CI/CD (Jenkins/GitHub Actions)?**
**A:** You typically store the Vault password as a "Secret" in the CI tool (e.g., GitHub Secrets). During the build pipeline, you inject this secret into a temporary file or environment variable so Ansible can read it non-interactively.
