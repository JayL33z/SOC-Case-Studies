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
The analyst checked by inspecting the email header using a parser such as     https://mxtoolbox.com/EmailHeaders.aspx to identify the true origins. Specifically, the following fields are checked: <br>
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

### Identification source: Network


### 2.2. Containment
### 2.3. Eradication
### 2.4. Recovery

## 3. Lessons Learned<a id='lessons-learned'></a>
