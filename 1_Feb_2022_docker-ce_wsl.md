# After Uninstalling Docker Desktop

Date: 1-Feb-2022

## Overview

Like some enterprise companies who did not take the Docker Desktop License grace period seriously, it would have hit with some impact, e.g my team. I had sent a reminder last November but 28th Jan I got a mail to uninstall Docker Desktop, as my company has not bought the enterprise license for the Docker Desktop installed on my Windows 10 office laptop. It hit hard at many levels, first we did not have any alternatives, the mail gave alternatives for mac and left out windows. There were many critical deliverable and despite Docker & Docker Desktop being an essential developer tool our company completely ignored the warning bells. I had to find a solution fast and a solution that would have minimum disruptions to our workflow.

## Quick Search

I did a quick search for alternatives. For Windows though there were not that many, I just found the below options:
- minikube with docker ce cli
- podman with docker compose on wsl
- vagrant, virtualbox, minikube or k3s/minikube

While there many be many other options in teh wild I had to try something and test at the earliest, and minikube seemed to be a mature project to try out.
Hence I tried downloading minikube and tried running native on Windows 10.

## Failed Attempt

The main reason for selecting minikube was because of its support for virtualbox and Hyper-v VM configuration apart from docker to run a single host Kubernetes cluster. Also it has a docker host process which could act as a container registry. My plan was to use minikube native with docker window cli provided in the below link.
https://github.com/StefanScherer/docker-cli-builder

For installation for minikube on windows I used Windows Package Manager as I had Admin right for my PC, you can choose to download the setup binaries as well.

```
winget install minikube
```

More details on minikube installation can be found [here](https://minikube.sigs.k8s.io/docs/start/)

The installation was uneventful, while running I ran it in an elevated powershell and setting up a minikube cluster with `minikube start`, as the default driver is Hyper-v I expected it to be relatively fast. But to my dismay first needed a Hyper V

```
C:\WINDOWS\system32>minikube start
* minikube v1.25.1 on Microsoft Windows 10 Home Single Language 10.0.19043 Build 19043
* Using the hyperv driver based on existing profile
* Starting control plane node minikube in cluster minikube
* Creating hyperv VM (CPUs=2, Memory=2200MB, Disk=20000MB) ...
! StartHost failed, but will try again: creating host: create: precreate: Hyper-V PowerShell Module is not available
* Creating hyperv VM (CPUs=2, Memory=2200MB, Disk=20000MB) ...
* Failed to start hyperv VM. Running "minikube delete" may fix it: creating host: create: precreate: Hyper-V PowerShell Module is not available

X Exiting due to PR_HYPERV_MODULE_NOT_INSTALLED: Failed to start host: creating host: create: precreate: Hyper-V PowerShell Module is not available
* Suggestion: Run: 'Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-Tools-All'
* Documentation: https://www.altaro.com/hyper-v/install-hyper-v-powershell-module/
* Related issue: https://github.com/kubernetes/minikube/issues/9040
```

After enabling the Hyper-V Module I got the below error

```
PS C:\windows\system32> minikube start
* minikube v1.25.1 on Microsoft Windows 10 Enterprise 10.0.18363 Build 18363
* Unable to pick a default driver. Here is what was considered, in preference order:
  - hyperv: Not healthy: C:\windows\System32\WindowsPowerShell\v1.0\powershell.exe -NoProfile -NonInteractive @([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole(([System.Security.Principal.SecurityIdentifier]::new("S-1-5-32-578"))) returned "False\r\n"
  - hyperv: Suggestion: Unable to determine current user's Hyper-V administrator privileges. <>
* Alternatively you could install one of these drivers:
  - docker: Not installed: exec: "docker": executable file not found in %PATH%
  - vmware: Not installed: exec: "docker-machine-driver-vmware": executable file not found in %PATH%
  - virtualbox: Not installed: unable to find VBoxManage in $PATH
  - podman: Not installed: exec: "podman": executable file not found in %PATH%

X Exiting due to DRV_NOT_HEALTHY: Found driver(s) but none were healthy. See above for suggestions how to fix installed drivers.
```

While I could have dug further, with so many failures it seemed a non starter for the quick reliable option that I wanted to give to my team.

## Docker in Wsl 2 to the rescue

After some Googling, I found the eureka moment which I was searching for, installing Docker Engine Ce (Community edition) on Wsl. 

I already had wsl 2 installed on my Windows 10 and thanks to Docker Desktop supporting running containers in wsl as back-end. More on how to install wsl is out of scope of this blog but you can find the simple installation option on official [wsl site](https://docs.microsoft.com/en-us/windows/wsl/install).

I was using Ubuntu 18.4 Linux in wsl which is a very popular version for installing Linux. It can we installed from Windows Store, more details can be found here, a bit more details installation guide is given [here](https://ubuntu.com/blog/ubuntu-on-wsl-2-is-generally-available).

But finally when installed you can verify the setup by running the following command.

```
C:\Users\dashp>wsl -l
Windows Subsystem for Linux Distributions:
Ubuntu-18.04 (Default)
```

The next step was to install Docker onto ubuntu which was much like the standard Docker engine installation process as given in this [link](https://docs.docker.com/engine/install/ubuntu/)

While the steps I give here is current, it  may change in future as Docker is working on Docker Desktop for Linux which may be closed source.

### Set up the package repository

```
sudo apt-get update

sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

I faced some errors but running below command fixed those issues

```
sudo apt-get update --fix-missing
```

### Adding GPG key & Install

The Docker GPG key was added by the below step for apt to validate the docker repositories.

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

And used the below command to point to the stable remote repository for docker binaries

```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```


Then ran the below command to install Docker

```
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

Both the above steps ran without any issues, the `update` ensured the local repositories had the latest list repositories from the docker remote repo for docker binaries & make docker installation a smooth process


Post the installation I tested if docker worked, but I got the below error

```
(base) priyab@LAPTOP-TSUCQBB3:/mnt/c/Users/dashp$ sudo docker run hello-world
docker: Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?.
See 'docker run --help'.
```

This was because docker server process had not started and since wsl has no systemd enabled, so docker service had to be manually started.

```
$ sudo service docker start
 * Starting Docker: docker                    [ OK ]
```

To check the status of the service we can run the below command.

```
$ sudo service docker status
 * Docker is running
```

After that I ran the docker `hello world` utility to check if docker is running fine.

```
$ sudo docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
2db29710123e: Pull complete
Digest: sha256:507ecde44b8eb741278274653120c2bf793b174c06ff4eaa672b713b3263477b
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

And here most of the blogs I read stopped, I needed to test it with a real world load to make sure if it was the suitable solution.

### Check Docker Compose

Since Docker compose was an important component of my workflow, I had to make sure it worked.

```
$ docker-compose

The command 'docker-compose' could not be found in this WSL 2 distro.
We recommend to activate the WSL integration in Docker Desktop settings.

For details about using Docker Desktop with WSL 2, visit:

https://docs.docker.com/go/wsl2/

```

Thankfully I had `miniconda` installed, installing latest version of docker-compose was just a `conda` call.

```
conda install -c conda-forge docker-compose
```


### Time for Real Test

Now it was time to test docker-compose integration with the docker installed. I had an earlier project to install Jupyer Data science notebook.
More details [here](https://jupyter-docker-stacks.readthedocs.io/en/latest/index.html).

```
$ docker-compose up -d
Creating datascience-notebook-container ... done
```

And voila it worked. Now it was time to see it was running

```
docker-compose ps
             Name                           Command              State                    Ports
-----------------------------------------------------------------------------------------------------------------
datascience-notebook-container   tini -g -- start-notebook.sh    Up      0.0.0.0:8080->8888/tcp,:::8080->8888/tcp
```

It was running find and pointing to 8080 port on host machine. Or so I thought

## Connection Issue & Solution

Generally when I start a docker-compose service in Docker Desktop it maps the ports to the host machine in my case my Laptop, but since I was running in wls2, unfortunately the way wsl is configured, the post mapping maps to 0.0.0.0:8080 port of wsl machine and which in turn is mapped to `localhost` or
`127.0.0.1:8080` on my laptop. Blame it on how Microsoft has implemented wsl 2 and this issue as a "feature", I had to make do with accessing the url with
`http://127.0.0.1:8080` in my web browser.

There is one solution to map the localhost port to 0.0.0.0 using `netsh` more details [here](https://stackoverflow.com/questions/61002681/connecting-to-wsl2-server-via-local-network).

We can do something as below:

```
netsh interface portproxy add v4tov4 listenport=<port-to-listen> listenaddress=0.0.0.0 connectport=<port-to-forward> connectaddress=<forward-to-this-IP-address>

e.g.
netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=3000 connectaddress=172.30.16.3
```

Where `172.30.16.3` probablyis the ip address given by the virtual network bridge assigned to wsl by Hyper-v. 

But as my need was to run and test my api development locally using `localhost`, the given solution was more than fine for me.


## Risks & Future

As already mentioned Docker Inc is working on Docker Desktop for Linux, given the way it has chosen to go with Docker Desktop from a free to a licensed solution and killing docker engine on Windows, its anyone's guess. While I am not arguing the merits of what Docker Inc is doing, but I feel my current work around is temporary.

I feel in long term a better solution can be either of two, but either it will need me or someone to build some smooth script or automation to make it work:

- podman + docker-compose in wsl
- minikube + podman in wsl
- minikube + kvm in wsl, though to be honest I am not sure of the performance implication
- vagrant/terraform + podman

While the developer container development work flow is disrupted for many, I am sure this is also an opportunity for competitors to come up and offer a good alternative to docker which may be better aligned to kubernetes and oci spec and still beginner and developer friendly.

While my larger adventure to replace docker continues, if you have some suggestions, ping me on twitter at [twitmyreview](https://twitter.com/twitmyreview). Or reach out to me on [linkedin](https://www.linkedin.com/in/priyab-dash-21616b15/).

Please do like share and comment.
