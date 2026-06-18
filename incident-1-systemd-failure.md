# Incident 1: systemd Service Startup Failure

## Service Description
Python HTTP server running on port 8080, managed by systemd.

## Symptoms
- `systemctl status` showed "active (auto-restart)" with status=200/CHDIR
- `curl localhost:8080` returned "connection refused"
- Restart counter reached 9062

## Investigation
- Read systemd status: exit code 200/CHDIR means chdir() syscall failed
- Checked service file: User=ubuntu, WorkingDirectory pointed to /home/ubuntu
- Verified /home/ubuntu does not exist on this machine
- Actual username is simranagrawal

## Root Cause
User and WorkingDirectory pointed to a nonexistent home directory. systemd could not change into the target directory, so the ExecStart command never ran.

## Fix
Updated User=simranagrawal and WorkingDirectory=/home/simranagrawal/troubleshooting-lab/app in /etc/systemd/system/troubleshoot.service, then ran systemctl daemon-reload and systemctl restart.

## Lessons Learned
- Read error codes precisely. CHDIR told me exactly what failed.
- Always verify User and WorkingDirectory match the actual environment.
- systemd Restart=on-failure works relentlessly. Check restart counters to know how long a service has been failing.
