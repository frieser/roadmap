---
---

## Summary
Jinja2 is the templating engine used by Ansible. It allows you to write dynamic files (like configuration files) by injecting variables and using logic (loops/conditionals). It is also used inside Playbooks for variable substitution (e.g., `{{ var }}`).

## Detailed Explanation

### Syntax
*   **`{{ variable }}`**: Output a variable (Print).
*   **`{% control %}`**: Logic (Loop/If).
*   **`{# comment #}`**: Comments (not rendered).

### Control Structures
```jinja
# Loop
{% for host in groups['webservers'] %}
server {{ host }};
{% endfor %}

# Conditional
{% if ansible_os_family == 'Debian' %}
user www-data;
{% else %}
user apache;
{% endif %}
```

### Filters
Filters transform data using pipes `|`.
*   `{{ my_var | default('fallback') }}`: Use default if undefined.
*   `{{ my_list | join(', ') }}`: Join list into string.
*   `{{ my_path | basename }}`: Get filename from path.

## Go-Specific Context/Examples

Jinja2 is very similar to Go's `text/template` (and `html/template`).

### Analogy
**Jinja2**: `Hello {{ name | upper }}`
**Go**: `Hello {{ .Name | ToUpper }}`

Both use double curly braces `{{ }}` and a pipeline concept for functions/filters.

## Interview Questions

**Q: What is the difference between `default(value)` and `default(value, true)`?**
**A:** `default('foo')` uses the default only if the variable is *undefined*. `default('foo', true)` uses the default if the variable is undefined OR if it evaluates to *false/empty string* (boolean true = "use default if it evaluates to false").

**Q: How do you prevent newlines from being added in loops?**
**A:** Use whitespace control characters `{%-` and `-%}`.
`{%- for item in list -%}` removes whitespace before/after the block.

**Q: Can you execute arbitrary Python code in Jinja2?**
**A:** No. Jinja2 is sandboxed. You can only use the variables passed to the template and the registered filters/tests.
