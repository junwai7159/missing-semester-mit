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
- `head`, `tail` 

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

### Exercises
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
- help, man, tldr

#### Finding files
**find file or directory**:
```
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
```
# Delete all files with .tmp extension
find . -name '*.tmp' -exec rm {} \;
# Find all PNG files and convert them to JPG
find . -name '*.png' -exec convert {} {}.jpg \;
```

**`fd`**:
- alternative to `find`
- offers some nice defaults like colorized output, default regex matching, and Unicode support
```
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
```
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
- `vimtutor`
- install anid configure a plugin
- advanced vivimm

## Lecture 4: Data Wrangling
### Introduction
**`less`**:
```
ssh myserver 'journalctl | grep sshd | grep "Disconnected from"' > ssh.log
less ssh.log
```
- single quote to do the filtering on the remote server
- `less`: gives a "pager"

**stream editor**:
```
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
```
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
```
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
```
 | awk '$1 == 1 && $2 ~ /^c[^ ]*e$/ { print $2 }' | wc -l
```
or 
```
BEGIN { rows = 0 }
$1 == 1 && $2 ~ /^c[^ ]*e$/ { rows += $1 }
END { print rows }
```

### Analyzing data
**Calculator**:
- `bc`
```
echo "1+2" | bc -l
# 3
```

**Stats**:
- `st`
- `R`
```
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
 | awk '{print $1}' | R --no-echo -e 'x <- scan(file="stdin", quiet=TRUE); summary(x)'
```
- `gnuplot` for simple plotting
```
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
```
rustup toolchain list | grep nightly | grep -vE "nightly-x86" | sed 's/-x86.*//' | xargs rustup toolchain uninstall
```

### Wrangling binary data
use ffmpeg to capture an image from our camera, convert it to grayscale, compress it, send it to a remote machine over SSH, decompress it there, make a copy, and then display it
```
ffmpeg -loglevel panic -i /dev/video0 -frames 1 -f image2 -
 | convert - -colorspace gray -
 | gzip
 | ssh mymachine 'gzip -d | tee copy.jpg | env DISPLAY=:0 feh -'
```

### Exercises
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
```
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
```
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
```
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
```
ssh foobar@server ls | grep PATTERN
```

- `grep` remotely the local output of `ls`
```
ls | ssh foobar@server grep PATTERN
```

#### SSH keys
- Public-key cryptography to prove to the server that, the client owns the secret private key without revealing the key
- Do not need to reenter password every time
- Private key (`~/.ssh/id_rsa` or `~/.ssh/id_ed25519`) is effectively your password

**Key generation**:
- To generate public-private key pair
```
ssh-keygen -o -a 100 -t ed25519 -f ~/.ssh/id_ed25519
```

- To check if passphrase is available and validate
```
ssh-keygen -y -f /path/to/key
```

**Key-based authentication**:
- `ssh` will look into public key (`.ssh/authorized_keys`) to determine which clients to let in

Copy a public key over
```
cat .ssh/id_ed25519.pub | ssh foobar@remote 'cat >> ~/.ssh/authorized_keys'
```
or
```
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
```
alias my_server="ssh -i ~/.id_ed25519 --port 2222 -L 9999:localhost:8888 foobar@remote_server
```

**`~/.ssh/config`**:
- dotfile, able to be read by other programs like `scp`, `rsync`, `mosh`
- server side config: `/etc/ssh/sshd_config`
```
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
- visuliaze `baz.txt`: `git cat-file -p 4448adbf7ecd394f42ae135bbeed9676e894af85`
```
git is wonderful
```

#### References
- human-readable names for SHA-1 hashes
- pointers to commits
- mutable (update to point new commit)
- e.g. `master` reference points to the latest commit in the main branch
- `HEAD`: reference to current check-out commit
    - detached `HEAD`: when `HEAD` points to a commit rather than a branch 

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
- `git status`: tells you what’s going on
- `git add <filename>`: adds files to staging area
- `git commit`: creates a new commit
- `git log`: shows a flattened log of history
- `git log --all --graph --decorate --oneline`: visualizes history as a DAG
- `git diff <filename>`: show what has changed but hasn't been added to the index yet via `git add`
- `git diff --cached`: show what has been added to the index  via `git add` but not yet committed
- `git diff <revision> <filename>`: shows differences in a file between snapshots
- `git checkout <revision>`: updates HEAD and current branch

#### Branching and merging
- `git branch`: shows branches
    - `git branch -vv` for verbose display
- `git branch <name>`: creates a branch
- `git checkout -b <name>`: creates a branch and switches to it
    - `same as git branch <name>; git checkout <name>`
- `git merge <revision>`: merges into current branch
    - need to stage the file to mark the conflict as resolved
    - `git merge --continue` instead of `git commit` to complete the merge
    - `git merge --abort` to abort the merge
- `git mergetool`: use a fancy tool to help resolve merge conflicts
- `git rebase`: rebase set of patches onto a new base

#### Remotes
- `git remote`: list remotes
- `git remote add <name> <url>`: add a remote
- `git push <remote> <local branch>:<remote branch>`: send objects to remote, and update remote reference
- `git branch --set-upstream-to=<remote>/<remote branch>`: set up correspondence between local and remote branch
- `git fetch`: retrieve objects/references from a remote
- `git pull`: same as git fetch; git merge
- `git clone`: download repository from remote

#### Undo
- `git commit --amend`: edit a commit’s contents/message
- `git reset HEAD <file>`: unstage a file
- `git checkout -- <file>`: discard changes

### Advanced Git
- `git config`: Git is highly customizable, `~/.gitconfig`
- `git clone --depth=1`: shallow clone, without entire version history
- `git add -p`: interactive staging
- `git rebase` -i: interactive rebasing
- `git blame`: show who last edited which line
- `git stash`: temporarily remove modifications to working directory
    - `git stash pop` to undo the stash
- `git bisect`: binary search history (e.g. for regressions)
- `.gitignore`: specify intentionally untracked files to ignore
- `git show <commit>`: show commit

### Workflows
- Feature branch workflow
- Forking workflow
- Gitflow workflow 

![git-flow](./media/git-flow.png)

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

## Lecture 7: Debugging and Profiling

## Lecture 8: Metaprogramming

## Lecture 9: Security and Cryptography

## Lecture 10: Potpourri

## References
- https://missing.csail.mit.edu/
