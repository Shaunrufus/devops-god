# Phase 1 — Topic 3: Process Management

## What Is a Process?

A **process** is a running instance of a program. Every process has:
- **PID** (Process ID) — unique number
- **PPID** (Parent Process ID) — who spawned it
- **State** — running, sleeping, zombie, stopped
- **Resources** — CPU %, memory, priority

## Key Concepts

| Term | Meaning |
|---|---|
| **PID** | Process ID — unique identifier |
| **PPID** | Parent Process ID |
| **Daemon** | Background service (e.g., nginx, docker) |
| **Zombie** | Dead process not cleaned up by parent |
| **Orphan** | Process whose parent died |
| **Signal** | Message sent to process (kill, stop, resume) |

## Process States

| State | Symbol | Meaning |
|---|---|---|
| Running | R | Actively executing |
| Sleeping | S | Waiting for event |
| Stopped | T | Paused (Ctrl+Z) |
| Zombie | Z | Finished but not reaped |
| Idle | I | Kernel thread |

## Viewing Processes

### ps — Snapshot of processes

```bash
ps aux          # All processes, detailed
ps -ef          # Full format
ps -ef --forest # Tree view
ps -u shaun     # Processes by user
ps -p 1234      # Specific PID
```

**Key columns:**
- `USER` — who owns it
- `PID` — process ID
- `%CPU` — CPU usage
- `%MEM` — memory usage
- `COMMAND` — what's running

### top — Live process monitor

```bash
top
# Press:
# q - quit
# k - kill process
# M - sort by memory
# P - sort by CPU
# 1 - show individual CPUs
```

### htop — Better top (install first)

```bash
sudo apt install htop -y
htop
# F9 - kill
# F6 - sort by column
# F5 - tree view
```

## Killing Processes

### Signals

| Signal | Number | Action |
|---|---|---|
| SIGTERM | 15 | Graceful shutdown (default) |
| SIGKILL | 9 | Force kill (no cleanup) |
| SIGHUP | 1 | Reload config |
| SIGSTOP | 19 | Pause process |
| SIGCONT | 18 | Resume process |

### Commands

```bash
# Kill by PID
kill 1234           # SIGTERM (graceful)
kill -9 1234        # SIGKILL (force)

# Kill by name
pkill -f nginx      # Kill all nginx processes
killall nginx       # Same as pkill

# Interactive kill
top                 # Then press 'k' and enter PID
```

## Background & Foreground Jobs

```bash
# Run in background
sleep 100 &

# List jobs
jobs

# Bring to foreground
fg %1

# Send to background (Ctrl+Z first to stop)
bg %1

# Run immune to hangup
nohup python app.py &

# Detach from terminal
screen          # or tmux
```

## Managing Services (systemd)

```bash
# Check status
systemctl status nginx

# Start/stop/restart
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx

# Enable/disable on boot
sudo systemctl enable nginx
sudo systemctl disable nginx

# View logs
journalctl -u nginx -f          # Live tail
journalctl -u nginx -n 100      # Last 100 lines
journalctl -u nginx --since today
```

## Process Priority

```bash
# View priority
ps -eo pid,ni,comm

# Run with lower priority (nice value 10)
nice -n 10 ./script.sh

# Change priority of running process
renice -n 5 -p 1234

# Nice values: -20 (highest) to 19 (lowest)
# Default: 0
```

## Real DevOps Scenarios

### Scenario 1: Server Unresponsive — High CPU

```bash
# 1. Find the culprit
top -o %CPU

# 2. Investigate the process
ps -p <PID> -f
lsof -p <PID>       # What files is it using?
strace -p <PID>     # What syscalls is it making?

# 3. Kill if malicious, restart if legit
kill -9 <PID>
```

### Scenario 2: Memory Leak — App Consuming RAM

```bash
# 1. Sort by memory
top -o %MEM

# 2. Check memory details
ps aux | grep <process>
cat /proc/<PID>/status | grep VmRSS

# 3. Restart the service
sudo systemctl restart app
```

### Scenario 3: Stuck Deployment — Process Won't Die

```bash
# 1. Try graceful kill
kill <PID>

# 2. Wait 5 seconds, check if alive
ps -p <PID>

# 3. Force kill if still running
kill -9 <PID>

# 4. Check for zombies
ps aux | grep defunct
```

### Scenario 4: Service Crashed — Auto-Restart

```bash
# Check why it crashed
journalctl -u myapp -n 50 --no-pager

# Restart it
sudo systemctl restart myapp

# Check status
systemctl status myapp
```

## Common Mistakes

❌ **Using kill -9 first** → Always try `kill` (SIGTERM) before `kill -9`  
❌ **Killing PID 1** → That's init/systemd — system crash  
❌ **Not checking children** → Kill parent, orphan children still run  
❌ **Forgetting & for background** → Terminal hangs  
✅ **Check logs before killing** → Understand why it's stuck  
✅ **Use systemctl for services** → Better than raw kill  
✅ **Monitor after restart** → Ensure it stays up

## My Lab Observations

1. **What is PID 1?** init or systemd
2. **First time `top`:** sorted by CPU
3. **Pressing `M` in `top`:** sort by memory
4. **Difference between `kill 123` and `kill -9 123`?** First is graceful (SIGTERM), second is force-kill (SIGKILL)
5. **The script I killed with `Ctrl+C`:** it was a python script printing hello world 50 times
6. **My `script.sh` didn't run at first because:** it didn't have execute permissions
7. **The output of `ps -ef --forest` showed:** A tree view of processes

## My Lab Observations

1. **My current shell's PID:** 696
2. **What command shows live CPU/memory usage?** htop
3. **Difference between `kill` and `kill -9`:** kill - graceful termination allowing process to release resources and then trminate it , kill -9 kills directly
4. **What happens when you run `sleep 300 &`?** jobs runs in the background , sleep 300 %3 job runs in the foreground 
5. **Real scenario:** If nginx crashed at 3am, what's your first command? 

sudo systemctl status nginx (to check status and see the latest logs/error snippet) or sudo journalctl -u nginx -n 50 --no-pager (to view recent error messages).


