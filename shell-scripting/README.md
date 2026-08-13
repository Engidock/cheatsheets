# Shell Scripting Cheatsheet

Complete quick reference guide for Bash & Shell programming.

## 🎯 Basics & Syntax

### Script Structure

Basic script:

```bash
#!/bin/bash
# This is a comment
echo "Hello World"

# Variables
name="John"
echo "Hello, $name"

# Variable expansion
echo ${name}
echo ${name:0:3}                 # First 3 characters
```

Shebang lines:

```bash
#!/bin/bash                    # Bash shell
#!/bin/sh                      # POSIX shell
#!/usr/bin/env bash             # Portable bash
#!/bin/zsh                      # Zsh shell
```

### Variables & Arrays

Variables:

```bash
name="John"
age=30
file_path="/path/to/file"

# Unset variable
unset name

# Read-only variable
readonly API_KEY="12345"

# Local variable (in functions)
local var="local_value"
```

Arrays:

```bash
arr=(1 2 3 4 5)
arr[0]="first"
arr+=(6 7)                      # Append
echo ${arr[0]}                  # First element
echo ${arr[@]}                  # All elements
echo ${#arr[@]}                 # Array length
echo ${arr[@]:1:3}              # Slice
```

## 🔀 Conditionals

### If-Else Statements

Basic if-else:

```bash
if [ $age -ge 18 ]; then
    echo "Adult"
elif [ $age -ge 13 ]; then
    echo "Teenager"
else
    echo "Child"
fi
```

Test operators:

```bash
# Numeric comparisons
[ $a -eq $b ]                   # Equal
[ $a -ne $b ]                   # Not equal
[ $a -lt $b ]                   # Less than
[ $a -le $b ]                   # Less or equal
[ $a -gt $b ]                   # Greater than
[ $a -ge $b ]                   # Greater or equal
```

String tests:

```bash
[ -z "$str" ]                   # Empty string
[ -n "$str" ]                   # Non-empty string
[ "$str1" = "$str2" ]           # Strings equal
[ "$str1" != "$str2" ]          # Strings not equal
[[ $str == pattern* ]]          # Pattern matching
```

File tests:

```bash
[ -f "$file" ]                  # Is file
[ -d "$dir" ]                   # Is directory
[ -e "$path" ]                  # Path exists
[ -r "$file" ]                  # Readable
[ -w "$file" ]                  # Writable
[ -x "$file" ]                  # Executable
```

### Case Statement

Case statement:

```bash
case $color in
    red|crimson)
        echo "Stop"
        ;;
    green)
        echo "Go"
        ;;
    yellow)
        echo "Caution"
        ;;
    *)
        echo "Unknown"
        ;;
esac
```

## 🔁 Loops

### Loop Types

For loop:

```bash
for i in 1 2 3 4 5; do
    echo $i
done

for i in {1..10}; do
    echo $i
done

for ((i=0; i<10; i++)); do
    echo $i
done
```

While loop:

```bash
i=0
while [ $i -lt 10 ]; do
    echo $i
    ((i++))
done

while IFS= read -r line; do
    echo "$line"
done < file.txt
```

Until & break/continue:

```bash
i=0
until [ $i -ge 10 ]; do
    echo $i
    ((i++))
done

for i in {1..10}; do
    if [ $i -eq 5 ]; then
        break                    # Exit loop
    fi
    if [ $i -eq 2 ]; then
        continue                 # Skip iteration
    fi
    echo $i
done
```

## ⚙️ Functions

### Function Definition

Basic function:

```bash
function greet() {
    echo "Hello, $1"
}

# Alternative syntax
greet() {
    echo "Hello, $1"
}

# Call function
greet "Alice"
```

Function parameters:

```bash
add() {
    echo $(($1 + $2))
}

sum_all() {
    local total=0
    for num in "$@"; do          # $@ all parameters
        total=$((total + num))
    done
    echo $total
}

sum_all 1 2 3 4 5                # Returns 15
```

Return values:

```bash
is_even() {
    if [ $((${1} % 2)) -eq 0 ]; then
        return 0                 # Success
    else
        return 1                 # Failure
    fi
}

if is_even 4; then
    echo "Even"
fi
```

## 📝 String Operations

### String Manipulation

String methods:

```bash
str="Hello World"

# Length
echo ${#str}                     # 11

# Substring
echo ${str:0:5}                  # Hello (from pos 0, 5 chars)
echo ${str:6}                    # World (from pos 6)

# Replace
echo ${str/World/Universe}       # Hello Universe
echo ${str//o/0}                 # Hell0 W0rld (replace all)
```

String expansion:

```bash
str="hello"

# Upper/lowercase (bash 4+)
echo ${str^^}                    # HELLO
echo ${str,,}                    # hello

# Remove prefix/suffix
path="/home/user/file.txt"
echo ${path#*/}                  # home/user/file.txt
echo ${path%/*}                  # /home/user

# Default values
echo ${var:-"default"}           # Use default if empty
echo ${var:="default"}           # Assign default if empty
```

## 📂 File Operations

### File Commands

Reading files:

```bash
while IFS= read -r line; do
    echo "$line"
done < file.txt

mapfile -t lines < file.txt      # Read into array
for line in "${lines[@]}"; do
    echo "$line"
done
```

Writing files:

```bash
echo "text" > file.txt           # Overwrite
echo "text" >> file.txt          # Append

cat > file.txt << EOF
Line 1
Line 2
EOF

printf "formatted: %d\n" 42 > file.txt
```

## 🔨 Text Processing

### Common Tools

grep - Search:

```bash
grep "pattern" file.txt
grep -i "pattern" file.txt       # Case insensitive
grep -n "pattern" file.txt       # Show line numbers
grep -v "pattern" file.txt       # Invert match
grep -E "regex" file.txt         # Extended regex
```

sed - Stream editor:

```bash
sed 's/old/new/' file.txt        # Replace first
sed 's/old/new/g' file.txt       # Replace all
sed '5d' file.txt                # Delete line 5
sed -n '2,4p' file.txt           # Print lines 2-4
```

awk - Text processing:

```bash
awk '{print $1}' file.txt        # Print first column
awk -F',' '{print $2}' file.csv  # Use comma as separator
awk '{sum+=$1} END {print sum}' file.txt
awk 'NR==5' file.txt             # Print line 5
```

cut & sort:

```bash
cut -d' ' -f1 file.txt           # Cut first field
cut -c1-5 file.txt               # Cut characters 1-5
sort file.txt
sort -n file.txt                 # Numeric sort
sort -r file.txt                 # Reverse sort
sort -u file.txt                 # Unique lines
```

## ⚡ Advanced Topics

### Advanced Concepts

Piping & redirection:

```bash
command1 | command2              # Pipe output
command > file.txt               # Redirect stdout
command 2> error.txt             # Redirect stderr
command &> all.txt               # Redirect both
command1 && command2             # Run if success
command1 || command2             # Run if failure
```

Command substitution:

```bash
files=$(ls *.txt)
date_now=$(date +%Y-%m-%d)

# Or use backticks (older)
files=`ls *.txt`
```

Arithmetic:

```bash
result=$((2 + 2))
result=$((10 * 5))
result=$((20 / 4))
result=$((10 % 3))
result=$((2 ** 3))
((result++))
((result += 5))
```

## 🐛 Debugging & Best Practices

- ✓ **Always quote variables** - `"$var"` prevents word splitting
- ✓ **Use `set -e`** - Exit on error
- ✓ **Use `set -u`** - Error on undefined variables
- ✓ **Use `set -o pipefail`** - Detect errors in pipes
- ✓ **Check exit codes** - Use `$?` to check return value

### Debugging Tips

🔍 Debug mode:

```bash
bash -x script.sh               # Print commands before execution
set -x                           # Enable debug in script
```

🔍 Echo debugging:

```bash
echo "DEBUG: variable=$variable"
```

💡 Common Patterns:

```bash
set -euo pipefail                # Safe mode
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"   # Get script directory
```

## 📋 Quick Reference

| Command | Purpose | Example |
|---|---|---|
| `echo` | Print text | `echo "Hello"` |
| `read` | Read input | `read -p "Enter: " var` |
| `grep` | Search text | `grep "pattern" file` |
| `sed` | Edit stream | `sed 's/old/new/' file` |
| `awk` | Process text | `awk '{print $1}' file` |
| `find` | Find files | `find . -name "*.txt"` |
| `ls` | List files | `ls -la directory` |
| `wc` | Count lines | `wc -l file.txt` |

---

*Source: adapted from the Shell Scripting cheatsheet on [engidock.com](https://www.engidock.com/cheatsheets).*
