---
title:  "Analyzing Shadowbrokers Implants"
tags: ShadowBrokers Splunk Security Blogs
---

### Who are the Shadowbrokers?
- In $DATE the shadowbrokers attempted to auction off zero day tools allegedly developed by the NSA. These tools used undisclosed vulns to target other adversarial nation states and surveil targets with the goal of improving national security. The tools were leaked and subsequently came into possesion of the shadowbrokers. the encrypted set of tools was hosted on github and other sites before they were removed. On $DATE the shadowbrokers released the decryption key (medium.com/shadowbrokers) in response to President Trump's military intervention in Syria and "broken promises". True motives aside, the key released exposed the numerous zero day exploits still unresolved by numerous vendors.

### What is the impact?
- Many of the affected systems will take time to patch while vendors and security researchers attempt to determine how to address each vuln. Due to this disclosure, anyone with the capacity to download the tools can make use of these exploits for their own kits. As information is shared amongst the security community, indictors of compromise tied to this disclosure will develop to scope how to best detect and mitigate these attacks.

### Analysis of implants
- Security researches have "identified" hosts supposedly exploited by reviewing the code dump by the Shadowbrokers. This first blog post will look at the composition of the systems which have been targeted to begin to identify patterns. Subsequent blog posts will analyze indicators of compromise as security researchers dig through the code and disclose signatures to the community.

```
$ wget "https://gist.githubusercontent.com/anthonygtellez/737fed2cebdec5a803ced2d713a7f7d5/raw/a082d2b8bf105e1bfd90639b92221872c4e5e322/dump.csv"
```

Modify transforms.conf to give the lookup a more friendly name:
```
[shadowbrokers]
filename = dump.csv

```

| inputlookup shadowbrokers

| inputlookup shadowbrokers
| stats count by OS
| sort -count


| inputlookup shadowbrokers
| stats count by Implant
| sort -count

### Geographical
| inputlookup shadowbrokers
| rename "IP Address" as ip_address
| iplocation ip_address

| stats count by Country | sort -count

| inputlookup shadowbrokers
| eval _time= (Year-1970) * 31557600 + (Month-1) * 2629800 + (Day * 86400)

### How to detect?
