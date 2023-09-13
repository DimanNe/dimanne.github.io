title: Fail2Ban

# **fail2ban**


* **Get the status**: `fail2ban-client status`
* **Get the status of a specific jail**: `fail2ban-client status sshd`
* **Unban**: `fail2ban-client set sshd unbanip 1.2.3.4`
* **Ban**: `fail2ban-client set sshd banip 1.2.3.4`
