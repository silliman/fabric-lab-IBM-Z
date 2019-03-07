Section 1:  Lab Overview
========================
In this lab, you will start with a basic Ubuntu 16.04.5 server instance running on an IBM z13 Server that resides in the IBM Washington Systems Center in Herndon, Virginia.  
During the installation process for this instance, no extra packages were selected for installation beyond the minimum offered by the installer.  
Ubuntu updates were applied such that at this time the level of Ubuntu is *16.04.5 LTS* and the Linux kernel is at level *44.4.0-135*.

You will install the necessary software prerequisites to build and test a Hyperledger Fabric network, including

*	Docker and Docker Compose
*	Node.js and npm
*	Golang compiler
*	other necessary packages

During the lab, you will download three Hyperledger Fabric source code repositories, build the necessary artifacts from the source code, and run two comprehensive end-to-end tests to verify that your Hyperledger Fabric installation is in working order.

You will first download the *fabric* repository, which contains the source code for the core functionality of Hyperledger Fabric.  
Using this source code, you will build the Docker images that comprise the core Hyperledger Fabric services. 
Then you will run an end-to-end test that utilizes the Hyperledger Fabric Command Line Interface (CLI) to verify that everything is working correctly.

Then you will download two more Hyperledger Fabric source code repositories, *fabric-ca* and *fabric-sdk-node*, and build from this source the artifacts necessary to run a second, more comprehensive end-to-end test of the Fabric Node.js SDK.

Section 2: Install prerequisite software needed by Hyperledger Fabric
=====================================================================

In this section, you will install a subset of the software prerequisites mentioned in the Lab Overview section-  the major software prerequisites that will be necessary to run the end-to-end command line interface (CLI) test in Section 3. 
You will install:

*	Docker
*	PIP, a python installer program (needed to install Docker Compose)
*	curl (used to download PIP) 
*	Docker Compose
*	Golang compiler

**Step 2.1:** Log in to your assigned Ubuntu 16.04.5 Linux on IBM Z instance using the PuTTY icon on your workstation using the IP address assigned to your team.  You will be greeted with a message like this::

 Welcome to Ubuntu 16.04.5 LTS (GNU/Linux 4.4.0-139-generic s390x)

  * Documentation:  https://help.ubuntu.com
  * Management:     https://landscape.canonical.com
  * Support:        https://ubuntu.com/advantage
 New release '18.04.2 LTS' available.
 Run 'do-release-upgrade' to upgrade to it.

 Last login: Thu Mar  7 11:24:35 2019 from 192.168.215.249
 bcuser@ubuntu16045:~$ 

**Step 2.2:** This system is a relatively “clean” Ubuntu 16.04.5 instance- “clean” in the sense that after this instance was created, no additional software was installed.  
You will therefore need to install several software prerequisites.  

The first thing you will install is Docker. 
Docker is a software container platform that is an integral part of Hyperledger Fabric.  
*Smart contracts*, also known as *chaincode*, run as Docker containers.  
In addition, you will run the Hyperledger Fabric processes themselves in Docker containers.  
You could choose to run the Hyperledger Fabric processes natively on your host operating system, that is, *not* in Docker 
containers, but even if you did this you would still need Docker to run chaincode.  
For this lab, you will use Docker containers for *both* the chaincode and the Hyperledger Fabric processes.  

Issue this *which* command, which attempts to find the *docker* executable. 
The fact that it gives no response other than returning to the command prompt indicates that the program *docker* is not found::

 bcuser@ubuntu16045:~$ which docker
 bcuser@ubuntu16045:~$ 

**Note:** Some Linux distributions produce no output if the executable is not found, as in the above example.  
Other Linux distributions may produce an error message stating that the executable is not found.
   
**Step 2.3:** You will be using root authority for several commands throughout this lab.  
You can tell when you have root authority by observing the command prompt-  it will end with a ‘#’ when you have root authority and it will end with a ‘$’ when you do not.  
Use the *sudo* command to switch to the root user.  
Observe the change in the command prompt after you enter this command::

 bcuser@ubuntu16045:~$ sudo su -
 root@ubuntu16045:~# 

**Step 2.4:** Ubuntu provides a popular software package manager named *apt*, which stands for *Advanced Package Tool*. 
You will be using *apt* throughout this lab to install various software packages. 
The first thing you need to do is install the Docker Community Edition (CE).  
This software is provided by Docker, so the next several steps will add a repository managed by Docker to your system’s list of repositories so that you can install Docker CE. 
Enter this command to update the *apt* package index::

 root@ubuntu16045:~# apt-get update
 Hit:1 http://us.ports.ubuntu.com/ubuntu-ports xenial InRelease
 Get:2 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates InRelease [109 kB]                    
 Get:3 http://us.ports.ubuntu.com/ubuntu-ports xenial-backports InRelease [107 kB]                           
 Get:4 http://ports.ubuntu.com/ubuntu-ports xenial-security InRelease [109 kB]    
 Get:5 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates/main s390x Packages [664 kB]
 Get:6 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates/main Translation-en [370 kB]   
 Get:7 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates/universe s390x Packages [605 kB]
 Get:8 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates/universe Translation-en [305 kB]
 Get:9 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates/multiverse s390x Packages [11.0 kB]
 Get:10 http://us.ports.ubuntu.com/ubuntu-ports xenial-backports/main s390x Packages [7280 B]
 Get:11 http://ports.ubuntu.com/ubuntu-ports xenial-security/main s390x Packages [426 kB]
 Get:12 http://ports.ubuntu.com/ubuntu-ports xenial-security/main Translation-en [256 kB]
 Get:13 http://ports.ubuntu.com/ubuntu-ports xenial-security/universe s390x Packages [336 kB]
 Get:14 http://ports.ubuntu.com/ubuntu-ports xenial-security/universe Translation-en [172 kB]
 Get:15 http://ports.ubuntu.com/ubuntu-ports xenial-security/multiverse s390x Packages [1716 B]
 Get:16 http://ports.ubuntu.com/ubuntu-ports xenial-security/multiverse Translation-en [2676 B]
 Fetched 3481 kB in 1s (3328 kB/s)   
 root@ubuntu16045:~#     
 
**Step 2.5:** Install packages to allow *apt* to use a repository over HTTPS::

 root@ubuntu16045:~# apt-get install -y apt-transport-https ca-certificates curl software-properties-common
 Reading package lists... Done
 Building dependency tree       
 Reading state information... Done
 The following additional packages will be installed:
   libcurl3-gnutls python3-pycurl python3-software-properties unattended-upgrades xz-utils
 Suggested packages:
   libcurl4-gnutls-dev python-pycurl-doc python3-pycurl-dbg bsd-mailx mail-transport-agent
 The following NEW packages will be installed:
   curl python3-pycurl python3-software-properties software-properties-common unattended-upgrades xz-utils
 The following packages will be upgraded:
   apt-transport-https ca-certificates libcurl3-gnutls
 3 upgraded, 6 newly installed, 0 to remove and 55 not upgraded.
 Need to get 684 kB of archives.
 After this operation, 1552 kB of additional disk space will be used.
 Get:1 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates/main s390x libcurl3-gnutls s390x 7.47.0-1ubuntu2.12 [175 kB]
 Get:2 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates/main s390x apt-transport-https s390x 1.2.29ubuntu0.1 [25.0 kB]
 Get:3 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates/main s390x ca-certificates all 20170717~16.04.2 [167 kB]
 Get:4 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates/main s390x curl s390x 7.47.0-1ubuntu2.12 [137 kB]
 Get:5 http://us.ports.ubuntu.com/ubuntu-ports xenial/main s390x python3-pycurl s390x 7.43.0-1ubuntu1 [39.9 kB]
 Get:6 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates/main s390x python3-software-properties all 0.96.20.8 [20.2 kB]
 Get:7 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates/main s390x software-properties-common all 0.96.20.8 [9440 B]
 Get:8 http://us.ports.ubuntu.com/ubuntu-ports xenial/main s390x xz-utils s390x 5.1.1alpha+20120614-2ubuntu2 [78.4 kB]
 Get:9 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates/main s390x unattended-upgrades all 0.90ubuntu0.10 [32.3 kB]
 Fetched 684 kB in 0s (3250 kB/s)               
 Preconfiguring packages ...
 (Reading database ... 64431 files and directories currently installed.)
 Preparing to unpack .../libcurl3-gnutls_7.47.0-1ubuntu2.12_s390x.deb ...
 Unpacking libcurl3-gnutls:s390x (7.47.0-1ubuntu2.12) over (7.47.0-1ubuntu2.11) ...
 Preparing to unpack .../apt-transport-https_1.2.29ubuntu0.1_s390x.deb ...
 Unpacking apt-transport-https (1.2.29ubuntu0.1) over (1.2.29) ...
 Preparing to unpack .../ca-certificates_20170717~16.04.2_all.deb ...
 Unpacking ca-certificates (20170717~16.04.2) over (20170717~16.04.1) ...
 Selecting previously unselected package curl.
 Preparing to unpack .../curl_7.47.0-1ubuntu2.12_s390x.deb ...
 Unpacking curl (7.47.0-1ubuntu2.12) ...
 Selecting previously unselected package python3-pycurl.
 Preparing to unpack .../python3-pycurl_7.43.0-1ubuntu1_s390x.deb ...
 Unpacking python3-pycurl (7.43.0-1ubuntu1) ...
 Selecting previously unselected package python3-software-properties.
 Preparing to unpack .../python3-software-properties_0.96.20.8_all.deb ...
 Unpacking python3-software-properties (0.96.20.8) ...
 Selecting previously unselected package software-properties-common.
 Preparing to unpack .../software-properties-common_0.96.20.8_all.deb ...
 Unpacking software-properties-common (0.96.20.8) ...
 Selecting previously unselected package xz-utils.
 Preparing to unpack .../xz-utils_5.1.1alpha+20120614-2ubuntu2_s390x.deb ...
 Unpacking xz-utils (5.1.1alpha+20120614-2ubuntu2) ...
 Selecting previously unselected package unattended-upgrades.
 Preparing to unpack .../unattended-upgrades_0.90ubuntu0.10_all.deb ...
 Unpacking unattended-upgrades (0.90ubuntu0.10) ...
 Processing triggers for libc-bin (2.23-0ubuntu10) ...
 Processing triggers for man-db (2.7.5-1) ...
 Processing triggers for dbus (1.10.6-1ubuntu3.3) ...
 Processing triggers for systemd (229-4ubuntu21.10) ...
 Processing triggers for ureadahead (0.100.0-19) ...
 Setting up libcurl3-gnutls:s390x (7.47.0-1ubuntu2.12) ...
 Setting up apt-transport-https (1.2.29ubuntu0.1) ...
 Setting up ca-certificates (20170717~16.04.2) ...
 Setting up curl (7.47.0-1ubuntu2.12) ...
 Setting up python3-pycurl (7.43.0-1ubuntu1) ...
 Setting up python3-software-properties (0.96.20.8) ...
 Setting up software-properties-common (0.96.20.8) ...
 Setting up xz-utils (5.1.1alpha+20120614-2ubuntu2) ...
 update-alternatives: using /usr/bin/xz to provide /usr/bin/lzma (lzma) in auto mode
 Setting up unattended-upgrades (0.90ubuntu0.10) ...

 Creating config file /etc/apt/apt.conf.d/50unattended-upgrades with new version
 Synchronizing state of unattended-upgrades.service with SysV init with /lib/systemd/systemd-sysv-install...
 Executing /lib/systemd/systemd-sysv-install enable unattended-upgrades
 Processing triggers for libc-bin (2.23-0ubuntu10) ...
 Processing triggers for ca-certificates (20170717~16.04.2) ...
 Updating certificates in /etc/ssl/certs...
 0 added, 0 removed; done.
 Running hooks in /etc/ca-certificates/update.d...
 done.
 Processing triggers for dbus (1.10.6-1ubuntu3.3) ...
 Processing triggers for systemd (229-4ubuntu21.10) ...
 Processing triggers for ureadahead (0.100.0-19) ...
 root@ubuntu16045:~# 

**Step 2.6:**  Add Docker’s official GPG key::

 root@ubuntu16045:~# curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
 OK
 root@ubuntu16045:~# 

**Step 2.7:** Verify that the key fingerprint is *9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88*::
 
 root@ubuntu16045:~# apt-key fingerprint 0EBFCD88
 pub   4096R/0EBFCD88 2017-02-22
       Key fingerprint = 9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
 uid                  Docker Release (CE deb) <docker@docker.com>
 sub   4096R/F273FCD8 2017-02-22

 root@ubuntu16045:~# 

**Step 2.8:** Enter the following command to add the *stable* repository that is provided by Docker::

 root@ubuntu16045:~# add-apt-repository "deb [arch=s390x] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
 root@ubuntu16045:~#

**Step 2.9:** Update the *apt* package index again:: 

 root@ubuntu16045:~# apt-get update
 Hit:1 http://us.ports.ubuntu.com/ubuntu-ports xenial InRelease
 Hit:2 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates InRelease                             
 Hit:3 http://us.ports.ubuntu.com/ubuntu-ports xenial-backports InRelease                           
 Hit:4 http://ports.ubuntu.com/ubuntu-ports xenial-security InRelease                               
 Get:5 https://download.docker.com/linux/ubuntu xenial InRelease [66.2 kB]
 Get:6 https://download.docker.com/linux/ubuntu xenial/stable s390x Packages [3880 B]
 Fetched 70.1 kB in 0s (200 kB/s)    
 Reading package lists... Done
 root@ubuntu16045:~# )     

**Step 2.10:** Enter this command to show some information about the Docker package.  This command won’t actually install anything::
 
 root@ubuntu16045:~# apt-cache policy docker-ce
 docker-ce:
   Installed: (none)
   Candidate: 18.06.3~ce~3-0~ubuntu
   Version table:
      18.06.3~ce~3-0~ubuntu 500
         500 https://download.docker.com/linux/ubuntu xenial/stable s390x Packages
      18.06.2~ce~3-0~ubuntu 500
         500 https://download.docker.com/linux/ubuntu xenial/stable s390x Packages
      18.06.1~ce~3-0~ubuntu 500
         500 https://download.docker.com/linux/ubuntu xenial/stable s390x Packages
      18.06.0~ce~3-0~ubuntu 500
         500 https://download.docker.com/linux/ubuntu xenial/stable s390x Packages
      18.03.1~ce-0~ubuntu 500
         500 https://download.docker.com/linux/ubuntu xenial/stable s390x Packages
      18.03.0~ce-0~ubuntu 500
         500 https://download.docker.com/linux/ubuntu xenial/stable s390x Packages
      17.12.1~ce-0~ubuntu 500
         500 https://download.docker.com/linux/ubuntu xenial/stable s390x Packages
      17.12.0~ce-0~ubuntu 500
         500 https://download.docker.com/linux/ubuntu xenial/stable s390x Packages
      17.09.1~ce-0~ubuntu 500
         500 https://download.docker.com/linux/ubuntu xenial/stable s390x Packages
      17.09.0~ce-0~ubuntu 500
         500 https://download.docker.com/linux/ubuntu xenial/stable s390x Packages
      17.06.2~ce-0~ubuntu 500
         500 https://download.docker.com/linux/ubuntu xenial/stable s390x Packages
      17.06.1~ce-0~ubuntu 500
         500 https://download.docker.com/linux/ubuntu xenial/stable s390x Packages
      17.06.0~ce-0~ubuntu 500
         500 https://download.docker.com/linux/ubuntu xenial/stable s390x Packages
 root@ubuntu16045:~# 

Some key takeaways from the command output:

*	Docker is not currently installed *(Installed: (none))*
*	*18.06.3~ce~3-0~ubuntu* is the candidate version to install- it is the latest version available
*	When you install the software, you will be going out to the Internet to the *download.docker.com* domain to get the software.

**Step 2.11:** Enter this *apt-get* command to install the candidate version.  (Enter Y when prompted to continue)::

 root@ubuntu16045:~# apt-get install docker-ce=18.06.3~ce~3-0~ubuntu
 Reading package lists... Done
 Building dependency tree       
 Reading state information... Done
 The following additional packages will be installed:
   aufs-tools cgroupfs-mount git git-man liberror-perl libltdl7 patch pigz
 Suggested packages:
   mountall git-daemon-run | git-daemon-sysvinit git-doc git-el git-email git-gui gitk gitweb git-arch git-cvs git-mediawiki
   git-svn diffutils-doc
 The following NEW packages will be installed:
   aufs-tools cgroupfs-mount docker-ce git git-man liberror-perl libltdl7 patch pigz
 0 upgraded, 9 newly installed, 0 to remove and 55 not upgraded.
 Need to get 33.8 MB of archives.
 After this operation, 202 MB of additional disk space will be used.
 Do you want to continue? [Y/n] Y
 Get:1 http://us.ports.ubuntu.com/ubuntu-ports xenial/universe s390x pigz s390x 2.3.1-2 [56.4 kB]
 Get:2 http://us.ports.ubuntu.com/ubuntu-ports xenial/universe s390x aufs-tools s390x 1:3.2+20130722-1.1ubuntu1 [92.8 kB]
 Get:3 http://us.ports.ubuntu.com/ubuntu-ports xenial/universe s390x cgroupfs-mount all 1.2 [4970 B]
 Get:4 http://us.ports.ubuntu.com/ubuntu-ports xenial/main s390x libltdl7 s390x 2.4.6-0.1 [37.6 kB]
 Get:5 http://us.ports.ubuntu.com/ubuntu-ports xenial/main s390x liberror-perl all 0.17-1.2 [19.6 kB]
 Get:6 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates/main s390x git-man all 1:2.7.4-0ubuntu1.6 [736 kB]
 Get:7 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates/main s390x git s390x 1:2.7.4-0ubuntu1.6 [3008 kB]
 Get:8 https://download.docker.com/linux/ubuntu xenial/stable s390x docker-ce s390x 18.06.3~ce~3-0~ubuntu [29.8 MB]
 Get:9 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates/main s390x patch s390x 2.7.5-1ubuntu0.16.04.1 [92.5 kB]
 Fetched 33.8 MB in 1s (20.7 MB/s)                           
 Selecting previously unselected package pigz.
 (Reading database ... 64536 files and directories currently installed.)
 Preparing to unpack .../pigz_2.3.1-2_s390x.deb ...
 Unpacking pigz (2.3.1-2) ...
 Selecting previously unselected package aufs-tools.
 Preparing to unpack .../aufs-tools_1%3a3.2+20130722-1.1ubuntu1_s390x.deb ...
 Unpacking aufs-tools (1:3.2+20130722-1.1ubuntu1) ...
 Selecting previously unselected package cgroupfs-mount.
 Preparing to unpack .../cgroupfs-mount_1.2_all.deb ...
 Unpacking cgroupfs-mount (1.2) ...
 Selecting previously unselected package libltdl7:s390x.
 Preparing to unpack .../libltdl7_2.4.6-0.1_s390x.deb ...
 Unpacking libltdl7:s390x (2.4.6-0.1) ...
 Selecting previously unselected package docker-ce.
 Preparing to unpack .../docker-ce_18.06.3~ce~3-0~ubuntu_s390x.deb ...
 Unpacking docker-ce (18.06.3~ce~3-0~ubuntu) ...
 Selecting previously unselected package liberror-perl.
 Preparing to unpack .../liberror-perl_0.17-1.2_all.deb ...
 Unpacking liberror-perl (0.17-1.2) ...
 Selecting previously unselected package git-man.
 Preparing to unpack .../git-man_1%3a2.7.4-0ubuntu1.6_all.deb ...
 Unpacking git-man (1:2.7.4-0ubuntu1.6) ...
 Selecting previously unselected package git.
 Preparing to unpack .../git_1%3a2.7.4-0ubuntu1.6_s390x.deb ...
 Unpacking git (1:2.7.4-0ubuntu1.6) ...
 Selecting previously unselected package patch.
 Preparing to unpack .../patch_2.7.5-1ubuntu0.16.04.1_s390x.deb ...
 Unpacking patch (2.7.5-1ubuntu0.16.04.1) ...
 Processing triggers for man-db (2.7.5-1) ...
 Processing triggers for libc-bin (2.23-0ubuntu10) ...
 Processing triggers for ureadahead (0.100.0-19) ...
 Processing triggers for systemd (229-4ubuntu21.10) ...
 Setting up pigz (2.3.1-2) ...
 Setting up aufs-tools (1:3.2+20130722-1.1ubuntu1) ...
 Setting up cgroupfs-mount (1.2) ...
 Setting up libltdl7:s390x (2.4.6-0.1) ...
 Setting up docker-ce (18.06.3~ce~3-0~ubuntu) ...
 Setting up liberror-perl (0.17-1.2) ...
 Setting up git-man (1:2.7.4-0ubuntu1.6) ...
 Setting up git (1:2.7.4-0ubuntu1.6) ...
 Setting up patch (2.7.5-1ubuntu0.16.04.1) ...
 Processing triggers for libc-bin (2.23-0ubuntu10) ...
 Processing triggers for systemd (229-4ubuntu21.10) ...
 Processing triggers for ureadahead (0.100.0-19) ...

Observe that not only was Docker installed, but so were its prerequisites that were not already installed.

**NOTE:** The *=18.06.3~ce~3-0~ubuntu* part of the above install command was not necessary in this case because this is the candidate version that would have been installed by default.  But I wanted to show you the syntax you would use if for some reason you wanted to install a version that was not the candidate version.

**Step 2.12:** Issue the *which* command again and this time it will tell you where it found the just-installed docker program::

 root@ubuntu16045:~# which docker
 /usr/bin/docker

**Step 2.13:** Enter the *docker version* command and you should see that version *17.06.2-ce* was installed::

 root@ubuntu16045:~# docker version
 Client:
  Version:           18.06.3-ce
  API version:       1.38
  Go version:        go1.10.3
  Git commit:        d7080c1
  Built:             Wed Feb 20 02:27:09 2019
  OS/Arch:           linux/s390x
  Experimental:      false

 Server:
  Engine:
   Version:          18.06.3-ce
   API version:      1.38 (minimum version 1.12)
   Go version:       go1.10.3
   Git commit:       d7080c1
   Built:            Wed Feb 20 02:26:03 2019
   OS/Arch:          linux/s390x
   Experimental:     false

**Step 2.14:** Enter *docker info* to see even more information about your Docker environment::

 root@ubuntu16045:~# docker info
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 0
 Server Version: 18.06.3-ce
 Storage Driver: overlay2
  Backing Filesystem: extfs
  Supports d_type: true
  Native Overlay Diff: true
 Logging Driver: json-file
 Cgroup Driver: cgroupfs
 Plugins:
  Volume: local
  Network: bridge host macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file logentries splunk syslog
 Swarm: inactive
 Runtimes: runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: 468a545b9edcd5932818eb9de8e72413e616e86e
 runc version: a592beb5bc4c4092b1b1bac971afed27687340c5
 init version: fec3683
 Security Options:
  apparmor
  seccomp
   Profile: default
 Kernel Version: 4.4.0-139-generic
 Operating System: Ubuntu 16.04.5 LTS
 OSType: linux
 Architecture: s390x
 CPUs: 2
 Total Memory: 3.733GiB
 Name: ubuntu16045
 ID: 5PYQ:47LO:VPA6:6PFY:XXF4:PFOK:2APP:K27S:VMFQ:F2UB:2RI5:7W6K
 Docker Root Dir: /var/lib/docker
 Debug Mode (client): false
 Debug Mode (server): false
 Registry: https://index.docker.io/v1/
 Labels:
 Experimental: false
 Insecure Registries:
  127.0.0.0/8
 Live Restore Enabled: false

 WARNING: No swap limit support


**Step 2.15:** After the Docker installation, non-root users cannot run Docker commands. One way to get around this for a non-root userid is to add that userid to a group named *docker*.  Enter this command to 
add the *bcuser* userid to the group *docker*::

 root@ubuntu16045:~# usermod -aG docker bcuser
 
**Note:** This method of authorizing a non-root userid to enter Docker commands, while suitable for a controlled sandbox environment, may not be suitable for a production environemnt due to security considerations. 

**Step 2.16:** Exit so that you are no longer running as root::

 root@ubuntu16045:~# exit
 logout
 bcuser@ubuntu16045:~$
 
**Step 2.17:** Even though *bcuser* was just added to the *docker* group, you will have to log out and then log back in again for this change to take effect.  
To prove this, before you log out, enter the *docker info* command and you will receive a permissions error::

 bcuser@ubuntu16045:~$ docker info
 Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.38/info: dial unix /var/run/docker.sock: connect: permission denied

**Step 2.18:** Now log out::

 bcuser@ubuntu16045:~$ exit
 logout
 Connection to 192.168.22.118 closed.

**Step 2.19:** Log in again.  (These instructions show logging in again using *ssh* from a command shell.  If you are using PuTTY you may need to start a new PuTTY session and log in)::

 $ ssh bcuser@192.168.22.108
 Welcome to Ubuntu 16.04.5 LTS (GNU/Linux 4.4.0-139-generic s390x)

  * Documentation:  https://help.ubuntu.com
  * Management:     https://landscape.canonical.com
  * Support:        https://ubuntu.com/advantage
 Last login: Wed Jan 30 14:46:41 2019 from 192.168.215.31
 bcuser@ubuntu16045:~$ 

**Step 2.20:** Now try *docker info* and this time it should work from your non-root userid::

 bcuser@ubuntu16045:~$ docker info
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 0
 Server Version: 18.06.3-ce
 Storage Driver: overlay2
  Backing Filesystem: extfs
  Supports d_type: true
  Native Overlay Diff: true
 Logging Driver: json-file
 Cgroup Driver: cgroupfs
 Plugins:
  Volume: local
  Network: bridge host macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file logentries splunk syslog
 Swarm: inactive
 Runtimes: runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: 468a545b9edcd5932818eb9de8e72413e616e86e
 runc version: a592beb5bc4c4092b1b1bac971afed27687340c5
 init version: fec3683
 Security Options:
  apparmor
  seccomp
   Profile: default
 Kernel Version: 4.4.0-139-generic
 Operating System: Ubuntu 16.04.5 LTS
 OSType: linux
 Architecture: s390x
 CPUs: 2
 Total Memory: 3.733GiB
 Name: ubuntu16045
 ID: 5PYQ:47LO:VPA6:6PFY:XXF4:PFOK:2APP:K27S:VMFQ:F2UB:2RI5:7W6K
 Docker Root Dir: /var/lib/docker
 Debug Mode (client): false
 Debug Mode (server): false
 Registry: https://index.docker.io/v1/
 Labels:
 Experimental: false
 Insecure Registries:
  127.0.0.0/8
 Live Restore Enabled: false

 WARNING: No swap limit support

**Step 2.21:** You will need to get right back in as root to install *Docker Compose*.  
Docker Compose is a tool provided by Docker to help make it easier to run an application that consists of multiple Docker containers.  
On some platforms, it is installed along with the Docker package but on Linux on IBM Z it is installed separately.  
It is written in Python and you will install it with a tool called Pip.  
But first you will install Pip itself!  
You will do this as root, so enter this again::

 bcuser@ubuntu16045:~$ sudo su -
 root@ubuntu16045:~#

**Step 2.22:** Install the *python-pip* package which will provide a tool named *Pip* which is used to install Python packages from a public repository::

 root@ubuntu16045:~# apt-get -y install python-pip

This will bring in a lot of prerequisites and will produce a lot of output which is not shown here.

**Step 2.23:** Run this command just to verify that *docker-compose* is not currently available on the system::

 root@ubuntu16045:~# which docker-compose
 root@ubuntu16045:~# 

**Step 2.24:** Use Pip to install Docker Compose::

 root@ubuntu16045:~# pip install docker-compose
 
You can ignore the suggestion at the end of the output of this command to consider upgrading *pip*- that isn't necessary for this lab.

**Step 2.25:** There was a bunch of output from the prior step I didn’t show, but if your install works, you should feel pretty good about the output from this command::

 root@ubuntu16045:~# docker-compose --version
 docker-compose version 1.23.2, build 1110ad0

**Note:** If the version of Docker Compose shown in your output differs from what is shown here, that's okay, as long as it is at least 1.13 or higher.

**Step 2.26:** Leave root behind and become a normal user again::

 root@ubuntu16045:~# exit
 logout
 bcuser@ubuntu16045:~$

**Step 2.27:** You won’t have to log out and log back in, like you did with Docker, in order to use Docker Compose, and to prove it, check for the version again now that you are no longer root::

 bcuser@ubuntu16045:~$ docker-compose --version
 docker-compose version 1.23.2, build 1110ad0

**Step 2.28:** The next thing you are going to install is the *Golang* programming language. 
You are going to install Golang version 1.11.1.  
Go to the /tmp directory::

 bcuser@ubuntu16045:~$ cd /tmp
 bcuser@ubuntu16045:/tmp$

**Step 2.29:** Use *wget* to get the compressed file that contains the Golang compiler and tools.  
And now is a good time to tell you that from here on out I will just call Golang what everybody else usually calls it-  *Go*.  Go figure.
::
 bcuser@ubuntu16044:/tmp$ wget --no-check-certificate https://storage.googleapis.com/golang/go1.11.1.linux-s390x.tar.gz
 --2019-03-07 12:28:05--  https://storage.googleapis.com/golang/go1.11.1.linux-s390x.tar.gz
 Resolving storage.googleapis.com (storage.googleapis.com)... 172.217.164.144, 2607:f8b0:4004:814::2010
 Connecting to storage.googleapis.com (storage.googleapis.com)|172.217.164.144|:443... connected.
 HTTP request sent, awaiting response... 200 OK
 Length: 100474359 (96M) [application/octet-stream]
 Saving to: 'go1.11.1.linux-s390x.tar.gz'

 go1.11.1.linux-s390x.tar.gz      100%[=======================================================>]  95.82M  79.7MB/s    in 1.2s    

 2019-03-07 12:28:07 (79.7 MB/s) - 'go1.11.1.linux-s390x.tar.gz' saved [100474359/100474359]

**Step 2.30:** Enter the following command which will extract the files into the /tmp directory, and provide lots and lots of output.
(It’s the *‘v’* in *-xvf* which got all chatty, or *verbose*, on you)::

 bcuser@ubuntu16045:/tmp$ tar -xvf go1.11.1.linux-s390x.tar.gz 
   .
   .  (output not shown here)
   .

**Step 2.31:** You will move the extracted stuff, which is all under */tmp/go*, into */opt*, and for that you will need root authority.
Whereas before you were instructed to enter *sudo su* – which effectively logged you in as root until you exited, you can issue a single command with *sudo* which executes it as root and then returns control back to you in non-root mode.   
Enter this command::

 bcuser@ubuntu16045:/tmp$ sudo mv -iv go /opt
 'go' -> '/opt/go'

**Step 2.32:** You need to set a couple of Go-related environment variables.  First check to verify that they are not set already::

 bcuser@ubuntu16045:/tmp$ env | grep GO

That command, *grep*, is looking for any lines of input that contain the characters *GO*.  
Its input is the output of the previous *env*command, which prints all of your environment variables. 
Right now you should not see any output.

**Step 2.33:**  You will set these values now.  You will make these changes in a special hidden file named *.bashrc* in your home 
directory.  Change to your home directory::

 bcuser@ubuntu16045:/tmp$ cd ~  # that is a tilde ~ character I know it is hard to see 
 bcuser@ubuntu16045:~$

**Step 2.34:** Enter the *cp* command to make a backup copy of *.bashrc* to allow a recovery in the infinitesimally slim chance that you make a mistake in the subsequent five steps which will append information to *.bashrc*.  
I know you wouldn't ever make a mistake, but the joker sitting next to you will.  
Trust me.::

 bcuser@ubuntu16045:~$ cp -ipv .bashrc .bashrc_orig
 '.bashrc' -> '.bashrc_orig'

**Step 2.35:** The next five steps- *Steps 2.35 through 2.39* - are each *echo* commands which will append to the end of *.bashrc*.  
The first and last of these steps just add a blank line for readability.  
Enter these exactly as shown in each step.  
It is critical that you use two ‘greater-than’ signs, i.e., ‘>>’, when you enter them.  
This appends the arguments of the *echo* commands to the end of the *.bashrc* file.  
If you only enter one ‘>’ sign, you will overwrite the file’s contents.  
I’d rather you not do that. 
Although *Step 2.34* does create a backup copy of the file,just in case.  
So first, add a blank line::

 bcuser@ubuntu16045:~$ echo '' >> .bashrc   # that is two single quotes, not one double-quote
 
**Step 2.36:** Add this line to set your *GOPATH* environment variable::

 bcuser@ubuntu16045:~$ echo export GOPATH=/home/bcuser/git >> .bashrc
 
**Step 2.37:** Add this line to set your *GOROOT* environment variable::

 bcuser@ubuntu16045:~$ echo export GOROOT=/opt/go >> .bashrc
 
**Step 2.38:** Add this line to update your *PATH* environment variable::

 bcuser@ubuntu16045:~$ echo export PATH=/opt/go/bin:/home/bcuser/bin:\$PATH >> .bashrc
 
**Step 2.39:** Finally, add another blank line for readability::

 bcuser@ubuntu16045:~$ echo '' >> .bashrc  

**Step 2.40:** Let’s see how you did.  Enter this command::

 bcuser@ubuntu16045:~$ head .bashrc
 # ~/.bashrc: executed by bash(1) for non-login shells.
 # see /usr/share/doc/bash/examples/startup-files (in the package bash-doc)
 # for examples 

 # If not running interactively, don't do anything
 case $- in
     *i*) ;;
       *) return;;
 esac

If your output looked like the above, congratulations, you did not stomp all over your file. 
*head* prints the top of the file.  
Had you made and mistake and used a single '>' instead two ‘>>’ like I told you, you would have whacked this stuff.  Y
our stuff is at the bottom.  
If *head* prints the top of the file, guess what command prints the bottom of the file.

**Step 2.41:** Try this::

 bcuser@ubuntu16045:~$ tail -5 .bashrc
 
 export GOPATH=/home/bcuser/git
 export GOROOT=/opt/go
 export PATH=/opt/go/bin:$PATH

**Step 2.42:** These changes will take effect next time you log in, but you can make them take effect immediately by entering this::

 bcuser@ubuntu16045:~$ source .bashrc

**Step 2.43:** Try this to see if your changes took::

 bcuser@ubuntu16045:~$ env | grep GO
 GOROOT=/opt/go
 GOPATH=/home/bcuser/git

**Step 2.44:**  Then try this::

 bcuser@ubuntu16045:~$ go version
go version go1.11.1 linux/s390x

**Recap:** Before you move on, here is a summary of the major tasks you performed in this section.

*	You installed Docker and added *bcuser* to the *docker* group so that *bcuser* can issue Docker commands
*	You installed Docker Compose (and Pip, which was needed to install it)
*	You installed Go
*	You updated your *.bashrc* profile to make necessary environment changes

In the next section, you will download the Hyperledger Fabric source code, build it, and run a comprehensive verification test using the Hyperledger Fabric Command Line Interface, or CLI.
 
Section 3: Download, build and test the Hyperledger Fabric CLI
==============================================================

In this section, you will:

* Download the source code repository containing the core Hyperledger Fabric functionality
* Use the source code to build Docker images that contain the core Hyperledger Fabric functionality
* Use the source code to build native executable binaries that provide Hyperledger Fabric utilities and that also provide an alternative to using Docker images.
* Download a repository containing Hyperledger Fabric samples
* Test for success by running one of the samples called "Build Your First Network", which will provide a test of core Hyperledger Fabric functionality.

**Step 3.1:** There are some software packages necessary to be able to successfully build the Hyperledger Fabric source code.  Install them with this command. 
Observe the output, not shown here, to see the different packages installed::

 bcuser@ubuntu16045:~$ sudo apt-get install -y build-essential libltdl3-dev
 
**Step 3.2:** Create the following directory path with this command.  
Make sure you are in your home directory when you enter it. 
If you are following these steps exactly, you already are.  
If you strayed away from your home directory, I'm assuming you're smart enough to get back there. 
(Or see *Step 2.33* if you accidentally left home and are too embarrassed to ask for help)::

 bcuser@ubuntu16045:~$ mkdir -p git/src/github.com/hyperledger
 bcuser@ubuntu16045:~$
 
**Step 3.3:** Navigate to the directory you just created::

 bcuser@ubuntu16045:~$ cd git/src/github.com/hyperledger/
 bcuser@ubuntu16045:~/git/src/github.com/hyperledger$
 
**Step 3.4:** Use the software tool *git* to download the source code of the Hyperledger Fabric core package from the official place where it lives.  
The *-b v1.4.0* argument specifies that you want Hyperledger Fabric v1.4.0, the most recent release at the time of this writing::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger$ git clone -b v1.4.0 https://gerrit.hyperledger.org/r/fabric
 Cloning into 'fabric'...
 remote: Counting objects: 5893, done
 remote: Finding sources: 100% (19/19)
 remote: Total 94844 (delta 0), reused 94831 (delta 0)
 Receiving objects: 100% (94844/94844), 114.78 MiB | 28.56 MiB/s, done.
 Resolving deltas: 100% (41532/41532), done.
 Checking connectivity... done.
 Note: checking out 'd700b43476e803c864c48021e63a78543b60e17e'.

 You are in 'detached HEAD' state. You can look around, make experimental
 changes and commit them, and you can discard any commits you make in this
 state without impacting any branches by performing another checkout.

 If you want to create a new branch to retain commits you create, you may
 do so (now or later) by using -b with the checkout command again. Example:

   git checkout -b <new-branch-name>


*Note:* The numbers in the various output messages may differ from what you see listed here, and this may be the case for any other times you do a *git clone* in the remainder of these labs.

**Step 3.5:** Switch to the *fabric* directory, which is the top-level directory of where the *git* command put the code it just downloaded::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger$ cd fabric
 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric$

**Step 3.6:** You will use a program called *make*, which is used to build software projects, in order to build Docker images for Hyperledger Fabric.  
But first, run this command to show that your system does not currently have any Docker images stored on it.  
The only output you will see is the column headings::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric$ docker images
 REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE

**Step 3.7:** That will change in a few minutes.  
Enter the following command, which will build the Hyperledger Fabric images.  
You can ‘wrap’ the *make* command, which is what will do all the work, in a *time* command, which will give you a measure of the time, including ‘wall clock’ time, required to build the images 
(See how it took several minutes on my system.  
It will probably take you a similar amount of time, so either check your email, fiddle with your smartphone, watch the output scroll by, or go to the bathroom really really quick)::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric$ time make docker release
   .
   .  (output not shown here)
   .
 real	4m4.679s
 user	0m13.729s
 sys	0m1.255s
 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric$ 

The *docker* argument to the *make* command will make several Docker images for you.
Docker images are often based on other, pre-existing Docker images, and you will observe that some Docker images
are downloaded for you from Hyperledger's public Docker Hub account.
This is expected behavior.

The *release* argument to the *make* command will compile Hyperledger Fabric binary executable files that can be run natively on Linux, outside of a Docker container.  
You will use some of those programs later in this lab so I am having you build them now.

**Step 3.8:** Run *docker images* again and you will see several Docker images that were just created. 
You will notice that many of the Docker images at the top of the output were created in the last few minutes.  
These were created by the *make docker* command.  
The Docker images that are several days or weeks or months old were downloaded from the Hyperledger Fabric's public 
DockerHub repository.  
Your output should look similar to that shown here, although the tags will be different if your instructor gave you a different level to checkout, and your *image ids* will be different either way, for those images that were created in the last few minutes::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric$ docker images
 REPOSITORY                     TAG                 IMAGE ID            CREATED              SIZE
 hyperledger/fabric-tools       latest                         d08700aec890        About a minute ago   1.52GB
 hyperledger/fabric-tools       s390x-1.4.0-snapshot-d700b43   d08700aec890        About a minute ago   1.52GB
 hyperledger/fabric-tools       s390x-latest                   d08700aec890        About a minute ago   1.52GB
 <none>                         <none>                         fbbfa64f6939        About a minute ago   1.68GB
 hyperledger/fabric-buildenv    latest                         c869d419df0e        2 minutes ago        1.47GB
 hyperledger/fabric-buildenv    s390x-1.4.0-snapshot-d700b43   c869d419df0e        2 minutes ago        1.47GB
 hyperledger/fabric-buildenv    s390x-latest                   c869d419df0e        2 minutes ago        1.47GB
 hyperledger/fabric-ccenv       latest                         609d68d7d5cb        3 minutes ago        1.41GB
 hyperledger/fabric-ccenv       s390x-1.4.0-snapshot-d700b43   609d68d7d5cb        3 minutes ago        1.41GB
 hyperledger/fabric-ccenv       s390x-latest                   609d68d7d5cb        3 minutes ago        1.41GB
 hyperledger/fabric-orderer     latest                         3943ef06d94c        3 minutes ago        147MB
 hyperledger/fabric-orderer     s390x-1.4.0-snapshot-d700b43   3943ef06d94c        3 minutes ago        147MB
 hyperledger/fabric-orderer     s390x-latest                   3943ef06d94c        3 minutes ago        147MB
 hyperledger/fabric-peer        latest                         cf05de3480f5        4 minutes ago        153MB
 hyperledger/fabric-peer        s390x-1.4.0-snapshot-d700b43   cf05de3480f5        4 minutes ago        153MB
 hyperledger/fabric-peer        s390x-latest                   cf05de3480f5        4 minutes ago        153MB
 hyperledger/fabric-baseimage   s390x-0.4.14                   6e4e09df1428        4 months ago         1.38GB
 hyperledger/fabric-baseos      s390x-0.4.14                   4834a1e3ce1c        4 months ago         120MB

**Step 3.9:** Navigate to up one directory level::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric$ cd ..
 bcuser@ubuntu16045:~/git/src/github.com/hyperledger$

**Step 3.10:** Download the *fabric-samples* repository, which, if I told you it contains Hyperledger Fabric samples, would you believe me?::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger$ git clone -b v1.4.0 --depth 1 https://gerrit.hyperledger.org/r/fabric-samples
 Cloning into 'fabric-samples'...
 remote: Counting objects: 570, done
 remote: Finding sources: 100% (570/570)
 remote: Total 570 (delta 49), reused 498 (delta 49)
 Receiving objects: 100% (570/570), 382.69 KiB | 0 bytes/s, done.
 Resolving deltas: 100% (49/49), done.
 Checking connectivity... done.
 Note: checking out 'bb39b6ed0947aa6f5d23d64b8ad65b1a64384bbb'.

 You are in 'detached HEAD' state. You can look around, make experimental
 changes and commit them, and you can discard any commits you make in this
 state without impacting any branches by performing another checkout.

 If you want to create a new branch to retain commits you create, you may
 do so (now or later) by using -b with the checkout command again. Example:

   git checkout -b <new-branch-name>

**Step 3.11:** Navigate to the *fabric-samples* directory which contains the repository you just cloned::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger$ cd fabric-samples/
 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-samples$ 

**Step 3.12:** Back in *Step 3.7* the *release* argument to the *make* command created some binary executables for you.
THh next few steps will copy these files from where the *make* command put them into the directory where the sample you are about to run expects to find them.
Create a directory named *bin* with this command::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-samples$ mkdir bin
 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-samples$ 

**Step 3.13:** The sample script you will run soon expects to find a program named *cryptogen*, a program named *configtxgen* , and a program named *configtxglator* in the *fabric-samples/bin* directory.
Let's oblige it by copying these three programs from the directory where there were placed courtesy of the *release* argument of the *make docker release* command from *Step 3.7*::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-samples$ cp -ipv ../fabric/release/linux-s390x/bin/c* bin/.
 '../fabric/release/linux-s390x/bin/configtxgen' -> 'bin/./configtxgen'
 '../fabric/release/linux-s390x/bin/configtxlator' -> 'bin/./configtxlator'
 '../fabric/release/linux-s390x/bin/cryptogen' -> 'bin/./cryptogen'

**Step 3.14:** Navigate to the *first-network* directory because this is the sample that you will use in this lab::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-samples$ cd first-network/
 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-samples/first-network$

**Step 3.15:** You will run the test using a script named *byfn.sh* in this directory. There are several arguments to this script. Invoke the script with the *-h* argument in order to display some help text::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-samples/first-network$ ./byfn.sh -h
 Usage: 
   byfn.sh <mode> [-c <channel name>] [-t <timeout>] [-d <delay>] [-f <docker-compose-file>] [-s <dbtype>] [-l <language>] [-o <consensus-type>] [-i <imagetag>] [-v]
     <mode> - one of 'up', 'down', 'restart', 'generate' or 'upgrade'
       - 'up' - bring up the network with docker-compose up
       - 'down' - clear the network with docker-compose down
       - 'restart' - restart the network
       - 'generate' - generate required certificates and genesis block
       - 'upgrade'  - upgrade the network from version 1.3.x to 1.4.0
     -c <channel name> - channel name to use (defaults to "mychannel")
     -t <timeout> - CLI timeout duration in seconds (defaults to 10)
     -d <delay> - delay duration in seconds (defaults to 3)
     -f <docker-compose-file> - specify which docker-compose file use (defaults to docker-compose-cli.yaml)
     -s <dbtype> - the database backend to use: goleveldb (default) or couchdb
     -l <language> - the chaincode language: golang (default) or node
     -o <consensus-type> - the consensus-type of the ordering service: solo (default) or kafka
     -i <imagetag> - the tag to be used to launch the network (defaults to "latest")
     -v - verbose mode
   byfn.sh -h (print this message)

 Typically, one would first generate the required certificates and 
 genesis block, then bring up the network. e.g.:

	byfn.sh generate -c mychannel
	byfn.sh up -c mychannel -s couchdb
 byfn.sh up -c mychannel -s couchdb -i 1.4.0
	byfn.sh up -l node
	byfn.sh down -c mychannel
 byfn.sh upgrade -c mychannel

 Taking all defaults:
 	byfn.sh generate
 	byfn.sh up
 	byfn.sh down

The last three lines of the output from the help shows the simplest way to perform this test.
If you took all defaults, you would be generating artifacts to create a channel named *mychannel*, using chaincode written in the *Go* language, an orderering consensus type of *solo*, and using *levelDB* as the state database.

**Step 3.16:** Run *byfn.sh* with the *generate* argument in order to generate the necessary cryptographic material and channel configuration for a channel named *mychannel*.  If you wish to use a different channel name, maybe named after your favorite child or pet or car or football team or spouse, use the *-c* argument as well, but be aware that channel names must start with a lowercase character, and contain only lowercase characters, numbers, and a small number of punctuation characters such as a dash (*-*) and a couple others I can't remember right now.  You'll also have to use this channel name on other inovcations of *byfn.sh* in subsequent steps.  Just don't use uppercase letters, okay?  (You can use uppercase numbers if that makes you feel better).  The below examples show creating a channel named *tim*. Use *tim* or create your own name or leave the *-c* argument off altogether to use the default of *mychannel*.  You decide.  I trust your judgement. So go ahead and issue this command or your customized version ::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-samples/first-network$ ./byfn.sh generate -c tim
 Generating certs and genesis block for channel 'tim' with CLI timeout of '10' seconds and CLI delay of '3' seconds
 Continue? [Y/n] Y
 proceeding ...
 /home/bcuser/git/src/github.com/hyperledger/fabric-samples/first-network/../bin/cryptogen

 ##########################################################
 ##### Generate certificates using cryptogen tool #########
 ##########################################################
 + cryptogen generate --config=./crypto-config.yaml
 org1.example.com
 org2.example.com
 + res=0
 + set +x

 /home/bcuser/git/src/github.com/hyperledger/fabric-samples/first-network/../bin/configtxgen
 ##########################################################
 #########  Generating Orderer Genesis block ##############
 ##########################################################
 CONSENSUS_TYPE=solo
 + '[' solo == solo ']'
 + configtxgen -profile TwoOrgsOrdererGenesis -channelID byfn-sys-channel -outputBlock ./channel-artifacts/genesis.block
 2019-01-15 11:35:34.252 EST [common.tools.configtxgen] main -> INFO 001 Loading configuration
 2019-01-15 11:35:34.284 EST [common.tools.configtxgen.localconfig] completeInitialization -> INFO 002 orderer type: solo
 2019-01-15 11:35:34.284 EST [common.tools.configtxgen.localconfig] Load -> INFO 003 Loaded configuration: /home/bcuser /git/src/github.com/hyperledger/fabric-samples/first-network/configtx.yaml
 2019-01-15 11:35:34.317 EST [common.tools.configtxgen.localconfig] completeInitialization -> INFO 004 orderer type: solo
 2019-01-15 11:35:34.317 EST [common.tools.configtxgen.localconfig] LoadTopLevel -> INFO 005 Loaded configuration: /home/bcuser/git/src/github.com/hyperledger/fabric-samples/first-network/configtx.yaml
 2019-01-15 11:35:34.319 EST [common.tools.configtxgen] doOutputBlock -> INFO 006 Generating genesis block
 2019-01-15 11:35:34.319 EST [common.tools.configtxgen] doOutputBlock -> INFO 007 Writing genesis block
 + res=0
 + set +x

 #################################################################
 ### Generating channel configuration transaction 'channel.tx' ###
 #################################################################
 + configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID tim
 2019-01-15 11:35:34.390 EST [common.tools.configtxgen] main -> INFO 001 Loading configuration
 2019-01-15 11:35:34.422 EST [common.tools.configtxgen.localconfig] Load -> INFO 002 Loaded configuration: /home/bcuser /git/src/github.com/hyperledger/fabric-samples/first-network/configtx.yaml
 2019-01-15 11:35:34.452 EST [common.tools.configtxgen.localconfig] completeInitialization -> INFO 003 orderer type: solo
 2019-01-15 11:35:34.452 EST [common.tools.configtxgen.localconfig] LoadTopLevel -> INFO 004 Loaded configuration: /home/bcuser/git/src/github.com/hyperledger/fabric-samples/first-network/configtx.yaml
 2019-01-15 11:35:34.452 EST [common.tools.configtxgen] doOutputChannelCreateTx -> INFO 005 Generating new channel configtx
 2019-01-15 11:35:34.453 EST [common.tools.configtxgen] doOutputChannelCreateTx -> INFO 006 Writing new channel tx
 + res=0
 + set +x

 #################################################################
 #######    Generating anchor peer update for Org1MSP   ##########
 #################################################################
 + configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID tim -asOrg Org1MSP
 2019-01-15 11:35:34.518 EST [common.tools.configtxgen] main -> INFO 001 Loading configuration
 2019-01-15 11:35:34.548 EST [common.tools.configtxgen.localconfig] Load -> INFO 002 Loaded configuration: /home/bcuser/git/src/github.com/hyperledger/fabric-samples/first-network/configtx.yaml
 2019-01-15 11:35:34.579 EST [common.tools.configtxgen.localconfig] completeInitialization -> INFO 003 orderer type: solo
 2019-01-15 11:35:34.579 EST [common.tools.configtxgen.localconfig] LoadTopLevel -> INFO 004 Loaded configuration: /home/bcuser/git/src/github.com/hyperledger/fabric-samples/first-network/configtx.yaml
 2019-01-15 11:35:34.579 EST [common.tools.configtxgen] doOutputAnchorPeersUpdate -> INFO 005 Generating anchor peer update
 2019-01-15 11:35:34.579 EST [common.tools.configtxgen] doOutputAnchorPeersUpdate -> INFO 006 Writing anchor peer update
 + res=0
 + set +x

 #################################################################
 #######    Generating anchor peer update for Org2MSP   ##########
 #################################################################
 + configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors.tx -channelID tim -asOrg Org2MSP
 2019-01-15 11:35:34.656 EST [common.tools.configtxgen] main -> INFO 001 Loading configuration
 2019-01-15 11:35:34.703 EST [common.tools.configtxgen.localconfig] Load -> INFO 002 Loaded configuration: /home/bcuser/git/src/github.com/hyperledger/fabric-samples/first-network/configtx.yaml
 2019-01-15 11:35:34.737 EST [common.tools.configtxgen.localconfig] completeInitialization -> INFO 003 orderer type: solo
 2019-01-15 11:35:34.737 EST [common.tools.configtxgen.localconfig] LoadTopLevel -> INFO 004 Loaded configuration: /home/bcuser/git/src/github.com/hyperledger/fabric-samples/first-network/configtx.yaml 
 2019-01-15 11:35:34.737 EST [common.tools.configtxgen] doOutputAnchorPeersUpdate -> INFO 005 Generating anchor peer update
 2019-01-15 11:35:34.737 EST [common.tools.configtxgen] doOutputAnchorPeersUpdate -> INFO 006 Writing anchor peer update
 + res=0
 + set +x

**Step 3.17:** Issue the following command to start the Hyperledger Fabric network using Node.js chaincode (the *-l node* argument and value), using CouchDB as the state database (the *-s couchdb* argument and value), the Kafka orderer consensus algorithm (the *-o kafka* argument and value, and the channel named *tim* (the *-c tim* argument and value).
If your channel is not named *tim* use your correct channel name instead.
If you want to try one of the other options for chaincode language, state database, or consensus algorithm, I applaud your curiosity and leave the determination of which options to choose as an exercise for the reader::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-samples/first-network$ ./byfn.sh up -l node -s couchdb -o kafka -c tim 
 Starting for channel 'tim' with CLI timeout of '10' seconds and CLI delay of '3' seconds and using database 'couchdb'
 Continue? [Y/n] Y
  .
  . (output not shown)
  .
 ========= All GOOD, BYFN execution completed =========== 

  _____   _   _   ____   
 | ____| | \ | | |  _ \  
 |  _|   |  \| | | | | | 
 | |___  | |\  | | |_| | 
 |_____| |_| \_| |____/  

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-samples/first-network$ 

**Step 3.18:** If you were following the output closely you may have noticed a few spots where the output paused for a little bit. 
This occurred at the point where a peer was building a new Docker image for chaincode.
This occurs three times during the script.
Additionally, Node.js chaincode takes longer for its Docker images to be built than does Go chaincode, so if you chose *-l node* the pause may have been long enough for you to worry a bit.
But it's all good.

Run this command to see the three Docker images created for the chaincode.
These images will start with *dev* because this is the default name of a Hyperledger Fabric network and this script does not override the default::
 
 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-samples/first-network$ docker images dev-*
 REPOSITORY                                                                                             TAG                  IMAGE ID            CREATED             SIZE
 dev-peer1.org2.example.com-mycc-1.0-26c2ef32838554aac4f7ad6f100aca865e87959c9a126e86d764c8d01f8346ab   latest              a8774579a9fc        2 minutes ago       1.55GB
 dev-peer0.org1.example.com-mycc-1.0-384f11f484b9302df90b453200cfb25174305fce8f53f4e94d45ee3b6cab0ce9   latest              4f27f3f88bdb        4 minutes ago       1.55GB
 dev-peer0.org2.example.com-mycc-1.0-15b571b3ce849066b7ec74497da3b27e54e0df1345daff3951b94245ce09c42b   latest              01039107ff4d        6 minutes ago       1.55GB

The second part of the Docker images name is the name of the peer for which the chaincode was created.
This sample uses chaincode in three peers.

**Step 3.19:** List the Docker containers that are running as part of this sample with this command::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-samples/first-network$ docker ps --all
 CONTAINER ID        IMAGE                                                                                                  COMMAND                  CREATED             STATUS              PORTS                                              NAMES
 5187acd227a1        dev-peer1.org2.example.com-mycc-1.0-26c2ef32838554aac4f7ad6f100aca865e87959c9a126e86d764c8d01f8346ab   "/bin/sh -c 'cd /usr…"   3 minutes ago       Up 3 minutes                                                           dev-peer1.org2.example.com-mycc-1.0
 2799f7a78fed        dev-peer0.org1.example.com-mycc-1.0-384f11f484b9302df90b453200cfb25174305fce8f53f4e94d45ee3b6cab0ce9   "/bin/sh -c 'cd /usr…"   5 minutes ago       Up 5 minutes                                                           dev-peer0.org1.example.com-mycc-1.0
 90932096a694        dev-peer0.org2.example.com-mycc-1.0-15b571b3ce849066b7ec74497da3b27e54e0df1345daff3951b94245ce09c42b   "/bin/sh -c 'cd /usr…"   7 minutes ago       Up 7 minutes                                                           dev-peer0.org2.example.com-mycc-1.0
 bc7c4fc1a6d4        hyperledger/fabric-tools:latest                                                                        "/bin/bash"              9 minutes ago       Up 9 minutes                                                           cli
 3eb36dabb079        hyperledger/fabric-kafka:latest                                                                        "/docker-entrypoint.…"   9 minutes ago       Up 9 minutes        9092-9093/tcp                                      kafka.example.com
 08fa1f55c4fd        hyperledger/fabric-peer:latest                                                                         "peer node start"        9 minutes ago       Up 9 minutes        0.0.0.0:7051->7051/tcp, 0.0.0.0:7053->7053/tcp     peer0.org1.example.com
 b88117066a91        hyperledger/fabric-peer:latest                                                                         "peer node start"        9 minutes ago       Up 9 minutes        0.0.0.0:9051->7051/tcp, 0.0.0.0:9053->7053/tcp     peer0.org2.example.com
 1090c22396bc        hyperledger/fabric-peer:latest                                                                         "peer node start"        9 minutes ago       Up 9 minutes        0.0.0.0:10051->7051/tcp, 0.0.0.0:10053->7053/tcp   peer1.org2.example.com
 15b622c0141e        hyperledger/fabric-peer:latest                                                                         "peer node start"        9 minutes ago       Up 9 minutes        0.0.0.0:8051->7051/tcp, 0.0.0.0:8053->7053/tcp     peer1.org1.example.com
 f6b983c81454        hyperledger/fabric-couchdb                                                                             "tini -- /docker-ent…"   9 minutes ago       Up 9 minutes        4369/tcp, 9100/tcp, 0.0.0.0:5984->5984/tcp         couchdb0
 65b3f6f4c73b        hyperledger/fabric-couchdb                                                                             "tini -- /docker-ent…"   9 minutes ago       Up 9 minutes        4369/tcp, 9100/tcp, 0.0.0.0:7984->5984/tcp         couchdb2
 677e9ecf4cdd        hyperledger/fabric-zookeeper:latest                                                                    "/docker-entrypoint.…"   9 minutes ago       Up 9 minutes        2181/tcp, 2888/tcp, 3888/tcp                       zookeeper.example.com
 0d370344a2b6        hyperledger/fabric-couchdb                                                                             "tini -- /docker-ent…"   9 minutes ago       Up 9 minutes        4369/tcp, 9100/tcp, 0.0.0.0:8984->5984/tcp         couchdb3
 807ff8200896        hyperledger/fabric-couchdb                                                                             "tini -- /docker-ent…"   9 minutes ago       Up 9 minutes        4369/tcp, 9100/tcp, 0.0.0.0:6984->5984/tcp         couchdb1
 a13a43f5a97e        hyperledger/fabric-orderer:latest                                                                      "orderer"                9 minutes ago       Up 9 minutes        0.0.0.0:7050->7050/tcp                             orderer.example.com

You may have a different number of containers running depending on what options you chose for your *byfn up* command.
If you did not choose *couchdb* as your state databse with the *-s* argument then you will not see the four *couchdb* containers (one for each peer).
If you did not choose the *Kafka* orderer consensus algorithm with the *-o* argument then you will not see the *kafka* and *zookeeper* containers.

**Step 3.20:** Run the following command pipe to show a more verbose output of the *docker ps* command which comes courtesy of the *--no-trunc* argument and then pipes the output through *grep* with a judicious string, *dev-peer* to filter the output such that you only see the output for the three chaincode containers::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-samples/first-network$ docker ps --no-trunc | grep dev-peer
 5187acd227a16d6aac4db1e83b5e2a0e1887bcac385f176a96533644d1406e2d   dev-peer1.org2.example.com-mycc-1.0-26c2ef32838554aac4f7ad6f100aca865e87959c9a126e86d764c8d01f8346ab   "/bin/sh -c 'cd /usr/local/src; npm start -- --peer.address peer1.org2.example.com:7052'"   4 minutes ago       Up 4 minutes                                                           dev-peer1.org2.example.com-mycc-1.0
 2799f7a78fedee73a688ba207ca8e0e498c3fdc0e2fbf641b76ff678fcac8ba7   dev-peer0.org1.example.com-mycc-1.0-384f11f484b9302df90b453200cfb25174305fce8f53f4e94d45ee3b6cab0ce9   "/bin/sh -c 'cd /usr/local/src; npm start -- --peer.address peer0.org1.example.com:7052'"   6 minutes ago       Up 6 minutes                                                           dev-peer0.org1.example.com-mycc-1.0
 90932096a6941374e751b35b0aaf8bc3fb55e080c001eac3cc4fb5d9c370ab5f   dev-peer0.org2.example.com-mycc-1.0-15b571b3ce849066b7ec74497da3b27e54e0df1345daff3951b94245ce09c42b   "/bin/sh -c 'cd /usr/local/src; npm start -- --peer.address peer0.org2.example.com:7052'"   8 minutes ago       Up 8 minutes                                                           dev-peer0.org2.example.com-mycc-1.0

In the above output you'll see the command used to start the container-  e.g., *"/bin/sh -c 'cd /usr/local/src; npm start -- --peer.address peer1.org2.example.com:7052'"* for the first container listed.
The *npm start* command is your clue that this is a Node.js container- *npm* is the de facto Node.js package manage, despite the authors of *npm* claiming that *npm* is not an acronym.  (I'm pretty sure this is true, and if not I'll modify this when the npm lawyers deliver the cease-and-desist letter to me).

**Step 3.21:** Run this command to stop the Hyperledger Network stood up in *Step 3.17*::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-samples/first-network$ ./byfn.sh down 
 Stopping for channel 'mychannel' with CLI timeout of '10' seconds and CLI delay of '3' seconds
 Continue? [Y/n] Y
  .
  . (output not shown)
  .
  
**Step 3.22:** Run this command to see if there are any running Docker containers::
  
 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-samples/first-network$ docker ps --all                     
 CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
 
There ain't no Docker containers no more.

**Step 3.23:** Run this command to see if there are any Docker images for the chaincode::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-samples/first-network$ docker images dev-*
 REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
 
There are no Docker chaincode images remaining as well-  the *byfn.sh down* command cleaned up nicer than a debonair dandy getting ready for a night on the town!

**Recap:** In this section, you:

*	Downloaded the main Hyperledger Fabric source code repository
*	Ran *make* to build the project’s Docker images and native binaries
* Downloaded the Hyperledger Fabric *fabric-samples* respository
*	Ran the *first-network* sample to verify core Hyperledger functionality
*	Cleaned up afterwards
 
Section 4: Install the Hyperledger Fabric Certificate Authority
===============================================================

In the prior section, the end-to-end test that you ran supplied its own security-related material such as keys and certificates- everything it needed to perform its test.  
Therefore it did not need the services of a Certificate Authority.

Almost all "real world" Hyperledger Fabric networks will not be this static-  new users, peers and organizations will probably join the network.  
They will need PKI x.509 certificates in order to participate.  
The Hyperledger Fabric Certificate Authority (CA) is provided by the Hyperledger Fabric project in order to issue these certificates.

The next major goal in this lab is to run the Hyperledger Fabric Node.js SDK’s end-to-end test.  
This test makes calls to the Hyperledger Fabric Certificate Authority (CA). 
Therefore, before we can run that test, you will get started by downloading and building the Hyperledger Fabric CA.

**Step 4.1:** Use *cd* to navigate three directory levels up, to the *hyperledger* directory::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric/examples/e2e_cli$ cd ~/git/src/github.com/hyperledger
 bcuser@ubuntu16045:~/git/src/github.com/hyperledger$ 

**Step 4.2:** Get the source code for the v1.3.0-rc1 release of the Fabric CA using *git*::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger$ git clone -b v1.3.0-rc1 https://gerrit.hyperledger.org/r/fabric-ca
 Cloning into 'fabric-ca'...
 remote: Counting objects: 18, done
 remote: Total 11768 (delta 0), reused 11768 (delta 0)
 Receiving objects: 100% (11768/11768), 26.70 MiB | 16.76 MiB/s, done.
 Resolving deltas: 100% (4132/4132), done.
 Checking connectivity... done.
 Note: checking out 'edb0015bcbe2a8add8f5c50f14b917cb4ddf9cb7'.

 You are in 'detached HEAD' state. You can look around, make experimental
 changes and commit them, and you can discard any commits you make in this
 state without impacting any branches by performing another checkout.

 If you want to create a new branch to retain commits you create, you may
 do so (now or later) by using -b with the checkout command again. Example:

   git checkout -b <new-branch-name>

**Step 4.3:** Navigate to the *fabric-ca* directory, which is the top directory of where the *git* command put the code it just downloaded::

 bcuser@ubuntu16044:~/git/src/github.com/hyperledger$ cd fabric-ca
 bcuser@ubuntu16044:~/git/src/github.com/hyperledger/fabric-ca$

**Step 4.4:** Enter the following command, which will build the Hyperledger Fabric CA image.  
Just like you did with the *fabric* repo, ‘wrap’ the *make* command, which is what will do all the work, in a *time* command, which will give you a measure of the time, including ‘wall clock’ time, required to build the image::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-ca $ time make docker
   .
   .  (output not shown here)
   .
 real	1m22.338s
 user	0m0.320s
 sys	0m0.152s
 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-ca$

**Step 4.5:** Enter the *docker images* command and you will see at the top of the output the Docker image that were just created for the Fabric Certificate Authority::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-ca$ docker images
 REPOSITORY                      TAG                 IMAGE ID            CREATED              SIZE
 hyperledger/fabric-ca          latest                             91082da5dc41        43 seconds ago      218MB
 hyperledger/fabric-ca          s390x-1.3.0-rc1                    91082da5dc41        43 seconds ago      218MB
   .
   . (remaining output not shown here)
   .

You may have noticed that for many of the images, the *Image ID* appears twice, once with a tag of *latest*, and once with a tag such as *s390x-1.1.0*. 
An image can be actually be given any number of tags. 
Think of these *tags* as nicknames, or aliases.  
In our case the *make* process first gave the Docker image it created a descriptive tag, *s390x-1.3.0-rc1*, and then it also ‘tagged’ it with a new tag, *latest*.  
It did that for a reason.  
When you are working with Docker images, if you specify an image without specifying a tag, the tag defaults to the name *latest*. 
So, for example, using the above output, you can specify either *hyperledger/fabric-ca*, *hyperledger/fabric-ca:latest*, or *hyperledger/fabric-ca:s390x-1.3.0-rc1*, and in all three cases you are asking for the same image, the image with ID *91082da5dc41*.

**Recap:** In this section, you downloaded the source code for the Hyperledger Fabric Certificate Authority and built it.  That was easy.
 
Section 5: Install Hyperledger Fabric Node.js SDK and its prerequisite software
===============================================================================
The preferred way for an application to interact with a Hyperledger Fabric chaincode is through a Software Development Kit (SDK) that 
exposes APIs.  The Hyperledger Fabric Node.js SDK is very popular among developers, due to the popularity of JavaScript as a programming 
language for developing web applications and the popularity of Node.js as a runtime platform for running server-side JavaScript.

In this section, you will install and configure Node.js, which also includes a program called *npm*, which is the de facto Node.js 
package manager.  

Then you will download the Hyperledger Fabric Node.js SDK and install npm packages that it requires.

**Step 5.1:** Change to the */tmp* directory::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-ca$ cd /tmp
 bcuser@ubuntu16045:/tmp$

**Step 5.2:** Retrieve the *Node.js* package with this command::

 bcuser@ubuntu16045:/tmp$ wget https://nodejs.org/dist/v8.11.3/node-v8.11.3-linux-s390x.tar.xz
 --2018-09-27 15:05:04--  https://nodejs.org/dist/v8.11.3/node-v8.11.3-linux-s390x.tar.xz
 Resolving nodejs.org (nodejs.org)... 104.20.22.46, 104.20.23.46, 2400:cb00:2048:1::6814:172e, ...
 Connecting to nodejs.org (nodejs.org)|104.20.22.46|:443... connected.
 HTTP request sent, awaiting response... 200 OK
 Length: 10939304 (10M) [application/x-xz]
 Saving to: 'node-v8.11.3-linux-s390x.tar.xz'

 node-v8.11.3-linux-s390x.tar.xz            100%[=====================================================================================>]  10.43M  46.9MB/s    in 0.2s    

 2018-09-27 15:05:04 (46.9 MB/s) - 'node-v8.11.3-linux-s390x.tar.xz' saved [10939304/10939304]


**Step 5.3:** Extract the package underneath your home directory, */home/bcuser*. 
This will cause the executables to wind up in */home/bcuser/bin*, which is in your path::

 bcuser@ubuntu16045:/tmp$ cd /home/bcuser && tar --strip-components=1 -xf /tmp/node-v8.11.3-linux-s390x.tar.xz
 bcuser@ubuntu16045:~$ 

**Step 5.4:** Issue this command to see where *node* resides within your path::

 bcuser@ubuntu16045:/tmp$ which node
 /home/bcuser/bin/node
 
**Step 5.5:** Issue this command to see where *npm* resides within your path::
 
 bcuser@ubuntu16045:/tmp $ which npm
 /home/bcuser/bin/npm
 
**Step 5.6:** Issue this command to see which version of *node* is installed::

 bcuser@ubuntu16045:~$ node --version
 v8.11.3
 
**Step 5.7:** Issue this command to see which version of *npm* is installed::
 
 bcuser@ubuntu16045:/tmp$ npm --version
 5.6.0

**Step 5.8:** Switch to the *~/git/src/github.com/hyperledger* directory::

 bcuser@ubuntu16045:~$ cd ~/git/src/github.com/hyperledger/
 bcuser@ubuntu16045:~/git/src/github.com/hyperledger$

**Step 5.9:** Now you will download the version 1.2.2 release of the Hyperledger Fabric Node SDK source code from its official repository::

 bcuser@ubuntu16045: ~/git/src/github.com/hyperledger $ git clone -b v1.2.2 https://gerrit.hyperledger.org/r/fabric-sdk-node
 Cloning into 'fabric-sdk-node'...
 remote: Counting objects: 21, done
 remote: Total 10603 (delta 0), reused 10603 (delta 0)
 Receiving objects: 100% (10603/10603), 7.49 MiB | 9.48 MiB/s, done.
 Resolving deltas: 100% (5225/5225), done.
 Checking connectivity... done.
 Note: checking out 'f5b275e9e6c78e25f26b431c5314f04b0b234122'.

 You are in 'detached HEAD' state. You can look around, make experimental
 changes and commit them, and you can discard any commits you make in this
 state without impacting any branches by performing another checkout.

 If you want to create a new branch to retain commits you create, you may
 do so (now or later) by using -b with the checkout command again. Example:

   git checkout -b <new-branch-name>

**Step 5.10:** Change to the *fabric-sdk-node* directory which was just created::

 bcuser@ubuntu16045: ~/git/src/github.com/hyperledger $ cd fabric-sdk-node
 bcuser@ubuntu16045: ~/git/src/github.com/hyperledger/fabric-sdk-node$

**Step 5.11:** Run *npm install* to install the required packages that the Hyperledger Fabric Node SDK would like to use.
This will take a few minutes and will produce a lot of output::

 bcuser@ubuntu16044: ~/git/src/github.com/hyperledger/fabric-sdk-node$ npm install
   .
   . (output not shown here)
   .
 added 1438 packages in 68.335s
 
bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$

You may ignore the *WARN* messages throughout the output, and there may even be some messages that look like error messages, but the npm installation program may be expecting such conditions and working through it.  
If there is a serious error, the end of the output will leave little doubt about it.

**Step 5.12:** Run the *npm list* command.  
The output, although not shown here, will show a long list of installed packages.  
This just proves what 
everyone suspected-  programmers would much rather use other peoples’ code than write their own.  
Not that there’s anything wrong with that. 
You can even steal this lab if you want to.

There may be several messages regarding unmet dependencies and missing prerequisites and the like, but you can ignore those for the purposes of this lab
::

 bcuser@ubuntu16045: ~/git/src/github.com/hyperledger/fabric-sdk-node$ npm list
   .
   . (output not shown here)
   .
 bcuser@ubuntu16045: ~/git/src/github.com/hyperledger/fabric-sdk-node$

**Step 5.13:** The tests use an automation tool named *gulp*, but before you install it, 
run the *which* command. The silent treatment it gives you confirms it is not available to you::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ which gulp
 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ 

**Step 5.14:** Now install *gulp* at a global level, using the *-g* argument to the *npm install*. 
This makes the package  available on a system-wide basis::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ npm install -g gulp
 npm WARN deprecated gulp-util@3.0.8: gulp-util is deprecated - replace it, following the guidelines at https://medium.com/gulpjs/gulp-util-ca3b1f9f9ac5
 npm WARN deprecated graceful-fs@3.0.11: please upgrade to graceful-fs 4 for compatibility with current and future versions of Node.js
 npm WARN deprecated minimatch@2.0.10: Please update to minimatch 3.0.2 or higher to avoid a RegExp DoS issue
 npm WARN deprecated minimatch@0.2.14: Please update to minimatch 3.0.2 or higher to avoid a RegExp DoS issue
 npm WARN deprecated graceful-fs@1.2.3: please upgrade to graceful-fs 4 for compatibility with current and future versions of Node.js
 /home/bcuser/bin/gulp -> /home/bcuser/lib/node_modules/gulp/bin/gulp.js
 + gulp@3.9.1
 added 253 packages in 4.726s

**NOTE:** These warnings can be ignored for the purposes of this lab, but in a production environment you should probably take the warnings more seriously.
 
**Step 5.15:** Running *which* again shows that *gulp* is available to you now::
 
 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ which gulp
 /home/bcuser/bin/gulp
 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$

**Step 5.16:** Next you will install a code coverage testing tool named *istanbul*, also at a global level.  
But first, use *which* to prove it isn't there yet::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ which istanbul
 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ 
 
**Step 5.17:** Install it globally::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ npm install -g istanbul
 /home/bcuser/bin/istanbul -> /home/bcuser/lib/node_modules/istanbul/lib/cli.js
 + istanbul@0.4.5
 added 48 packages in 1.677s

**Step 5.18:** Prove it worked::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ which istanbul
 /home/bcuser/bin/istanbul
 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$

**Recap:** In this section, you:

*	Installed Node.js and npm
*	Downloaded the Hyperledger Fabric Node.js SDK
*	Installed the *npm* packages required by the Hyperledger Fabric Node.js SDK
*	Installed the *gulp* and *istanbul* packages so that you are ready to run the Hyperledger Fabric Node.js SDK end-to-end test (which you will do in the next section)
 
Section 6: Run the Hyperledger Fabric Node.js SDK end-to-end test
=================================================================
In this section, you will run two tests provided by the Hyperledger Fabric Node.js SDK, verify their successful operation, and clean up afterwards.

The first test is a quick test that takes abot a minute, and does not bring up any chaincode containers.  
The second test is the "end-to-end" test, as it is much more comprehensive and will bring up several chaincode containers and will take several minutes.

**Step 6.1:** The first test is very simple and can be run simply by running *npm test*::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ npm test
   .
   . (initial output not shown)
   .
 1..1236
 # tests 1236
 # pass  1236

 # ok

 [16:17:49] Finished 'run-test-headless' after 1.1 min
 ---------------------------------|----------|----------|----------|----------|-------------------|
 File                             |  % Stmts | % Branch |  % Funcs |  % Lines | Uncovered Line #s |
 ---------------------------------|----------|----------|----------|----------|-------------------|
 All files                        |    64.95 |    57.66 |    66.93 |       65 |                   |
  fabric-ca-client/lib            |    63.68 |    59.59 |    51.85 |    63.68 |                   |
   AffiliationService.js          |    66.67 |       70 |      100 |    66.67 |... 82,185,186,189 |
   FabricCAClientImpl.js          |    62.58 |    59.24 |    44.62 |    62.58 |... 54,955,957,960 |
   IdentityService.js             |    63.64 |    46.15 |    66.67 |    63.64 |... 46,248,249,252 |
   helper.js                      |      100 |      100 |      100 |      100 |                   |
  fabric-client/lib               |    63.95 |    57.49 |    68.25 |       64 |                   |
   BaseClient.js                  |    93.33 |    91.67 |    92.86 |    93.33 |           119,196 |
   BlockDecoder.js                |    69.78 |    52.94 |    72.41 |    70.11 |... 1393,1395,1396 |
   CertificateAuthority.js        |       75 |      100 |       50 |       75 |... 60,167,174,181 |
   Channel.js                     |    44.59 |    38.13 |     58.4 |    44.53 |... 3379,3381,3384 |
   ChannelEventHub.js             |    64.35 |    55.83 |    68.75 |    64.72 |... 1317,1318,1320 |
   Client.js                      |    75.25 |    76.06 |    80.23 |    75.07 |... 1925,1928,1931 |
   Config.js                      |    91.18 |       75 |      100 |    91.18 |          55,73,90 |
   Constants.js                   |      100 |      100 |      100 |      100 |                   |
   EventHub.js                    |    69.18 |    63.78 |    68.75 |     69.3 |... 18,819,824,843 |
   Orderer.js                     |    24.26 |    17.24 |    31.58 |    24.26 |... 82,283,284,286 |
   Organization.js                |     97.5 |      100 |    94.12 |      100 |                   |
   Packager.js                    |    90.91 |    91.67 |      100 |    90.91 |             50,51 |
   Peer.js                        |    50.67 |    31.25 |    38.46 |    50.67 |... 71,172,174,175 |
   Policy.js                      |    89.83 |    88.68 |       80 |    89.83 |... 36,237,239,245 |
   Remote.js                      |    84.85 |    79.69 |       90 |    84.85 |... 82,183,184,234 |
   SideDB.js                      |    74.14 |    96.15 |       80 |    74.14 |... 19,121,127,129 |
   TransactionID.js               |    95.83 |    83.33 |      100 |    95.83 |                39 |
   User.js                        |     82.8 |    64.29 |    78.95 |     82.8 |... 33,235,256,263 |
   api.js                         |    41.94 |        0 |    13.79 |    41.94 |... 69,401,408,417 |
   client-utils.js                |    88.79 |    72.97 |    86.67 |    88.79 |... 28,186,199,201 |
   hash.js                        |      100 |      100 |      100 |      100 |                   |
   utils.js                       |    75.78 |    72.66 |    75.61 |    75.78 |... 93,595,597,600 |
  fabric-client/lib/impl          |    68.72 |    61.69 |    70.08 |     68.6 |                   |
   BasicCommitHandler.js          |    76.47 |       70 |      100 |    76.47 |... 27,128,131,132 |
   CouchDBKeyValueStore.js        |    76.71 |       60 |    93.33 |    77.46 |... 46,147,160,161 |
   CryptoKeyStore.js              |      100 |     87.5 |      100 |      100 |             42,76 |
   CryptoSuite_ECDSA_AES.js       |     84.4 |    71.84 |    78.95 |       85 |... 78,307,324,330 |
   DiscoveryEndorsementHandler.js |    79.44 |    69.33 |      100 |    79.44 |... 94,296,304,306 |
   FileKeyValueStore.js           |    91.89 |    83.33 |      100 |    91.89 |          47,48,65 |
   NetworkConfig_1_0.js           |    98.06 |    85.05 |      100 |    98.03 |... 02,416,449,450 |
   bccsp_pkcs11.js                |     25.8 |    30.97 |     8.33 |    24.25 |... 1051,1055,1056 |
  fabric-client/lib/impl/aes      |    11.11 |        0 |        0 |    11.11 |                   |
   pkcs11_key.js                  |    11.11 |        0 |        0 |    11.11 |... 39,43,47,51,55 |
  fabric-client/lib/impl/ecdsa    |       50 |    31.25 |       45 |    51.82 |                   |
   key.js                         |    98.46 |    96.15 |      100 |    98.46 |               182 |
   pkcs11_key.js                  |     9.09 |        0 |        0 |     9.72 |... 54,158,159,161 |
  fabric-client/lib/msp           |    75.44 |    60.49 |       70 |    75.74 |                   |
   identity.js                    |    88.24 |    64.52 |    76.92 |    88.24 |... 86,105,106,214 |
   msp-manager.js                 |       75 |    72.73 |    85.71 |       76 |... 15,116,117,146 |
   msp.js                         |    66.18 |    46.43 |       50 |    66.18 |... 38,139,181,182 |
  fabric-client/lib/packager      |     88.7 |    60.71 |    87.18 |     88.6 |                   |
   BasePackager.js                |    81.48 |    44.44 |    78.95 |    81.48 |... 30,145,168,186 |
   Car.js                         |       60 |      100 |        0 |       60 |             16,17 |
   Golang.js                      |      100 |      100 |      100 |      100 |                   |
   Node.js                        |    96.43 |       75 |      100 |     96.3 |                75 |
 ---------------------------------|----------|----------|----------|----------|-------------------| 

 =============================== Coverage summary ===============================
 Statements   : 64.95% ( 4369/6727 )
 Branches     : 57.66% ( 1845/3200 )
 Functions    : 66.93% ( 597/892 )
 Lines        : 65% ( 4339/6675 )
 ================================================================================
 [16:17:50] Finished 'test-headless' after 1.2 min


You may have seen some messages scroll by that looked like errors or exceptions, but chances are they were expected to occur within the test cases-  the key indicator of this is that of the 1,236 tests, all of them passed.  


**Step 6.2:** Run the end-to-end tests with the *gulp test* command.  
While this command is running, a little bit of the output may look like errors, but some of the tests expect errors, so the real indicator is, again, like the first test, whether or not all tests passed::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ gulp test
   .
   . (lots of output not shown here)
   . 
 
 1..2256
 # tests 2256
 # pass  2256

 # ok

 [16:34:20] Finished 'run-test' after 12 min
 ---------------------------------|----------|----------|----------|----------|-------------------|
 File                             |  % Stmts | % Branch |  % Funcs |  % Lines | Uncovered Line #s |
 ---------------------------------|----------|----------|----------|----------|-------------------|
 All files                        |    85.13 |    73.31 |    83.63 |    85.29 |                   |
  fabric-ca-client/lib            |    94.87 |    89.38 |    93.83 |    94.87 |                   |
   AffiliationService.js          |    98.33 |       96 |      100 |    98.33 |               186 |
   FabricCAClientImpl.js          |    95.09 |    89.67 |    93.85 |    95.09 |... 31,949,957,960 |
   IdentityService.js             |    90.91 |    80.77 |    88.89 |    90.91 |... 37,240,243,249 |
   helper.js                      |      100 |      100 |      100 |      100 |                   |
  fabric-client/lib               |    89.05 |    76.96 |    87.44 |    89.16 |                   |
   BaseClient.js                  |    93.33 |    91.67 |    92.86 |    93.33 |           119,196 |
   BlockDecoder.js                |    91.37 |    65.69 |    98.28 |    91.85 |... 1377,1392,1393 |
   CertificateAuthority.js        |    80.56 |      100 |    61.11 |    80.56 |... 60,167,174,181 |
   Channel.js                     |    87.17 |    71.06 |     90.4 |    87.26 |... 3379,3381,3384 |
   ChannelEventHub.js             |    88.56 |    82.92 |    89.58 |    88.68 |... 1256,1313,1320 |
   Client.js                      |    92.91 |    87.09 |    93.02 |    92.86 |... 1925,1928,1931 |
   Config.js                      |    94.12 |     87.5 |      100 |    94.12 |             73,90 |
   Constants.js                   |      100 |      100 |      100 |      100 |                   |
   EventHub.js                    |    91.54 |    81.89 |    96.88 |    91.79 |... 01,602,815,824 |
   Orderer.js                     |    71.32 |    48.28 |    84.21 |    71.32 |... 82,283,284,286 |
   Organization.js                |     97.5 |      100 |    94.12 |      100 |                   |
   Packager.js                    |    90.91 |    91.67 |      100 |    90.91 |             50,51 |
   Peer.js                        |    89.33 |    78.13 |      100 |    89.33 |... 21,167,174,175 |
   Policy.js                      |    99.15 |    92.45 |      100 |    99.15 |               149 |
   Remote.js                      |    90.91 |    90.63 |       90 |    90.91 |... 82,183,184,234 |
   SideDB.js                      |      100 |      100 |      100 |      100 |                   |
   TransactionID.js               |    95.83 |    83.33 |      100 |    95.83 |                39 |
   User.js                        |     91.4 |    73.21 |    89.47 |     91.4 |... 13,218,219,235 |
   api.js                         |    41.94 |        0 |    13.79 |    41.94 |... 69,401,408,417 |
   client-utils.js                |    95.33 |    81.08 |      100 |    95.33 |56,128,186,199,201 |
   hash.js                        |      100 |      100 |      100 |      100 |                   |
   utils.js                       |    80.47 |    75.78 |    78.05 |    80.47 |... 93,452,531,597 |
  fabric-client/lib/impl          |    71.62 |    64.35 |    70.08 |    71.56 |                   |
   BasicCommitHandler.js          |    89.71 |       85 |      100 |    89.71 |... 27,128,131,132 |
   CouchDBKeyValueStore.js        |    80.82 |    63.33 |    93.33 |    81.69 |... 46,147,160,161 |
   CryptoKeyStore.js              |      100 |     87.5 |      100 |      100 |             42,76 |
   CryptoSuite_ECDSA_AES.js       |     84.4 |    71.84 |    78.95 |       85 |... 78,307,324,330 |
   DiscoveryEndorsementHandler.js |    91.11 |       84 |      100 |    91.11 |... 94,296,304,306 |
   FileKeyValueStore.js           |    91.89 |    83.33 |      100 |    91.89 |          47,48,65 |
   NetworkConfig_1_0.js           |    98.06 |     86.6 |      100 |    98.03 |... 02,416,449,450 |
   bccsp_pkcs11.js                |     25.8 |    30.97 |     8.33 |    24.25 |... 1051,1055,1056 |
  fabric-client/lib/impl/aes      |    11.11 |        0 |        0 |    11.11 |                   |
   pkcs11_key.js                  |    11.11 |        0 |        0 |    11.11 |... 39,43,47,51,55 |
  fabric-client/lib/impl/ecdsa    |       50 |    31.25 |       45 |    51.82 |                   |
   key.js                         |    98.46 |    96.15 |      100 |    98.46 |               182 |
   pkcs11_key.js                  |     9.09 |        0 |        0 |     9.72 |... 54,158,159,161 |
  fabric-client/lib/msp           |    80.12 |    62.96 |    76.67 |    79.88 |                   |
   identity.js                    |    92.16 |    67.74 |    84.62 |    92.16 |     42,86,105,106 |
   msp-manager.js                 |    86.54 |    77.27 |      100 |       86 |... 5,76,77,78,146 |
   msp.js                         |    66.18 |    46.43 |       50 |    66.18 |... 38,139,181,182 |
  fabric-client/lib/packager      |     88.7 |    60.71 |    87.18 |     88.6 |                   |
   BasePackager.js                |    81.48 |    44.44 |    78.95 |    81.48 |... 30,145,168,186 |
   Car.js                         |       60 |      100 |        0 |       60 |             16,17 |
   Golang.js                      |      100 |      100 |      100 |      100 |                   |
   Node.js                        |    96.43 |       75 |      100 |     96.3 |                75 |
 ---------------------------------|----------|----------|----------|----------|-------------------|

 =============================== Coverage summary ===============================
 Statements   : 85.13% ( 5727/6727 )
 Branches     : 73.31% ( 2346/3200 )
 Functions    : 83.63% ( 746/892 )
 Lines        : 85.29% ( 5693/6675 )
 ================================================================================
 [16:34:31] Finished 'test' after 13 min
 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$

**NOTE:** When this lab was first written, this test would use the Docker images that you created in the first sections of this lab.  
Since then, this test will now retrieve the images you created from the public Docker Hub repository. 
It would be possible to tailor this test so that it uses the images that you built earlier, but that is an advanced topic beyond the scope of this lab.  
In fact, as of this update (late September 2018) the most recent tag in the *fabric-node-sdk* repo, *v1.2.2*, is looking for Hyperledger Fabric v1.2 images, whereas in the prior steps I had you build using the most recent tag (again, as of late September 2018) in the *fabric* and *fabric-ca repo*, which was *v1.3.0-rc1* for both.

**Step 6.3:** (Optional) What I really like about the second end-to-end test is that it cleans itself up really well at the beginning- that is, it will remove any artifacts left running at the end of the prior test, so if you wanted to, you could simply enter *gulp test* again if you'd like to see this for yourself and have several minutes to spare.  
If you're pressed for time, skip this step::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ gulp test
   .
   . (output not shown here)
   . 

**Step 6.4:** Enter this command to see what Docker containers were created as part of the test::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ docker ps -a
 CONTAINER ID        IMAGE                                                                                                                                        COMMAND                  CREATED             STATUS              PORTS                                            NAMES
 8bdd2cd86477        dev-peer0.org2.example.com-e2enodecc-v1538081208749-b629728e22b7e99aeaea14164075177fe3ac2625212d91a50145d5707d23fab6                         "/bin/sh -c 'cd /usr…"   12 minutes ago      Up 12 minutes                                                        dev-peer0.org2.example.com-e2enodecc-v1538081208749
 6b1c612e5adc        dev-peer0.org1.example.com-e2enodecc-v1538081208749-0e76ffce65a1a7f16af8360ed94a6946d0ca855faaf0e031a9c6e2879905ae91                         "/bin/sh -c 'cd /usr…"   12 minutes ago      Up 12 minutes                                                        dev-peer0.org1.example.com-e2enodecc-v1538081208749
 0e7fcf5dc1d8        dev-peer0.org2.example.com-end2endnodesdk-v1538081208749-4b86007dcf45ab90bdb6c7a12b48fe009ec79091e304e277a819611d421a7547                    "chaincode -peer.add…"   13 minutes ago      Up 13 minutes                                                        dev-peer0.org2.example.com-end2endnodesdk-v1538081208749
 e0a64f787844        dev-peer0.org1.example.com-end2endnodesdk-v1538081208749-6a9f39aaaf432465c015c135a1bee7a9f701c4a220f50912ab51f773e5d41755                    "chaincode -peer.add…"   13 minutes ago      Up 13 minutes                                                        dev-peer0.org1.example.com-end2endnodesdk-v1538081208749
 1c22fc838156        dev-peer0.org2.example.com-example-v2-90bdd5067516ca8ff658962fb11e84e3894c43f587b7ee58fb3aa67b3f8c1281                                       "chaincode -peer.add…"   14 minutes ago      Up 14 minutes                                                        dev-peer0.org2.example.com-example-v2
 aa7506650662        dev-peer0.org1.example.com-example-v2-2998a364b4084289621eed7b56196ada935299d7a677e7182298a70fac3ae9fc                                       "chaincode -peer.add…"   14 minutes ago      Up 14 minutes                                                        dev-peer0.org1.example.com-example-v2
 ffb90c6a65aa        dev-peer0.org1.example.com-example-v1-23d6c8a7edc0c13919f6ebd42e8bdb11d048860ba202fa89226d0a9b9ab031ec                                       "chaincode -peer.add…"   15 minutes ago      Up 15 minutes                                                        dev-peer0.org1.example.com-example-v1
 eb35065cd232        dev-peer0.org2.example.com-example-v1-5e945f2a4bda672df2b593545e20e6bdcf2e2f196f718358a9a13286857000f7                                       "chaincode -peer.add…"   15 minutes ago      Up 15 minutes                                                        dev-peer0.org2.example.com-example-v1
 a70d3796d5ef        dev-peer0.org1.example.com-end2endnodesdk-v3-78c36fcdd427a2cedc3441743b894733bfd2f1440c7ad6bd8c7b825981f2e5c9                                "chaincode -peer.add…"   16 minutes ago      Up 16 minutes                                                        dev-peer0.org1.example.com-end2endnodesdk-v3
 b898ef4b4344        dev-peer0.org2.example.com-end2endnodesdk-v3-d83d9a69fa471c4cd45b511a29301b5ed30b2a9f07847b9ae5a34f8f99c7f141                                "chaincode -peer.add…"   16 minutes ago      Up 16 minutes                                                        dev-peer0.org2.example.com-end2endnodesdk-v3
 f5563ff7f50a        dev-peer0.org1.example.com-events_unit_test_v1538081016245-v1538081016245-bb9c386389d57c26eea93f05d81e7aaee5898dd832d78a48ddcd88f653145c42   "chaincode -peer.add…"   16 minutes ago      Up 16 minutes                                                        dev-peer0.org1.example.com-events_unit_test_v1538081016245-v1538081016245
 914b95b8e356        dev-peer0.org1.example.com-events_unit_test1538080641590-v1538080641590-082aadf860511fc1d404ed3f70a7d6878db53bbe8cc8221562255c13ca9751ec     "chaincode -peer.add…"   17 minutes ago      Up 17 minutes                                                        dev-peer0.org1.example.com-events_unit_test1538080641590-v1538080641590
 3146398ef735        dev-peer0.org2.example.com-end2endnodesdk-v1-28ad5b85f1199c9112eb1ecc700a3d1df6e02826c37d26e0fa4b7435c6970156                                "chaincode -peer.add…"   17 minutes ago      Up 17 minutes                                                        dev-peer0.org2.example.com-end2endnodesdk-v1
 b18f605e4040        dev-peer0.org1.example.com-end2endnodesdk-v1-cb50140fc38a1dbff755119ff4f1af9c21ea51dd33e23af11035622c35921bd4                                "chaincode -peer.add…"   17 minutes ago      Up 17 minutes                                                        dev-peer0.org1.example.com-end2endnodesdk-v1
 57e4624291fa        dev-peer0.org1.example.com-end2endnodesdk_privatedata-v0-51a96f60e00d7b9f6d88c8707ea5590906fcd18dbbad8b8cb02060d7795f86b2                    "chaincode -peer.add…"   18 minutes ago      Up 18 minutes                                                        dev-peer0.org1.example.com-end2endnodesdk_privatedata-v0
 c8da41505595        dev-peer0.org2.example.com-end2endnodesdk_privatedata-v0-7c0b39eb135217fa341a9bb9f8ec0defd59bfb2cbdf25559bb281fc6d4ad5851                    "chaincode -peer.add…"   18 minutes ago      Up 18 minutes                                                        dev-peer0.org2.example.com-end2endnodesdk_privatedata-v0
 3e3d05f1b87e        dev-peer0.org2.example.com-end2endnodesdk-v0-cffecf663c4cac97a99d46282042dc47e9b6b306eb3e1e3d271cf3b25f1e9958                                "chaincode -peer.add…"   19 minutes ago      Up 19 minutes                                                        dev-peer0.org2.example.com-end2endnodesdk-v0
 e57f8135cc6a        dev-peer0.org1.example.com-end2endnodesdk-v0-2c1b3eb0a77138303953abb093fcd1df798601e1dc45c1d0d76fd23d671f44ad                                "chaincode -peer.add…"   19 minutes ago      Up 19 minutes                                                        dev-peer0.org1.example.com-end2endnodesdk-v0
 c06865edb826        dev-peer0.org2.example.com-e2enodecc-v1-d8837a85ad58d7fdaaeabc0e9ba1f3afa23697653b401c04755945ca06e8799a                                     "/bin/sh -c 'cd /usr…"   19 minutes ago      Up 19 minutes                                                        dev-peer0.org2.example.com-e2enodecc-v1
 7a9e40d28823        dev-peer0.org1.example.com-e2enodecc-v1-51c979938c8b1894cd6d7283f286f4e8d7c8459241e8df9db618f7b184a527c7                                     "/bin/sh -c 'cd /usr…"   19 minutes ago      Up 19 minutes                                                        dev-peer0.org1.example.com-e2enodecc-v1
 efa6673912dd        dev-peer0.org2.example.com-e2enodecc-v0-ea065bdcb166a15ec4bc1565e18f5f0361f3f7cae214b1d4447192fd1378bdf6                                     "/bin/sh -c 'cd /usr…"   21 minutes ago      Up 21 minutes                                                        dev-peer0.org2.example.com-e2enodecc-v0
 9e6e96e173c4        dev-peer0.org1.example.com-e2enodecc-v0-70ad6a959a7accfe9e547d7648e65307b18d05a80f055bf1de425b3b4d61f4b6                                     "/bin/sh -c 'cd /usr…"   21 minutes ago      Up 21 minutes                                                        dev-peer0.org1.example.com-e2enodecc-v0
 38cc0d007dc3        hyperledger/fabric-peer:s390x-1.2.0                                                                                                          "peer node start"        23 minutes ago      Up 23 minutes       0.0.0.0:8051->8051/tcp, 0.0.0.0:8053->8053/tcp   peer0.org2.example.com
 eec5f98d831e        hyperledger/fabric-peer:s390x-1.2.0                                                                                                          "peer node start"        23 minutes ago      Up 23 minutes       0.0.0.0:7051->7051/tcp, 0.0.0.0:7053->7053/tcp   peer0.org1.example.com
 f25bf3b8ea1d        hyperledger/fabric-couchdb:s390x-0.4.10                                                                                                      "tini -- /docker-ent…"   23 minutes ago      Up 23 minutes       4369/tcp, 9100/tcp, 0.0.0.0:5984->5984/tcp       couchdb
 1c084cb29490        hyperledger/fabric-ca:s390x-1.2.0                                                                                                            "sh -c 'fabric-ca-se…"   23 minutes ago      Up 23 minutes       0.0.0.0:8054->7054/tcp                           ca_peerOrg2
 95d4c0875c73        hyperledger/fabric-orderer:s390x-1.2.0                                                                                                       "orderer"                23 minutes ago      Up 23 minutes       0.0.0.0:7050->7050/tcp                           orderer.example.com
 8b4b6ee98e73        hyperledger/fabric-ca:s390x-1.2.0                                                                                                            "sh -c 'fabric-ca-se…"   23 minutes ago      Up 23 minutes       0.0.0.0:7054->7054/tcp                           ca_peerOrg1
 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ 

**Step 6.5:** Enter this command to see that several Docker images for chaincode have been created as part of the test.  
These are the images that start with *dev-*::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ docker images
 REPOSITORY                                                                                                                                   TAG                                IMAGE ID            CREATED             SIZE
 dev-peer0.org2.example.com-e2enodecc-v1538081208749-b629728e22b7e99aeaea14164075177fe3ac2625212d91a50145d5707d23fab6                         latest                             c96634c914c4        15 minutes ago      1.55GB
 dev-peer0.org1.example.com-e2enodecc-v1538081208749-0e76ffce65a1a7f16af8360ed94a6946d0ca855faaf0e031a9c6e2879905ae91                         latest                             7ad21b78879e        15 minutes ago      1.55GB
 dev-peer0.org1.example.com-end2endnodesdk-v1538081208749-6a9f39aaaf432465c015c135a1bee7a9f701c4a220f50912ab51f773e5d41755                    latest                             37eae1e15067        16 minutes ago      137MB
 dev-peer0.org2.example.com-end2endnodesdk-v1538081208749-4b86007dcf45ab90bdb6c7a12b48fe009ec79091e304e277a819611d421a7547                    latest                             ed5c69e28ebe        16 minutes ago      137MB
 dev-peer0.org1.example.com-example-v2-2998a364b4084289621eed7b56196ada935299d7a677e7182298a70fac3ae9fc                                       latest                             1b6f00e8dbc7        17 minutes ago      137MB
 dev-peer0.org2.example.com-example-v2-90bdd5067516ca8ff658962fb11e84e3894c43f587b7ee58fb3aa67b3f8c1281                                       latest                             8da2381befa5        17 minutes ago      137MB
 dev-peer0.org2.example.com-example-v1-5e945f2a4bda672df2b593545e20e6bdcf2e2f196f718358a9a13286857000f7                                       latest                             df7df00475b1        18 minutes ago      137MB
 dev-peer0.org1.example.com-example-v1-23d6c8a7edc0c13919f6ebd42e8bdb11d048860ba202fa89226d0a9b9ab031ec                                       latest                             7e95cdced795        18 minutes ago      137MB
 dev-peer0.org2.example.com-end2endnodesdk-v3-d83d9a69fa471c4cd45b511a29301b5ed30b2a9f07847b9ae5a34f8f99c7f141                                latest                             dc0a911b4060        19 minutes ago      137MB
 dev-peer0.org1.example.com-end2endnodesdk-v3-78c36fcdd427a2cedc3441743b894733bfd2f1440c7ad6bd8c7b825981f2e5c9                                latest                             20583453f180        19 minutes ago      137MB
 dev-peer0.org1.example.com-events_unit_test_v1538081016245-v1538081016245-bb9c386389d57c26eea93f05d81e7aaee5898dd832d78a48ddcd88f653145c42   latest                             24c01fb00a9c        19 minutes ago      137MB
 dev-peer0.org1.example.com-events_unit_test1538080641590-v1538080641590-082aadf860511fc1d404ed3f70a7d6878db53bbe8cc8221562255c13ca9751ec     latest                             ec8c9a92aa37        20 minutes ago      137MB
 dev-peer0.org1.example.com-end2endnodesdk-v1-cb50140fc38a1dbff755119ff4f1af9c21ea51dd33e23af11035622c35921bd4                                latest                             50638eef91e0        20 minutes ago      137MB
 dev-peer0.org2.example.com-end2endnodesdk-v1-28ad5b85f1199c9112eb1ecc700a3d1df6e02826c37d26e0fa4b7435c6970156                                latest                             6d4d9ad8d043        20 minutes ago      137MB
 dev-peer0.org2.example.com-end2endnodesdk_privatedata-v0-7c0b39eb135217fa341a9bb9f8ec0defd59bfb2cbdf25559bb281fc6d4ad5851                    latest                             672e205e2a5c        21 minutes ago      137MB
 dev-peer0.org1.example.com-end2endnodesdk_privatedata-v0-51a96f60e00d7b9f6d88c8707ea5590906fcd18dbbad8b8cb02060d7795f86b2                    latest                             067be94c6813        21 minutes ago      137MB
 dev-peer0.org2.example.com-end2endnodesdk-v0-cffecf663c4cac97a99d46282042dc47e9b6b306eb3e1e3d271cf3b25f1e9958                                latest                             b2ed753b75c1        21 minutes ago      137MB
 dev-peer0.org1.example.com-end2endnodesdk-v0-2c1b3eb0a77138303953abb093fcd1df798601e1dc45c1d0d76fd23d671f44ad                                latest                             22786db0e0d0        21 minutes ago      137MB
 dev-peer0.org1.example.com-e2enodecc-v1-51c979938c8b1894cd6d7283f286f4e8d7c8459241e8df9db618f7b184a527c7                                     latest                             bb8e1b6d20f5        22 minutes ago      1.55GB
 dev-peer0.org2.example.com-e2enodecc-v1-d8837a85ad58d7fdaaeabc0e9ba1f3afa23697653b401c04755945ca06e8799a                                     latest                             1e3fd93e38d2        22 minutes ago      1.55GB
 dev-peer0.org1.example.com-e2enodecc-v0-70ad6a959a7accfe9e547d7648e65307b18d05a80f055bf1de425b3b4d61f4b6                                     latest                             e21636660582        23 minutes ago      1.55GB
 dev-peer0.org2.example.com-e2enodecc-v0-ea065bdcb166a15ec4bc1565e18f5f0361f3f7cae214b1d4447192fd1378bdf6                                     latest                             86b8c578804a        23 minutes ago      1.55GB
 hyperledger/fabric-ca                                                                                                                        latest                             91082da5dc41        About an hour ago   218MB
 hyperledger/fabric-ca                                                                                                                        s390x-1.3.0-rc1                    91082da5dc41        About an hour ago   218MB
 hyperledger/fabric-tools                                                                                                                     latest                             2f932afc62cf        2 hours ago         1.48GB
 hyperledger/fabric-tools                                                                                                                     s390x-1.3.0-rc1-snapshot-d5c1514   2f932afc62cf        2 hours ago         1.48GB
 hyperledger/fabric-tools                                                                                                                     s390x-latest                       2f932afc62cf        2 hours ago         1.48GB
 hyperledger/fabric-testenv                                                                                                                   latest                             94f5cf342fd1        2 hours ago         1.53GB
 hyperledger/fabric-testenv                                                                                                                   s390x-1.3.0-rc1-snapshot-d5c1514   94f5cf342fd1        2 hours ago         1.53GB
 hyperledger/fabric-testenv                                                                                                                   s390x-latest                       94f5cf342fd1        2 hours ago         1.53GB
 hyperledger/fabric-buildenv                                                                                                                  latest                             811f6744c029        2 hours ago         1.44GB
 hyperledger/fabric-buildenv                                                                                                                  s390x-1.3.0-rc1-snapshot-d5c1514   811f6744c029        2 hours ago         1.44GB
 hyperledger/fabric-buildenv                                                                                                                  s390x-latest                       811f6744c029        2 hours ago         1.44GB
 hyperledger/fabric-ccenv                                                                                                                     latest                             446ba9534733        2 hours ago         1.39GB
 hyperledger/fabric-ccenv                                                                                                                     s390x-1.3.0-rc1-snapshot-d5c1514   446ba9534733        2 hours ago         1.39GB
 hyperledger/fabric-ccenv                                                                                                                     s390x-latest                       446ba9534733        2 hours ago         1.39GB
 hyperledger/fabric-orderer                                                                                                                   latest                             402795f7129d        2 hours ago         142MB
 hyperledger/fabric-orderer                                                                                                                   s390x-1.3.0-rc1-snapshot-d5c1514   402795f7129d        2 hours ago         142MB
 hyperledger/fabric-orderer                                                                                                                   s390x-latest                       402795f7129d        2 hours ago         142MB
 hyperledger/fabric-peer                                                                                                                      latest                             f28d66c114d4        2 hours ago         149MB
 hyperledger/fabric-peer                                                                                                                      s390x-1.3.0-rc1-snapshot-d5c1514   f28d66c114d4        2 hours ago         149MB
 hyperledger/fabric-peer                                                                                                                      s390x-latest                       f28d66c114d4        2 hours ago         149MB
 hyperledger/fabric-zookeeper                                                                                                                 latest                             6a9cfd5ac47a        9 days ago          1.39GB
 hyperledger/fabric-kafka                                                                                                                     latest                             8ce4902cb68e        9 days ago          1.4GB
 hyperledger/fabric-couchdb                                                                                                                   latest                             2bc434828917        9 days ago          1.52GB
 hyperledger/fabric-baseimage                                                                                                                 s390x-0.4.12                       a2d3919231fa        9 days ago          1.35GB
 hyperledger/fabric-baseos                                                                                                                    s390x-0.4.12                       54e371e1a6ee        9 days ago          120MB
 hyperledger/fabric-ca                                                                                                                        s390x-1.2.0                        7a9a8d2589c9        2 months ago        217MB
 hyperledger/fabric-orderer                                                                                                                   s390x-1.2.0                        75b2abac5351        2 months ago        140MB
 hyperledger/fabric-peer                                                                                                                      s390x-1.2.0                        85d0c3e2c248        2 months ago        147MB
 hyperledger/fabric-couchdb                                                                                                                   s390x-0.4.10                       76a35badf382        3 months ago        1.76GB
 hyperledger/fabric-baseimage                                                                                                                 s390x-0.4.10                       33ca13b136f1        3 months ago        1.4GB
 hyperledger/fabric-baseos                                                                                                                    s390x-0.4.10                       4bf08bb910ad        3 months ago        120MB

 
**Step 6.6:** You will now clean up. 
You will do this by running only the parts "hidden" within the *gulp test* command execution that do the initial cleanup::
 
 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ gulp clean-up pre-test docker-clean
 
**Step 6.7:** Now observe that all Docker containers have been stopped and removed by entering this command::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ docker ps -a
 CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
 
**Step 6.8:** And enter this comand and see that all chaincode images (those starting with *dev-*) have been removed::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ docker images
 REPOSITORY                     TAG                                IMAGE ID            CREATED             SIZE
 hyperledger/fabric-ca          latest                             91082da5dc41        About an hour ago   218MB
 hyperledger/fabric-ca          s390x-1.3.0-rc1                    91082da5dc41        About an hour ago   218MB
 hyperledger/fabric-tools       latest                             2f932afc62cf        2 hours ago         1.48GB
 hyperledger/fabric-tools       s390x-1.3.0-rc1-snapshot-d5c1514   2f932afc62cf        2 hours ago         1.48GB
 hyperledger/fabric-tools       s390x-latest                       2f932afc62cf        2 hours ago         1.48GB
 hyperledger/fabric-testenv     latest                             94f5cf342fd1        2 hours ago         1.53GB
 hyperledger/fabric-testenv     s390x-1.3.0-rc1-snapshot-d5c1514   94f5cf342fd1        2 hours ago         1.53GB
 hyperledger/fabric-testenv     s390x-latest                       94f5cf342fd1        2 hours ago         1.53GB
 hyperledger/fabric-buildenv    latest                             811f6744c029        2 hours ago         1.44GB
 hyperledger/fabric-buildenv    s390x-1.3.0-rc1-snapshot-d5c1514   811f6744c029        2 hours ago         1.44GB
 hyperledger/fabric-buildenv    s390x-latest                       811f6744c029        2 hours ago         1.44GB
 hyperledger/fabric-ccenv       latest                             446ba9534733        2 hours ago         1.39GB
 hyperledger/fabric-ccenv       s390x-1.3.0-rc1-snapshot-d5c1514   446ba9534733        2 hours ago         1.39GB
 hyperledger/fabric-ccenv       s390x-latest                       446ba9534733        2 hours ago         1.39GB
 hyperledger/fabric-orderer     latest                             402795f7129d        2 hours ago         142MB
 hyperledger/fabric-orderer     s390x-1.3.0-rc1-snapshot-d5c1514   402795f7129d        2 hours ago         142MB
 hyperledger/fabric-orderer     s390x-latest                       402795f7129d        2 hours ago         142MB
 hyperledger/fabric-peer        latest                             f28d66c114d4        2 hours ago         149MB
 hyperledger/fabric-peer        s390x-1.3.0-rc1-snapshot-d5c1514   f28d66c114d4        2 hours ago         149MB
 hyperledger/fabric-peer        s390x-latest                       f28d66c114d4        2 hours ago         149MB
 hyperledger/fabric-zookeeper   latest                             6a9cfd5ac47a        9 days ago          1.39GB
 hyperledger/fabric-kafka       latest                             8ce4902cb68e        9 days ago          1.4GB
 hyperledger/fabric-couchdb     latest                             2bc434828917        9 days ago          1.52GB
 hyperledger/fabric-baseimage   s390x-0.4.12                       a2d3919231fa        9 days ago          1.35GB
 hyperledger/fabric-baseos      s390x-0.4.12                       54e371e1a6ee        9 days ago          120MB
 hyperledger/fabric-ca          s390x-1.2.0                        7a9a8d2589c9        2 months ago        217MB
 hyperledger/fabric-orderer     s390x-1.2.0                        75b2abac5351        2 months ago        140MB
 hyperledger/fabric-peer        s390x-1.2.0                        85d0c3e2c248        2 months ago        147MB
 hyperledger/fabric-couchdb     s390x-0.4.10                       76a35badf382        3 months ago        1.76GB
 hyperledger/fabric-baseimage   s390x-0.4.10                       33ca13b136f1        3 months ago        1.4GB
 hyperledger/fabric-baseos      s390x-0.4.10                       4bf08bb910ad        3 months ago        120MB

**Recap:** In this section, you ran the Hyperledger Fabric Node.js SDK end-to-end tests and then you cleaned up its leftover artifacts afterward.
This completes this lab.  
You have downloaded and built a Hyperledger Fabric network and verified that the setup is correct by successfully running two end-to-end tests-  the CLI end-to-end test and the Node.js SDK end-to-end test- and the shorter Node.js SDK test.

If you really wanted to dig into the details of how the Hyperledger Fabric works, you could do worse than to drill down into the details of each of these tests.  

*** End of Lab! ***
