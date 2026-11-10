*This project has been created as part of the 42 curriculum by oalsmadi.*

# Born2beRoot

## Description

Born2beRoot is a system administration project. It uses a Debian 13 virtual machine in Oracle VirtualBox to practice encrypted storage, local account management, SSH, firewall rules, and system monitoring.

### Operating system choice

I chose Debian stable for its minimal installer, package availability, and familiar APT tools. Debian is a good fit for this server because it can be installed with a text console and a small set of services. Stable releases prioritize predictable updates, so some packages are older than their upstream releases.

Rocky Linux is a stable alternative with an enterprise Linux ecosystem, long support cycles, and tools used on RHEL-compatible servers. Its DNF and SELinux workflows are useful in enterprise settings, but they would require me to learn a different package manager and security configuration for this project.

### Main setup choices

| Area | Implementation |
|---|---|
| Storage | One LUKS-encrypted container backs LVM logical volumes for the root filesystem, var, home, tmp, and swap. The boot partition is separate. LVM lets the system manage these volumes independently. |
| Accounts | The hostname is oalsmadi42. The project account is oalsmadi and belongs to sudo and user42. |
| Passwords | PAM checks length, character classes, repeated characters, and the username. The regular password-change flow checks the difference from the old password. Passwords expire after 30 days, with a two-day minimum between changes and a seven-day warning. |
| Privilege escalation | Sudo requires a terminal, allows three password attempts, shows a custom error, restricts the command search path, and records command and session logs under /var/log/sudo. |
| Services | AppArmor is enabled. OpenSSH listens on TCP 4242 in the guest and disallows root login. UFW denies incoming connections by default and allows TCP 4242. |
| Monitoring | A Bash script reports system, resource, session, network, and sudo activity. Cron is configured to launch it on boot and every ten minutes. The script sends the report to the terminals associated with logged-in sessions. |

### Platform and security comparisons

| Comparison | Differences |
|---|---|
| Debian vs Rocky Linux | Debian uses DEB packages and APT. Rocky uses RPM packages and DNF and targets compatibility with the RHEL ecosystem. Both offer stable server releases. |
| AppArmor vs SELinux | AppArmor profiles commonly identify files by path. SELinux applies policy using labels and contexts attached to processes and resources. Both add access controls beyond ordinary Unix permissions. |
| UFW vs firewalld | UFW provides a simple rule interface for allowing or blocking traffic. firewalld organizes policy around zones and services, which helps when network interfaces need different policies. |
| VirtualBox vs UTM | VirtualBox is a desktop hypervisor used here on a Linux host, with NAT and port forwarding. UTM runs on Apple platforms and supports virtualization or emulation depending on the guest architecture and backend. |

## Instructions

1. Open the configured Debian VM in VirtualBox, start it, and enter the disk encryption passphrase if prompted.
2. Sign in on the console as oalsmadi. From the host, the configured NAT rule forwards loopback TCP 14242 to TCP 4242 in the guest:

   ~~~sh
   ssh -p 14242 oalsmadi@127.0.0.1
   ~~~

3. Inspect the relevant settings and run a monitoring report from the Debian shell:

   ~~~sh
   hostname
   id -nG
   chage -l oalsmadi
   lsblk -f
   systemctl is-active apparmor ssh cron
   sudo visudo -c
   sudo ufw status verbose
   sudo sshd -T | grep -E '^(port|permitrootlogin) '
   sudo /usr/local/bin/monitoring.sh
   ~~~

4. The monitoring schedule is defined in /etc/cron.d/born2beroot. To pause it during an evaluation, move that schedule file outside /etc/cron.d and restore it afterward.
5. For submission, place this README and signature.txt at the repository root. Calculate the SHA-1 of the powered-off VM's VDI on the host after the final verification. Keep the virtual disk outside the Git repository.

## Resources

- [Debian Installation Guide](https://www.debian.org/releases/stable/installmanual)
- [Debian manual for OpenSSH server configuration](https://manpages.debian.org/trixie/openssh-server/sshd_config.5.en.html)
- [Debian sudoers manual](https://manpages.debian.org/trixie/sudo/sudoers.5.en.html)
- [Debian password quality configuration manual](https://manpages.debian.org/trixie/libpwquality-common/pwquality.conf.5.en.html)
- [Debian Cron file format](https://manpages.debian.org/trixie/cron/crontab.5.en.html)
- [VirtualBox User Manual](https://www.virtualbox.org/manual/)
- [Rocky Linux documentation](https://docs.rockylinux.org/) and [UTM documentation](https://docs.getutm.app/)

### AI use

ChatGPT helped interpret the subject, suggest Debian configuration commands, and troubleshoot terminal input, sudoers syntax, SSH port forwarding, and monitoring output. It proposed the monitoring script and helped draft this README. I applied the commands and configuration in the VM and used terminal output to check the results.
