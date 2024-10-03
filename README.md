# ./missing-semester

## Topic 1: The Shell
### What is the shell?
- textual interface
- shell vs terminal
- bash (Bourne Again Shell)

### Using the shell
Launch terminal to open shell prompt:
```
missing:~$
```
- `missing`: machine
- `~`: short for home
- `$`: you are not the root user

Echo:
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

`ls` command:
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

`ls --help`:
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

Handy programs:
- `mv` (rename/move file), `cp` (copy file), `rm -r` (remove recursively), `rmdir` (remove directory), `mkdir` (make directory)
- `ctrl+l` to clear
- `man ls`: show manual page of `ls`

## References
- https://missing.csail.mit.edu/
