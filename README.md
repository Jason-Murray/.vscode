
# .vscode Dotfile for CND


# Getting Started

## Infrastructure

**Workspace Droplet**
- Ubuntu
- Shared CPU - Regular Intel with SSD - Suitable size
- No block storage
- Suitable region
- No additional options
- Add SSH key
- Change hostname to something more friendly, e.g. `workspace-1`
- Give it the tag `workspace`
- No backups
- Create!

Once it's running:
- Set up floating IP for the workspace droplet
- Add IP to your hosts file: `<floating ip> workspace`
- Add SSH config for the droplet:
  ```
  Host workspace
    HostName workspace
    User root
    ForwardAgent yes
  ```
  (ForwardAgent will pass through your local SSH key to the session, allowing you to pull from GitHub etc immediately without copying the keys in to the droplet.)
  
**Firewall Droplet**
- Marketplace Image: `OpenVPN + Pihole`
- Smallest standard size is fine
- No block storage
- In the same region/VPC as workspace
- No additional options
- Add same SSH key as workspace
- Name `openvpn-pihole`, tag `vpn`
- Follow the getting started directions for this droplet, you can `cat /root/client.ovpn` and copy the contents to a local file.
- Add your workspace floating IP to the VPN conf (last line of config) to route only that IP.
```none
route-nopull
route <workspace floating ip> 255.255.255.255
```
- You can now use this file to connect to the VPN droplet, which will provide a secure tunnel to the workspace droplet once we set up the firewall in the following steps. For macOS you could use https://tunnelblick.net/ as your VPN client. There are some further settings that can be configured in Tunnelblick to make this process even better see the Notes section.

**Create a DO Network Firewall**
- Name `workspace`
- IN: allow [`ICMP`, `All TCP`, `All UDP`] from `vpn` tag.
- OUT: allow [`ICMP`, `All TCP`, `All UDP`] from [`All IPv4`, `All IPv6`].
- Apply this firewall to the `workspace` droplet (does not need to be applied to the `vpn` droplet).

## Provisioning
- update & upgrade
	- [Install Docker CE](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository)
	- [Install docker-compose](https://docs.docker.com/compose/install/#install-compose-on-linux-systems)
	- [Install Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-kubectl-binary-with-curl-on-linux)
	- [Install k9s](https://github.com/derailed/k9s)
	- [Install aws](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
	- [Install aws-iam-authenticator](https://docs.aws.amazon.com/eks/latest/userguide/install-aws-iam-authenticator.html)
	- Install `apt-get install -y ncdu unzip`
    - [set file watchers](https://code.visualstudio.com/docs/setup/linux#_visual-studio-code-is-unable-to-watch-for-file-changes-in-this-large-workspace-error-enospc) because vscode needs em
	- clone git repo to /mnt/workspace/.vscode

## Connecting VSCode
- start local vscode
	- "install remote development"
	- "Remote-SSH: Connect to Remote Host"
	- "open workspace from file" /mnt/workspace/.vscode/workspace.code-workspace
	- "Extensions: Show Recommended Extensions" click on the little cloud symbol
	- thas it

# Cleaning up from time to time
- `docker system prune --all`
- `docker volume rm $(docker volume ls)`
- delete vendors
	```bash
	find . -type d -name node_modules -prune -exec rm -rf {} \;
	find . -type d -name .venv -prune -exec rm -rf {} \;
	find . -type d -name .tmp -prune -exec rm -rf {} \;
	find . -type d -name .vscode-server -prune -exec rm -rf {} \;
	```
- `ncdu /`

# Moving to a new machine
- add a disk to droplet named workspace
  - cp -R /root/workspace /mnt/workspace/
  - power down the machine
  - detach the volume and the floating IP
- create and provision a new droplet as seen above
  - cp -R /mnt/workspace/workspace /root/
- detach and destroy the volume

# Notes
TODO: Tunnelblick warnings, nameserver setting, etc.
