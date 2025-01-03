![GitHub last commit](https://img.shields.io/github/last-commit/njarajames/Fail2Ban-Postfix)
[![Years Badge](https://badges.pufler.dev/years/njarajames)](https://badges.pufler.dev)
[![Repos Badge](https://badges.pufler.dev/repos/njarajames)](https://badges.pufler.dev)
[![Commits Badge](https://badges.pufler.dev/commits/monthly/njarajames)](https://badges.pufler.dev)



 `# Fail2Ban Installation and Configuration Guide This guide will walk you through the installation and configuration of Fail2Ban on an Ubuntu server, including setting up a custom action to send email notifications using the `echo` command.`

## Configuration of Postfix for Email Sending

To send email notifications, you need to configure Postfix to use your Gmail account. Follow these steps:

1. Install the necessary packages:
    
    `sudo apt-get install postfix mailutils libsasl2-2 ca-certificates libsasl2-modules`
    
2. Configure Postfix by editing the `/etc/postfix/main.cf` file:
    
    `sudo nano /etc/postfix/main.cf`
    
    Add the following lines in end of the file:
    
    ```bash
   relayhost = [smtp.gmail.com]:587
    smtp_use_tls = yes
    smtp_sasl_auth_enable = yes
    smtp_sasl_security_options =
    smtp_sasl_tls_security_options = noanonymous
    smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
    smtp_tls_CAfile = /etc/ssl/certs/ca-certificates.crt
    
4. Create the SASL password file:
    
    `sudo nano /etc/postfix/sasl_passwd`
    
    Add the following line, replacing `USERNAME` and `PASSWORD` with your Gmail credentials from (https://myaccount.google.com/security):
    
    `[smtp.gmail.com]:587    USERNAME@gmail.com:PASSWORD`
    
5. Secure the password file:
    
    `sudo chmod 400 /etc/postfix/sasl_passwd && sudo postmap /etc/postfix/sasl_passwd`
    
6. Restart Postfix:
    
    `sudo systemctl restart postfix && /etc/init.d/postfix status` 
    

## Configuration of Fail2Ban

Edit the Fail2Ban jail configuration file:

`sudo nano /etc/fail2ban/jail.local`

Add or modify the [sshd] section as follows:


[sshd]
enabled  = true
port     = ssh
filter   = sshd
logpath  = /var/log/auth.log
maxretry = 3
findtime = 300
bantime  = 60
ignoreip = 127.0.0.1
Creating a Custom Action
Create a custom action file to send email notifications using the echo command:


sudo nano /etc/fail2ban/action.d/sendmail-echo.conf
Add the following content:


[Definition]
actionstart =
actionstop =
actioncheck =
actionban = echo "Subject: Fail2Ban Alert: <name> ban of <ip>" | /usr/bin/mail -a "From: fail2ban@example.com" your_email@example.com
actionunban =
[Init]
name = default
Updating the Jail Configuration
Update the jail.local file to use the custom action:


sudo nano /etc/fail2ban/jail.local
Modify the [sshd] section to include the custom action:


[sshd]
enabled  = true
port     = ssh
filter   = sshd
logpath  = /var/log/auth.log
maxretry = 3
findtime = 300
bantime  = 60
ignoreip = 127.0.0.1
action   = sendmail-echo[name=SSH, dest=your_email@example.com]


This configuration snippet is for setting up Fail2Ban to monitor and protect the SSH service. Here's a breakdown of each parameter:

[sshd] enabled = true: This enables the Fail2Ban jail for the SSH service.
port = ssh: Specifies the port that Fail2Ban will monitor. In this case, it's the default SSH port.
filter = sshd: Defines the filter to be used for detecting failed login attempts. The sshd filter is specifically designed for SSH.
logpath = /var/log/auth.log: Specifies the log file that Fail2Ban will monitor for failed login attempts. This is typically where SSH logs are stored.
maxretry = 3: Sets the number of failed login attempts before an IP is banned. Here, it's set to 3 attempts.
findtime = 300: Defines the time window (in seconds) within which the maxretry attempts must occur to trigger a ban. In this case, it's 300 seconds (5 minutes).
bantime = 60: Specifies the duration (in seconds) for which an IP will be banned after exceeding maxretry attempts. Here, it's set to 60 seconds (1 minute).
ignoreip = 127.0.0.1: Lists IP addresses that Fail2Ban should ignore. In this case, the localhost IP (127.0.0.1) is ignored.
action = sendmail-echo[name=SSH, dest=your_email@example.com]: Defines the action to take when an IP is banned. Here, it sends an email notification to your_email@example.com with the subject "SSH".
This configuration helps in securing the SSH service by automatically banning IPs that exhibit suspicious activity, thus preventing potential brute-force attacks.

Restarting Fail2Ban
Restart Fail2Ban to apply the new configurations:


sudo systemctl restart fail2ban
Verification
To verify that everything is working correctly, try to log in with incorrect credentials multiple times to trigger the Fail2Ban rule and check if you receive an email notification.

Conclusion
Now, only the Bash code snippets are included in the sections where they can be copied and pasted.

You have successfully installed and configured Fail2Ban with a custom email notification action using the `echo` command. If you have any questions or encounter any issues, feel free to ask!


