# Network-Traffic-Analysis

## Introduction : 
Network traffic analysis plays a crucial role in cybersecurity by identifying malicious activities, detecting unauthorized access, and ensuring the integrity of network communications. This project focuses on an in-depth analysis of a real-world scenario from the BTLO: Network Analysis – Web Shell Challenge. Wireshark Tool has been used to conduct the analysis. After the completion of analysis, a report of key findings is prepared. 

## Case : 
The SOC received an alert in their SIEM for ‘Local to Local Port Scanning’ where an internal private IP began scanning another internal system. Can you investigate and determine if this activity is malicious or not? You have been provided a PCAP, investigate using any tools you wish.

## Investigation : 

1. Begin by ensuring the proper formatting of the Time field for accurate analysis.

![image](https://github.com/user-attachments/assets/52dcfe04-6f79-49bc-bd17-35a7a19263e0)


2. In the statistics tab, check the capture file properties. It is important to first verify whether it is the right pcap file to analyze the incident. 

![image](https://github.com/user-attachments/assets/bb0cadc2-bc88-4457-86ba-b9052a877f7d)


3. In Statistics → Conversations → TCP (Port 1284), it is observed that a single IP, 10.251.96.4, repeatedly sends packets from port 41675 to various ports on 10.251.96.5.
This pattern indicates a port scanning attempt.
Notably, the packet sizes for port 80 (HTTP) and port 22 (SSH) are larger than the rest, suggesting that these ports were found open.
It is likely that 10.251.96.5 is a web server.

![image](https://github.com/user-attachments/assets/e630342a-1673-4404-a129-071bbfdf51bb)


4. Frame 73 : It is a DNS Request to the Ubuntu Server  which replies with the 3 addresses. These addresses were also observed in the conversations view indicating it being legitimate traffic. 

![image](https://github.com/user-attachments/assets/d66ceb7b-9c34-4943-bcb9-f9b15a17b6ee)


5. From Frame 117 onwards, 10.251.96.5 starts scanning different ports of 10.251.96.4, confirming the port scanning attack hypothesis.
The scanner sends SYN requests, and most responses are RST, ACK, meaning the TCP handshake is being denied (ports likely closed).

![image](https://github.com/user-attachments/assets/b28c3822-8256-4498-80a1-8a93b9d310f3)

However, port 80 and 22 respond with SYN, ACK, further confirming they are open.

![image](https://github.com/user-attachments/assets/180c3211-2b47-4e48-8c82-0603850626d3)

The scanning activity concludes at Frame 2166, with a timestamp of 08:33:06

![image](https://github.com/user-attachments/assets/cdaf265c-d5de-4950-987a-605da281934c)


6. Gobuster Tool Detection (Frame 2215 - 08:34:05):
The HTTP stream reveals the User-Agent field as Gobuster 3.0.1.
Gobuster is a tool used by attackers to crawl web directories.

![image](https://github.com/user-attachments/assets/89a483bf-13d1-4ad1-ad68-01846a21e88c)


7. Applying the filter (ip.src == 10.251.96.5) && (http.response.code == 200) reveals abnormally large response sizes at Frames 7725 and 13894.

![image](https://github.com/user-attachments/assets/86b01f8e-6fca-4e9b-9da6-06016e983211)

Analysis of Frame 7725 confirms that Gobuster successfully extracted directory information. The php version is 7.2. It is useful for the attacker to know the version because it can then look up for vulnerabilities existing for that version. 

![image](https://github.com/user-attachments/assets/54d9c990-f029-4087-9c36-d956ce310a9a)

Frame 13894 shows a similar response, but with a different user-agent, indicating that a manual user may have accessed the directories.
At Frame 13661 (08:34:06), Gobuster completes its scan, stopping at the letter Z.

![image](https://github.com/user-attachments/assets/1dd31ba9-5a15-4009-ae25-ddc65f7f73f4)


8. At 13914, the scan reveals an upload directory, which could be a potential entry point for an attacker.

![image](https://github.com/user-attachments/assets/a66f5d22-7076-42cf-b0cf-a0977b488da0)

Let’s check for a POST request to see if there is an actual upload. At 13979, ·  A POST request is detected, indicating a possible file upload. The HTTP stream reveals the use of SQLMap, a tool for performing SQL Injection attacks. Timestamp : 08:36:17

![image](https://github.com/user-attachments/assets/1eb972a2-0b71-4638-a685-908cee5d001f)

At 14060, The URL contains unusual parameters.

![image](https://github.com/user-attachments/assets/e52ba9f2-7fcc-42e4-a299-f254a86049eb)

Decoded URL analysis reveals attempts at SQL Injection and Cross-Site Scripting (XSS). The attacker also attempted to open a web shell to access the passwd file.

![image](https://github.com/user-attachments/assets/5736556b-6f7c-4468-a4dc-b1d72b61e6c2)


9. Frame 16102 - 08:40:39: 
After going through it HTTP stream, we can make an inference that the attacker clicked on “editprofile” where there was an “upload” button and they started uploading file “dbfunctions.php”. 

![image](https://github.com/user-attachments/assets/bf28815b-478c-4565-bc8b-7f24b1d9a21f)


10. Frame : 16144, The attacker executed the whoami command, which displays the currently logged-in user on a Linux system.

![image](https://github.com/user-attachments/assets/51dfc6ff-dddb-4e28-a8c0-916e4cddee9f)


11. Frame : 16201, a suspicious URL is observed. 

![image](https://github.com/user-attachments/assets/9a5309ae-3497-4ba5-8773-9385fee3c7e8)

On using the URL Decoder, following result is obtained, 

![image](https://github.com/user-attachments/assets/caae6613-1e87-4b99-b536-815f56099f6e)

The decoded URL reveals that a connection was opened to 10.251.96.4 on port 4422 using bin.sh -I. This suggests the attacker attempted to establish a reverse shell.


12. Frame 16203 - 16205
A successful TCP handshake is observed, confirming that a reverse shell connection was established.

![image](https://github.com/user-attachments/assets/3d7a47e3-87df-4697-8793-e56a1a152789)

Further analysis reveals the attacker executing commands on the compromised web server.

![image](https://github.com/user-attachments/assets/6f5603ea-c5fa-4e56-9f93-a263486eabac)

---

[Read the Analysis Report](Network%20Traffic%20Analysis%20Report.md)

