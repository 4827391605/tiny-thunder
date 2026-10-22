# Debian Linux Command Notes

Personal notes for people learning their way around Debian. This is an independent community cheat sheet, not official Debian documentation and not endorsed by the Debian project.

Commands below are examples to adapt. Replace example names and paths with ones that exist on your system. Read a command before running it, especially when it uses `sudo`, changes permissions, or removes files. These are shell snippets for reference, not a script to run all at once.

## Find your way around

```bash
pwd                         # print the current directory
ls                          # list visible files
ls -lah                     # include hidden files and show readable sizes
cd Documents                # enter a directory inside the current one
cd ..                       # go up one directory
cd -                        # return to the previous directory
```

## Create, copy, move, and remove files

```bash
mkdir music-notes            # create a directory
mkdir -p projects/audio/demo # create parent directories as needed
touch session-notes.txt      # create an empty file if it does not exist
cp mix-notes.txt mix-backup.txt
cp -r sample-project/ sample-project-copy/
mv rough-draft.txt final-draft.txt
mv final-draft.txt Documents/
rm old-export.txt            # remove one file
rm -i old-export.txt         # ask before removing the file
```

`rm` removes files directly rather than moving them to Trash. Check the path before using it. Use `rm -r folder-name/` only when you intend to remove that directory and everything inside it; avoid adding `-f` unless you fully understand the effect.

## Search for files and text

```bash
find . -type f -name '*.log'         # find matching files below this directory
find . -type d -name 'cache*'        # find matching directories
find . -type f -size +20M            # find files larger than 20 MB
find . -type f -mtime -2             # files changed in the last two days
grep -n 'warning' system.log         # find a word and show line numbers
grep -Rni 'sample rate' notes/       # search text in a directory tree
```

Quote wildcard patterns like `'*.log'` so `find` receives the pattern itself.

## Read and edit text from the terminal

```bash
cat session-notes.txt                # print a short file
less system.log                      # read a file a screen at a time; press q to quit
head -n 15 session-notes.txt         # show the first 15 lines
tail -n 30 system.log                # show the last 30 lines
tail -f application.log              # follow new lines; press Ctrl+C to stop
```

Some handy `sed` examples:

```bash
sed -n '7p' session-notes.txt                 # print line 7
sed -n '4,11p' session-notes.txt              # print lines 4 through 11
sed '/^[[:space:]]*#/d' settings.conf         # omit comment lines from the output
sed '/^[[:space:]]*$/d' session-notes.txt     # omit blank lines from the output
sed 's/grey/gray/g' color-notes.txt           # replace every match on each line in the output
sed -i.bak 's/grey/gray/g' color-notes.txt    # edit in place and keep a .bak copy
```

Without `-i`, `sed` prints changed text but leaves the file alone. With `-i`, it edits the file; the `.bak` suffix above keeps a backup. Check the result before using an in-place edit on important files.

## Permissions and ownership

```bash
ls -l session-notes.txt             # inspect permissions and ownership
chmod u+x helper-script.sh          # let the owner execute a script
chmod 640 private-notes.txt         # owner: read/write; group: read; others: none
chmod 755 shared-tool.sh            # owner: full access; group/others: read and execute
```

Use `chown` to change ownership, usually with administrator privileges:

```bash
sudo chown mira:mira session-notes.txt
```

Avoid broad permissions such as `777`; they let every local user write to the item. Be careful with recursive permission or ownership changes (`-R`), because they affect everything underneath the chosen directory.

## Install and update software with APT

```bash
sudo apt update                       # refresh package lists
apt search audio-editor               # search package names and descriptions
apt show audacity                     # view information about a package
sudo apt install audacity             # install a package
sudo apt remove audacity              # remove a package, keeping some configuration
sudo apt purge audacity               # remove package and its system configuration
sudo apt upgrade                      # upgrade installed packages
sudo apt autoremove                   # remove dependencies no longer needed
```

`apt update` refreshes the package index; it does not upgrade installed software. Review APT's proposed changes before confirming them. Do not paste commands that add repositories or signing keys unless you trust and understand the source.

## Check system status

```bash
cat /etc/debian_version               # Debian release version
uname -r                              # running kernel version
hostnamectl                           # host and operating-system details
lscpu                                 # processor details
free -h                               # memory and swap use
df -h                                 # space available on mounted filesystems
du -sh ~/Downloads                    # total size of one directory
lsblk -f                              # disks, partitions, and filesystems
```

## Processes and services

```bash
top                                   # live process and resource view; press q to quit
ps -u "$USER" -o pid,stat,comm       # processes owned by your account
pgrep -a pulsar                       # find a process by name
kill 2468                             # ask process 2468 to exit normally
```

Use the actual process ID shown on your computer instead of `2468`. Try a normal `kill` first; force-stopping a process with `kill -9` prevents it from cleaning up, so reserve that for a process that will not exit otherwise.

For system services managed by systemd:

```bash
systemctl status NetworkManager       # inspect a service
sudo systemctl restart NetworkManager # restart a service
systemctl --user status               # show services for your logged-in user
journalctl -b -p warning              # warnings and errors from this boot
journalctl -u NetworkManager -b       # log entries for a service this boot
```

Restarting a network service can interrupt your connection. Use the actual service name installed on your machine; not every Debian installation has the same services.

## Network checks

```bash
ip address                            # show network interfaces and addresses
ip route                              # show routing table
ss -tulpn                             # show listening TCP/UDP sockets
ping -c 4 debian.org                  # check basic reachability
getent hosts debian.org               # check name lookup through system settings
```

Some process details in `ss -p` may require `sudo`. A failed ping alone does not prove the network is down; a network may block ping traffic.

## Archives and transfers

```bash
tar -czf music-notes.tar.gz music-notes/    # create a compressed archive
tar -tzf music-notes.tar.gz                 # list archive contents
tar -xzf music-notes.tar.gz                 # extract into the current directory
```

Inspect an archive's contents before extracting it, especially if it came from someone you do not know.

For a remote computer you control, `scp` copies a file over SSH:

```bash
scp session-notes.txt alex@server.example:/home/alex/
```

Replace the example account, host, and destination with details supplied by the server owner. Do not publish private hostnames, usernames, keys, or access tokens in a public notes file.

## A few shell conveniences

```bash
history                               # show recent commands
clear                                 # clear the terminal display
command --help                        # show a command's built-in help, when available
man find                              # open the manual page for find; press q to quit
```

An alias can give a frequently used command a short name. Add an alias to `~/.bashrc` to keep it for future Bash terminals:

```bash
alias ll='ls -lah'
```

After editing `~/.bashrc`, load the change into the current Bash session with `source ~/.bashrc`.

## Before running a command

- Read the full command and understand each option.
- Treat commands with `sudo`, `rm`, `chmod`, `chown`, disk tools, and `-i` file edits with extra care.
- Check paths and review proposed changes before confirming them.
- Use the manual page (`man command`) or `command --help` when an option is unfamiliar.

These notes are a learning aid. Check Debian's documentation and the relevant package's documentation for details about your specific release and setup.
