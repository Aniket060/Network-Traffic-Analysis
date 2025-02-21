# **Incident Analysis Report**  

## **1. Executive Summary**  
On **February 7, 2021**, an alert for **Local to Local Port Scanning** was triggered in the SIEM, indicating potential malicious activity. Upon investigation of the provided PCAP file, it was determined that an internal system (**10.251.96.4**) performed reconnaissance and successfully compromised another internal system (**10.251.96.5**). The attacker executed commands via a web shell, leading to unauthorized access.  

### **Key Takeaways:**  
- **Attack Type:** Internal Port Scan, Web Exploitation, Reverse Shell Access  
- **Source (Attacker):** `10.251.96.4`  
- **Destination (Victim):** `10.251.96.5`  
- **Tools Used:** `gobuster`, `sqlmap`  
- **Successful Reverse Shell:** Yes  

---

## **2. Attack Timeline**  

| **Timestamp**        | **Activity**                  | **Details**                                          |
|----------------------|------------------------------|------------------------------------------------------|
| **08:33:06**        | Port Scanning Initiated      | `10.251.96.4` scanned `10.251.96.5` on ports **22 & 80** |
| **08:34:05**        | Attack Tool Execution        | `gobuster 3.0.1` used for directory enumeration     |
| **08:34:17**        | SQL Injection Attempt        | `sqlmap 1.4.7` executed against `10.251.96.5`       |
| **08:40:39**        | Webshell Uploaded            | File: `dbfunctions.php` uploaded via `editprofile.php` |
| **08:42:35**        | Reverse Shell Established    | Connection from `10.251.96.5` to `10.251.96.4:4422` |
| **08:45:56**        | Last Observed Activity       | Attacker ran commands: `bash -i`, `whoami`, `cd`, `ls`, `python`, `rm db` |

---

## **3. Technical Analysis**  

### **3.1 Port Scanning**  
- **Source:** `10.251.96.4:41675`  
- **Destination:** `10.251.96.5 (Ports 22, 80)`  
- **Scan Type:** **TCP SYN Scan** (since the source port remains the same)  
- **Range of Ports Scanned:** **1-1024**  

### **3.2 Exploitation Tools Used**  
- **Gobuster (3.0.1)** → Directory enumeration  
- **SQLMap (1.4.7)** → SQL Injection attempt  

### **3.3 Webshell Deployment**  
- **File Uploaded:** `dbfunctions.php`  
- **Uploaded Via:** `editprofile.php`  
- **Parameter Used:** `cmd` (for command execution)  

### **3.4 Reverse Shell Execution**  
- **Callback to Attacker:** `10.251.96.4:4422` (TCP)  
- **Commands Executed:**  
  ```sh
  id
  whoami
  bash -i
  cd
  ls
  rm db

---

### **4. Indicators of Compromise (IoCs)**

| **Type**            | **Value**                      |
|----------------------|------------------------------|
| Attacker IP        | 10.251.96.4   | 
| Victim IP        | 	10.251.96.5       | 
| Malicious Files       | dbfunctions.php, editprofile.php        | 
| Ports Scanned        | 1-1024           | 
| Reverse Shell Port       | 	4422   | 

---

### **5. Conclusion**
The investigation confirms that 10.251.96.4 conducted an unauthorized internal scan and compromised 10.251.96.5 using directory enumeration, SQL injection, and a web shell upload. The attacker successfully gained a reverse shell, allowing remote execution of commands on the compromised server.
