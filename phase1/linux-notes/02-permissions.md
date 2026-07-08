# Phase 1 — Topic 2: File Permissions & Ownership

## What Are Linux Permissions?

Every file/folder in Linux has 3 permission sets:
- **Owner (u)** — the user who created it
- **Group (g)** — users in the same group
- **Others (o)** — everyone else

Each set has 3 permission types:
- **r (read)** — 4
- **w (write)** — 2  
- **x (execute)** — 1

## Reading Permission Notation

Example: `-rwxr-xr--`

| Symbol | Type | Owner | Group | Others |
|---|---|---|---|---|
| `-` | file type | rwx (7) | r-x (5) | r-- (4) |

**Breakdown:**
- First `-` = regular file (`d` = directory, `l` = symlink)
- Owner: `rwx` = read + write + execute = 7
- Group: `r-x` = read + execute = 5
- Others: `r--` = read only = 4

So `-rwxr-xr--` = **754** in numeric notation.

## Common Permission Patterns

| Numeric | Symbolic | Meaning | Use Case |
|---|---|---|---|
| 755 | rwxr-xr-x | Owner full, others read/exec | Scripts, binaries |
| 644 | rw-r--r-- | Owner write, others read | Config files, logs |
| 600 | rw------- | Owner only | SSH keys, passwords |
| 777 | rwxrwxrwx | Everyone full access | **NEVER use in prod** |
| 700 | rwx------ | Owner only, full access | Private scripts |

## Commands Cheat Sheet

| Command | Purpose | Example |
|---|---|---|
| `ls -l` | View permissions | `ls -l file.sh` |
| `chmod 755 file` | Set permissions (numeric) | `chmod 755 deploy.sh` |
| `chmod u+x file` | Add execute for owner | `chmod u+x script.sh` |
| `chmod go-w file` | Remove write from group/others | `chmod go-w config.yaml` |
| `chown user:group file` | Change ownership | `sudo chown shaun:shaun app.py` |
| `chown -R user dir/` | Recursive ownership change | `sudo chown -R shaun:shaun ~/app/` |
| `umask` | Default permission mask | `umask 022` |

## Special Permissions (Advanced)

- **SUID (4)** — Run as file owner (e.g., `passwd` command)
- **SGID (2)** — Run as group owner
- **Sticky Bit (1)** — Only owner can delete (e.g., `/tmp`)

Example: `chmod 4755` = SUID + rwxr-xr-x

## Why Permissions Matter in DevOps

❌ **SSH key with 644 perms** → SSH rejects it (must be 600)  
❌ **Script with 644** → Can't execute, deployment fails  
❌ **Config file with 777** → Security audit fail, anyone can edit secrets  
✅ **Dockerfile with 644** → Safe, readable, not executable  
✅ **deploy.sh with 755** → Owner can run, team can read/execute


## My Lab Observations

1. **Default permission for new files:**644 -rw-r--r--
2. **What happened when I tried to run script.sh before chmod +x?** permission denied as ther is no x(executable )
3. **The permission notation for `chmod 755`:** -rwxr-xr-x
4. **Why does SSH reject keys with 644 permissions?** 
it allows other to see the key and only owner should have the acess, so 600 is correct
5. **Real-world scenario:** If I deploy a Python app to a server, what permissions should `app.py` have?644
none of them need execute acess ,as python interpreter can execute it 