# chtc-containers

CHTC (the Center for High Throughput Computing) (https://chtc.cs.wisc.edu/) is a computing resource with High Throughput Computing (HTC) and High Performance Computing (HPC) options available to UW-Madison researchers. Please visit their website for more information.
Most researchers in our department will have access to HTC (the default) if creating an account with them. 

Containers are used to install and run software on CHTC. 

# What you will need

To run a typical bioinformatics workflow on CHTC you will need:

1. A **definition** file: instructions to build a Apptainer container for software
2. A **sif** file: an Apptainer container containing the software
3. An **executable script**: a bash script with lines that CHTC will run
4. A **submit script**: a text file containing information about resources requested, and that will refer to the SIF file you build

You do NOT need to create a `.sif` image file everytime you run a job, you can reuse the same images over time.

# Instructions

## Installing software: creating a container for your bioinformatics software

### Create a container image using the `.def` file using these instructions: https://chtc.cs.wisc.edu/uw-research-computing/apptainer-htc.html

If you have previously used `conda` to run bioinformatics software, the process used to involve creating a conda environment in your `/home/username` folder, then packing the conda environment into a tar.gz file, and writing in the tar.gz file inside of the executable script (.sh) when runnign a job on CHTC.

The website anaconda.org is where we can search for software to install. `Bioconda` (https://anaconda.org/bioconda/repo) is the name of a conda channel that contains many bioinformatics software, ready to install (over 10,000 of them to be exact!). 
On CHTC, we need to use Containers, such as Docker, Apptainer/Singularity images to install and run any software.
One challenge is that the repository for pre-built installable container images (e.g. Docker: hub.docker.com) is much less than on conda.

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

The lines after `%post` are the `conda install` instructions that you would have typed into your terminal. Note that you do not need to write `conda create` or `conda activate` in this `.def` file.

--> On this GitHub repository, I have created `.def` files of common bioinformatics software that can be used to create `.sif` container images to be used within CHTC. You can use them to build your own .sif images, OR use the template above and replace what follows the `%post` line with your own tool you want to install from conda.

### Starting an interactive job to create a container:
Once you have a `.def` file, create a `build.sub` file, that will be used to start an interactive job `condor_submit -i build.sub`:
In the code below, change the line `transfer_input_files = image.def` to correspond to the name of your `image.def` file (e.g. `spades.def`, `fastqc.def`, etc.)

```
# build.sub
# For building an Apptainer container

universe = vanilla
log = build.log

# In the latest version of HTCondor on CHTC, interactive jobs require an executable.
# If you do not have an existing executable, use a generic linux command like hostname as shown below.
executable = /usr/bin/hostname

# If you have additional files in your /home directory that are required for your container, add them to the transfer_input_files line as a comma-separated list.
transfer_input_files = image.def

requirements = (HasCHTCStaging == true)

+IsBuildJob = true
request_cpus = 4
request_memory = 16GB
request_disk = 16GB

queue
```

Follow the instructions on : https://chtc.cs.wisc.edu/uw-research-computing/apptainer-htc.html#start-an-interactive-build-job

in summary the command are:
```
# Build the container using the instructions in the def file, write it to the file name of your choice with the extension .sif
apptainer build <container.sif> <container.def>
# Test the container:
apptainer shell -e <container.sif>
# the prompt will change to Apptainer>
# Check installation by typing -h (or other ways to access the program) next to the program name
fastqc -h
# Once you saw that it works, exit the container:
exit
# Move the .sif file to your staging folder:
mv <container.sif> /staging/netid/.
# exit the interactive Build job
exit
```

## Getting Ready to Submit your Actual Job

### Create a submit file for your job

This is a DIFFERENT `.sub` file than in the previous step. Here, the submit file contains the resource requested to run the actual computational job. 
You will likely need to set custom cpus, memory and disk space.

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

### Writing your executable script for CHTC (.sh)

In this script, include the usual shebang line (`#!/bin/bash`) followed by the code to run your software for example:

```
#!/bin/bash

fastqc -h

fastqc *.fastq
```

In this .sh file, you do no need to activate the conda environment anymore. 

### Submit your job
You can submit your job using `condor_submit <file.sub>` as usual, making sure that you are using the <file.sub> file that uses the .sif file in the container line.
If you are using `condor_submit` with the `-i` flag to interactively test your code, you will need to type `conda activate` (no need to specify any environment name)

# How to build a Apptainer container from an existing conda environment

1) First activate your environment, and export it as a yml file:
Let's say I have an environment called `checkm2`

```
conda activate $ENVNAME
conda env export > $ENVNAME.yml
conda deactivate
ls
```
If you were to open your yml file, it might look something like this: https://github.com/UW-Madison-Bacteriology-Bioinformatics/chtc-containers/blob/main/recipes/from_yml/checkm2.yml

2) Send the yml file to another computer, another laptop, or in this case, your CHTC home folder.
3) On CHTC, create a definition file (recipe file) that looks like this: https://github.com/UW-Madison-Bacteriology-Bioinformatics/chtc-containers/blob/main/recipes/from_yml/checkm2.def
   You can copy and paste this into CHTC, and replace the yml file for your own file name.
4) The steps to build your container using `condor_submit -i` are going to be very similar to what we've seen previously, but make sure to include the `yml` file in the `transfer_input_files` line like this:

```
   [ptran5@ap2002 apptainer_def]$ cat build.sub
# build.sub
# For building an Apptainer container

universe = vanilla
log = build.log

# In the latest version of HTCondor on CHTC, interactive jobs require an executable.
# If you do not have an existing executable, use a generic linux command like hostname as shown below.
executable = /usr/bin/hostname

# If you have additional files in your /home directory that are required for your container, add them to the transfer_input_files line as a comma-separated list.
transfer_input_files = checkm2.yml, checkm2.def


requirements = (HasCHTCStaging == true)

+IsBuildJob = true
request_cpus = 4
request_memory = 16GB
request_disk = 16GB

queue
```

5) From there, submit your condor job interactively and use the `Apptainer build` commands to build your container (.sif file)
   



