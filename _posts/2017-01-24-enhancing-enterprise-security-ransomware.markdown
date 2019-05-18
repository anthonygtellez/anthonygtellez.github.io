---
title:  "Enhancing Enterprise Security for Ransomware"
tags: Ransomware Splunk Security ThreatIntel Blogs
---

### Ransomware isn't going away
Ransomware is a profitable business model for cyber criminals with 2016 payments closed at the billon dollar mark. According to a recent survey by [IBM](https://ibm.biz/RansomwareReport), nearly 70% of executives hit by ransomware have paid to get their data back. Those survey results do not include smaller organizations and consumers who are also paying to get their data back.

With the threat from ransomware growing, aside from prevention, detection is key to removing compromised devices from the network. Unfortunately, signature based detection alone will not catch everything, instead using it in combination with hunting techniques in Splunk can enhance your security posture.  In this blog, we will walkthrough adding the free ransomware intelligence feed from abuse.ch to Splunk Enterprise Security.
### Requirements

    - Internet Access for the Splunk Enterprise Security Instance
    - Splunk Enterprise Security
    - Knowlege of updating Splunk Configurations

### Configuration
There are two paths forward, which will depend on the level of access you have to the enterprise security search head. Command line is the simplest option since you can copy paste the configuration from this page, while using the GUI will require you to manually input the data via Splunkweb.

The configuration file walkthrough requires you to create a new inputs.conf file or add to an existing one in the SA-ThreatIntelligence app’s local directory. 

`$ vi /opt/splunk/etc/apps/SA-ThreatIntelligence/local/inputs.conf`

inputs.conf

```
[threatlist://ransomware_ip_blocklist] 
delim_regex = : 
description = abuse.ch Ransomware Blocklist 
disabled = false 
fields = ip:$1,description:Ransomware_ip_blocklist 
type = threatlist 
url = https://ransomwaretracker.abuse.ch/downloads/RW_IPBL.txt

```
Once completed, restart the splunkd service.
`$ /opt/splunk/bin/splunk restart`

GUI Walkthrough:

Locate the Enterprise Security Configuration Page

From the Enterprise Security Configuration page, select `Threat Intelligence Downloads`.
![Data Enrichment](http://tellez.sfo2.digitaloceanspaces.com/ess_configuration.png)

Click new, and fill in the various text fields on the resulting page:
![Threat Intel](http://tellez.sfo2.digitaloceanspaces.com/threatlist.png)
![Threat Intel Settings](http://tellez.sfo2.digitaloceanspaces.com/threatlist_edit.png)

Once configured, Enterprise Security will download the threat intelligence and begin alerting on any events found which match the threatlist. These can be reviewed and triaged as part of your workflow in the notable events page.

Note: If are not seeing the threat intelligence feed take a look at the troubleshooting guide on docs: [Verify that you added threat intelligence successfully](http://docs.splunk.com/Documentation/ES/4.6.0/User/Configureblocklists#Verify_that_you_added_threat_intelligence_successfully).
![Incident Review](http://tellez.sfo2.digitaloceanspaces.com/notable_event.png)

If you need an additional example, take a peek at docs.splunk.com for adding the domain ransomware threat feed: [Add a ransomware threat feed to Splunk Enterprise Security.](http://docs.splunk.com/Documentation/ES/4.6.0/User/Configureblocklists#Add_a_ransomware_threat_feed_to_Splunk_Enterprise_Security)