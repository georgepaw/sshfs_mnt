Scripts for helping mounting/managing remote drives.

Config for mount points is stored in `~/.mnt_script/mnt_points`.
Example config (similiar to `.ssh/config`:

```
s1:
        hostname server1
        user george
        remotepath /home/george/
        localmount ~/s1
s2:
        hostname server2
        user georgep
        remotepath /mnt/storage/home/georgep
        localmount ~/s2
```
This assumes you have ssh keys set up with your identify file (private key) being `~/.ssh/id_rsa`.
Note that this script does not have any error checking.
I'd recommend adding `mnt` and `umnt` to your path, then example usage is:
```bash
mnt s1
```
Note that these scripts use `sudo` for calling `sshfs` and `umount` (I found them more reliable this way).
To do so run `sudo visudo` and add:
```bash
## Read drop-in files from /private/etc/sudoers.d
## (the '#' here does not indicate a comment)
#includedir /private/etc/sudoers.d
```
to the bottom of the file.
And then create dropins for `sshfs` and `umount`:
```bash
sudo touch /private/etc/sudoers.d/sshfs
sudo touch /private/etc/sudoers.d/umount
```

For auto complete in zsh add this to you `~/.zshrc`
```bash
_mnt_complete() {
  local word completions
  word="$1"
  g=()
  if [[ -r  ~/.mnt_script/mnt_points ]]; then
    while IFS= read -r line || [[ -n "$line" ]]; do
      if [[ "$line" =~ ^(.*):$ ]]; then
        g+=("${MATCH: : -1}")
      fi
    done < "$HOME/.mnt_script/mnt_points"
  fi
  completions="$(printf '%s\n' "${g[@]}" )"
  reply=( "${(ps:\n:)completions}" )
}

compctl -K _mnt_complete mnt
compctl -K _mnt_complete umnt
```