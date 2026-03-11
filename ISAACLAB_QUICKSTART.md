# IsaacLab Quickstart Guide

## Prerequisites (first time only)
1. Fill in `session_config.sh` with your credentials and resource settings
2. Fix line endings: `sed -i 's/\r$//' start_interactive_session.sh session_config.sh`
3. Run `bash ssh_helpers/setup_slurm_ssh.sh` to set up the `asd-dev-session` SSH alias
4. Add your public key to WATcloud: `cat ~/.ssh/id_ed25519.pub | ssh wato-login1 "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"`

## Starting IsaacLab

**Terminal 1** — start the SLURM job (keep this open):
```bash
bash start_interactive_session.sh
```

**Terminal 2** — connect to the compute node and enter the container:
```bash
ssh -L 5900:localhost:5900 asd-dev-session
cd IsaacLab && export DISPLAY=:1
export DOCKER_HOST=unix:///tmp/run/docker.sock
./docker/container.py start ros2
./docker/container.py enter ros2
```

You are now inside the container at `root@<node>:/workspace/isaaclab#`.

## Stopping IsaacLab

Inside the container:
```bash
exit
./docker/container.py stop ros2
exit
```

Then in Terminal 1, press `Ctrl+C` to end the SLURM job.

## Notes
- **Session timeout**: SLURM job expires after `USAGE_TIME` (default 6h). Restart from Terminal 1 if it expires.
- **Docker persists between sessions**: The container images are cached, so `./docker/container.py start ros2` is fast after the first build.
- **`export DOCKER_HOST`**: Must be run every new SSH session before using `container.py`.
- **Multiple jobs**: If `ssh asd-dev-session` fails with an `nc` error, check for stale jobs: `ssh wato-login1 "/opt/slurm/bin/squeue --user <username> --name=wato_slurm_dev --states=R"` and cancel old ones with `scancel <jobid>`.
- **Isaac Sim version**: If the build fails with package errors, check `~/IsaacLab/docker/.env.base` — `ISAACSIM_VERSION=5.0.0` is known to work.
