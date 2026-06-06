# habrunasuser

`habrunasuser` is a utility designed to securely launch a program as a specified user, even when initiated from root. habridge is running as root
but root will not use any keys of the user, hence sudo in
[ha-bridge](https://github.com/bwssytems/ha-bridge) will not work. It uses a two-stage process to pass the program name and execute it under the desired user's environment.

## Features
- Run any program as a specific user.
- Uses the target user's SSH keys and SSH agent session.
- Preserves the user's environment (e.g., `.bashrc`).
- Supports SSH-based automation using the target user's authenticated session.
- Securely bridges execution from root to the target user.

## How It Works
1. A script (e.g., `launchprog`) is run with root privileges (typically via HA Bridge).
2. The program name and arguments are passed through the system.
3. The utility ensures the program runs as the specified user with the correct environment.
4. SSH commands rely on an existing user session SSH agent (not a newly created root agent).

## SSH Agent Requirements (IMPORTANT)

When used with HA Bridge, `habrunasuser` executes commands as the target user so that the user's SSH configuration and keys are used instead of root's environment.

For SSH-based automation to work reliably, the target user's SSH agent must already be running and have the required keys loaded.

During testing it was discovered that HA Bridge may start before the desktop session, keyring and SSH agent have fully initialised. In that situation, SSH commands executed through `habrunasuser` may prompt for the private key passphrase instead of using the existing authenticated SSH session.

To avoid this problem, HA Bridge should only execute automation once the user's desktop session and SSH agent are available.

A common solution is to delay HA Bridge startup until the desktop environment is ready, for example:

```bash
sleep 10
sudo systemctl start ha-bridge

