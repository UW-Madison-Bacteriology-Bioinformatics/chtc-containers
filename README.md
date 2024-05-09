# chtc-containers
CHTC-containers

Containers are used to install and run software on CHTC. On this page, I have created `.def` files that can be used to create `.sif` container images to be used within CHTC.

# What you will need

To run a workflow on CHTC you will need:

1. A **definition** file: instructions to build a Apptainer container for software
2. A **sif** file: an Apptainer container containing the software
3. An **executable script**: a bash script with lines that CHTC will run
4. A **submit script**: a text file containing information about resources requested, and that will refer to the SIF file you build

# Instructions

## Create a container image using the `.def` file using these instructions: https://chtc.cs.wisc.edu/uw-research-computing/apptainer-htc.html

If you have previously used `conda` to run bioinformatics software, the process used to involve creating a conda environment in your `/home/username` folder, then packing the conda environment into a tar.gz file, and writing in the tar.gz file inside of the executable script (.sh) when runnign a job on CHTC.

The website anaconda.org has lots of bioinformatics software (over 10,000 of them!), compared to pre-built images in hub.docker.com, so it is very convenient to install software this way.
However, on CHTC, we need to use containers to install software. 

> **_NOTE:_**  Conda: a way to install software along with all its dependencies, but is specific to different computer architectures (e.g. Mac, Windows, Linux). Container: A way to install softawre along with all its dependencies, IN ADDITION to the installation being built on a specific computer architecture.

Thankfully, it only takes a few steps to convert a conda environment into a container image, by creating a `.def` file of this format:
```
Bootstrap: docker
From: continuumio/miniconda3:latest

%post
        conda install -c conda-forge -c bioconda bowtie fastx_toolkit
```

The line `Bootstrap` tells you that this is a Docker image.
The `From` line is so that the container can be built by using miniconda. You don't have to install miniconda3 yourself, because an image exists here: https://hub.docker.com/r/continuumio/miniconda3
The lines after `%post` are the `conda install` instructions that you would have typed into your terminal.

## Create a submit file

Here is an example of the `sub` file to be used to actually use the container in a job on CHTC:

```
# apptainer.sub

# Provide HTCondor with the name of your .sif file and universe information
container_image = file:///staging/path/to/my-container.sif

executable = myExecutable.sh

# Include other files that need to be transferred here.
# transfer_input_files = other_job_files

log = job.log
error = job.err
output = job.out

requirements = (HasCHTCStaging == true)

# Make sure you request enough disk for the container image in addition to your other input files
request_cpus = 1
request_memory = 4GB
request_disk = 10GB      

queue
```

You will need to change the line `container_image` to correspond to the file path of the `.sif` file you created in step 1.

# Writing your executable script for CHTC (.sh)

In this script, include the usual shebang line (`#!/bin/bash`) followed by the code to run your software for example:

```
#!/bin/bash

fastqc -h

fastqc *.fastq
```

In this .sh file, you do no need to activate the conda environment anymore. 

# Submit your job
You can submit your job using `condor_submit <file.sub>` as usual, making sure that you are using the <file.sub> file that uses the .sif file in the container line.
If you are using `condor_submit` with the `-i` flag to interactively test your code, you will need to type `conda activate` (no need to specify any environment name)




