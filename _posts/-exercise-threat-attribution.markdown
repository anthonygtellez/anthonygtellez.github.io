---
title:  "An exercise in threat attribution"
subtitle: "GRIZZLY STEPPE"
author: "Anthony Tellez"
avatar: "img/authors/atellez.jpg"
image: "img/suricata.png"
date:   2016-02-28 22:00:00
---

### What
On Dec 29, 2016, the Department of Homeland Security jointly released a report[https://www.us-cert.gov/sites/default/files/publications/JAR_16-20296A_GRIZZLY%20STEPPE-2016-1229.pdf] with the Directorate of National Intelligence regarding supposed "Russian interface" in the 2016 Presidential Election. The report mostly detailed malicious actor behaviors not uncommon to most cyber security professionals. Included in the release were recommendations to blunt intrusion attempts by state actors and the indicators of compromise related to the "GRIZZLY STEPPE" campaign carried out by Russian Military and Civilian Intelligence Services. [https://www.dni.gov/files/documents/ICA_2017_01.pdf]. The report has circulated widely since the election of a new President with various experts weighing in as to the validity of the report. While this author finds the report dubious, I am leaving it you the reader to make your own judgment through the exercise.

## Request for Assistance 
DHS & DNI jointly requested assistance from the public to contribute additional information they may have related to the threat. Using the indicators provided by the report I will show you how to detect this activity in Splunk and decide as to impact.

### Indicators of Compromise
The first step is to download the csv hosted by us-cert to your own Splunk instance. There are two methods of doing so: splunkweb or command line using wget. Depending on how your environment one may be easier to do than the other. 

Select an app to save the lookup file, it could be search or a custom application. In this case, I have my own app called "security_viz". 

`$ cd /opt/splunk/etc/apps/security_viz/lookups`

From the lookups directory of your app download the lookup from us-cert.
``` 
$ wget https://www.us-cert.gov/sites/default/files/publications/JAR-16-20296A.csv
```
Check the permissions of your knowledge objects by reviewing the configuration of your local.meta or default.meta in the app you selected.

`$ cat /opt/splunk/etc/apps/security_viz/metadata/default.meta`

```
# Application-level permissions

[]
access = read : [ * ], write : [ admin, power ]

### EVENT TYPES

[eventtypes]
export = system

### PROPS

[props]
export = system

### TRANSFORMS

[transforms]
export = system

### LOOKUPS

[lookups]
export = system

### VIEWSTATES: even normal users should be able to create shared viewstates

[viewstates]
access = read : [ * ], write : [ * ]
export = system
```

For the GUI method download the csv to your local desktop and follow the documentation Configure Lookups Splunk Enterprise.

#### Searching in Splunk:
Filter to threat intel where only IPV4 Address are provided. Remove the left and right brackets around the octets from the value of the INDICATOR_VALUE field.

```
| inputlookup JAR-16-20296A.csv where TYPE=IPV4ADDR 
| rex field=INDICATOR_VALUE mode=sed "s/\[|\]//g"

```

Use the crafted lookup as sub search to correlate traffic from any data source. 
```
index=suricata [ | inputlookup JAR-16-20296A.csv where TYPE=IPV4ADDR | rex field=INDICATOR_VALUE mode=sed "s/\[|\]//g" | rename INDICATOR_VALUE as src_ip] 
```

## Known Tor Addresses?
One claim made by many journalists and security experts is the list includes known tor addresses in the indicators of compromise, which makes the report dubious in its attempt to nail Russia as the sole suspect [https://theintercept.com/2017/01/04/the-u-s-government-thinks-thousands-of-russian-hackers-are-reading-my-blog-they-arent/]. This claim should be taken seriously, as various exit nodes in the tor network were previously used by Ghost Net (Chinese based APT) for exfiltration of sensitive governmental information which were intercepted by WikiLeaks [https://www.wired.com/2010/06/wikileaks-documents/]. 

To test this claim we can run compared against known tor addresses and the IOC's provided by the report in Splunk. 


First download the list of known tor addresses from the time specified in the report, merge them into a csv:
`$ wget https://collector.torproject.org/archive/exit-lists/exit-list-2016-05.tar.xz`

Second configure your lookup

Third we write a search which compares the two lookup files against one another. 


## Conclusion


