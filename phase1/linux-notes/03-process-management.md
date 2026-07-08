# Linux Process Management Lab Observations

1. **My current shell's PID:** `696` (can be confirmed by running `echo $$` inside the active shell).
2. **What command shows live CPU/memory usage?** `top` (or the more interactive, user-friendly `htop`).
3. **Difference between `kill` and `kill -9`:** `kill` (SIGTERM, signal 15) politely requests a process to terminate, allowing it to clean up resources, save its state, and exit gracefully. `kill -9` (SIGKILL, signal 9) immediately forces the OS to terminate the process, not allowing it to clean up or run exit handlers.
4. **What happens when you run `sleep 300 &`?** It runs the `sleep 300` command in the background (asynchronously), freeing up the shell prompt immediately. The terminal will output the background job number and process ID (PID) (e.g., `[1] 12345`).
5. **Real scenario:** If nginx crashed at 3am, what's your first command? `sudo systemctl status nginx` (to check the current service status and read the last few log lines) or `sudo journalctl -u nginx -n 50 --no-pager` (to view the recent logs and identify the cause of the crash).
