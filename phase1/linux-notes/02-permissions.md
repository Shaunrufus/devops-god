# Linux Permissions Lab Observations

1. **Default permission for new files:** `644` (or `rw-r--r--`), which is determined by the system's default `umask` of `0022` applied to the maximum file permission of `666`.
2. **What happened when I tried to run script.sh before chmod +x?** Trying to run the script directly with `./script.sh` resulted in a `Permission denied` error because the file lacked the execute (`+x`) permission.
3. **The permission notation for `chmod 755`:** `rwxr-xr-x` (Owner: read/write/execute, Group: read/execute, Others: read/execute).
4. **Why does SSH reject keys with 644 permissions?** A permission of `644` allows anyone on the system (group and others) to read the private key. Since private SSH keys contain highly sensitive cryptographic data, SSH strictly requires the file to be inaccessible to others (typically `600`, readable/writable only by the owner) to prevent unauthorized access.
5. **Real-world scenario:** If I deploy a Python app to a server, what permissions should `app.py` have? It should have `644` (read/write for owner, read-only for group and others) or more restrictively `600`/`640` if it contains sensitive info. It does not need execute (`+x`) permissions, as it is executed by passing it to the Python interpreter (e.g., `python app.py`).
