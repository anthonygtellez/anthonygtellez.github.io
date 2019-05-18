---
title:  "Proactively Responding to #CloudBleed with Splunk"
tags: cloudbleed splunk security SSL Blogs
---
![Cloudbleed](http://tellez.sfo2.digitaloceanspaces.com/cloudbleed.png)

### What is CloudBleed?
Cloudbleed is a serious flaw in the Cloudflare content delivery network (CDN) discovered by Google Project Zero security researcher Tavis Ormany. This vulnerability means that Cloudflare leaked data stored in memory in response to specifically-formed requests. The vulnerability behavior is similar to Heartbleed, but Cloudbleed is considered worse because Cloudflare accelerates the performance of nearly 5.5 million websites globally. This vulnerability might have exposed sensitive information such as passwords, tokens, and cookies used to authenticate users to web crawlers used by search engines or nefarious actors. In some cases, the information exposed included messages exchanged by users on a popular dating site.


### Understanding the severity of Cloudbleed 
![Cloudbleed Diagram](http://tellez.sfo2.digitaloceanspaces.com/clouldbleed-CDN_diagram.png)
CDNs primarily act as a proxy between the user and web server, caching content locally to reduce the number of requests made to the original host server. In this case, [edge servers in the Cloudflare infrastructure](https://blog.cloudflare.com/incident-report-on-memory-leak-caused-by-cloudflare-parser-bug/) were susceptible to a buffer overflow vulnerability, exposing sensitive user information like authentication tokens. Technical details of the disclosure can be viewed on the [Project Zero bug forum.](https://bugs.chromium.org/p/project-zero/issues/detail?id=1139)


### Evaluating risk with Splunk
The most obvious risk introduced by this vulnerability includes any exposed user data, which could be the same credentials users are using for corporate authentication. An easy way to enumerate the scope of this problem is to compare the [list of domains using Cloudflare DNS](https://github.com/pirate/sites-using-cloudflare) against your proxy or DNS logs. This can give you some insight into how often users could be using the affected websites and the relative risk associated with using the same credentials for multiple accounts.

To do this analysis, we first need to download the list of Cloudflare domains and modify the file slightly so we can use it as a lookup.

`$ git clone https://github.com/pirate/sites-using-cloudflare.git`

Convert txt list to csv

```
$ cat sorted_unique_cf.txt | sed -e 's/^/"/' > sorted_unique_cf.csv
$ cat sorted_unique_cf.csv | sed -e 's/$/","true"/' > sorted_unique_cf_final.csv
```

Using a text editor, update first line of file:
Change: `"","true" to "domain","CloudflareDNS"`

Finally, copy the formatted file to the lookups directory of the search app or a custom app that you use for security analysis.
`$ cp sorted_unique_cf_final.csv /opt/splunk/etc/apps/security_viz/lookups/`

After that step is complete, validate the lookup works:
`|inputlookup sorted_unique_cf_final.csv`

This might take some time because there are nearly 4.3 million domains in the lookup.
![Inputlookup image](http://tellez.sfo2.digitaloceanspaces.com/clouldbleed-inputlookup.png)

The domains on the list are not fully qualified domain names (FQDN) so will be harder to match against your proxy and IPS logs that include subdomains. Use [URL Toolbox](https://splunkbase.splunk.com/app/2734/) to parse the DNS queries or HTTP URLs in your IPS or proxy logs.

![Image of Queries](http://tellez.sfo2.digitaloceanspaces.com/cloudbleed-dns_queries.png)

This is an example search of how to use URL Toolbox to parse Suricata DNS queries:

```
index=suricata event_type=dns
| lookup ut_parse_extended_lookup url AS query
```

![Parsed Domains](http://tellez.sfo2.digitaloceanspaces.com/cloudbleed-ut_domain.png)


In the example below, we are parsing DNS queries and comparing them against the Cloudflare lookup. When a domain in the DNS query events matches a domain in the lookup, that event gets a new field called `CloudflareDNS` with a value of “True”.


```
index=suricata event_type=dns 
| lookup ut_parse_extended_lookup url AS query 
| lookup sorted_unique_cf_final.csv domain AS ut_domain OUTPUT CloudflareDNS
```
![Matched True](http://tellez.sfo2.digitaloceanspaces.com/clouldbleed-lookup-DNS.png)

Although the above search is helpful to identify whether or not a domain uses Cloudflare DNS, now we need to go a step further and use the new field to see only DNS requests for Cloudflare domains.

```
index=suricata event_type=dns 
| lookup ut_parse_extended_lookup url AS query 
| lookup sorted_unique_cf_final.csv domain AS ut_domain OUTPUT CloudflareDNS
| search CloudflareDNS=true
```
In my environment, I am checking for about 4.3 million domains for a nine-month period (the entire span of my data). I am doing this to determine if a user has ever visited any of the domains. This is important because random user data was leaked, the credentials could have been compromised at any time.

Another trick I am going to use is to save the results to a CSV file, because this search can take a long time. Saving it to a lookup, allows you to review the results later and filter through the results without needing to run the search again.

```
index=suricata event_type=dns 
| lookup ut_parse_extended_lookup url AS query
| lookup sorted_unique_cf_final.csv domain AS ut_domain OUTPUT CloudflareDNS
| search CloudflareDNS=true
| stats count by src_ip ut_domain
| outputlookup all_affected_domains.csv
```

![Splunk Search](http://tellez.sfo2.digitaloceanspaces.com/clouldbleed-output_affected_domains.png)

The final output from the search shows that during the time period specified, we had nearly half a million visits to websites using Cloudflare. Breaking it down by src_ip, we can see specific users were more frequent visitors to impacted sites than others. These users should change any of their reused credentials as a precaution.

#### Bonus: Use Dig to determine geography of impacted domains
Using a subset of data from the lookup, we can use sed and a script to dig for the ip_address associated with each domain.

Reduce our results to only the domains:

```
|inputlookup all_affected_domains.csv 
| dedup ut_domain 
| rename ut_domain AS domain 
| fields domain
| outputlookup cloud-bleed_domain-only.csv
```

Remove the quotes surrounding each domain in the file:
`$ cat cloud-bleed_domain-only.csv | sed -e 's/"//g' > my_cloud-bleed_domains.csv`

Run this script to find all the IP Address for each Domain:
`$ bash domains.sh`

After running the script, you will have a new lookup called “dig_cloud-bleed_domains.csv” which you can further analyze in Splunk.

```
| inputlookup dig_cloud-bleed_domains.csv where "IP Address"=* 
| rename "IP Address" AS ip_address 
| iplocation ip_address
| stats count by Country
| geom geo_countries featureIdField=Country 
| eval count="count: "+count
```

![Cloud Bleed Map](http://tellez.sfo2.digitaloceanspaces.com/Cloudbleed_domains_Choropleth_map.jpg)

This choropleth map showcases the geographical location of each of the affected domains visited by users in my environment. The majority of the traffic was to sites based in the United States, with traffic to Germany in a distant second place.