# Content
1. [Scenario](#scenario)
2. [Incident Response Narrative](#incident-response)
3. [Lessons Learned](#lessons-learned)

## 1. Scenario<a id='scenario'></a>
As per advisory from external threat intelligence, a cybersecurity engineer urgently configured a rudimentary update of the signature-based rules on the endpoint security solution. Then from the endpoint security solution, SOC was alerted there was match for IP address (23[.]254[.]226[.]90) on the web server hosting the organization’s corporate website. Although based on the threat intelligence, this IP address is an indicator-of-comprise (IOC) that was just known to be malicious in the wild, the match was based from past events/logs on the web server before the IOC was added to the signature-based rule. Due to a technical bug on the endpoint security solution, its logs cannot be remotely acquired by the SOC analyst for investigation. However, since the logs from the web server are piped to the Security Information and Event Management (SIEM) solution as well, the SOC analyst can use the SIEM to investigate this.
<br>

### Key concerns from stakeholders: 
I.	Has the web server already been compromised before the alert? <br>
II.	Did the adversary breach into the internal network? <br>
III.	What could have been done to detect this even before the ingestion of IOCs from external threat intelligence?

## 2. Incident Response Narrative<a id='incident-response'></a>
### 2.1. Identification

### Identification source: SIEM

SOC searched on the SIEM (Splunk Search) with a date/time range that covers the period which the match for the IP address was detected on the EDR.
Then a general query with the specific index and host name for the web server in question.

<br> Splunk Query: ```index="XXXX-web" host="WEB-XXXX-01" ``` <br>

As per below screenshot, the logs which are piped from the web server to the SIEM are *access.log*.

<img width="848" height="537" alt="image" src="https://github.com/user-attachments/assets/13ec5732-e926-4aab-aba5-a321e1c5e68e" />

Using the *Interesting Fields* feature under *clientip*,  SOC observed that the IP address 23[.]254[.]226[.]90 from external threat intelligence has abnormal high log volume on the web server within this time as compared to the other client IP addresses.
<img width="1220" height="531" alt="image" src="https://github.com/user-attachments/assets/94d5f7e1-05a8-4b8a-85a6-da268b06021a" />


This suggests that 23[.]254[.]226[.]90 had accessed the webserver the most during that period which is highly suspicious.
>Analyst notes: The first thing that comes to mind from such high volume of web requests are either attempts for brute-force attack or Denial-of-Service (DoS) attackers.
Then SOC did a search for 23[.]254[.]226[.]90 on VirusTotal to check its reputation.

<img width="1024" height="534" alt="image" src="https://github.com/user-attachments/assets/6671b4a4-ea20-4a16-b684-5440a439f691" />

From the above VT results, there is high confidence that web requests originates from an adversary at the time of investigation.

### 2.2. Containment
### 2.3. Eradication
### 2.4. Recovery

## 3. Lessons Learned<a id='lessons-learned'></a>
