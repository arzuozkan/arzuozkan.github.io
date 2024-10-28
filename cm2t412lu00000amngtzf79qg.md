---
title: "WordPress TakeOver Investigation"
datePublished: Mon Oct 28 2024 14:24:22 GMT+0000 (Coordinated Universal Time)
cuid: cm2t412lu00000amngtzf79qg
slug: wordpress-takeover-investigation
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1730126320824/c353579b-1f6a-4856-ad99-f17b1181221c.png
tags: wordpress-plugins, wordpress, log-analysis, incident-response, cyberexam

---

Lab link: [https://learn.cyberexam.io/challenges/blue-team/incident-response/wordpress-takeover-investigation](https://learn.cyberexam.io/challenges/blue-team/incident-response/wordpress-takeover-investigation)

---

## Mission Statement

An advesary attacked a web server. The logs from the server after the attack are given to us. Analyze the data collected and solve the incident by gathering information about the attacker and the attack.

Log file is in Desktop after connected lab machine.

## Lab Solution

### Which platform under attack?

The targeted platform name is given in incident.log file.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1730060940990/1e4bf890-2a9f-43f1-b77b-9b7accaa0973.png align="center")

### Which tool was used for attack? (First letter is uppercase)

To find the used tool for the attack, we can check for the user-agent value in the captured network packets incident.pcapng file since most of the attack tools is recognized by this method if user agent values doesnt be specifically modified by the attacker to reduce own visibility in the system.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1730062313666/30117640-1958-445c-b2df-9c4c4de4ceb3.png align="center")

### Who performed attack? (IP address)

We can see the IP address that is performed attack after the open network traffic pcapng file with Wireshark tool applying filter `http.user_agent`.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1730062585394/e5d47929-c7c5-4931-a35d-f53264548e6d.png align="center")

According to the picture abovee, the first IP represents source IP address which is the attacker IP address. Second is the destination address which gives the our victim machine IP.

### What is the password of the admin panel?

The attacker requested with SQL payload for credentials wp admin panel. We can observe in both log and pcap file. Apply the `http.request.method=="POST"` filter to list POST HTTP requests. Then examine the packets.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1730063595715/595b5244-295b-42e1-a153-c0d02542ee9d.png align="center")

For the more detailed filter, we can do right click and select apply as a filter option.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1730063690279/6482316f-42bc-4dd9-9fdf-92dfd38f0118.png align="center")

To see hte response of these requests, right click and select the `Follow > HTTP Stream` option. The response code for the right password value will be 302 Found status but this may not always be the case.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1730064032714/4a31add4-fd8c-4194-b69e-eb57785c7937.png align="center")

### What is the password of the alex?

The attacker got a reverse shell by exploiting the SQL injection vulnerability that was found in the wordpress plugin used. The SQL queries were injected rest\_route parameter with GET request and content type of the response was JSON format. Apply filter `http.content_type contains "application/json"`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1730116227993/4a2ef315-5213-42fb-bb0e-27a85428141f.png align="center")

Filter the empty response by adding `frame.len > 663` since the response packet length is that value.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1730116589569/f8d3e458-5010-4f0d-82cb-bc0e265a583a.png align="center")

Notice that a file was modified. Right click the response and Follow &gt; HTTP Stream. We can see the attacker had a reverse shell from 1234 port number by editing 404.php file.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1730117377803/be7b7df5-424b-4c02-9691-213fb21bcc37.png align="center")

In a basic way, it might be detected by filtering POST request `http.request.method == "POST"`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1730117677790/3ef58b43-4b7a-4956-b0cf-85e6b073c445.png align="center")

We found the port number that was connected. Apply filter `tcp.port==1234 && data contains "alex"`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1730118116024/e64f629d-78cc-4d33-8043-6b580e672bd8.png align="center")

Right click the packet and select Follow &gt; TCP Stream option to see whole conversation.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1730118174385/bc383b9e-f5a5-4ba1-97c6-06bd02279c78.png align="center")

### What is the content of the secret.txt?

The content of the secret file is included in the TCP stream.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1730118612796/310ec299-e38a-4a5e-b325-953bae842311.png align="center")

### Which attack technique used? (\*\*\*e-b\*\*\*\* \*\*\*\*\* \*\*\*\*) (lowercase)

The attacker used many attack technique in fact. Exploiting SQLi vulnerability to find wordpress admin panel credentials and having reverse shell with modified 404.php file. Otherwise when we consider the attack type, we can come to the solution. It may be useful to examine the used SQLi payloads to identify the attack type.

HTTP requests can be displayed Statistics &gt; HTTP &gt; Requests

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1730124018380/c15224f6-5bb0-4e8d-88a9-0a57dc5bf8de.png align="center")

There are many requests to review manually.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1730124130664/779c84a9-4d03-4e98-a0e2-215ed67af984.png align="center")

Apply filter the response code is not 500 and request path contains sqli payload `http.request.uri contains "SELECT" && !(http.response.code == 500)` .After that select a random packet and follow TCP stream.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1730124896839/95f1e972-0244-4dfa-9e7b-5e275a311b8e.png align="center")

The payload gives the information about type of the exploited vulnerability.

### What is the CVE number for these attack?

The attacker used rest\_route parameter to inject SQL queries with the tool to find admin panel credentials. We can check for the version or used plugins to identify the vulnerability. Filter incident.log file with grep tool. Run the command `cat incident.log | grep -i -E "plugin|version"` to list plugins.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1730077869340/7b65d256-22c5-4465-8848-cf00cc919cc7.png align="center")

Then search with keywords like `wordpress paid-memberships-pro plugin sqli` or `wordpress rest_route sqli` for the vulnerability.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1730120683955/cbaa495b-15b6-4165-834f-0073ed15fa1b.png align="center")