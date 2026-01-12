---
---

## Summary
Playbooks are the core of Ansible's configuration management. Unlike ad-hoc commands, Playbooks are YAML files that describe a policy to be enforced on your remote systems. They allow for complex orchestration, variables, and multi-step deployments.

## Detailed Explanation

### Structure
A Playbook consists of one or more **Plays**. A Play maps a group of **Hosts** to a list of **Tasks**.

```yaml
---
- name: Configure Webservers        # Play Name
  hosts: webservers                 # Target Group
  become: yes                       # Run as sudo
  
  vars:                             # Variables
    http_port: 80

  tasks:                            # List of Tasks
    - name: Install Nginx
      apt:
        name: nginx
        state: present

    - name: Start Nginx
      service:
        name: nginx
        state: started
```

### YAML Syntax Rules
*   Start with `---`.
*   Indentation is critical (usually 2 spaces). Tabs are forbidden.
*   Lists use hyphens `- item`.
*   Dictionaries use `key: value`.

## Go-Specific Context/Examples

If you are building a tool in Go to lint or analyze Ansible playbooks, you can use `gopkg.in/yaml.v3`.

### Example: Parsing a Playbook in Go

```go
package main

import (
	"fmt"
	"log"
	"os"

	"gopkg.in/yaml.v3"
)

type Task struct {
	Name string `yaml:"name"`
	Apt  map[string]interface{} `yaml:"apt,omitempty"`
}

type Play struct {
	Name  string `yaml:"name"`
	Hosts string `yaml:"hosts"`
	Tasks []Task `yaml:"tasks"`
}

func main() {
	data, _ := os.ReadFile("playbook.yml")
	
	var playbook []Play
	err := yaml.Unmarshal(data, &playbook)
	if err != nil {
		log.Fatal(err)
	}

	for _, play := range playbook {
		fmt.Printf("Play: %s (Targets: %s)\n", play.Name, play.Hosts)
		for _, task := range play.Tasks {
			fmt.Printf(" - Task: %s\n", task.Name)
		}
	}
}
```

## Interview Questions

**Q: What is the difference between a Play and a Playbook?**
**A:** A **Playbook** is the file (YAML) that contains the code. A **Play** is a logical section inside that file that targets a specific group of hosts. A Playbook can contain multiple Plays (e.g., one play for DB servers, followed by one play for Web servers).

**Q: How do you verify the syntax of a playbook without running it?**
**A:** Run `ansible-playbook --syntax-check myplaybook.yml`.

**Q: Why is YAML used instead of JSON?**
**A:** YAML is designed to be human-readable and supports comments, which are essential for documenting infrastructure code. JSON is harder to read/write manually and doesn't support comments.
