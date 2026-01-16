# Content
1. [Scenario](#scenario)
2. [Incident Response Narrative](#incident-response)
3. [Lessons Learned](#lessons-learned)

## 1. Scenario<a id='scenario'></a>
A system administrator reported to SOC that after downloading a technical document from an email, he suspected that the downloaded file might not have been from the legitimate vendor. He only realized this after he tried but could not open to read the document and thus, he immediately called the vendor directly to check if the file was broken. However, the vendor replied that they did not send any file to him during that time.
Upon SOC’s request, the system administrator provided the estimated time frame of which these events occurred.
<br>
### Key concerns from stakeholders: 
I. Could the file be a malware? What is it trying to do? <br>
II. Why did none of the cybersecurity technologies detected and stopped this? <br>
III. How to prevent this from happening? <br>

## 2. Incident Response Narrative<a id='incident-response'></a>
### 2.1. Identification

### Identification source: Email
As an alternative to the system administrator manually handing over the suspicious email to the SOC for investigation through a secure channel, the SOC analyst can also retrieve a copy of emails from the email security solution which acts as a gateway for inbound emails to the company’s domain in a controlled, auditable way that balances security needs, privacy, and legal requirements. The email body is as follows:

```
Hello John, 

Marcus here! Check out this document here: https://file.io/XXXXXX.
It can help you and your team do troubleshooting for some common issues while using our products.

Hope it helps!

Regards,
Marcus
Solution Consultant
```
The email was further analysed based on the below methodology:


#### 1.	Content/Contextual Examination
While the language used is in a casual tone, the email body looks legitimate as the vendor’s consultant Marcus is known in-person by the system administrator.
Also, there are no signs of social engineering red flags such as the request for urgent action, the promise of personal rewards etc.

#### 2.	Sender and Header Examination
The analyst checked by inspecting the email header using a parser such as https://mxtoolbox.com/EmailHeaders.aspx to identify the true origins. Specifically, the following fields are checked: <br>
i.	Date <br>
ii.	From <br>
iii.	Subject <br>
iv.	To <br>
v.	Reply-To <br>
vi.	Return-Path  <br>
vii.	X-Sender-IP <br>
viii.	Received <br>
However, nothing out of the ordinary was observed compared to other prior emails from the sender.

The analyst further checked sender authenticity such as Sender Policy Framework (SPF), DomainKeys Identified Mail (DKIM), Domain-based Message Authentication, Reporting & Conformance (DMARC) in MX records (https://mxtoolbox.com/SuperTool.aspx). However, these records were found to be clean at the time of investigation.
This is indicative with high confidence that the email indeed came from the vendor’s email domain with clean reputation.

#### 3.	URL Examination
The sender uses a legitimate file sharing service File.IO (https://www.file.io/). However, during the time of investigation, the file being shared had been removed online and was no longer downloadable by the analyst.



### Identification source: Host
From the endpoint detection and response (EDR) solution, the analyst can acquire Sysmon event logs (sysmon.evtx) during specific time frames from the system administrator’s endpoint to investigate. 
This would provide evidence to the series of events which occurred.

The below events indicates that the file was downloaded from hxxps://file.io/XXXXXX by the information from the zone identifier.
The file is named XXXX_Troubleshooting_Guidev2.pdf.exe and it was downloaded using Microsoft Edge web browser.
<img width="840" height="397" alt="image" src="https://github.com/user-attachments/assets/1bd36f53-9ca1-4df5-8a89-1865b69af7b4" />
<img width="1262" height="732" alt="image" src="https://github.com/user-attachments/assets/7d530816-7072-4469-b0c0-68e7426bfa55" />

> Analyst's note: The name of the file appeared to be a simple attempt to trick a user to think that it is a pdf document although it is an executable.

From these events, the hash was identified and searching it on VirusTotal (https://www.virustotal.com/gui/home/search) indicated that it could be a publicly unknown artifact since there were no search results based on hash itself.

Further analysis down the timeline found that malware was eventually executed (Sysmon Event ID 1: Process Create).
<img width="1133" height="660" alt="image" src="https://github.com/user-attachments/assets/20ca3a2e-7bab-4e18-950e-a589f28e9eba" />

Then the malware made connection to external IP address X.X.X.X on destination port XXXX as per (Sysmon Event ID 3: Network Connection Detected)
<img width="989" height="575" alt="image" src="https://github.com/user-attachments/assets/d39107a4-26d7-436a-9cb7-1f96aa97cbc7" />

And from the event logs, it was observed that the attacker added a new local user account XXXXX_support. <br>
<img width="673" height="444" alt="image" src="https://github.com/user-attachments/assets/fe221f33-ad2a-4bed-8df1-f6efb6a8a7a9" />

And added the new local user account to the local Administrators group. <br>
<img width="657" height="431" alt="image" src="https://github.com/user-attachments/assets/d1a9056f-23a0-47d4-99d7-a22d67375ab4" />
<br>
However, then there was a time gap of about two minutes before the next event which showed that the attacker-added user account was deleted. <br>
<img width="905" height="591" alt="image" src="https://github.com/user-attachments/assets/8ef23c60-ea89-4f38-b6d8-6c355e188a71" />

In between these time gaps, there were no events captured. Here are some possibilities why: <br>
i. There was a technical bug with the endpoint security solution or OS which is more common than vendors are willing to admit. <br>
ii. The attacker had hastily deleted some event logs. <br>
iii. Certain activities are not configured to be logged based on how existing configurations is set. <br>

Thus, between the time period of creation and deletion of the user cisco_support, it is very likely that the attacker would have taken further actions on the system before deleting this user account.

> Analyst notes: Whatever the true reason for gaps in system events should be left for further investigation during the incident post mortem. This may involve re-evaluating the Preparation Phase of the IR plan

**Due to time-urgency of an IR, the analyst needs to be adaptable and re-orient the investigation to other data sources such as network logs etc..**


### Identification source: Network


### 2.2. Containment
### 2.3. Eradication
### 2.4. Recovery

## 3. Lessons Learned<a id='lessons-learned'></a>
