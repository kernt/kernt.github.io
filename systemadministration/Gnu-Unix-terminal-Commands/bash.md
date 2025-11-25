---
tags:
  - bash
  - konsole
  - gnu-tools
  - system-administration
---
# bash Uebersicht

## Pathnamen erweiterung (globbing)

* - means 'match any number of characters'. '/' is not matched (and depending on your settings, things like '.' may or may not be matched, see above)
? - means 'match any single character'
[abc] - match any of the characters listed. This syntax also supports ranges, like [0-9]
## Datein Auswerten

```sh
for filename in *.txt; do
  echo "=== BEGIN: $filename ==="
  cat "$filename"
  echo "=== END: $filename ==="
done
```
## Compound kommandos

Grouping
{ …; }	command grouping
( … )	command grouping in a subshell
Conditionals
[[ ... ]]	conditional expression
if …; then …; fi	conditional branching
case … esac	pattern-based branching
Loops
for word in …; do …; done	classic for-loop
for ((x=1; x<=10; x++)); do ...; done	C-style for-loop
while …; do …; done	while loop
until …; do …; done	until loop
Misc
(( ... ))	arithmetic evaluation
select word in …; do …; done	user selections

## Expansion und substitution

`{A,B,C} {A..C}`	Brace expansion

`touch file{1,2,3}`

`mkdir directory{1,2,3}{a,b,c}`

`touch file{a..z}`

`~/ ~root/`	Tilde expansion

`$FOO ${BAR%.mp3}`	Parameter expansion

`command` $(command)	Command substitution

`<(command) >(command)`	Process substitution

`$((1 + 2 + 3)) $[4 + 5 + 6]`	Arithmetic expansion

`Hello <---> Word!`	Word splitting

`/data/*-av/*.mp?`	Pathname expansion

## Process substitution

```sh
<( <LIST> )

>( <LIST> )
```

## Example mit diff

`diff <(ls "$first_directory") <(ls "$second_directory")`

## Example als counter

```sh
counter=0

while IFS= read -rN1 _; do
    ((counter++))
done < <(find /etc -printf ' ')

echo "$counter files"
```

## Zuweisung via Parameter

```sh
f() {
    cat "$1" >"$x"
}

x=>(tr '[:lower:]' '[:upper:]') f <(echo 'hi there')
```
## Parameter expansion

**Simple usage**
        $PARAMETER
        ${PARAMETER}

**Indirection**
        ${!PARAMETER}

**Case modification**
        ${PARAMETER^}
        ${PARAMETER^^}
        ${PARAMETER,}
        ${PARAMETER,,}
        ${PARAMETER~}
        ${PARAMETER~~}

**Variable name expansion**
        ${!PREFIX*}
        ${!PREFIX@}

**Substring removal (also for filename manipulation!)**
        ${PARAMETER#PATTERN}
        ${PARAMETER##PATTERN}
        ${PARAMETER%PATTERN}
        ${PARAMETER%%PATTERN}

**Search and replace**
        ${PARAMETER/PATTERN/STRING}
        ${PARAMETER//PATTERN/STRING}
        ${PARAMETER/PATTERN}
        ${PARAMETER//PATTERN}

**String length**
        ${#PARAMETER}

**Substring expansion**
        ${PARAMETER:OFFSET}
        ${PARAMETER:OFFSET:LENGTH}

**Use a default value**
        ${PARAMETER:-WORD}
        ${PARAMETER-WORD}

**Assign a default value**
        ${PARAMETER:=WORD}
        ${PARAMETER=WORD}

**Use an alternate value**
        ${PARAMETER:+WORD}
        ${PARAMETER+WORD}
## Display error if null or unset

* `${PARAMETER:?WORD}`
* `${PARAMETER?WORD}`

**Get name without extension**
`${FILENAME%.*}`

**Get extension**
`${FILENAME##*.}`

**Get directory name**
`${PATHNAME%/*}`

**Get filename**
`${PATHNAME##*/}`
## Brace expansion

```sh
$ echo _{I,want,my,money,back}
_I _want _my _money _back

$ echo {I,want,my,money,back}_
I_ want_ my_ money_ back_

$ echo _{I,want,my,money,back}-
_I- _want- _my- _money- _back-
```

`echo {{A..Z},{a..z}}`

`echo {A..Z}{0..9}`

`{<START>..<END>}`
## Search and replace

```sh
${PARAMETER/PATTERN/STRING}
${PARAMETER//PATTERN/STRING}
${PARAMETER/PATTERN}
${PARAMETER//PATTERN}
```
# Bash shortcuts nach emacs

```
STRG+a = Anfang der eingabe
STRG+e = Ende der Eingabe
ALT+b = Word zurück
ALT+F = Word vor
STRG+b = Zeichen zurück 
STRG+f = Zeichen vor
STRG+k = Lösche ein
STRG+u = Lösche vom cursor bis anfang
STRG+w = Löscht links vom cusor
STRG+t = Vertauscht Zeichen
ALT+t = Vertauscht Wörter
STRG+[ = Löscht inhalt des Festers
```
## Arrays

**Quellen:**

* [bash-hackers.org](https://wiki.bash-hackers.org/syntax/pe)

## Tools

* [bash-templater](https://github.com/lavoiesl/bash-templater)

**Process Substitution with Named Pipes (**`**<(...)**` **and** `**>(...)**`**)**

Process substitution allows you to use the output of a command as a temporary file. This is particularly useful when a command requires a filename as an argument.

```
diff <(sort file1.txt) <(sort file2.txt)
```

This compares the sorted versions of two files without creating intermediate files.

**Using** `**/dev/tcp**` **and** `**/dev/udp**` **for Network Communications**

Bash can perform TCP and UDP network operations without external utilities.
# Check if a port is open

`timeout 1 bash -c "</dev/tcp/google.com/80" && echo "Open" || echo "Closed"`

This attempts to open a TCP connection to `google.com` on port `80`.

**Recursive Globbing with** `**globstar**` **(**`******`**)**

By enabling the `globstar` option, you can use `**` to match directories recursively.

```
shopt -s globstar  
ls **/*.txt
```

This lists all `.txt` files in the current directory and all subdirectories.

**String Manipulation with Parameter Expansion**

Bash provides powerful string manipulation without invoking external commands.

```
FILENAME="archive.tar.gz"  
echo "${FILENAME%.tar.gz}"  # Output: archive
```

This removes the `.tar.gz` suffix from the filename.

**Extended Pattern Matching with** `**extglob**`

Enable `extglob` to use advanced pattern matching in your scripts.

```
shopt -s extglob  
rm !(*.txt)
```

This command deletes all files except those ending with `.txt`.

**Arithmetic Evaluation with** `**let**`**,** `**$((...))**`**, and** `**((...))**`

Perform arithmetic operations without external tools.

```
COUNT=0  
let COUNT++  
echo $COUNT  # Output: 1  
  
# Or using ((...))  
((COUNT+=5))  
echo $COUNT  # Output: 6

```
**Creating Anonymous Functions**

Define functions without names for quick, one-time use.

```
(function(){ echo "Hello from an anonymous function!"; })
```

This executes the function immediately without polluting the namespace.

**Associative Arrays for Key-Value Storage**

Bash 4+ supports associative arrays, allowing you to map strings to values.

```
declare -A fruits  
fruits[apple]="red"  
fruits[banana]="yellow"  
echo "${fruits[apple]}"  # Output: red
```

**Dynamic Variable Names with Indirect Expansion**

Use the value of a variable as the name of another variable.

```
VAR_NAME="USER"  
echo "${!VAR_NAME}"  # Equivalent to echo "$USER"
```

**Temporary Files with** `**mktemp**`

Securely create temporary files or directories in your scripts.

```sh
TMPFILE=$(mktemp)  
echo "Data" > "$TMPFILE"  
# Do something with $TMPFILE  
rm "$TMPFILE"
```

This avoids race conditions and security issues associated with predictable filenames.

These tricks leverage built-in Bash features to perform tasks more efficiently and elegantly. They can help you write more powerful scripts and use the command line more effectively.

You can directly run it on Linux and macOS.

You have a few options if you’re using Windows:

- Windows Subsystem for Linux
- Git Bash
- Cygwin
- MSYS2
- PowerShell (native Windows scripting capabilities)

Bash scripting is a superpower for engineers. Whether automating repetitive tasks, gluing together tools, or managing systems, Bash is always there, simple yet powerful.

But like any power, it requires mastery. Let me walk you through 10 essential Bash constructs through the lens of a plausible scenario.

![](https://miro.medium.com/v2/resize:fit:700/1*U1hsROtF2MpcvuNBrO5yxA.png)

Bash

# The Scenario

You’re tasked with analyzing server logs from multiple files, extracting failed login attempts, and generating a report. It’s a routine problem, but with Bash, we’ll make it elegant and reusable.

# 1. Setting the Stage with a Script

We begin our journey by writing the skeleton of our script:

#!/bin/bash  
  
set -e # Exit on errors  
trap 'echo "Error on line $LINENO"; exit 1' ERR

**Why?**:

- `**set -e**` ensures the script stops at the first sign of trouble.
- `**trap**` catches errors, giving us helpful debugging information.

# 2. Modularize with Functions

Good scripts are modular. Let’s define a function to parse log files:

parse_logs() {  
  
  local file="$1"  
  local output="$2"  
  
  while read -r line; do  
    if [[ "$line" == *"FAILED LOGIN"* ]]; then  
        echo "$line" >> "$output"  
    fi  
  
  done < "$file"  
}

**Why?**:

- Functions make scripts reusable and maintainable.
- `**local**` **variables** prevent accidental overwrites.

# 3. Arrays: Managing Multiple Logs

We need to process logs from several servers:

log_files=("server1.log" "server2.log" "server3.log")  
results=()  
  
for file in "${log_files[@]}"; do  
    output="${file%.log}_failed.log"  
    parse_logs "$file" "$output"  
    results+=("$output")  
done

**Why?**:

- Arrays help manage lists of items efficiently.
- We append processed results to an array for future steps.

# 4. Command Substitution: Adding Timestamps

Let’s add timestamps to our output files using `date`:

timestamp=$(date "+%Y-%m-%d")  
final_report="failed_logins_$timestamp.txt"

**Why?**:

- Command substitution integrates dynamic values into scripts seamlessly.

# 5. String Manipulation

Before combining the logs, we sanitize the output filenames:

for file in "${results[@]}"; do  
    sanitized_name="${file// /_}"  # Replace spaces with underscores  
    mv "$file" "$sanitized_name"  
done

**Why?**:

- Bash’s parameter expansion simplifies string transformations without external tools.

# 6. Process Substitution: Combining Files

To merge the logs efficiently:

cat "${results[@]}" > "$final_report"

**Why?**:

- Process substitution and array expansion enable concise, efficient handling of multiple files.

# 7. Conditional Logic: Tailoring Reports

Let’s customize the final report based on its content:

if [[ -s "$final_report" ]]; then  
    echo "Report generated: $final_report"  
else  
    echo "No failed logins found."  
    rm "$final_report"  
fi

**Why?**:

- `**if**` ensures actions depend on context, such as whether the report is empty.

# 8. Case Statements: Default Ports

Imagine we need to identify default SSH and HTTPS ports based on server type:

get_port() {  
    local server="$1"  
    case "$server" in  
        "prod"*) echo 22 ;;  
        "staging"*) echo 2222 ;;  
        *) echo 80 ;;  
    esac  
}

**Why?**:

- `**case**` is ideal for handling multiple specific patterns elegantly.

# 9. Debugging with `set -x`

Before deploying the script, let’s debug it:

set -x # Enable debugging  
# Run the main script here  
set +x # Disable debugging

**Why?**:

- Debugging tools like `set -x` make it easy to trace and fix errors.

# 10. File Descriptors for Advanced I/O

Let’s imagine we’re reading and processing logs from a special input stream:

exec 3<"$final_report"  
  
while read -u3 line; do  
    echo "Processed: $line"  
done  
  
exec 3<&-

**Why?**:

- File descriptors give precise control over inputs and outputs, enabling parallel processing.

# The Final Script

Here’s what the polished script might look like:

#!/bin/bash  
  
set -e  
trap 'echo "Error on line $LINENO"; exit 1' ERR  
parse_logs() {  
    local file="$1"  
    local output="$2"  
    while read -r line; do  
        if [[ "$line" == *"FAILED LOGIN"* ]]; then  
            echo "$line" >> "$output"  
        fi  
    done < "$file"  
}  
log_files=("server1.log" "server2.log" "server3.log")  
results=()  
for file in "${log_files[@]}"; do  
    output="${file%.log}_failed.log"  
    parse_logs "$file" "$output"  
    results+=("$output")  
done  
timestamp=$(date "+%Y-%m-%d")  
final_report="failed_logins_$timestamp.txt"  
cat "${results[@]}" > "$final_report"  
if [[ -s "$final_report" ]]; then  
    echo "Report generated: $final_report"  
else  
    echo "No failed logins found."  
    rm "$final_report"  
fi

# Takeaways

This script ties together “almost” everything an engineer needs for professional Bash scripting: modularity, error handling, efficient data processing, and debugging tools.

By mastering these constructs, you’ll not only write better scripts but also transform mundane tasks into elegant solutions.

But, there is one more thing (maybe 5). I am feeling too excited now and I will include 5 more extra that I find myself using quite often:

# Five More Bash Constructs Every Engineer Should Know

Here are five additional constructs:

# 11. Associative Arrays

**What They Are:** Associative arrays are key-value pairs in Bash, available starting with Bash 4. They allow efficient lookups and data organization.

**Example:** Imagine you’re mapping server names to their IP addresses:

declare -A servers  
  
servers=( ["web"]="192.168.1.10" ["db"]="192.168.1.20" ["cache"]="192.168.1.30" )  
  
# Access values  
echo "Web server IP: ${servers[web]}"  
# Iterate over keys  
for key in "${!servers[@]}"; do  
    echo "$key -> ${servers[$key]}"  
done

**Why Use Them:**

- Associative arrays provide a natural way to handle structured data without relying on external tools like `awk` or `sed`.
- Useful for configurations, lookups, and organizing data dynamically in scripts.

# 12. Heredocs for Multi-line Input

**What They Are:** Heredocs allow multi-line strings or input directly in your scripts, improving readability when dealing with templates or bulk data.

**Example:** Generating an email template dynamically:

email_body=$(cat <<EOF  
Hello Team,  
  
This is a reminder for the upcoming deployment at midnight.  
  
Regards,  
DevOps  
EOF)  
  
echo "$email_body" | mail -s "Deployment Reminder" team@example.com  

**Why Use Them:**

- They eliminate the need for complex string concatenations or external files.
- Heredocs simplify handling multi-line content, like logs, templates, or commands, directly within your script.

# 13. `eval` for Dynamic Command Execution

**What It Is:** The `eval` command lets you execute a dynamically constructed string as a Bash command.

**Example:** Suppose you need to execute a command stored in a variable:

cmd="ls -l"  
eval "$cmd"

Or dynamically set variables:

var_name="greeting"  
eval "$var_name='Hello, World!'"  
echo "$greeting"

**Why Use It:**

- `eval` provides flexibility for handling dynamically generated commands or input.
- ⚠ Use with caution: While powerful, improper use of `eval` can lead to security risks if handling untrusted input.

# 14. Subshells for Isolated Execution

**What They Are:** A subshell is a child process in which commands can be executed without affecting the parent shell.

**Example:** Suppose you want to temporarily change directories and execute commands:

(current_dir=$(pwd)  
cd /tmp  
echo "Now in $(pwd)"  
)  
echo "Back in $current_dir"

**Why Use Them:**

- Subshells allow temporary changes to variables, environments, or directories without impacting the main shell.
- Ideal for running isolated operations that don’t pollute or modify the parent environment.

# 15. Named Pipes (FIFOs)

**What They Are:** Named pipes (or FIFOs) are special files that facilitate inter-process communication by acting as a buffer between commands.

**Example:** Let’s create a named pipe to transfer data between processes:

mkfifo my_pipe  
  
# In one terminal: Write to the pipe  
echo "Hello from process 1" > my_pipe  
  
# In another terminal: Read from the pipe  
cat < my_pipe  
  
# Clean up  
rm my_pipe

**Why Use Them:**

- Named pipes enable asynchronous communication between processes, allowing data to flow without temporary files.
- Useful for real-time processing scenarios, such as feeding logs or streaming data between commands.