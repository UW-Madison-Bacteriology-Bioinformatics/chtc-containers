# Sharing containers with other researchers

Did you build some `.sif` containers image files that you want to share with other people at UW-Madison? 
Yes, you could email the `.sif` file to your colleagues, but this requires a lot of "having to know the right people".
Why not make general software tools available to the UW-Madison research community and beyond?
Assuming that you have these files on hand, this guide will show you how to push (publish) them on the UW-Madison GitLab Container registry, and how these container images can be used within CHTC `.sub` files.
All the steps in this guide are done on the Terminal, logged into CHTC, and on a web browser.

# Gap

This github page chtc-containers contains recipes to software (`.def`) files, but still requires each user to built their own `sif` file and have it in their `staging` folder.
The purpose of the guide is to document how `sif` files can be published online, such that they can be used directly in a HTCondor `.submit` file. This aims to make it easier for researcher to use software, while still using best practices (e.g. containers) to running them on CHTC.
Additionally, this links multiple UW-Madison cyberinfrastructure tools, such as the Uw-Madison GitLab registry instance, and CHTC resources. Hopefully, this will make it easier for researchers here to make the most of the resources available to them to enable their research!

# Prerequisites
1. A container image you want to share, in a `.sif` format, already located on CHTC
2. A CHTC account (https://chtc.cs.wisc.edu/uw-research-computing/account-details)
3. A Uw-Madison GitLab Account (https://kb.wisc.edu/shared-tools/page.php?id=121442)

# Step by Step
## Create a GitLab Repository, and enable container 

Login using your UW-Madison credentials, and click on New Project, Create Blank Project. Set the permissions to **public**

![Screenshot 2025-02-28 at 11 58 50 AM](https://github.com/user-attachments/assets/b5562ecd-afa2-4740-958c-9c3ae1aed0f1)

Navigate to your project, click on Deploy, and select Container Registry. Enable the container registry

![Screenshot 2025-02-28 at 11 59 41 AM](https://github.com/user-attachments/assets/7270bcfe-4fc3-4b30-89fe-428d9316f386)

## Log into CHTC and set up apptainer
```
ssh netid@ap2002.chtc.wisc.edu
# enter password
apptainer -h
```

Check remotes available:
```
apptainer remote list
```

You might only see this:
```
NAME           URI                     DEFAULT?  GLOBAL?  EXCLUSIVE?  SECURE?
DefaultRemote  cloud.apptainer.org               ✓                    ✓
```
We want to add the GitLab remote and URI to our list of remote. To do this, type:

```
apptainer remote add GitLab registry.doit.wisc.edu
```
To verify that it worked, type:
```
apptainer remote list
```

### Login into apptainer from the terminal:

Next, we want to use our netID credentials to log into the registry.doit.wisc.edu. Note that we do not need the specific path to our project here.
Type:
```
apptainer registry login --username yourNetID docker://registry.doit.wisc.edu
```
You might be prompted to enter a token at the address:
`
Generate an access token at https://registry.doit.wisc.edu/auth/tokens, and paste it here.
`

The web link might not work. To generate a token, leave your terminal window open, but open a new tab on your web browser. Link on your profile icon in the upper left part of the page, then click on User Settings > Access Token > New Token. 
Give it an informative name such as `chtc access point` so you know what machine this token is associated with.
Click the checkboxes to enable these settings
`read_repository, write_repository, read_registry, write_registry`

![Screenshot 2025-02-28 at 12 06 05 PM](https://github.com/user-attachments/assets/924be0cb-d5d6-4b8b-bb87-d4aa49153e77)

Copy and paste the token into the terminal.
You will not see anything on the screen, but press enter.

If successful, you will see:
```
[ptran5@ap2002 ~]$ apptainer registry login --username ptran5 docker://registry.doit.wisc.edu
Password / Token: 
INFO:    Token stored in /home/netid/.apptainer/remote.yaml
```

## Push the container to the registry

From your terminal, navigate to the folder with your containers. This is likely to be in your staging folder.
For example, I have a folder called `/staging/ptran5/apptainer`

To push a container to the registry, type:

```
apptainer push flye.sif oras://registry.doit.wisc.edu/ptran5/containers/flye:1.0
```

In this example, I am pushing a container image called `flye.sif` using the `oras` protocol, to the address registry.doit.wisc.edu/ptran5/containers/. This path corresponds to my `netid/projectname`.
the next part of the command is the image name and tag (`1.0`). You mush use the `oras` protocol : typing `docker://` will not work with apptainer.

If successful you will see:

```
182.8MiB / 182.8MiB [===============================================================================================================] 100 % 565.4 MiB/s 0s
INFO:    Upload complete
```

## Check you container online!
Go to the web address:
`https://git.doit.wisc.edu/yourusername/yourproject/container_registry`. Refresh the page, and you should see your container updated there!

## Use the container in a submit file

To use a container image from the GitLab registry, use the path in your `container_image` line:
```
container_image = docker://registry.doit.wisc.edu/ptran5/containers/flye:1.0
```


### Futher readings


### Future steps
The current container repo is associated with a user (e.g. ptran5), but I would like to make a group one (e.g. Bioinformatics-UW) such as a people (not just me) can contribute (push) files.


