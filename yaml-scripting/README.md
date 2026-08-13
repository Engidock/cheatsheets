# YAML Scripting Cheatsheet

Complete quick reference guide for YAML configuration — syntax, collections, strings, anchors, and real-world examples.

## 🎯 YAML Basics

### Syntax Fundamentals

Key-value pairs:

```yaml
name: John
age: 30
active: true
version: 1.5
```

Comments:

```yaml
# This is a comment
key: value              # Inline comment
# YAML version
%YAML 1.2
```

Data types:

```yaml
string: Hello World
number: 42
float: 3.14
boolean: true
null_value: null
date: 2025-12-16
timestamp: 2025-12-16T10:30:00Z
```

### Indentation & Structure

Proper indentation (use spaces):

```yaml
parent:
  child1: value1
  child2: value2
  nested:
    level3: value3
```

Important rules:

```yaml
# ✓ Use 2 or 4 spaces (not tabs!)
# ✓ Consistent indentation throughout
# ✗ Never use tabs
# ✗ Inconsistent indentation causes errors
# ✓ Empty lines are ignored
# ✓ Keys must be unique at same level
```

## 📦 Collections (Lists & Maps)

### Lists (Arrays)

Basic list:

```yaml
fruits:
  - apple
  - banana
  - orange
# Or inline
colors: [red, green, blue]
```

List of objects:

```yaml
people:
  - name: Alice
    age: 30
    city: NYC
  - name: Bob
    age: 25
    city: LA
```

Nested lists:

```yaml
matrix:
  - [1, 2, 3]
  - [4, 5, 6]
  - [7, 8, 9]
# Or verbose
matrix:
  - - 1
    - 2
    - 3
  - - 4
    - 5
    - 6
```

### Maps (Dictionaries)

Basic map:

```yaml
person:
  name: John
  age: 30
  email: john@example.com
# Or inline
person: {name: John, age: 30, email: john@example.com}
```

Nested maps:

```yaml
company:
  name: TechCorp
  address:
    street: 123 Main St
    city: NYC
    zip: 10001
  employees:
    - name: Alice
      role: Engineer
```

## 📝 Strings & Text

### String Types

Simple strings:

```yaml
simple: hello
quoted: "hello world"
single: 'single quoted'
```

Multiline strings (literal):

```yaml
description: |
  This is a literal block.
  It preserves newlines.
  And indentation matters.
```

Multiline strings (folded):

```yaml
summary: >
  This is a folded block.
  Long lines are folded into
  a single line.
  Unless there's a blank line.

  New paragraph starts here.
```

Special characters:

```yaml
path: C:\Users\name\file.txt
quote: "He said: \"Hello\""
newline: "Line 1\nLine 2"
special: "~!@#$%^&*()"
```

## 🔗 Anchors & Aliases

### Reference & Reuse

Anchors & aliases:

```yaml
defaults: &default-settings
  timeout: 30
  retries: 3
  log_level: info
production:
  <<: *default-settings
  timeout: 60
staging:
  <<: *default-settings
```

Merge keys:

```yaml
base: &base
  name: myapp
  version: 1.0
config:
  <<: *base
  environment: production
  port: 8080
```

## ⚡ Advanced Features

### Advanced Techniques

Quoted keys:

```yaml
"key with spaces": value
"key:with:colons": value
'single:quoted:key': value
```

Complex values:

```yaml
list_of_maps:
  - id: 1
    name: Item1
  - id: 2
    name: Item2
map_of_lists:
  colors:
    - red
    - green
    - blue
  numbers:
    - 1
    - 2
    - 3
```

Tags (type hints):

```yaml
string: !!str 123
integer: !!int "456"
float: !!float 3.14
boolean: !!bool yes
null: !!null
```

## 🏗️ Real-World Examples

### Common Use Cases

Docker Compose style:

```yaml
version: '3.8'
services:
  web:
    image: nginx:latest
    ports:
      - "80:80"
    environment:
      - ENVIRONMENT=production
    volumes:
      - ./html:/usr/share/nginx/html

  db:
    image: mysql:5.7
    environment:
      - MYSQL_ROOT_PASSWORD=secret
    ports:
      - "3306:3306"
```

Kubernetes manifest:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: myapp
spec:
  containers:
  - name: container1
    image: myimage:1.0
    ports:
    - containerPort: 8080
    env:
    - name: ENV_VAR
      value: "value"
```

Configuration file:

```yaml
app:
  name: MyApp
  version: 1.0.0
database:
  host: localhost
  port: 5432
  username: admin
  password: secret

server:
  host: 0.0.0.0
  port: 8080
  workers: 4

logging:
  level: INFO
  format: json
```

## ✅ Best Practices

- **Use consistent indentation** — 2 or 4 spaces, never tabs
- **Quote strings with special chars** — use quotes for safety
- **Use anchors for DRY** — don't repeat yourself with aliases
- **Clear structure** — organize logically with comments
- **Validate YAML** — use linters before deploying
- **Use maps for config** — better than nested lists

### Common Issues & Tips

**⚠️ Indentation issues:**

- Use spaces, not tabs
- Consistent indentation required
- Each level adds 2-4 spaces

**⚠️ Boolean values:**

- `true`/`false` (lowercase)
- `yes`/`no` (lowercase)
- Quote if you need a literal string

**⚠️ Numbers as strings:**

- Quote numbers that should be strings
- `"123"` is a string, `123` is an integer

**💡 Validation:**

- Online: `yamllint.com`
- CLI: `yamllint file.yaml`
- Python: `yaml.safe_load()`

## 📋 Quick Reference

| Feature | Syntax | Example |
|---|---|---|
| String | `key: value` | `name: Alice` |
| Number | `key: 123` | `age: 30` |
| Boolean | `key: true/false` | `active: true` |
| List | `key: [a, b, c]` | `colors: [red, blue]` |
| Map | `{key: value}` | `{x: 1, y: 2}` |
| Multiline | `key: \|` | preserves newlines |
| Anchor | `&anchor_name` | `&default` |
| Alias | `*anchor_name` | `*default` |

---
*Source: adapted from the YAML Scripting cheatsheet on [engidock.com](https://www.engidock.com/cheatsheets).*
