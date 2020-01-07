---
title: Using Docker and Splunk to Operationalize the Splunk Machine Learning Toolkit
tags: MachineLearning Security DataScience Splunk Docker
layout: article
mathjax: true
date: 2019-03-19
---

Configuring and maintaining a Splunk Dev environment can be challenging as new releases of apps and the software are made available. Leveraging the official [Docker image](https://www.splunk.com/blog/2018/10/24/announcing-splunk-on-docker.html), the newest versions of [Splunk Enterprise](https://www.splunk.com/en_us/software/splunk-enterprise.html) and various apps can be made available without a time commitment or worries about future updates.

The requirements for this tutorial are:

Docker engine installed
- Root/Sudo Access for the server running docker
- Internet connectivity for the server or workstation
- Basic understanding of docker
- Splunkbase account username & password

To check if Docker is installed on your server or workstation simply type:

```
$ docker --version
Docker version 18.09.2, build 6247962
```

If you see a similiar output from the command above, you can move onto the next step. Otherwise checkout the documentation at Docker.com to [install for your platform of choice](https://docs.docker.com/install/). 

Using the Splunk Supported Docker Image
You can pull the latest version of the [Splunk supported Docker image](https://www.splunk.com/blog/2018/10/24/announcing-splunk-on-docker.html) using the following command syntax:

`$ docker pull splunk/splunk:latest`

More information on the officially supported Docker image can be found at [Docker Hub](https://hub.docker.com/r/splunk/splunk/).

Using the following command syntax, more advanced environmental variables will allow you to pre-install apps into the container running Splunk Enterprise.

```
$ docker run -d -p 8000:8000 -e 'SPLUNK_START_ARGS=--accept-license' -e 'SPLUNK_PASSWORD=splunk123' -e SPLUNK_APPS_URL=https://splunkbase.splunk.com/app/2882/release/1.3/download,https://splunkbase.splunk.com/app/2890/release/4.1.0/download  -e SPLUNKBASE_USERNAME=<redacted@domain.com> -e SPLUNKBASE_PASSWORD=<redacted> splunk/splunk:latest
```

In the above example there are two parameters that will need to be changed:

```
SPLUNKBASE_USERNAME=<redacted@domain.com> -e 
SPLUNKBASE_PASSWORD=<redacted>
```

These will need to be changed to your [Splunkbase](https://splunkbase.splunk.com/) username and password credentials in order for the apps ([Python for Scientific Computing](https://splunkbase.splunk.com/app/2882/) v 1.3 & [Splunk Machine Learning Toolkit](https://splunkbase.splunk.com/app/2890/) v 4.1.0) to be downloaded as part of launching the container.

With the parameters set, you should be able to login to the Splunk instance using the credentials admin:splunk123. If you prefer a different password, you can update the parameter in the docker command line or change it after logging in. Splunk web can be accessed using a web browser: http://127.0.0.1:8000 or http://localhost:8000

If you’d like to dive deeper into the Splunk Machine Learning Toolkit and the showcase examples in the product, you can leverage our [Splunk Machine Learning](https://www.youtube.com/playlist?list=PLxkFdMSHYh3Q1jwpgJJ0ZSnRzZIx2c_KM) channel on YouTube for more.