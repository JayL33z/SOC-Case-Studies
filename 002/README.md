# Content
1. [Scenario](#scenario)
2. [Incident Response](#incident-response)
3. [Lessons Learned](#lessons-learned)

## 1. Scenario<a id='scenario'></a>
As per advisory from external threat intelligence, a cybersecurity engineer urgently configured a rudimentary update of the signature-based rules on the Security Information and Event Management (SIEM) solution. Then from the solution, SOC was alerted there was match for IP address (23[.]254[.]226[.]90) on the web server hosting the organization’s corporate website. Although based on the threat intelligence, this IP address is an indicator-of-comprise (IOC) that was just known to be malicious in the wild, the match on the SIEM was based from past events/logs on the web server before the IOC was added.
<br>
### Key concerns from stakeholders: 
I.	Has the web server already been compromised before the alert? <br>
II.	Did the adversary breach into the internal network? <br>
III.	What could have been done to detect this even before the ingestion of IOCs from external threat intelligence?

## 2. Incident Response<a id='incident-response'></a>
### 2.1. Identification

### Identification source: Email

### Identification source: Host

### Identification source: Network


### 2.2. Containment
### 2.3. Eradication
### 2.4. Recovery

## 3. Lessons Learned<a id='lessons-learned'></a>
