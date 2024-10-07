# ./missing-semester
My notes for **The Missing Semester of Your CS Education**.

## Contents
- [Lecture 1: The Shell](#lecture-1-the-shell)
- [Lecture 2: Shell Tools and Scripting](#lecture-2-shell-tools-and-scripting)
- [Lecture 3: Editors (Vim)](#lecture-3-editors-vim)
- [Lecture 4: Data Wrangling](#lecture-4-data-wrangling)
- [Lecture 5: Command-line Environment](#lecture-5-command-line-environment)
- [Lecture 6: Version Control (Git)](#lecture-6-version-control-git)
- [Lecture 7: Debugging and Profiling](#lecture-7-debugging-and-profiling)
- [Lecture 8: Metaprogramming](#lecture-8-metaprogramming)
- [Lecture 9: Security and Cryptography](#lecture-9-security-and-cryptography)
- [Lecture 10: Potpourri](#lecture-9-security-and-cryptography)

## Lecture 1: The Shell
### What is the shell?
- textual interface
- shell vs terminal
- bash (Bourne Again Shell)

### Using the shell
**Launch terminal to open shell prompt:**
```
missing:~$
```
- `missing`: machine
- `~`: short for home
- `$`: you are not the root user

**Echo:**
```
missing:~$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
missing:~$ which echo
/bin/echo
missing:~$ /bin/echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```
- `$PATH`: lists which directories the shell should search for programs
- Colon separated list
- `which`: which file is executed for a given program name

### Navigating in the shell
```
missing:~$ pwd
/home/missing
missing:~$ cd /home
missing:/home$ pwd
/home
missing:/home$ cd ..
missing:/$ pwd
/
missing:/$ cd ./home
missing:/home$ pwd
/home
missing:/home$ cd missing
missing:~$ pwd
/home/missing
missing:~$ ../../bin/echo hello
hello
```
- Linux & macOS: path separated by `/`; the path `/` is the root of the file system
- Windows: path separated by `\`; one root for each disk partition (e.g. `C:\`)
- `cd ~`: to home directory
- `cd -`: to previous directory

**`ls` command:**
```
missing:~$ ls
missing:~$ cd ..
missing:/home$ ls
missing
missing:/home$ cd ..
missing:/$ ls
bin
boot
dev
etc
home
...
```
- commands accept flags and options (flags with values)

**`ls --help`:**
```
  -l                         use a long listing format
```
```
missing:~$ ls -l /home
drwxr-xr-x 1 missing  users  4096 Jun 15  2019 missing
```
- permissions: owner (`missing`) - group (`users`) - everyone else
- `rwx`: read, write, execute
- files: read; write; execute
- directory: `ls`; rename/create/remove files in the directory; "search", to enter the a directory

**Handy programs:**
- `mv` (rename/move file), `cp` (copy file), `rm -r` (remove recursively), `rmdir` (remove directory), `mkdir` (make directory)
- `ctrl+l` to clear
- `man ls`: show manual page of `ls`

### Connecting programs
- Two primary "streams": input stream and output stream
- redirect streams: `< file` and `> file`
- append: `>>` 

**`cat`**
```
missing:~$ echo hello > hello.txt
missing:~$ cat hello.txt
hello
missing:~$ cat < hello.txt
hello
missing:~$ cat < hello.txt > hello2.txt
missing:~$ cat hello2.txt
hello
```

**Pipes**:
```
missing:~$ ls -l / | tail -n1
drwxr-xr-x 1 root  root  4096 Jun 20  2019 var
missing:~$ curl --head --silent google.com | grep --ignore-case content-length | cut --delimiter=' ' -f2
219
```
- to "chain" programs such that the output of one is the input of another

### A versatile and powerful tool
- `$`: regular user ; `#`: root user
- `sudo`: do as super user
- `sudo su`: super user do; shell as super user, `exit` to return
- `find`: search for file
- `tee`: read from standard input, write to both standard output and file
- `xdg-open`: open file with preferred application

**e.g. change brightness**:
```
$ sudo find -L /sys/class/backlight -maxdepth 2 -name '*brightness*'
/sys/class/backlight/thinkpad_screen/brightness
$ cd /sys/class/backlight/thinkpad_screen
$ sudo echo 3 > brightness
An error occurred while redirecting file 'brightness'
open: Permission denied
```
- `|`, `>` and `<` are done by the shell

workaround:
```
echo 3 | sudo tee brightness
```

### Exercise 
- `touch`: create an empty file
- single quotes (treated as literal string) vs double quote (allow variable expansion)
- `sh`: Bourne shell
- `chmod`: change file permissions
- shebang: `#!` beginning of a script, specifies the intepreter to be used, e.g. `#!/bin/bash`

## Lecture 2: Shell Tools and Scripting
### Shell Scripting
**Assign variables:**
```
foo=bar
echo "$foo"
# prints bar
echo '$foo'
# prints $foo
```
- spacing will perform argument spliting

**Control flow:**
```
mcd () {
    mkdir -p "$1"
    cd "$1"
}
```
- `source`: execute commands from a file in the current shell

**Special variables**:
- `$0` - Name of the script
- `$1` to `$9` - Arguments to the script. $1 is the first argument and so on.
- `$@` - All the arguments
- `$#` - Number of arguments
- `$?` - Return code of the previous command
- `$$` - Process identification number (PID) for the current script
- `!!` - Entire last command, including arguments. A common pattern is to execute a command only for it to fail due to missing permissions; you can quickly re-execute the command with `sudo` by doing `sudo !!`
- `$_` - Last argument from the last command. If you are in an interactive shell, you can also quickly get this value by typing `Esc` followed by . or `Alt+`.

**Command returns**:
- return output with `STDOUT`, errors with `STDERR`
- `>` to redirect `STDOUT`, `2>` to redirect `STDERR`
- Return Code: 0 means OK, else an error occured
- `true`: 0 return code, `false`: 1 return code

**Short-circuiting**:
```
false || echo "Oops, fail"
# Oops, fail

true || echo "Will not be printed"
#

true && echo "Things went well"
# Things went well

false && echo "Will not be printed"
#

true ; echo "This will always run"
# This will always run

false ; echo "This will always run"
# This will always run
```

**Substitution**:
- command substitution: `$(CMD)`, execute `CMD`, get the output of the command and substitute in place
- process substitution: `<(CMD)`, execute `CMD`, place the output in a temporary file and substitute `<()` with that file's name
- try to use double brackets `[[ ]]` in favor of single brackets `[ ]` in bash, although it won't be portable in `sh` 
```
#!/bin/bash

echo "Starting program at $(date)" # Date will be substituted

echo "Running program $0 with $# arguments with pid $$"

for file in "$@"; do
    grep foobar "$file" > /dev/null 2> /dev/null
    # When pattern is not found, grep has exit status 1
    # We redirect STDOUT and STDERR to a null register since we do not care about them
    if [[ $? -ne 0 ]]; then
        echo "File $file does not have any foobar, adding one"
        echo "# foobar" >> "$file"
    fi
done
```

**Globbing**:
- glob patterns specify sets of filenames with wildcard characters
- wildcards: `?` and `*` to match one or any amount of characters respectively
- curly braces `{}`: common substring in a series of command, to expand
```
convert image.{png,jpg}
# Will expand to
convert image.png image.jpg

cp /path/to/project/{foo,bar,baz}.sh /newpath
# Will expand to
cp /path/to/project/foo.sh /path/to/project/bar.sh /path/to/project/baz.sh /newpath

# Globbing techniques can also be combined
mv *{.py,.sh} folder
# Will move all *.py and *.sh files


mkdir foo bar
# This creates files foo/a, foo/b, ... foo/h, bar/a, bar/b, ... bar/h
touch {foo,bar}/{a..h}
touch foo/x bar/y
# Show differences between files in foo and bar
diff <(ls foo) <(ls bar)
# Outputs
# < x
# ---
# > y
```

**shebang**:
- `env` command to resolve to wherever the command lives in the system
- e.g. use `#!/usr/bin/env python` instead of `#!/usr/local/bin/python`
- `shellcheck` to find errors in sh/bash scripts
```
#!/usr/bin/env python
import sys
for arg in reversed(sys.argv[1:]):
    print(arg)
```

**Shell functions vs scripts**:
- Functions have to be in the same language as the shell, while scripts can be written in any language. This is why including a shebang for scripts is important.
- Functions are loaded once when their definition is read. Scripts are loaded every time they are executed. This makes functions slightly faster to load, but whenever you change them you will have to reload their definition.
- Functions are executed in the current shell environment whereas scripts execute in their own process. Thus, functions can modify environment variables, e.g. change your current directory, whereas scripts can’t. Scripts will be passed by value environment variables that have been exported using `export`
- As with any programming language, functions are a powerful construct to achieve modularity, code reuse, and clarity of shell code. Often shell scripts will include their own function definitions.

### Shell Tools
#### Finding how to use commands

#### Finding files

#### Finding code

#### Finding shell commands

#### Directory navigation

### Exercise

## Lecture 3: Editors (Vim)

## Lecture 4: Data Wrangling

## Lecture 5: Command-line Environment

## Lecture 6: Version Control (Git)

## Lecture 7: Debugging and Profiling

## Lecture 8: Metaprogramming

## Lecture 9: Security and Cryptography

## Lecture 10: Potpourri

## References
- https://missing.csail.mit.edu/
