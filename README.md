## BASH

``` bash
#!/usr/bin/env bash
set -euo pipefail  # exit on error | undefined var | pipe fails
IFS=$'\n\t'

name="world"
echo "Hello, $name"
```

## Python

```python
#!/usr/bin/env python3
import sys

name = "world"
print(f"Hello, {name}")
```


| Task        | Bash               | Python         |
| ----------- | ------------------ | -------------- |
| Define var  | `x=42` (no spaces) | `x = 42`       |
| String      | `s="hi"`           | `s = "hi"`     |
| Interpolate | `"Hi $name"`       | `f"Hi {name}"` |
| Length      | `${#s}`            | `len(s)`       |
| Substring   | `${s:0:2}`         | `s[0:2]`       |



# Conditionals

## BASH (prefer [[ ... ]])

``` bash
if [[ -f "$p" && -s "$p" ]]; then echo "file exists & nonempty"; fi

# string tests
[[ "$a" == "$b" ]]       # equals
[[ -z "$s" ]]            # empty
[[ -n "$s" ]]            # non-empty

# numeric tests
[[ $x -gt 10 ]]          # > (gt, lt, ge, le, eq, ne)
```

## Python

```python
from pathlib import Path
p = Path(path)
if p.is_file() and p.stat().st_size > 0:
    print("file exists & nonempty")

if a == b:
    ...
if not s:                # empty string is falsy
    ...
if x > 10:
    ...
```

# Loops

## BASH

``` bash
# over list
for x in a b c; do echo "$x"; done

# over array
arr=(a b c)
for x in "${arr[@]}"; do echo "$x"; done

# lines in file (safe)
while IFS= read -r line; do
  echo "$line"
done < file.txt
```

## Python

```python
for x in ["a", "b", "c"]:
    print(x)

with open("file.txt") as f:
    for line in f:
        print(line.rstrip("\n"))
```

# Arrays / Lists & Dicts

| Concept            | Bash                       | Python                |
| ------------------ | -------------------------- | --------------------- |
| List/Array         | `arr=(a b c)`              | `arr = ["a","b","c"]` |
| Index              | `${arr[0]}`                | `arr[0]`              |
| Length             | `${#arr[@]}`               | `len(arr)`            |
| Iterate            | `for x in "${arr[@]}"`     | `for x in arr:`       |
| Dict / Assoc array | `declare -A m; m[foo]=bar` | `m = {"foo": "bar"}`  |
| Read dict          | `${m[foo]}`                | `m["foo"]`            |



# Functions

## BASH

``` bash
greet() {                # or: function greet() { ... }
  local name=${1:-world} # args via $1, $2...
  echo "Hi $name"        # return strings via echo
  return 0               # numeric exit status
}
greet "Alice"
```

## Python

```python
def greet(name="world"):
    return f"Hi {name}"

print(greet("Alice"))
```

# | Task                  | Bash                   | Python                         |
| --------------------- | ---------------------- | ------------------------------ |
| Positional args       | `$1`, `$2`, `$@`, `$#` | `sys.argv[1:]` or `argparse`   |
| Exit code of last cmd | `$?`                   | `proc.returncode` (subprocess) |
| Script exit           | `exit 1`               | `sys.exit(1)`                  |


# Files 

| Task             | Bash               | Python                        |
| ---------------- | ------------------ | ----------------------------- |
| Read file        | `cat file` or loop | `open("file").read()`         |
| Write (truncate) | `echo "x" > file`  | `open("file","w").write("x")` |
| Append           | `echo "x" >> file` | `open("file","a").write("x")` |
| File exists      | `[[ -f "$p" ]]`    | `Path(p).is_file()`           |
| Directory exists | `[[ -d "$p" ]]`    | `Path(p).is_dir()`            |


# Arithmetic

## BASH

``` bash
(( x = 1 + 2 ))
echo $((x * 3))
```

## Python

```python
x = 1 + 2
print(x * 3)
# integer division vs float:
5 // 2  # 2
5 / 2   # 2.5
```

# Command subsitution

| Task           | Bash                   | Python                                                                      |
| -------------- | ---------------------- | --------------------------------------------------------------------------- |
| Run command    | `ls -l`                | `subprocess.run(["ls","-l"])`                                               |
| Capture output | `out=$(cmd)`           | `subprocess.run(cmd, capture_output=True, text=True).stdout`                |
| Pipe           | `cmd1 \| cmd2`         | `subprocess.run(..., stdout=PIPE); subprocess.run(..., stdin=prev.stdout)`  |
| Here-string    | `grep foo <<< "$text"` | Use Python logic or `subprocess.run(["grep","foo"], input=text, text=True)` |


# Env vars

| Task                  | Bash                    | Python                                                 |
| --------------------- | ----------------------- | ------------------------------------------------------ |
| Read                  | `$HOME`                 | `os.environ["HOME"]` (or `.get("HOME")`)               |
| Set (current proc)    | `export KEY=val`        | `os.environ["KEY"]="val"`                              |
| Child process sees it | `export` before running | `subprocess.run(..., env={**os.environ, "KEY":"val"})` |


# Strings, Regex, Globs

| Concept    | Bash                            | Python                    |
| ---------- | ------------------------------- | ------------------------- |
| Globs      | `*.log` (shell expansion)       | `glob.glob("*.log")`      |
| Regex test | `[[ "$s" =~ ^[0-9]+$ ]]`        | `re.fullmatch(r"\d+", s)` |
| Split      | `IFS=, read -ra parts <<< "$s"` | `s.split(",")`            |
| Join       | `printf "%s," "${arr[@]}"`      | `",".join(arr)`           |

# Json

## BASH

``` bash
val=$(jq -r '.key' config.json)
jq '.items[] | select(.enabled==true)' config.json
```

## Python

```python
import json
data = json.load(open("config.json"))
val = data["key"]
enabled = [x for x in data["items"] if x.get("enabled")]
```


# Examples

## Count lines containing “ERROR” (stdin or file)

``` bash
#!/usr/bin/env bash
# usage: ./count_error.sh [file]; if no file, reads stdin
set -euo pipefail
if [[ $# -gt 0 ]]; then
  grep -c "ERROR" -- "$1"
else
  grep -c "ERROR"
fi
```

## Python

```python
#!/usr/bin/env python3
# usage: ./count_error.py [file]; if no file, reads stdin
import sys, fileinput
count = sum(1 for line in fileinput.input() if "ERROR" in line)
print(count)
```



## Read JSON and print .service.region

``` bash
#!/usr/bin/env bash
# usage: ./read_region.sh config.json
set -euo pipefail
jq -r '.service.region' "$1"
```

## Python

```python
#!/usr/bin/env python3
# usage: ./read_region.py config.json
import sys, json
with open(sys.argv[1]) as f:
    data = json.load(f)
print(data["service"]["region"])
```



## List files > 1MB in current dir

``` bash
#!/usr/bin/env bash
# lists regular files > 1MB in .
set -euo pipefail
find . -maxdepth 1 -type f -size +1M -printf '%P\n' 2>/dev/null || \
find . -maxdepth 1 -type f -size +1048576c -exec basename {} \;   # fallback if -printf not available
```

## Python

```python
#!/usr/bin/env python3
from pathlib import Path

THRESH = 1 * 1024 * 1024
for p in Path(".").iterdir():
    if p.is_file() and p.stat().st_size > THRESH:
        print(p.name)
```


## Parse --name (default “world”) and print greeting

``` bash
#!/usr/bin/env bash
set -euo pipefail
NAME="world"
while [[ $# -gt 0 ]]; do
  case "$1" in
    --name)      NAME="$2"; shift 2 ;;
    --name=*)    NAME="${1#*=}"; shift ;;
    *)           shift ;;
  esac
done
echo "Hello, $NAME"
```

## Python

```python
#!/usr/bin/env python3
import argparse
p = argparse.ArgumentParser()
p.add_argument("--name", default="world")
args = p.parse_args()
print(f"Hello, {args.name}")
```


## Exit non-zero if a URL is not reachable (HTTP 200)

``` bash
#!/usr/bin/env bash
# usage: ./check_url.sh https://example.com
set -euo pipefail
URL="$1"
code=$(curl -sS -o /dev/null -w '%{http_code}' --max-time 10 "$URL" || echo 000)
if [[ "$code" == "200" ]]; then
  echo "OK 200"
  exit 0
else
  echo "FAIL $code" >&2
  exit 1
fi
```

## Python

```python
#!/usr/bin/env python3
# usage: ./check_url.py https://example.com
import sys, urllib.request, urllib.error
url = sys.argv[1]
try:
    with urllib.request.urlopen(urllib.request.Request(url, method="GET"), timeout=10) as r:
        code = r.getcode()
        if code == 200:
            print("OK 200"); sys.exit(0)
        else:
            print(f"FAIL {code}", file=sys.stderr); sys.exit(1)
except Exception as e:
    print(f"FAIL {e}", file=sys.stderr)
    sys.exit(1)
```


