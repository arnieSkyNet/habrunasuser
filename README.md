# habrunasuser

`habrunasuser` is a utility designed to securely launch a program as a specified user, even when initiated from root. habridge is running as root but root will not use any keys of the user, hence sudo in hadbridge will not work. It uses a two-stage process to pass the program name and execute it under the desired user's environment.

## Features
- Run any program as a specific user.
- Uses specific users ssh keys.
- Preserves the user's environment (e.g., `.bashrc`).
- Securely bridges execution from root to the target user.

## How It Works
1. A script (e.g., `launchprog`) is run with root privileges.
2. The program name is passed through the system.
3. The utility ensures the program runs as the specified user with the correct environment.

## Usage
```bash
# 1. Clone the repository:
git clone https://github.com/yourusername/habrunasuser.git

# 2. Place your desired script (e.g., launchprog) in the repository's main directory.

# 3. Edit launchprog to specify the program to execute:
# Example launchprog content:
#!/bin/bash
# Example: Launches a custom program as user "pi"
lxterminal --display=:0 -e "sudo -u pi env HOME=/home/pi bash -c '/path/to/your/program'"

# 4. Run the script with root privileges:
sudo ./launchprog

eg
/home/pi/sbin/launchprog /usr/bin/ssh -tt mark@flo projects/webcam/webcamctl -r
