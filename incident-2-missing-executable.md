# Incident 2: Missing Executable - Service Fails to Start

## Service Description
Same Python HTTP server on port 8080, managed by systemd.

## What I Broke
Renamed server.py to server.py.bak, then restarted the service.

## Symptoms
- systemctl status showed: active (auto-restart), code=exited, status=2
- Service would not stay running
- Restart counter began climbing

## Investigation
- status=2 typically means file not found (Python couldn't locate the script)
- Checked the WorkingDirectory: /home/simranagrawal/troubleshooting-lab/app
- Listed directory contents: server.py was missing, only server.py.bak present
- Confirmed the ExecStart line referenced server.py, not the .bak file

## Root Cause
The Python script specified in ExecStart did not exist at the expected path. systemd attempted to execute it, Python failed with exit code 2 (file not found), and systemd kept retrying per Restart=on-failure.

## Fix
Renamed server.py.bak back to server.py. Ran systemctl restart troubleshoot. Service returned to active (running) state.

## Lessons Learned
- Exit code 2 from Python means the script file wasn't found
- Always verify the file exists at the exact path in ExecStart
- systemd doesn't care why the process fails — it just reports the exit code. You must interpret it.
- Moving/renaming files under a running service doesn't kill it until restart. The old process kept running with the inode until I restarted.
