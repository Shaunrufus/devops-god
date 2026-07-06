# DevOps + Linux Cheat Sheet & Incident Response Playbook

## PART 1: LINUX FILESYSTEM & NAVIGATION

| Command | Purpose |
|---|---|
| `pwd` | Print current directory |
| `ls -la` | List all files incl. hidden, long format |
| `cd /path` | Change directory (absolute) |
| `cd ..` | Go up one level |
| `cd ~` | Go to home directory |
| `tree` | Show directory structure as tree |
| `find / -name "file"` | Find file by name from root |
| `locate file` | Fast file search (needs `updatedb`) |
| `du -sh *` | Disk usage of items in current dir |
| `df -h` | Disk free space, human-readable |

**Key Directories:** `/etc` configs, `/var/log` logs, `/home` user dirs, `/tmp` temp files, `/proc` live kernel info, `/bin` `/usr/bin` executables.

## PART 2: FILE PERMISSIONS & OWNERSHIP

| Command | Purpose |
|---|---|
| `ls -l` | View permissions |
| `chmod 755 file` | Set rwxr-xr-x |
| `chmod u+x file` | Add execute for owner |
| `chown user:group file` | Change ownership |
| `umask` | Default permission mask |
| `sudo -l` | List sudo privileges |

Permission digits: 4=read, 2=write, 1=execute (sum per role: owner/group/others)

## PART 3: PROCESS MANAGEMENT

| Command | Purpose |
|---|---|
| `ps aux` | List all running processes |
| `ps -ef --forest` | Process tree view |
| `top` / `htop` | Live process monitor |
| `kill -9 PID` | Force kill process |
| `pkill -f name` | Kill by process name |
| `nice -n 10 cmd` | Run with lower priority |
| `renice -n 5 -p PID` | Change priority of running process |
| `jobs` / `bg` / `fg` | Manage background/foreground jobs |
| `nohup cmd &` | Run immune to hangup signal |
| `systemctl status svc` | Check service status |
| `systemctl restart svc` | Restart a service |
| `journalctl -u svc -f` | Live logs for a systemd service |

## PART 4: NETWORKING

| Command | Purpose |
|---|---|
| `ip a` | Show IP addresses |
| `ping host` | Test reachability |
| `curl -I url` | Get HTTP headers only |
| `ss -tulnp` | List listening ports |
| `traceroute host` | Trace network path |
| `dig domain` / `nslookup domain` | DNS lookup |
| `nc -zv host port` | Test port open/closed |
| `scp file user@host:/path` | Secure copy over SSH |
| `rsync -avz src dest` | Sync files efficiently |

## PART 5: DISK, MEMORY, CPU

| Command | Purpose |
|---|---|
| `free -h` | Memory usage |
| `vmstat 1` | Virtual memory stats |
| `iostat -x 1` | Disk I/O stats |
| `lsblk` | List block devices |
| `dmesg | tail` | Kernel ring buffer |
| `uptime` | Load average + uptime |
| `nproc` | Number of CPU cores |

## PART 6: LOGS & DEBUGGING

| Command | Purpose |
|---|---|
| `tail -f /var/log/syslog` | Live tail a log |
| `grep -i error file.log` | Search case-insensitive |
| `grep -A 5 -B 5 "OOM" file.log` | Context lines around match |
| `awk '{print $1}' file` | Print column 1 |
| `sed 's/old/new/g' file` | Find & replace |
| `journalctl -xe` | Recent system errors |
| `strace -p PID` | Trace syscalls of a running process |
| `lsof -i :PORT` | What process is using a port |

## PART 7: GIT SHORTCUTS

| Command | Purpose |
|---|---|
| `git status` | Check changes |
| `git add . && git commit -m "msg"` | Stage + commit |
| `git log --oneline --graph --all` | Visual history |
| `git stash` / `git stash pop` | Save/restore uncommitted work |
| `git reset --hard HEAD~1` | Undo last commit (destructive) |
| `git rebase -i HEAD~3` | Interactive rebase |
| `git cherry-pick <hash>` | Apply specific commit |

## PART 8: DOCKER SHORTCUTS

| Command | Purpose |
|---|---|
| `docker ps -a` | All containers |
| `docker logs -f <id>` | Live container logs |
| `docker exec -it <id> bash` | Shell into running container |
| `docker system prune -a` | Clean unused images/containers |
| `docker inspect <id>` | Full container metadata |
| `docker stats` | Live resource usage |
| `docker compose up -d` | Start stack in background |

## PART 9: AWS CLI SHORTCUTS

| Command | Purpose |
|---|---|
| `aws sts get-caller-identity` | Confirm active AWS identity |
| `aws ec2 describe-instances` | List EC2 instances |
| `aws logs tail /log/group --follow` | Live tail CloudWatch logs |
| `aws s3 ls s3://bucket` | List bucket contents |

## PART 10: SHELL SHORTCUTS

`Ctrl+R` search history, `Ctrl+C` kill process, `Ctrl+Z` suspend, `Ctrl+L` clear screen, `!!` repeat last command, `alias ll='ls -la'` custom shortcut.

---

## PART 11: REAL-WORLD INCIDENT RESPONSE PLAYBOOKS

### Scenario A — Prod Server Down
1. `curl -I https://domain.com` + `ping <ip>` — confirm outage externally
2. `ssh user@server` — if fails, check cloud console instance status
3. `uptime`, `free -h`, `df -h` — resource check
4. `systemctl status nginx`, `journalctl -u nginx -n 100` — service check
5. `ps aux | grep nginx`, `ss -tulnp | grep :443` — process/port check
6. `dmesg | tail -50`, `journalctl -xe` — crash window logs
7. Classify: OOM kill / disk full / crash / firewall block
8. Fix: restart service, free disk, fix firewall/security group
9. Document in `incident-notes/`

### Scenario B — Database Deadlock
1. `SHOW ENGINE INNODB STATUS;` (MySQL) or `SELECT * FROM pg_locks WHERE NOT granted;` (Postgres)
2. Identify blocking vs blocked sessions via `pg_stat_activity`
3. Check long-running queries by duration
4. Classify: true deadlock / lock contention / missing index
5. Fix: `pg_terminate_backend(pid)` or `KILL <thread_id>`, fix lock ordering, add index
6. Prevent: query timeouts, connection pool limits, retry logic

### Scenario C — Kubernetes CrashLoopBackOff
1. `kubectl get pods -n <ns>`, `kubectl describe pod <name>`
2. `kubectl logs <pod> --previous`
3. Check `Last State` for OOMKilled
4. `kubectl get events --sort-by='.lastTimestamp'`
5. Fix: correct config/secret, adjust resource limits, `kubectl rollout undo`

### Scenario D — High CPU / Unresponsive Server
1. `top -o %CPU` — find PID
2. `ps -p <PID> -o cmd,%cpu,%mem`, `strace -p <PID> -c`
3. `ss -tan | grep :80 | wc -l` — check traffic spike vs runaway process
4. Fix: kill/patch runaway process, or scale horizontally if legit traffic

### Scenario E — Disk Full on Production
1. `df -h`, `du -sh /var/* | sort -rh | head -10`
2. `find / -type f -size +500M -exec ls -lh {} \;`
3. Fix: `logrotate -f`, `docker system prune -a`, delete old backups
4. Prevent: automated log rotation + disk alerts

### Universal Incident Framework
```
1. DETECT   → Alert fires
2. TRIAGE   → Confirm severity
3. ISOLATE  → Which layer (Network/OS/App/DB)
4. DIAGNOSE → Logs, metrics, root cause
5. MITIGATE → Stop the bleeding fast (restart/rollback/scale)
6. RESOLVE  → Permanent fix
7. VERIFY   → Confirm healthy metrics
8. DOCUMENT → Postmortem
```
**Golden Rule:** Mitigate first, diagnose in parallel, root-cause fix last — never let debugging delay recovery.
