---
title:  Creating Custom Custom Containers for the Deep Learning Toolkit
tags: MachineLearning Security DataScience Splunk DeepLearning Blogs
layout: article
mathjax: true
date: 2020-01-10
---

The [Deep Learning Toolkit](https://splunkbase.splunk.com/app/4607/) (DLTK) was launched at .conf19 with the intention of helping customers leverage additional Deep Learning frameworks as part of their machine learning workflows. The app ships with four separate containers: Tensorflow 2.0 - CPU, Tensorflow 2.0 GPU, Pytorch and SpaCy.

All of the containers provide a base install of Jupyter Lab & Tensorboard to help customers develop and create neural nets or custom algorithms. The DLTK was built with extensibility in mind, as there are many different Deep Learning Frameworks available to data scientists. This blog will cover the open-sourced framework and how to build a custom container with pre-built libraries. 

![Deep Learning](https://splunk.nyc3.digitaloceanspaces.com/deep-learning.png)

https://www.rtinsights.com/top-deep-learning-tools/


## DLTK Container Repository
The DLTK container repository can be accessed on github: [https://github.com/splunk/splunk-mltk-container-docker](https://github.com/splunk/splunk-mltk-container-docker) and the first step in creating a custom container is to copy this repository to your local development environment. 

`$ git clone https://github.com/splunk/splunk-mltk-container-docker`

Reviewing the README.md provides context on the different flavors available for building a base image. These are sorted into tags like tf-cpu, pytorch and match up with the` build.sh` located in the root of the splunk-mltk-container-docker directory. 

There are some key things to note:
- The bases in the build.sh use official images from tensorflow/tensorflow on DockerHub
- The Dockerfile uses pip to install new libraries to customize the image

Creating an image uses the following syntax:
`$ ./build.sh tf-cpu your_local_docker_repo/`

## Creating a Custom Image:
In this example guide we are going to create a custom container to install the Nvidia Rapids Framework [rapids.ai]. You can think of these libraries as similar to the libraries that ship with the Machine Learning Toolkit, but capable of running on Nvidia GPUs. 

In order to create this rapids container, we have to modify a few files in the repository. 
Specifically:
`Build.sh` to support a new tag
Dockerfile to use conda install instead of pip
`Bootstrap.sh`  to adjust the startup of the container for virtual environments

The change to `build.sh` is fairly simple, insert the following line after the nlp section:

```	
rapidsai)
		base="rapidsai/rapidsai:cuda10.0-base-ubuntu18.04"
		;;
```

Changes to the Dockerfile
- Remove the RUN pip install 
- Replace with conda’s syntax:

```
# Installing packages
RUN pip install Flask
RUN pip install h5py
RUN pip install pandas
RUN pip install scipy
RUN pip install scikit-learn
RUN pip install jupyterlab
RUN pip install shap
RUN pip install lime
RUN pip install matplotlib
RUN pip install networkx
```

```
RUN conda install -n rapids jupyterlab flask h5py tensorboard nb_conda
```

Note: The rapids container ships with specialized environments, in this case we want jupyterlab, etc to be installed into the rapids python path,  -n rapids is required to specify this. 

Changes to `bootstrap.sh`
- Fix shell settings in the first line `#!/bin/sh`
- Make rapids environment active

```
#!/bin/bash
source activate rapids
```

## Building the Custom Image
If the Splunk installation is on the same machine it is not required to set up DockerHub. If the machine you plan to deploy DLTK & the custom image is in the cloud or a different environment it’s recommended to configure docker hub to easily create, manage and deploy docker images. 
You can use the following guide to configure [DockerHub] (https://docs.docker.com/docker-hub/) in your development environment. 

You can use a variation of the following command to create your own custom image:
`$ ./build.sh rapidsai docker_repo_name/`

Example:
`$ ./build.sh rapidsai anthonygtellez/`

The created image will exist in your local server and can be deployed once you configure the DLTK. Skip to the configuration step if you’re deploying locally.

The following command is used to push the built image to your DockerHub repository:
```
$ docker push anthonygtellez/mltk-container-rapidsai:latest
The push refers to repository [docker.io/anthonygtellez/mltk-container-rapidsai]
8116c2c543aa: Pushed 
aea568d7d2e7: Pushed 
a4fd893c10d9: Pushed 
b887d71bb9c6: Pushed 
301da5ed4c59: Pushed 
983a89b54d3e: Pushing [================>                                  ]  1.462GB/4.375GB
4d642a1129a1: Pushed 
78db50750faa: Layer already exists 
805309d6b0e2: Layer already exists 
2db44bce66cd: Layer already exists 
digest: sha256:46278436db8c3471f246d2334cc05ad9c8093ab7f98011fc92dc7d807faf4047 size: 2418 
```

The existing layers in rapidsai will not be uploaded, only the changed layers, once the image is uploaded to DockerHub you can pull this image to your environment where DLTK is hosted
be sure to take note of the checksum if you need to rebuild or decide to redeploy the container later. 

### Configuring the DLTK
The DLTK exists in the app directory in a folder named mltk-container to add custom configurations such as images to the app you need to create a new file named images.conf in the local directory. 

Below is an example images.conf configuration for the Rapids container:

```
[conda]
title = Conda Rapids
image = mltk-container-rapidsai
repo = anthonygtellez/
runtime = none,nvidia
```

Once the configuration file is in place restart Splunk and open the container management page of the DLTK:

![Container Overview](http://splunk.nyc3.digitaloceanspaces.com/dltk-containers.png)

The new image will now be available via the container image dropdown. Be aware the first time you launch the container might take some time, as the docker agent will need to download the image from DockerHub for deployment, subsequent deployments will be much quicker. 

Once the container is up and running you can access Jupyter notebook and make use of the newly installed libraries to develop and configure algorithms. 

![Jupyter Notebook](http://splunk.nyc3.digitaloceanspaces.com/jupyter-notebook.png)

Hopefully with this new framework customers are able to make use of the DLTK to extend their machine learning pipelines. 

For more background on the DLTK if you missed the session check out the .conf website:
FN1409 - Advances in Deep Learning with the MLTK Container for TensorFlow 2.0, PyTorch and Jupyter Notebooks
https://conf.splunk.com/watch/conf-online.html?search=%20Tellez%20Deep%20Learning#/
