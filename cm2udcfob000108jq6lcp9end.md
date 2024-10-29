---
title: "Introduction to Forensics and Incident Response - Cyberexam Lab Writeup"
datePublished: Tue Oct 29 2024 11:32:55 GMT+0000 (Coordinated Universal Time)
cuid: cm2udcfob000108jq6lcp9end
slug: introduction-to-forensics-and-incident-response-cyberexam
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1730201470562/92d574a4-0a28-44cd-aadc-cfad7e4b50be.png
tags: log-analysis, dfir, cyberexam

---

[https://learn.cyberexam.io/cyber-academy/digital-forensics/introduction-to-forensics-and-incident-response/quiz-for-lab](https://learn.cyberexam.io/cyber-academy/digital-forensics/introduction-to-forensics-and-incident-response/quiz-for-lab)

---

## Mission Statement

Identify the suspicious activity in Linux systems. Connect with ssh on port 22. Run the commands;

```bash
chmod 400 key
ssh -i key ubuntu@server
```

The time when unusual activities were detected was at 08:14.

## Questions

### What is the account used by the attacker?

After connected the target machine with ssh, we can check the related log files in the target system. auth.log file includes the user authentication and authorization. Lets examine these logs, go to /var/log directory and run the following command, `cat auth.log`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1728231141327/dff10fd3-9bc8-43f1-a2aa-387cdfdabb5a.png align="center")

www-data user tried to run `/usr/sbin/ufw status` and `/usr/sbin/service --status-all` commands with sudo permissions at time 8:16 but the user does not have permission and is not in sudoers that contains users and groups allowed to run command using sudo.

### Which port is modified?

While analyzing the auth.log, we can see that the attacker executed a cronjob in the above picture. To check cronjobs for the user www-data, type command `sudo crontab -l -u www-data`. After the list cronjobs if any, display the cronjab content, go to related directory and examine its content. Realize that the attacker get a reverse shell by using netcat tool. Attacker’s IP address and port number were in the payload.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1728231543706/702067df-e638-47a6-928b-a7f9dad29a7c.png align="center")

Another way to detect suspicious cronjobs, we can analyze the cron.log file under the /var/log directory. It contains cronjobs information.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1728232518961/2e1b77b2-ac34-404e-a02a-7937f76a2159.png align="center")

The port number in the shell payload might add firewall rule. This opinion comes from the beginning of the analyze since the attacker checked for the firewall status by running the command `/usr/bin/ufw status`. This command gives us to modified port number.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1728235021272/3ae851ff-69af-46c6-8361-739c9cf0a4c4.png align="center")

### Which command was used to modify firewall rules?

Become root user with `sudo su` command and check for the command history file or run the `history` command to find commands run by the attacker.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1730199255835/74a4c03e-3513-4af6-b4f7-6ffc67e0e644.png align="center")

### What is the attacker's IP address?

The attacker’s IP address can be found the reverse shell payload which was executed as a cronjob.

### What is the modification date of the /etc/sudoers file? (Format: June 5 09:34)

To find the modification date of the sudoers file, we can use `stat` or `date` commands. stat command display the system status for given file.

`stat /etc/sudoers | grep -i "modify"` command shows us the modification date of the sudoers. Another command `date` can be used with -r parameter giving the date format `date -r /etc/sudoers +"%B %e %H:%M"` gives the modification time.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1730201408954/ef735a47-9a6a-48c4-a59b-4cc2c1033dd3.png align="center")

### Which user is added to the /etc/sudoers file?

Check /etc/sudoers file with command `sudo cat /etc/sudoers`, the added user located at the end of the file.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1728234151053/717d746e-87d0-4ee2-9151-24466b15a075.png align="center")

### **What is the executable file downloaded by the attacker for creating a backdoor?**

There is a bash history file in the /var/www directory.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1728238773068/4012af06-252f-44ab-8457-60695c652513.png align="center")

When we view the content of the file, we observe that the attacker download an executable file with wget tool and gave the executable permission the the payload.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1730199565648/0acb2c64-56fc-4c90-b903-e8c679e7348e.png align="center")

### **What is the pipe used by the attacker for reverse shell?**

It is required to analyze reverse shell code which creates as a cronjob. The reverse shell code includes the pipe name.

### Did the attacker create a new user(Yes/No)?

Check for the users in the system whether the user might be created by the attacker or not.

### When attackers gain access to root privileges? (Format: June 5 09:34:23)

The time of the root access log can be found in /var/log/auth.log file.

### What is the type of cron job created by the attacker?

I tried many keywords like persistent cronjob, recurrent cronjob but these are not the answers. When we consider the question like this, for example the cronjob is crond.hourly or cron.daily according to the cron job. Observe the cronjob with `crontab -l -u www-data` command or cron.log file to find the answer.