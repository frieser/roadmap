### 01-introduction → 02-popular-shells
- `01-what-is-a-shell` user→shell→kernel, `$SHELL`; `02-what-is-bash` GNU sh superset, `[[ ]]`, brace `{1..5}`
- `03-cli-vs-gui` pipes, ssh, `/dev/null`; `04-what-is-scripting` shebang, `set -euo pipefail`, `source`
- Shells: `01-bash` `Ctrl+R`/`!!`/`!$`; `02-zsh` 1-based arrays, Oh My Zsh, macOS default
- `03-fish` NOT POSIX, `set -U`, `abbr`; `04-dash` strict POSIX, 4x faster, NO `[[ ]]`
- `05-ksh` `print`/`typeset`, AIX; `06-tcsh` C-like, NOT for scripting; `07-cmd` Windows `%VAR%`; `08-powershell` .NET objects

### 03-setting-up-bash → 04-shell-fundamentals
- `~/.bashrc` (interactive) vs `~/.bash_profile` (login), PS1 `\u@\h:\w`, `alias`, `shopt`
- `01-stdin-stdout-stderr` fd 0/1/2; `02-03-04-redirection` `>`/`>>`/`<`, `2>&1` order matters
- `05-pipes` `|`, `xargs`, `-print0`; `06-tab-completion` `complete -W`; `07-repeat-commands` `history`/`Ctrl+R`
- `08-help-commands` `man`/`help`/`type`; `09-bash-alias` vs `function(){}`; `10-command-substitution` `$(cmd)`
- `11-process-substitution` `<(cmd)`, Bash-only; `12-stop-execution` `Ctrl+C/Z`, `fg`/`bg`/`kill`

### 05-files-and-directories
- `01-pwd` -L/-P, `$OLDPWD`; `02-mkdir -p` brace-expand; `03-touch -d`/`-r`; `04-cd` pushd/popd
- `05-ls -la` `drwxr-xr-x`; `06-echo -n -e`/`printf`; `07-rm -rf` dangerous; `08-rmdir` empty only
- `09-cat` UUOC → `grep pat file`; `10-find -name`/`-type f`/`-size +100M`/`-mtime -7`/`-exec {} +`

### 06-working-with-text
- View: `grep -inr -E -o -c -A/B/C --exclude-dir`, `less`, `head -n`/`tail -f`/`-F`, `find`, Go `filepath.WalkDir`
- Transform: `cut -d: -f`, `paste`, `join` (needs sort), `split -l N`, `sort -n -r -k -t -h`
- `tr 'a-z' 'A-Z'`/`-d`/`-s`, `uniq -c -u` (pipe from sort), `sed 's/a/b/g'`/`d`/`-i`
- `awk -F: '{print $NF}'`/`BEGIN`/`END`/`NR`/`count[$1]++`, `nl`, `wc -lwc -L`

### 07-text-editors
- `01-basic-editor-ops` modes/buffers; `02-nano` no modes, `Ctrl+O/X`; `03-vim` modal, `hjkl`, `dd`/`yy`/`p`, macros
- `04-vi` universal, recovery mode; `05-emacs` `C-x C-f`, `C-x C-s`, org-mode, Elisp

### 08-bash-scripting
- `01-bash-script-anatomy` shebang, `set -euo pipefail`, `trap cleanup EXIT`, `main "$@"`
- `02-running-shell-scripts` `./` needs `+x`, `bash` no shebang, `source` no subshell
- `03-bash-data-types` `local`, `declare -i/-r/-A`, `${var:-default}`; `$#`/`"$@"`/`$?`/`$$`/`$!`
- Arrays `"${arr[@]}"`, assoc `declare -A`, literals `'...'` vs `"..."` vs `$'...'`
- `04-input-output` `read -r -p -s`, `printf`, heredoc `<<'EOF'`, here-string `<<<`
- `05-bash-operators` `((++i))`, `[[ -eq -lt -gt ]]`, `=~` regex, `-f -d -z -n`, `&&`/`||`
- `06-script-arguments` `$1..$9`/`${10}`, `shift`, `while [ $# -gt 0 ]; do case`
- `07-file-permissions` rwx=7, `chmod 755`/`u+x`, `chown user:group`, `chgrp`, `umask`
- `08-exit-codes` `$?` 0=success, `PIPESTATUS`, `128+SIG`; `09-working-with-numerics` `$((...))` integer, `bc -l` floats
- `10-string-manipulation` `${#s}`/`${s:0:5}`/`${s//old/new}`/`${s,,}`; `11-conditionals` `if [[ ]]`/`case`
- `12-loops` `for`/`while`/`until`/`break N`; `13-functions` `local`, `return`, `$(func)` capture

### 09-advanced-scripting
- `01-debugging` `set -x`/`bash -x`, PS4, `bash -n`, `shellcheck`; `02-regular-expressions` BRE `\+` vs ERE `+`, `=~` in `[[ ]]`
- `03-error-handling` `set -euo pipefail`, `|| true`, `trap cleanup EXIT/ERR`, logging `2>&1`

### 10-system-admin
- `01-process-management` `jobs`/`fg`/`bg`, `ps aux`, `top`/`htop`, `uptime` load avg
- `02-networking` `ping`, `curl`/`wget`, `ssh`/`scp`/`rsync -avz`, `ss -tulpn`, `iptables`
- `03-package-management` `apt` (Debian), `dnf` (RHEL 8+), `brew` (macOS)
- `04-task-scheduling` `crontab -e`, `at`, `systemd .timer`; `05-file-compression` `gzip`, `tar -cvzf`, `bzip2`

| Bash → Go mapping |
|---|
| `${#s}`→`len(s)`, `"${arr[@]}"`→`[]string`, `declare -A`→`map[string]string` |
| `$(cmd)`→`exec.Command()`, `while read`→`bufio.Scanner`, `trap EXIT`→`defer` |
| `grep`→`strings.Contains`/`regexp`, `sed`→`regexp.ReplaceAll`, `awk`→`strings.Split` |
| `find`→`filepath.WalkDir`, `$((a+b))`→`a+b`, `printf`→`fmt.Printf` |
