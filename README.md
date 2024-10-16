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
```bash
missing:~$
```
- `missing`: machine
- `~`: short for home
- `$`: you are not the root user

**Echo:**
```bash
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
```bash
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
```bash
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
- `exa`: modern replacement for `ls`

**`ls --help`:**
```
  -l                         use a long listing format
```
```bash
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
- `tac`: reverses the output
```bash
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
```bash
missing:~$ ls -l / | tail -n1
drwxr-xr-x 1 root  root  4096 Jun 20  2019 var
missing:~$ curl --head --silent google.com | grep --ignore-case content-length | cut --delimiter=' ' -f2
219
```
- to "chain" programs such that the output of one is the input of another
- `head`, `tail` 

### A versatile and powerful tool
- `$`: regular user ; `#`: root user
- `sudo`: do as super user
- `sudo su`: super user do; shell as super user, `exit` to return
- `find`: search for file
- `tee`: read from standard input, write to both standard output and file
- `xdg-open`: open file with preferred application

**e.g. change brightness**:
```bash
$ sudo find -L /sys/class/backlight -maxdepth 2 -name '*brightness*'
/sys/class/backlight/thinkpad_screen/brightness
$ cd /sys/class/backlight/thinkpad_screen
$ sudo echo 3 > brightness
An error occurred while redirecting file 'brightness'
open: Permission denied
```
- `|`, `>` and `<` are done by the shell

workaround:
```bash
echo 3 | sudo tee brightness
```

### Exercises
- `touch`: create an empty file
- single quotes (treated as literal string) vs double quote (allow variable expansion)
- `sh`: Bourne shell
- `chmod`: change file permissions
    - `r=4, w=2, x=1`
    - e.g. `chmod 600`: `rw` permission for owner, denies permission for group members and others
- shebang: `#!` beginning of a script, specifies the intepreter to be used, e.g. `#!/bin/bash`

## Lecture 2: Shell Tools and Scripting
### Shell Scripting
**Assign variables:**
```bash
foo=bar
echo "$foo"
# prints bar
echo '$foo'
# prints $foo
```
- spacing will perform argument spliting

**Control flow:**
```bash
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
```bash
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
```bash
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
```bash
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
```python
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
- help, man, tldr

#### Finding files
**find file or directory**:
```bash
# Find all directories named src
find . -name src -type d
# Find all python files that have a folder named test in their path
find . -path '*/test/*.py' -type f
# Find all files modified in the last day
find . -mtime -1
# Find all zip files with size in range 500k to 10M
find . -size +500k -size -10M -name '*.tar.gz'
```

**Perform actions**:
```bash
# Delete all files with .tmp extension
find . -name '*.tmp' -exec rm {} \;
# Find all PNG files and convert them to JPG
find . -name '*.png' -exec convert {} {}.jpg \;
```

**`fd`**:
- alternative to `find`
- offers some nice defaults like colorized output, default regex matching, and Unicode support
```bash
fd "*.py"
```

**`locate`**:
- uses a database that is updated using `updatedb`
- `updatedb` is updated daily via `cron`

#### Finding code
**`grep`**:
- `-R`: recursive search
- `-C`: context lines
- `-v`: invert match

**`rg`**:
```bash
# Find all python files where I used the requests library
rg -t py 'import requests'
# Find all files (including hidden files) without a shebang line
rg -u --files-without-match "^#\!"
# Find all matches of foo and print the following 5 lines
rg foo -A 5
# Print statistics of matches (# of matched lines and files )
rg --stats PATTERN
```

#### Finding shell commands
- up arrow
- `history`, e.g. `history | grep`
- `Ctrl + R`
- `fzf`: fuzzy finder
- `fish`: history-based autosuggestions


#### Directory navigation
- shell aliases: `alias`
- symlinks: `ln -s`
- autojump: `fasd`, `autojump`
- overview of directory structure: `tree`, `broot`, `nnn`, `ranger`

### Exercises
- `xargs`: execute a command using `STDIN` as arguments

## Lecture 3: Editors (Vim)
### Modal editing
**Operating modes**:
- Normal: for moving around a file and making edits; `<ESC>`
- Insert: for inserting text; `i`
- Replace: for replacing text; `R`
- Visual (plain, line, or block): for selecting blocks of text; `v`, `V` for line, `<C-v>` for block
- Command-line: for running a command; `:`

### Basics
#### Buffers, tabs, and windows
- buffers: set of open files
- tabs: collection of one or more windows
- windows: each window shows a single buffer

#### Command-line
- `:q` quit (close window), `:qa` to close all windows, `q!` to discard changes
- `:w` save (“write”)
- `:wq` save and quit
- `:e` {name of file} open file for editing
- `:ls` show open buffers
- `:help` {topic} open help
    - `:help :w` opens help for the `:w` command
    - `:help w` opens help for the `w` movement
- `:sp`: split window
- `:tabnew`

### Vim's interface is a programming language
#### Movement (nouns)
- Basic movement: `hjkl` (left, down, up, right)
- Words: `w` (next word), `b` (beginning of word), `e` (end of word)
- Lines: `0` (beginning of line), `^` (first non-blank character), `$` (end of line)
- Screen: `H` (top of screen), `M` (middle of screen), `L` (bottom of screen)
- Scroll: `Ctrl-u` (up), `Ctrl-d` (down)
- File: `gg` (beginning of file), `G` (end of file)
- Line numbers: `:{number}<CR>` or `{number}G` (line {number})
- Misc: `%` (corresponding item)
- Find: `f{character}`, `t{character}`, `F{character}`, `T{character}`
    - find/to forward/backward {character} on the current line
    - `,`/ `;` for navigating matches
- Search: `/{regex}`, `n` / `N` for navigating matches

#### Selection
- Visual: `v`
- Visual Line: `V`
- Visual Block: `Ctrl-v`

#### Edits (verbs)
- `i` enter Insert mode
    - but for manipulating/deleting text, want to use something more than backspace
- `o` / `O` insert line below / above
- `d`{motion}` delete {motion}
    - e.g. `dw` is delete word, `d$` is delete to end of line, `d0` is delete to beginning of line
- `c{motion}` change {motion}
    - e.g. `cw` is change word
    - like `d{motion}` followed by i
- `x` delete character (equal do `dl`)
- `r` replace character
- `s` substitute character (equal to `cl`)
- Visual mode + manipulation
    - select text, `d` to delete it or `c` to change it
- `u` to undo, `<C-r>` to redo
- `y` to copy / “yank” (some other commands like `d` also copy); `yy` line, `yw` word
- `p` to paste
- `/` for next match
- `n` for next match
- `.` repeats previous editing command
- `A` append to end of line
- Lots more to learn: e.g. `~` flips the case of a character

#### Counts
- `3w` move 3 words forward
- `5j` move 5 lines down
- `7dw` delete 7 words

#### Modifiers
- `ci(` change the contents inside the current pair of parentheses
- `ci[` change the contents inside the current pair of square brackets
- `da'` delete a single-quoted string, including the surrounding single quotes

### Customizing Vim
- customize through `~/.vimrc`

### Extending Vim
- create the directory `~/.vim/pack/vendor/start/`
- put plugins in there (e.g. via `git clone`)

### Exercises
TO-DO
- `vimtutor`
- install anid configure a plugin
- advanced vivimm

## Lecture 4: Data Wrangling
### Introduction
**`less`**:
```bash
ssh myserver 'journalctl | grep sshd | grep "Disconnected from"' > ssh.log
less ssh.log
```
- single quote to do the filtering on the remote server
- `less`: gives a "pager"

**stream editor**:
```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed 's/.*Disconnected from //'
```
- to modify file, rather than manipulate its content directly
- `s/REGEX/SUBSTITUTION`

### Regular expressions
**Common patterns**:
- `.` means “any single character” except newline
- `*` zero or more of the preceding match
- `+` one or more of the preceding match
- `[abc]` any one character of a, b, and c
- `(RX1|RX2)` either something that matches RX1 or RX2
- `^` the start of the line
- `$` the end of the line

**Example**:
```bash
echo 'aba' | sed 's/[ab]//'
# ba
echo 'aba' | sed 's/[ab]//g'
#
echo 'abcaba' | sed -E 's/(ab)*//g'
# ca
echo 'abcaba' | sed 's/\(ab\)*//g'
# ca
echo 'Disconnected from invalid user Disconnected from 84.211' | sed 's/.*Disconnected from//'
# 84.211
echo 'Disconnected from invalid user Disconnected from 84.211' | perl -pe 's/.*?Disconnected from//'
# invalid user Disconnected from 84.211
```
- replace once by default, add `g` modifier for all occurences 
- `sed` is weird, need to add `\` or `-E` to give special meaning
- `*` and `+` are by default greedy: match the most text
- `perl`: suffix `*` or `+` with `?` to make them non-greedy (not available in `sed`)

**Misc**:
- `\d`: any digit
- `\D`: any non-digit
- `\.`: period
- `[^abc]`: any single character expect for a, b, c
- `[a-z]`: characters a to z
- `\w`: any alphanumeric character, equivalent to `[A-Za-z0-9_]`
- `\W`: any non-alphanumeric character
- `a{m,n}`: m to n repetitions of character a
- `.*` :zero or more of any character
- `a?`: optional character of a
- `\s`: any whitespace (space `..`, tab `\t`, new line `\n`, carriage return `\r`)
- `\S`: any non-whitespace character
- `(...)`: capture group (access with numbered capture group, e.g. `\1`, `\2`, `\3`)
- `(a(bc))`: capture sub-group

**regex debugger**:
https://regex101.com/r/qqbZqh/2

### Back to data wrangling
```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
 | sort -nk1,1 | tail -n10
 | awk '{print $2}' | paste -sd,
```
- `wc`: word count (`-l` lines)
- `sort`: sort output (`-r` sort in reverse, `-n` sort in numeric, `-kl,1` sort by only the first whitespace-separated column, sort until the 1st field)
- `uniq`: filter repeated lines (`-c` for number of occurences)
- `awk`: programming language for processing text streams
- `paste -sd,`: combine lines by character delimiter `,`

### awk - another editor
**example**:
the number of single-use usernames that start with c and end with e
```bash
 | awk '$1 == 1 && $2 ~ /^c[^ ]*e$/ { print $2 }' | wc -l
```
or 
```bash
BEGIN { rows = 0 }
$1 == 1 && $2 ~ /^c[^ ]*e$/ { rows += $1 }
END { print rows }
```

### Analyzing data
**Calculator**:
- `bc`
```bash
echo "1+2" | bc -l
# 3
```

**Stats**:
- `st`
- `R`
```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
 | awk '{print $1}' | R --no-echo -e 'x <- scan(file="stdin", quiet=TRUE); summary(x)'
```
- `gnuplot` for simple plotting
```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
 | sort -nk1,1 | tail -n10
 | gnuplot -p -e 'set boxwidth 0.5; plot "-" using 1:xtic(2) with boxes'
```

### Data wrangling to make arguments
Uninstall old nightly builds of Rust from my system by extracting the old build names using data wrangling tools and then passing them via `xargs` to the uninstaller
```bash
rustup toolchain list | grep nightly | grep -vE "nightly-x86" | sed 's/-x86.*//' | xargs rustup toolchain uninstall
```

### Wrangling binary data
use ffmpeg to capture an image from our camera, convert it to grayscale, compress it, send it to a remote machine over SSH, decompress it there, make a copy, and then display it
```bash
ffmpeg -loglevel panic -i /dev/video0 -frames 1 -f image2 -
 | convert - -colorspace gray -
 | gzip
 | ssh mymachine 'gzip -d | tee copy.jpg | env DISPLAY=:0 feh -'
```

### Exercises
TO-DO
- `curl`
- `pup`
- `jq`

## Lecture 5: Command-line Environment
### Job Control
- The shell uses a UNIX communication mechanism called a signal to communicate information to the process

#### Killing a process
**Signals**:
- `SIGINT`: 
    - `Ctrl+C`
    - interrupt program
- `SIGQUIT`: 
    - `Ctrl+\`
    - quit program
- `SIGTERM`: 
    - `kill -TERM <PID>`
    - software termination signal (most cases equivalent to `SIGINT` & `SIGQUIT`, not sent through terminal)
- `SIGKILL`:
    - `kill -KILL <PID>`
    - cannot be captured by the process
    - terminate process no immediately
    - leaving orphaned children processes

**Example**:
- captures `SIGINT` and ignores, no longer stopping
```python
#!/usr/bin/env python
import signal, time

def handler(signum, time):
    print("\nI got a SIGINT, but I am not stopping")

signal.signal(signal.SIGINT, handler)
i = 0
while True:
    time.sleep(.1)
    print("\r{}".format(i), end="")
    i += 1

#####

$ python sigint.py
24^C
I got a SIGINT, but I am not stopping
26^C
I got a SIGINT, but I am not stopping
30^\[1]    39913 quit       python sigint.py
```

#### Pausing and backgrounding processes
**Signals**:
- `SIGHUP`: 
    - `kill -SIGHUP <PID>`
    - terminal line hangup (close terminal, terminate running processes)
- `SIGSTOP`: 
    - `Ctrl+Z` will send `SIGTSTP` (terminal's version of `SIGSTOP`)
    - stop (cannot obe caught or ignored) / pause a process  
- `SIGCONT`: 
    - `fg <PID>`, `bg <PID>`
    - continue after stop

**Example**:
- `nohup`: a wrapper to ignore `SIGHUP`
- `&`: runs the command in the background
- `fg` & `bg` to continue paused job in foreground or background
- `jobs` list unfinished jobs associated with the current terminal session
```bash
$ sleep 1000
^Z
[1]  + 18653 suspended  sleep 1000

$ nohup sleep 2000 &
[2] 18745
appending output to nohup.out

$ jobs
[1]  + suspended  sleep 1000
[2]  - running    nohup sleep 2000

$ bg %1
[1]  - 18653 continued  sleep 1000

$ jobs
[1]  - running    sleep 1000
[2]  + running    nohup sleep 2000

$ kill -STOP %1
[1]  + 18653 suspended (signal)  sleep 1000

$ jobs
[1]  + suspended (signal)  sleep 1000
[2]  - running    nohup sleep 2000

$ kill -SIGHUP %1
[1]  + 18653 hangup     sleep 1000

$ jobs
[2]  + running    nohup sleep 2000

$ kill -SIGHUP %2

$ jobs
[2]  + running    nohup sleep 2000

$ kill %2
[2]  + 18745 terminated  nohup sleep 2000

$ jobs
```

### Terminal Multiplexers
**Sessions** - a session is an independent workspace with one or more windows
- `tmux` starts a new session.
- `tmux new -s NAME` starts it with that name.
- `tmux ls `lists the current sessions
- Within `tmux` typing `<C-b> d` detaches the current session
- `tmux a` attaches the last session. You can use `-t` flag to specify which
- kill session:
    - inside the session: `<C-b> :`, then type `kill-session`
    - else: `tmux kill-session -t`

**Windows** - Equivalent to tabs in editors or browsers, they are visually separate parts of the same session
- `<C-b> c` Creates a new window. To close it you can just terminate the shells doing `<C-d>`
- `<C-b> N` Go to the N th window. Note they are numbered
- `<C-b> p` Goes to the previous window
- `<C-b> n` Goes to the next window
- `<C-b> ,` Rename the current window
- `<C-b> w` List current windows

**Panes** - Like vim splits, panes let you have multiple shells in the same visual display.
 -`<C-b> "` Split the current pane horizontally
 -`<C-b> %` Split the current pane vertically
 -`<C-b> <direction>` Move to the pane in the specified direction. Direction here means arrow keys.
 -`<C-b> z` Toggle zoom for the current pane
 -`<C-b> [` Start scrollback. You can then press `<space>` to start a selection and `<enter>` to copy that selection.
 -`<C-b> <space>` Cycle through pane arrangements.

### Aliases
- short form for command
- make alias persistent: `.bashrc` or `.zshrc`

**Features**:
```bash
# Make shorthands for common flags
alias ll="ls -lh"

# Save a lot of typing for common commands
alias gs="git status"
alias gc="git commit"
alias v="vim"

# Save you from mistyping
alias sl=ls

# Overwrite existing commands for better defaults
alias mv="mv -i"           # -i prompts before overwrite
alias mkdir="mkdir -p"     # -p make parent dirs as needed
alias df="df -h"           # -h prints human readable format

# Alias can be composed
alias la="ls -A"
alias lla="la -l"

# To ignore an alias run it prepended with \
\ls
# Or disable an alias altogether with unalias
unalias la

# To get an alias definition just call it with alias
alias ll
# Will print ll='ls -lh'
```

### Dotfiles
- hidden in `ls` by default
- many programs will ask you to include a line like `export PATH="$PATH:/path/to/program/bin"` in your shell configuration file so their binaries can be found
- dotfile repos: https://github.com/mathiasbynens/dotfiles
- dotfiles should be in their own folder, under version control, and symlinked (`ln -s`) into place using a script

**Example of dotfiles**:
- `bash` - `~/.bashrc`, `~/.bash_profile`
- `git` - `~/.gitconfig`
- `vim` - `~/.vimrc` and the `~/.vim folder`
- `ssh` - `~/.ssh/config`
- `tmux` - `~/.tmux.conf`

### Remote Machines
#### Executing commands
- execute `ls` in the home of `foobar`, `grep` locally the remote output of `ls`
```bash
ssh foobar@server ls | grep PATTERN
```

- `grep` remotely the local output of `ls`
```bash
ls | ssh foobar@server grep PATTERN
```

#### SSH keys
- Public-key cryptography to prove to the server that, the client owns the secret private key without revealing the key
- Do not need to reenter password every time
- Private key (`~/.ssh/id_rsa` or `~/.ssh/id_ed25519`) is effectively your password

**Key generation**:
- To generate public-private key pair
```bash
ssh-keygen -o -a 100 -t ed25519 -f ~/.ssh/id_ed25519
```

- To check if passphrase is available and validate
```bash
ssh-keygen -y -f /path/to/key
```

**Key-based authentication**:
- `ssh` will look into public key (`.ssh/authorized_keys`) to determine which clients to let in

Copy a public key over
```bash
cat .ssh/id_ed25519.pub | ssh foobar@remote 'cat >> ~/.ssh/authorized_keys'
```
or
```bash
ssh-copy-id -i .ssh/id_ed25519 foobar@remote
```

#### Copying files over SSH
- `ssh+tee`
    - `cat localfile | ssh remote_server tee serverfile`
- `scp`
    - `scp path/to/local_file remote_host:path/to/remote_file`
    - copy large amounts of file
- `rcp`
    - `rcp -avP . remote_host:path/`
    - detect identical files in local and remote, prevent copying again

#### Port Forwarding
**Local Port Forwarding**:
- Specifies that the given port on the local (client) host is to be forwarded to the given host and port on the remote side
- `ssh -L sourcePort:forwardToHost:onPort connectToHost` means: connect with ssh to `connectToHost`, and forward all connection attempts to the local `sourcePort` to port `onPort` on the machine called `forwardToHost`, which can be reached from the `connectToHost` machine.
![local-port-forwarding](./media/local-port-forwarding.png)

**Remote Port Forwarding**:
- Specifies that the given port on the remote (server) host is to be forwarded to the given host and port on the local side
- `ssh -R sourcePort:forwardToHost:onPort connectToHost` means: connect with ssh to `connectToHost`, and forward all connection attempts to the remote `sourcePort` to port `onPort` on the machine called `forwardToHost`, which can be reached from your local machine.
![remote-port-forwarding](./media/remote-port-forwarding.png)

#### SSH Configuration
**Shell alias**:
```bash
alias my_server="ssh -i ~/.id_ed25519 --port 2222 -L 9999:localhost:8888 foobar@remote_server
```

**`~/.ssh/config`**:
- dotfile, able to be read by other programs like `scp`, `rsync`, `mosh`
- server side config: `/etc/ssh/sshd_config`
```bash
Host vm
    User foobar
    HostName 172.16.174.141
    Port 2222
    IdentityFile ~/.ssh/id_ed25519
    LocalForward 9999 localhost:8888

# Configs can also take wildcards
Host *.mit.edu
    User foobaz
```

### Exercises
TO-DO

## Lecture 6: Version Control (Git)
### Git's data model
#### Snapshots
- tree: directory, maps names to blobs or trees
- blob: file (bunch of bytes)
- snapshot / commit: top-level tree that is being tracked

**Example**:
- top-level tree has two elements: tree "foo", blob "baz.txt"
```
<root> (tree)
|
+- foo (tree)
|  |
|  + bar.txt (blob, contents = "hello world")
|
+- baz.txt (blob, contents = "git is wonderful")
```

#### Modeling history: relating snapshots
- history: directed acyclic graph (DAG) of snapshots
- each snapshot refers to a set of parents (the snapshots that preceded it)
- single parent: linear history
- multiple parents: merging two parallel branches
- commits are immutable, "edits" to the commit history are creating new commits

**Example**:
```
o <-- o <-- o <-- o <---- o
            ^            /
             \          v
              --- o <-- o
```

#### Data model, as pseudocode
```
// a file is a bunch of bytes
type blob = array<byte>

// a directory contains named files and directories
type tree = map<string, tree | blob>

// a commit has parents, metadata, and the top-level tree
type commit = struct {
    parents: array<commit>
    author: string
    message: string
    snapshot: tree
}
```

#### Objects and content-addressing
Object: blob, tree, or commit
```
type object = blob | tree | commit
```

In Git data store, all objects are content-addressed by their SHA-1 hash (160 bits, 40 hexadecimal characters)
```
objects = map<string, object>

def store(object):
    id = sha1(object)
    objects[id] = object

def load(id):
    return objects[id]
```

**Example**:
- When objects reference other object,  they don’t actually contain them in their on-disk representation, but have a reference to them by their hash
- visualize commit: `git cat-file -p 698281bc680d1995c5f4caaf3359721a5a58d48d`
```
100644 blob 4448adbf7ecd394f42ae135bbeed9676e894af85    baz.txt
040000 tree c68d233a33c5c06e0340e4c224f0afca87c8ce87    foo
```
- visualize `baz.txt`: `git cat-file -p 4448adbf7ecd394f42ae135bbeed9676e894af85`
```
git is wonderful
```

#### References
- human-readable names for SHA-1 hashes
- pointers to commits
- mutable (update to point new commit)
- e.g. `master` reference points to the latest commit in the main branch
- `HEAD`: reference to current check-out commit 
    - e.g. `HEAD -> main -> C1`; `git checkout main`
- detached `HEAD`: when `HEAD` points to a commit rather than a branch 
    - e.g. `HEAD -> C1`; `git checkout C1`

```
references = map<string, string>

def update_reference(name, id):
    references[name] = id

def read_reference(name):
    return references[name]

def load_reference(name_or_id):
    if name_or_id in references:
        return load(references[name_or_id])
    else:
        return load(name_or_id)
```

#### Repositories
- Git repository: it is the data `objects` and `references`
- all `git` commands map to some manipulation of the commit DAG by adding `objects` and adding/updating `references`

### Staging area
Allow to specify which modifications should be included in the next snapshot

### Git command-line interface
#### Basics
- `git help <command>`: get help for a git command
- `git init`: creates a new git repo, with data stored in the .git directory
    - `git init --bare`: create bare repositories (for central repositories, does not have working directory)
- `git status`: tells you what’s going on
- `git add <filename>`: adds files to staging area
- `git commit`: creates a new commit
- `git log`: shows a flattened log of history
- `git log --all --graph --decorate --oneline`: visualizes history as a DAG
- `git diff <filename>`: show what has changed but hasn't been added to the index yet via `git add`
- `git diff --cached`: show what has been added to the index  via `git add` but not yet committed
- `git diff <revision> <filename>`: shows differences in a file between snapshots
- `git checkout <revision>`: updates HEAD and current branch
    - `git checkout HEAD^`: check out to parent commit (move upwards 1 time); `^` (parent directly above); `^2` (2nd parent) 
    - `git checkout HEAD~n`: move upwards `n` times

#### Branching and merging
- `git branch`: shows branches
    - `git branch -vv` for verbose display
    - `git branch -f main HEAD~3`: moves (by force) the `main` branch 3 parents behind `HEAD`
- `git branch <name>`: creates a branch
- `git checkout -b <name>`: creates a branch and switches to it
    - `same as git branch <name>; git checkout <name>`
    - `git checkout -b foo origin/main`: set local branch `foo` to track `origin/main`
- `git merge <revision>`: merges into current branch
    - need to stage the file to mark the conflict as resolved
    - `git merge --continue` instead of `git commit` to complete the merge
    - `git merge --abort` to abort the merge
- `git mergetool`: use a fancy tool to help resolve merge conflicts
- `git rebase`: rebase set of patches onto a new base
- `git rebase <basebranch> <topicbranch>`: checks out the topic branch for you and replays it onto the base branch

#### Remotes
- `git remote`: list remotes
- `git remote add <name> <url>`: add a remote
- `git push <remote> <local branch>:<remote branch>`: send objects to remote, and update remote reference
    - `git push <remote> :<remote branch>`: deletes remote branch
- `git branch --set-upstream-to=<remote>/<remote branch>`: set up correspondence between local and remote branch
    - `git branch -u origin/main foo`: set local branch `foo` to track `origin/main`
- `git fetch`: retrieve objects/references from a remote (does not update local branch)
    - `git fetch <remote> <remote branch>:<local branch>`: fetch remote branch from remote to local branch
    - `git fetch <remote> :<local branch>`: fetching nothing makes new local branch 
- `git pull`: same as git fetch; git merge
    - `git pull --rebase`: fetch and rebase
    - `git pull origin foo`: `git fetch origin foo; git merge origin/foo`
    - `git pull origin bar:bugFix`: `git fetch origin bar:bugFix; git merge bugFix`
- `git clone`: download repository from remote

#### Undo
- `git commit --amend`: edit a commit’s contents/message
- `git reset HEAD <file>`: unstage a file
- `git reset HEAD~1`: move a branch backwards
- `git checkout -- <file>`: discard changes
- `git revert HEAD`: creates a new commit that effectively negates the changes introduced by the specified commit

### Advanced Git
- `git config`: Git is highly customizable, `~/.gitconfig`
- `git clone --depth=1`: shallow clone, without entire version history
- `git add -p`: interactive staging
- `git rebase -i`: interactive rebasing
- `git blame`: show who last edited which line
- `git stash`: temporarily remove modifications to working directory
    - `git stash pop` to undo the stash
- `git bisect`: binary search history (e.g. for regressions)
- `.gitignore`: specify intentionally untracked files to ignore
- `git show <commit>`: show commit
- `git cherry-pick <commit1> <commit2>`: apply the changes introduced by specific commits from one branch to another branch
- `git tag <tagname> <commit>`: add tag at commit
- `git describe <ref>`: describe where you are relative to the closest tag
    - outputs `<tag>_<numCommits>_g<hash>`

### Workflows
**Types of workflows**:
- Centralized workflow
    - centralized repository that individual developers will push and pull from 
- Feature branch workflow
    - all feature development should take place in a dedicated branch instead of the `main` branch
- Gitflow workflow
    - `develop`, `feature`, `release`, `hotfix` branches
- Forking workflow
    - instead of a single server-side repository to act as the "central" codebase, it gives every developer a server-side repository
    - each contributor has two Git repositories: a private local one and a public server-side one

**Gitflow**:
![git-flow](./media/git-flow.png)

**The overall flow of Gitflow**:
1. A `develop` branch is created from `main`
2. A `release` branch is created from `develop`
3. `Feature` branches are created from `develop`
4. When a `feature` is completed it is merged into the `develop` branch
5. When the `release` branch is done it is merged into `develop` and `main`
6. If an issue in `main` is detected a `hotfix` branch is created from `main`
7. Once the `hotfix` is complete it is merged to both `develop` and `main`

**References**:
- https://www.endoflineblog.com/gitflow-considered-harmful
- https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow
- https://nvie.com/posts/a-successful-git-branching-model/

### Write good commit messages
**Example**:
```
Summarize changes in around 50 characters or less

More detailed explanatory text, if necessary. Wrap it to about 72
characters or so. In some contexts, the first line is treated as the
subject of the commit and the rest of the text as the body. The
blank line separating the summary from the body is critical (unless
you omit the body entirely); various tools like `log`, `shortlog`
and `rebase` can get confused if you run the two together.

Explain the problem that this commit is solving. Focus on why you
are making this change as opposed to how (the code explains that).
Are there side effects or other unintuitive consequences of this
change? Here's the place to explain them.

Further paragraphs come after blank lines.

 - Bullet points are okay, too

 - Typically a hyphen or asterisk is used for the bullet, preceded
   by a single space, with blank lines in between, but conventions
   vary here

If you use an issue tracker, put references to them at the bottom,
like this:

Resolves: #123
See also: #456, #789
1. Separate subject fr
```

**The seven rules of a great Git commit message**
- One more thing: atomic commits!
1. Separate subject from body with a blank line
2. Limit the subject line to 50 characters
3. Capitalize the subject line
4. Do not end the subject line with a period
5. Use the imperative mood in the subject line
6. Wrap the body at 72 characters
7. Use the body to explain what and why vs. how

**References**:
- https://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html
- https://cbea.ms/git-commit/

### Exercises
TO-DO
- https://learngitbranching.js.org/
- https://git-scm.com/book/en/v2

## Lecture 7: Debugging and Profiling
### Printf debugging and Logging
#### Advantages of logging over ad hoc print statements
- Log to files, sockets, and even remote servers instead of standard ouput
- Supports severity levels (python: `NOTSET`, `DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL`)
    - Allows to filter the output
- New issues have a high chance of producing logs with sufficient information to detect the problem
- Logs can be colored

#### Third party logs
- Large software systems have dependencies that run as separate programs (e.g. web servers, databases, message brokers, etc.)
    - Read their logs, since client side error messages might not suffice
- UNIX systems: programs write logs under `/var/log`
    - e.g. to view logs `cat /var/log/syslog | lnav`
- System logs (Linux): use `systemd` as system daemon
    - place logs under `/var/log/journal`
    - `journalctl` to display the log
    - `logger` for logging under the systems logs
```bash
logger "Hello Logs"
# On macOS
log show --last 1m | grep Hello
# On Linux
journalctl --since "1m ago" | grep Hello
```

#### Debuggers
Debuggers are programs that let you interact with the execution of a program:
- Halt execution of the program when it reaches a certain line
- Step through the program one instruction at a time
- Inspect values of variables after the program crashed
- Conditionally halt the execution when a given condition is met

**Python Debugger `pdb`**:
- l(ist) - Displays 11 lines around the current line or continue the previous listing.
- s(tep) - Execute the current line, stop at the first possible occasion.
- n(ext) - Continue execution until the next line in the current function is reached or it returns.
- b(reak) - Set a breakpoint (depending on the argument provided).
- p(rint) - Evaluate the expression in the current context and print its value. There’s also pp to display using `pprint` instead.
    - `p locals()`: dictionary representing the current local symbol table (local variables in the current scope)
- r(eturn) - Continue execution until the current function returns.
- q(uit) - Quit the debugger.
- c(continue) - resumes execution of the program until the next breakpoint or the program finishes (e.g. error)
- restart - restarts the debugger and reruns the program from the beginning

**`ipdb`**:
- improved pdb that uses the `IPython` REPL 
- enabling tab completion, syntax highlighting, better tracebacks, and better introspection - retaining the same interface as the `pdb` module
```bash
python -m ipdb bubble.py
```

**Low level programming debugger**:
- `gdb`, `pwndbg`, `lldb`
- optimized for C-like language debugging
- probe pretty much any process and get its current machine state: registers, stack, program counter, etc.

#### Specialized Tools
- Programs use **system calls** (a program requests a service from the operating system) to perform actions that only the kernel can
- `strace` to trace the syscalls made by the program
```bash
# On Linux
sudo strace -e lstat ls -l > /dev/null
# On macOS
sudo dtruss -t lstat64_extended ls -l > /dev/null
```

##### Static Analysis
- Static analysis programs take source code as input and analyze it using coding rules to reason about its correctness
- Python: `pyflakes`, `mypy`
- shell: `shellcheck`

**Code linting**:
- e.g. display stylistic violations or insecure constructs
- vim: `ale`, `syntastic`
- Python: 
    - stylistic linters: `pylint`, `pep8`
    - find common security issues: `bandit`
    - code formatter: `black`

**References**:
- https://github.com/analysis-tools-dev/static-analysis
- https://github.com/caramelomartins/awesome-linters

### Profiling
- Premature optimization is the root of all evil
- Profilers & Monitoring Tools: 
    - helps to understand which part of the program is taking most of the time and/or resources
    - thus focus on optimizing those parts

#### Timing
**Python using the `time` module**:
- wall clock time (*Real*) is misleading
- computer might be running other process at the same time
- or waiting for events to happen
- *User* + *Sys*: how much time the process actually spent in the CPU
```python
import time, random
n = random.randint(1, 10) * 100

# Get current time
start = time.time()

# Do some work
print("Sleeping for {} ms".format(n))
time.sleep(n/1000)

# Compute time between start and now
print(time.time() - start)

# Output
# Sleeping for 500 ms
# 0.5713930130004883
```

**Real, User, Sys time**
1. *Real*
    - Wall clock elapsed time from start to finish of the program, including the time taken by other processes and time taken while blocked (e.g. waiting for I/O or network)
2. *User*
    - Amount of time spent in the CPU running user code
3. *Sys*
    - Amount of time spent in the CPU running kernel code

**Example**:
- 2 seconds for the request to complete
- 15ms of CPU user time
- 12ms of kenel CPU time
```bash
$ time curl https://missing.csail.mit.edu &> /dev/null
real    0m2.561s
user    0m0.015s
sys     0m0.012s
```

#### Profilers
**CPU**:
- Profilers usuallly mean CPU profilers
- Two main types: tracing and sampling profilers
    - Tracing: keep a record of every function call the program makes
    - Sampling: probe the program periodically and record the program's stack
- Profilers present aggregate statistics of what the program spent the most time doing

**Python `cProfile`**:
- to profile time per fuction call
```python
#!/usr/bin/env python

import sys, re

def grep(pattern, file):
    with open(file, 'r') as f:
        print(file)
        for i, line in enumerate(f.readlines()):
            pattern = re.compile(pattern)
            match = pattern.search(line)
            if match is not None:
                print("{}: {}".format(i, line), end="")

if __name__ == '__main__':
    times = int(sys.argv[1])
    pattern = sys.argv[2]
    for i in range(times):
        for file in sys.argv[3:]:
            grep(pattern, file)
```
- IO takes most of the time
- compiling the regex takes a fair amount of time
- caveat: display time per function call
```bash
$ python -m cProfile -s tottime grep.py 1000 '^(import|\s*def)[^,]*$' *.py

[omitted program output]

 ncalls  tottime  percall  cumtime  percall filename:lineno(function)
     8000    0.266    0.000    0.292    0.000 {built-in method io.open}
     8000    0.153    0.000    0.894    0.000 grep.py:5(grep)
    17000    0.101    0.000    0.101    0.000 {built-in method builtins.print}
     8000    0.100    0.000    0.129    0.000 {method 'readlines' of '_io._IOBase' objects}
    93000    0.097    0.000    0.111    0.000 re.py:286(_compile)
    93000    0.069    0.000    0.069    0.000 {method 'search' of '_sre.SRE_Pattern' objects}
    93000    0.030    0.000    0.141    0.000 re.py:231(compile)
    17000    0.019    0.000    0.029    0.000 codecs.py:318(decode)
        1    0.017    0.017    0.911    0.911 grep.py:3(<module>)

[omitted lines]
```

**Line profilers**"
- using `cProfile` would get over 2500 lines of output
- even with sorting, hard to understand where the time is spent
- line profiler: shows the time taken per line
```python
#!/usr/bin/env python
import requests
from bs4 import BeautifulSoup

# This is a decorator that tells line_profiler
# that we want to analyze this function
@profile
def get_urls():
    response = requests.get('https://missing.csail.mit.edu')
    s = BeautifulSoup(response.content, 'lxml')
    urls = []
    for url in s.find_all('a'):
        urls.append(url['href'])

if __name__ == '__main__':
    get_urls()
```
```bash
$ kernprof -l -v a.py
Wrote profile results to urls.py.lprof
Timer unit: 1e-06 s

Total time: 0.636188 s
File: a.py
Function: get_urls at line 5

Line #  Hits         Time  Per Hit   % Time  Line Contents
==============================================================
 5                                           @profile
 6                                           def get_urls():
 7         1     613909.0 613909.0     96.5      response = requests.get('https://missing.csail.mit.edu')
 8         1      21559.0  21559.0      3.4      s = BeautifulSoup(response.content, 'lxml')
 9         1          2.0      2.0      0.0      urls = []
10        25        685.0     27.4      0.1      for url in s.find_all('a'):
11        24         33.0      1.4      0.0          urls.append(url['href'])
```

**Memory**:
- C/C++:
    - memory leaks: program to never release memory
    - Valgrind: identify memory leaks
- Python:
    - garbage collected languages
    - as long as having pointers to objects to memory, they won't be garbage collected

```python
@profile
def my_func():
    a = [1] * (10 ** 6)
    b = [2] * (2 * 10 ** 7)
    del b
    return a

if __name__ == '__main__':
    my_func()
```
```bash
$ python -m memory_profiler example.py
Line #    Mem usage  Increment   Line Contents
==============================================
     3                           @profile
     4      5.97 MB    0.00 MB   def my_func():
     5     13.61 MB    7.64 MB       a = [1] * (10 ** 6)
     6    166.20 MB  152.59 MB       b = [2] * (2 * 10 ** 7)
     7     13.61 MB -152.59 MB       del b
     8     13.61 MB    0.00 MB       return a
```

**Event Profiling**:
- `perf`: abstracts CPU differences away and does not report time or memory, but instead it reports system eventes related to you program
- e.g. report poor cache locality, high amounts of page faults or livelocks

**Visualization**:
- Flame graph:
    - Y axis: hierarachy of function calls
    - X axis: time taken
- Call graph / control flow graph:
    - display the relationships between subroutines within a program by including functions as nodes and functions calls between them as directed edges
    - Python: `pycallgraph`

#### Resource Monitoring
`stress`: artificially impose loads on the loads to test these tools:
1. General monitoring: `htop`, `glances`, `dstat`
2. I/O operations: `iotop`
3. Disk usage: `df`, `du`, `ncdu`
4. Memory usage: `free`
5. Open files: `lsof`
6. Network connections and config: `ss`, `ip`
7. Network usage: `nethogs`, `iftop`

**Specialized tools**:
- `hyperfine`: quickly benchmark command line programs
- `fd` was 20x faster than `find`
```bash
$ hyperfine --warmup 3 'fd -e jpg' 'find . -iname "*.jpg"'
Benchmark #1: fd -e jpg
  Time (mean ± σ):      51.4 ms ±   2.9 ms    [User: 121.0 ms, System: 160.5 ms]
  Range (min … max):    44.2 ms …  60.1 ms    56 runs

Benchmark #2: find . -iname "*.jpg"
  Time (mean ± σ):      1.126 s ±  0.101 s    [User: 141.1 ms, System: 956.1 ms]
  Range (min … max):    0.975 s …  1.287 s    10 runs

Summary
  'fd -e jpg' ran
   21.89 ± 2.33 times faster than 'find . -iname "*.jpg"'
```

### Exercises
TO-DO

## Lecture 8: Metaprogramming
### Build systems
**`make`**:
- consults `Makefile`
-  the things named on the right-hand side are dependencies, and the left-hand side is the target
- the indented block is a sequence of programs to produce the target from those dependencies
- the first directive is the default goal
```bash
paper.pdf: paper.tex plot-data.png
	pdflatex paper.tex

plot-%.png: %.dat plot.py
	./plot.py -i $*.dat -o $@
```

### Dependency management
**Repository**
- hosts a large number of dependencies in a single place
- Examples: 
    - Ubuntu package repositories for Ubuntu system packages, accessed through `apt` tool
    - PyPI for Python librarires

**Sematic versioning**: 
-  major.minor.path
- e.g. code written for Python 3.5 might run fine on Python 3.7, but possibly not on 3.4

1. If a new release does not change the API, increase the patch version
2. If you add to your API in a backwards-compatible way, increase the minor version
3. If you change the API in a non-backwards-compatible way, increase the major version

**Lock files**
- a file that lists the exact version currently depending on of each dependency
- vendoring: copy all the code of your dependencies into your own project

### Continuous integration systems
- CI: umbrella term for "stuff that runs whenever your code changes"
- Examples:
    - event triggered: 
        - general: Travis CI, Azure Pipelines, GitHub Actions
        - website: GitHub Pages
    - dependency version: Dependabot, alert when there are updates or security vulnerabilities, keeping dependencies up to date
- How it works:
    - roughly: you add a file (rule/recipe) to your repository that describes what should happen when various things happen to that repository
    - more detailed: when the event triggers, the CI provider spins up virtual machines, runs the commands in your "recipe", and then usually notes down the results somewhere
- Most common rule/recipe: 
    - when someone pushes code (or submit pull requests), run the test suite
    - check your code style everytime you commit
-  You might set it up so that you are notified if the test suite stops passing, or so that a little badge appears on your repository (e.g. top of README.md) as long as the tests pass

**A brief aside on testing**:
- Test suite: a collective term for all the tests
- Unit test: a “micro-test” that tests a specific feature in isolation
- Integration test: a “macro-test” that runs a larger part of the system to check that different feature or components work together.
- Regression test: a test that implements a particular pattern that previously caused a bug to ensure that the bug does not resurface.
- Mocking: to replace a function, module, or type with a fake implementation to avoid testing unrelated functionality. For example, you might “mock the network” or “mock the disk”.

### Exercises
TO-DO

## Lecture 9: Security and Cryptography
### Entropy
- a measure of randomness
- measured in bits, determining the strength of a password
- equal to `log_2(# of possibilities)`
- the attacker knows the model of the password, not the randomness (e.g. Diceware) used to select a particular password
- how many bits of entropy is enough, depends on your threat model

### Hash functions
**Cryptographic hash function**
```
hash(value: array<byte>) -> vector<byte, N>  (for some fixed N)
```
- Maps data of arbitrary size to a fixed size
- SHA-1: maps arbitrary-sized inputs to 160-bit outputs (40 hexadecimal characters)
    - no longer a strong cryptographic hash function
- Properties:
    - deterministic
    - non-invertible
    - target collision resistant
    - collision resistant
- Applications:
    - Git, for content-addressed storage
    - short summary of the contents of a file
    - commitment schemes

**Example**:
```bash
$ printf 'hello' | sha1sum
aaf4c61ddcc5e8a2dabede0f3b482cd9aea9434d
$ printf 'hello' | sha1sum
aaf4c61ddcc5e8a2dabede0f3b482cd9aea9434d
$ printf 'Hello' | sha1sum 
f7ff9e8b7bb2e09b70935a5d785e0cc5d9d0abf0
```

### Key derivation functions (KDFs)
- Deliberately slow to compute, in order to slow down offline brute-force attacks
- Applications:
    - Producing keys (similar properties as hash, e.g. fixed-length) from passphrases for use in other cryptographic algorithms  
        - e.g. generate key for symmetric cryptography
    - Storing login credentials
        - generate and store a random salt `salt = random()` for each user
        - store `KDF(password + salt)`
        - verify login attempts by recomputing the KDF given the entered password and the stored salt

### Symmetric cryptography
```
keygen() -> key  (this function is randomized)

encrypt(plaintext: array<byte>, key) -> array<byte>  (the ciphertext)
decrypt(ciphertext: array<byte>, key) -> array<byte>  (the plaintext)
```
- AES: symmetric cryptosystem
- Properties:
    - given the output (ciphertext), hard to determine the input (plaintext) without the key
    - decrypt function has the obvious correctness property, that `decrypt(encrypt(m, k), k) = m`
- Applications:
    - Encrypting files for storage in an untrusted cloud service

### Asymmetric cryptography
```
keygen() -> (public key, private key)  (this function is randomized)

encrypt(plaintext: array<byte>, public key) -> array<byte>  (the ciphertext)
decrypt(ciphertext: array<byte>, private key) -> array<byte>  (the plaintext)

sign(message: array<byte>, private key) -> array<byte>  (the signature)
verify(message: array<byte>, signature: array<byte>, public key) -> bool  (whether or not the signature is valid)
```
- RSA: asymmetric cryptosystem
- Private key is meant to kept private
- Public key can be publicly shared and it won't affect security (unlike in a symmetric cryptosystem)
- Hard to forge a signature (without private key)
- Properties:
    - a message can be encrypted using the public key
    - given the output (ciphertext), hard to determine the input (plaintext) without the private key
    - decrypt function has the obvious correctness property, that `decrypt(encrypt(m, public key), private key) = m`
    - no matter the message, without the private key, hard to produce signature such that `verify(message, signature, public key)` returns true
    - verify function has the obvious correctness property, that `verify(message, sign(message, private key), public key) = true`
- Applications:
    - PGP email encryption
    - Private messaging
    - Signing software

### Case studies
Detailed explanation at https://missing.csail.mit.edu/2020/security/:
- Password managers
- Two-factor authentication (2FA)
- Full disk encryption
- Private messaging
- SSH

### Exercises
TO-DO

## Lecture 10: Potpourri
### Daemons
- processes that runs in the background rather than waiting for a user to launch them and interact with them
- programs that run as daemon ofthen end with a `d`, e.g. `sshd` (SSH daemon), `systemd` (running and setting up daemon processes)
- `systemctl status` to list current runing daemons
- e.g. managing the network, solving DNS queries or displaying the graphical interface for the system
- `cron`: a daemon your system already runs to perform scheduled tasks
    - if need to run some program with a given frequency
    - no need to build a custom daemon

**Example**:
a daemon for running a simple Python app
```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=My Custom App
After=network.target

[Service]
User=foo
Group=foo
WorkingDirectory=/home/foo/projects/mydaemon
ExecStart=/usr/bin/local/python3.7 app.py
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

### FUSE (Filesystem in User Space)
- UNIX filesystems are traditonally implemented as kernel modules and only the kernel is allowed to perform filesystem calls
- FUSE allows filesystems to be implemented by a user program
- Examples:
    - sshfs: open locally remote files/folder through an SSH connection
    - rclone: Mount cloud storage services like Dropbox, GDrive, Amazon S3 or Google Cloud Storage and open data locally

### Backups
- Any data that you haven’t backed up is data that could be gone at any moment, forever
- 3-2-1 rule:
    - at least 3 copies of your data
    - 2 copies in different mediums
    - 1 of the copies being offsite
- Bad solutions:
    - copy of the data on the same disk
    - an external drive in your home
    - synchronization solutions (e.g. Dropbox)
    - disk mirroring solutions (e.g. RAID)
- Good backups:
    - versioning
    - deduplication
    - security
    - having offline copies of data in the cloud (e.g. email)

### APIs
- Structured URLs:
    - often rooted at `api.service.com`
    - path and query parameters indicate what data you want to read or what action you want to perform
- `curl`: used to transfer data with URLs
- OAuth: a way to give you tokens that can "act as you" on a given service
    - some APIs require authentication: secret token to include with the request
- [IFTTT](https://ifttt.com/): provides integrations with tons of services, and lets you chain events from them in nearly arbitrary ways

### Common command-line flags/patterns
- `--help`: dispaly brief usage instructions
- `--version` or `-V`: print version
- `--verbose` or `-v`: produce more verbose output, e.g. `-vvv` to get more verbose outout
- `--quiet`: only print something on error
- "dry run": only print what the tools would have done, but do not actually perform the change
- "interactive" flag: prompt you for each destructive action
- `-r`: make destructive tools recursive
- `-` in place of a file name: "standard input" (keyboard by default) or "standard output" (terminal screen by default)
- `--`: makes a program stop processing flags and options (things starting with `-`):
    - e.g. remove a file called `-r`: `rm -- -r`

### Window managers
- "floating" window manager
- "tiling" window manager

### VPNs
- just a way for you to change your internet service provider as far as the internet is concerned
- all your traffic will look like it’s coming from the VPN provider instead of your “real” location
- and the network you are connected to will only see encrypted traffic
- when you use a VPN, all you are really doing is shifting your trust from you current ISP to the VPN hosting company, whatever your ISP could see, the VPN provider now sees instead
- much of your traffic, at least of a sensitive nature, is already encrypted through HTTPS or TLS more generally
- some VPN providers are malicious (or at the very least opportunist), and will log all your traffic, and possibly sell information about it to third parties
- WireGuard to roll your own VPN

### Booting + Live USBs
**Booting**:
- When machine boots up, before the OS is loaded, the BIOS/UEFI initializes the system
- “Press F9 to configure BIOS. Press F12 to enter boot menu.” during the boot process
- BIOS menu: configure all sorts of hardware-related settings
- Boot menu: to boot from an alternate device instead of your hard drive

**Live USBs**:
- USB flash drives containing an OS

### Docker, Vagrant, VMs, Cloud, OpenStack
https://missing.csail.mit.edu/2020/potpourri/#docker-vagrant-vms-cloud-openstack

## References
- https://missing.csail.mit.edu/
- https://missing.csail.mit.edu/2019/

## 2019 Lectures to Watch (TO-DO):
- https://missing.csail.mit.edu/2019/automation/
- https://missing.csail.mit.edu/2019/web/
- https://missing.csail.mit.edu/2019/virtual-machines/