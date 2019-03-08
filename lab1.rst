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
They will need Public Key Infrastructure (PKI) x.509 certificates in order to participate.  
The Hyperledger Fabric Certificate Authority (CA) is provided by the Hyperledger Fabric project in order to issue these certificates.

The next major goal in this lab is to run the Hyperledger Fabric Node.js SDK’s end-to-end test.  
This test makes calls to the Hyperledger Fabric Certificate Authority (CA). 
Therefore, in order to get ready for that test, you will download and build the Hyperledger Fabric CA.

**Step 4.1:** Use *cd* to navigate three directory levels up, to the *hyperledger* directory::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric/examples/e2e_cli$ cd ~/git/src/github.com/hyperledger
 bcuser@ubuntu16045:~/git/src/github.com/hyperledger$

**Step 4.2:** Get the source code for the Fabric CA using *git*::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger$ git clone -b v1.4.0 --depth 1 https://gerrit.hyperledger.org/r/fabric-ca
 Cloning into 'fabric-ca'...
 remote: Counting objects: 2089, done
 remote: Finding sources: 100% (2089/2089)
 remote: Total 2089 (delta 188), reused 1860 (delta 188)
 Receiving objects: 100% (2089/2089), 6.04 MiB | 0 bytes/s, done.
 Resolving deltas: 100% (188/188), done.
 Checking connectivity... done.
 Note: checking out '27fbd69ab2d0f07212b382eb04aa85c904d2c300'.

 You are in 'detached HEAD' state. You can look around, make experimental
 changes and commit them, and you can discard any commits you make in this
 state without impacting any branches by performing another checkout.

 If you want to create a new branch to retain commits you create, you may
 do so (now or later) by using -b with the checkout command again. Example:

   git checkout -b <new-branch-name>


**Step 4.3:** Navigate to the *fabric-ca* directory, which is the top directory of where the *git* command put the code it just downloaded::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger$ cd fabric-ca
 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-ca$

**Step 4.4:** Enter the following command, which will build the Hyperledger Fabric CA image.  Just like you did with the *fabric* repo, ‘wrap’ the *make* command, which is what will do all the work, in a *time* command, which will give you a measure of the time, including ‘wall clock’ time, required to build the image::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-ca $ time FABRIC_CA_DYNAMIC_LINK=true make docker
   .
   .  (output not shown here)
   .
 real	1m29.510s
 user	0m0.313s
 sys	0m0.160s
 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-ca$

**Step 4.5:** Enter the *docker images* command and you will see at the top of the output the Docker image that was just created for the Fabric Certificate Authority::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-ca$ docker images
 REPOSITORY                      TAG                 IMAGE ID            CREATED              SIZE
 hyperledger/fabric-ca          latest                         e40a18deb6d6        6 seconds ago       329MB
 hyperledger/fabric-ca          s390x-1.4.0                    e40a18deb6d6        6 seconds ago       329MB
 hyperledger/fabric-tools       latest                         d08700aec890        About an hour ago   1.52GB
 hyperledger/fabric-tools       s390x-1.4.0-snapshot-d700b43   d08700aec890        About an hour ago   1.52GB
 hyperledger/fabric-tools       s390x-latest                   d08700aec890        About an hour ago   1.52GB
 <none>                         <none>                         fbbfa64f6939        About an hour ago   1.68GB
 hyperledger/fabric-buildenv    latest                         c869d419df0e        About an hour ago   1.47GB
 hyperledger/fabric-buildenv    s390x-1.4.0-snapshot-d700b43   c869d419df0e        About an hour ago   1.47GB
 hyperledger/fabric-buildenv    s390x-latest                   c869d419df0e        About an hour ago   1.47GB
 hyperledger/fabric-ccenv       latest                         609d68d7d5cb        About an hour ago   1.41GB
 hyperledger/fabric-ccenv       s390x-1.4.0-snapshot-d700b43   609d68d7d5cb        About an hour ago   1.41GB
 hyperledger/fabric-ccenv       s390x-latest                   609d68d7d5cb        About an hour ago   1.41GB
 hyperledger/fabric-orderer     latest                         3943ef06d94c        About an hour ago   147MB
 hyperledger/fabric-orderer     s390x-1.4.0-snapshot-d700b43   3943ef06d94c        About an hour ago   147MB
 hyperledger/fabric-orderer     s390x-latest                   3943ef06d94c        About an hour ago   147MB
 hyperledger/fabric-peer        latest                         cf05de3480f5        About an hour ago   153MB
 hyperledger/fabric-peer        s390x-1.4.0-snapshot-d700b43   cf05de3480f5        About an hour ago   153MB
 hyperledger/fabric-peer        s390x-latest                   cf05de3480f5        About an hour ago   153MB
 hyperledger/fabric-zookeeper   latest                         5db059b03239        4 months ago        1.42GB
 hyperledger/fabric-kafka       latest                         3bbd80f55946        4 months ago        1.43GB
 hyperledger/fabric-couchdb     latest                         7afa6ce179e6        4 months ago        1.55GB
 hyperledger/fabric-baseimage   s390x-0.4.14                   6e4e09df1428        4 months ago        1.38GB
 hyperledger/fabric-baseos      s390x-0.4.14                   4834a1e3ce1c        4 months ago        120MB

You may have noticed that for many of the images, the *Image ID* appears more than once, e.g., once with a tag of *latest*,  once with a tag such as *s390x-1.4.0-snapshot-d700b43*, and once with a tag of *s390x-latest*. An image can actually be given any number of tags. Think of these *tags* as nicknames, or aliases.  In our case the *make* process first gave the Docker image it created a descriptive tag, *s390x-1.4.0-snapshot-d700b43* in the case of the *fabric* repo, and *s390x-1.4.0* in the case of the *fabric-ca* repo, and then it also ‘tagged’ it with a new tag, *latest*.  It did that for a reason.  When you are working with Docker images, if you specify an image without specifying a tag, the tag defaults to the name *latest*. So, for example, using the above output, you can specify either *hyperledger/fabric-ca*, *hyperledger/fabric-ca:latest*, or *hyperledger/fabric-ca:s390x-1.4.0*, and in all three cases you are asking for the same image, the image with ID *e40a18deb6d6*.

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
 --2019-03-07 17:58:29--  https://nodejs.org/dist/v8.11.3/node-v8.11.3-linux-s390x.tar.xz
 Resolving nodejs.org (nodejs.org)... 104.20.22.46, 104.20.23.46, 2606:4700:10::6814:162e, ...
 Connecting to nodejs.org (nodejs.org)|104.20.22.46|:443... connected.
 HTTP request sent, awaiting response... 200 OK
 Length: 10939304 (10M) [application/x-xz]
 Saving to: 'node-v8.11.3-linux-s390x.tar.xz'

 node-v8.11.3-linux-s390x.tar.xz 100%[======================================================>]  10.43M  68.6MB/s    in 0.2s    

 2019-03-07 17:58:29 (68.6 MB/s) - 'node-v8.11.3-linux-s390x.tar.xz' saved [10939304/10939304]

**Step 5.3:** Extract the package underneath your home directory, */home/bcuser*. 
This will cause the executables to wind up in */home/bcuser/bin*, which is in your path::

 bcuser@ubuntu16045:/tmp$ cd /home/bcuser && tar --strip-components=1 -xf /tmp/node-v8.11.3-linux-s390x.tar.xz
 bcuser@ubuntu16045:~$ 

*Confession time:* */home/bcuser/bin* is in your path because I snuck that in there in *Step 2.38* while you were setting up *Go* even though that directory has nothing to do with Go!  That's called "killing two birds with one stone", a phrase soon to be made illegal and to be replaced with "feeding two birds with one scone".

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

**Step 5.9:** Now you will download the Hyperledger Fabric Node SDK source code from its official repository::

 bcuser@ubuntu16045: ~/git/src/github.com/hyperledger $ git clone -b v1.4.0 --depth 1 https://gerrit.hyperledger.org/r/fabric-sdk-node
 Cloning into 'fabric-sdk-node'...
 remote: Counting objects: 711, done
 remote: Finding sources: 100% (711/711)
 remote: Total 711 (delta 36), reused 527 (delta 36)
 Receiving objects: 100% (711/711), 847.68 KiB | 0 bytes/s, done.
 Resolving deltas: 100% (36/36), done.
 Checking connectivity... done.
 Note: checking out 'a8d4c2bd564d61e94e24d2c422d9d9816ba40ed0'.

 You are in 'detached HEAD' state. You can look around, make experimental
 changes and commit them, and you can discard any commits you make in this
 state without impacting any branches by performing another checkout.

 If you want to create a new branch to retain commits you create, you may
 do so (now or later) by using -b with the checkout command again. Example:

   git checkout -b <new-branch-name>

**Step 5.10:** Change to the *fabric-sdk-node* directory which was just created::

 bcuser@ubuntu16045: ~/git/src/github.com/hyperledger $ cd fabric-sdk-node
 bcuser@ubuntu16045: ~/git/src/github.com/hyperledger/fabric-sdk-node$

**Step 5.11:** You are about to install the packages that the Hyperledger Fabric Node SDK would like to use. Before you start, 
run *npm list* to see that you are starting with a blank slate::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ npm list
 fabric-sdk-node@1.4.0 /home/bcuser/git/src/github.com/hyperledger/fabric-sdk-node
 `-- (empty)
 
 bcuser@ubuntu16045: ~/git/src/github.com/hyperledger/fabric-sdk-node$

**Step 5.12:** Run *npm install* to install the required packages.  This will take a few minutes and will produce a lot of output::

 bcuser@ubuntu16045: ~/git/src/github.com/hyperledger/fabric-sdk-node$ npm install
   .
   . (output not shown here)
   .
 npm notice created a lockfile as package-lock.json. You should commit this file.
 npm WARN gulp-debug@4.0.0 requires a peer of gulp@>=4 but none is installed. You must install peer dependencies yourself.
 npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@1.2.7 (node_modules/fsevents):
 npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@1.2.7: wanted {"os":"darwin","arch":"any"} (current: {"os":"linux","arch":"s390x"})

 added 1512 packages in 100.572s

You may ignore the *WARN* messages throughout the output, and there may even be some messages that look like error messages, but the npm installation program may be expecting such conditions and working through it.  If there is a serious error, the end of the output will leave little doubt about it.

**Step 5.13:** Repeat the *npm list* command.  The output, although not shown here, will be anything but empty.  This just proves what everyone suspected-  programmers would much rather use other peoples’ code than write their own.  Not that there’s anything wrong with that. You can even steal this lab if you want to.
::
 bcuser@ubuntu16045: ~/git/src/github.com/hyperledger/fabric-sdk-node$ npm list
   .
   . (output not shown here, but surely you will agree it is not empty)
   .
 bcuser@ubuntu16045: ~/git/src/github.com/hyperledger/fabric-sdk-node$

**Step 5.14:** The tests use an automation tool named *gulp*, but before you install it, 
run the *which* command. The silent treatment it gives you confirms it is not available to you::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ which gulp
 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ 

**Step 5.15:** Now install *gulp* at a global level, using the *-g* argument to the *npm install*. 
This makes the package  available on a system-wide basis::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ npm install -g gulp
 npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@1.2.7 (node_modules/gulp/node_modules/fsevents):
 npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@1.2.7: wanted {"os":"darwin","arch":"any"} (current: {"os":"linux","arch":"s390x"})

 + gulp@4.0.0
 added 316 packages in 9.875s

**NOTE:** These warnings can be ignored for the purposes of this lab, but in a production environment you should probably take the warnings more seriously.
 
**Step 5.16:** Running *which* again shows that *gulp* is available to you now::
 
 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ which gulp
 /home/bcuser/bin/gulp
 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$

**Step 5.17:** Next you will install a code coverage testing tool named *istanbul*, also at a global level.  
But first, use *which* to prove it isn't there yet::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ which istanbul
 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ 
 
**Step 5.18:** Install it globally::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ npm install -g istanbul
 npm WARN deprecated istanbul@0.4.5: This module is no longer maintained, try this instead:
 npm WARN deprecated   npm i nyc
 npm WARN deprecated Visit https://istanbul.js.org/integrations for other alternatives.
 /home/bcuser/bin/istanbul -> /home/bcuser/lib/node_modules/istanbul/lib/cli.js
 + istanbul@0.4.5
 added 48 packages in 1.93s

**Step 5.19:** Prove it worked::

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

The first test is a quick test that takes about a minute and does not bring up any chaincode containers.  The second test is the "end-to-end" test, as it is much more comprehensive and will bring up several chaincode containers and will take several minutes.

**Step 6.1:** The first test is very simple and can be run simply by running *npm test*.
I am going to show some snippets from the output below- there will be much more output between the snippets I'm showing here.
You can watch the test progress and then when it ends, see if you can scroll up through the output and find what I'm showing here::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ npm test
 > fabric-sdk-node@1.4.0 test /home/bcuser/git/src/github.com/hyperledger/fabric-sdk-node
 > gulp test-headless


 ####################################################
 # debug log: /tmp/hfc/test-log/debug.log
 ####################################################

 [ 16:28:49] Using gulpfile ~/git/src/github.com/hyperledger/fabric-sdk-node/gulpfile.js
 [ 16:28:49] Starting 'test-headless'...

 ####################################################
 # debug log: /tmp/hfc/test-log/debug.log
 ####################################################

 [16:28:50] Using gulpfile ~/git/src/github.com/hyperledger/fabric-sdk-node/gulpfile.js
 [16:28:50] Starting 'run-test-headless'...
 [16:28:50] Starting 'clean-up'...
 [16:28:50] Finished 'clean-up' after 967 μs
 [16:28:50] Starting 'pre-test'...
 [16:28:50] Finished 'pre-test' after 65 ms
 [16:28:50] Starting 'ca'...
   .
   . (output not shown)
   .
 
  188 passing (487ms)

 [16:29:01] Finished 'mocha-fabric-ca-client' after 2.2 s
 [16:29:01] Starting 'mocha-fabric-client'...

   .
   . (output not shown)
   .
 
  249 passing (2s)

 [16:29:17] Finished 'mocha-fabric-network' after 4.52 s
 [16:29:17] Finished 'run-test-mocha' after 18 s
 [16:29:17] Starting 'run-tape-unit'...

   .
   . (output not shown)
   .

 1..685
 # tests 685
 # pass  685

 # ok

 [16:30:00] Finished 'run-tape-unit' after 43 s
 [16:30:00] Starting 'run-logger-unit'...

 1..11
 # tests 11
 # pass  11

 # ok

 [16:30:01] Finished 'run-logger-unit' after 1.05 s
 ERROR: Coverage for lines (84.28%) does not meet global threshold (85%)
 ERROR: Coverage for branches (79.42%) does not meet global threshold (80%)
 ERROR: Coverage for statements (84.3%) does not meet global threshold (85%)
 -----------------------------------|----------|----------|----------|----------|-------------------|
 File                               |  % Stmts | % Branch |  % Funcs |  % Lines | Uncovered Line #s |
 -----------------------------------|----------|----------|----------|----------|-------------------|
 All files                          |     84.3 |    79.42 |    91.11 |    84.28 |                   |
  fabric-ca-client/lib              |      100 |    99.02 |      100 |      100 |                   |
   AffiliationService.js            |      100 |      100 |      100 |      100 |                   |
   IdentityService.js               |      100 |      100 |      100 |      100 |                   |
   helper.js                        |      100 |       95 |      100 |      100 |                67 |
  fabric-client/lib                 |    86.26 |    84.58 |    95.98 |    86.25 |                   |
   BaseClient.js                    |      100 |      100 |      100 |      100 |                   |
   BlockDecoder.js                  |      100 |      100 |      100 |      100 |                   |
   CertificateAuthority.js          |      100 |      100 |      100 |      100 |                   |
   Channel.js                       |    59.34 |    56.19 |    83.33 |    59.31 |... 4043,4045,4047 |
   ChannelEventHub.js               |    99.18 |       99 |      100 |    99.18 |... 12,413,547,548 |
   Client.js                        |      100 |    99.75 |      100 |      100 |               162 |
   Config.js                        |      100 |      100 |      100 |      100 |                   |
   Constants.js                     |      100 |      100 |      100 |      100 |                   |
   Orderer.js                       |      100 |      100 |      100 |      100 |                   |
   Organization.js                  |      100 |      100 |      100 |      100 |                   |
   Package.js                       |      100 |      100 |      100 |      100 |                   |
   Packager.js                      |      100 |      100 |      100 |      100 |                   |
   Peer.js                          |      100 |       95 |      100 |      100 |             74,79 |
   Policy.js                        |      100 |    93.88 |      100 |      100 |        81,171,191 |
   ProtoLoader.js                   |      100 |      100 |      100 |      100 |                   |
   Remote.js                        |      100 |      100 |      100 |      100 |                   |
   SideDB.js                        |      100 |      100 |      100 |      100 |                   |
   TransactionID.js                 |      100 |      100 |      100 |      100 |                   |
   User.js                          |      100 |    98.33 |      100 |      100 |                61 |
   api.js                           |      100 |      100 |      100 |      100 |                   |
   client-utils.js                  |      100 |      100 |      100 |      100 |                   |
   hash.js                          |      100 |      100 |      100 |      100 |                   |
   utils.js                         |    89.41 |     87.9 |    94.29 |    89.41 |... 59,561,563,566 |
  fabric-client/lib/impl            |    69.02 |    63.78 |    70.99 |    68.99 |                   |
   BasicCommitHandler.js            |    73.33 |       70 |      100 |    73.33 |... 19,120,123,124 |
   CouchDBKeyValueStore.js          |    76.71 |       60 |    93.33 |    76.71 |... 50,152,166,167 |
   CryptoKeyStore.js                |      100 |     87.5 |      100 |      100 |             43,77 |
   CryptoSuite_ECDSA_AES.js         |    86.23 |    83.91 |    78.95 |    86.23 |... 78,307,324,330 |
   DiscoveryEndorsementHandler.js   |    78.68 |    65.85 |      100 |    78.68 |... 42,444,452,454 |
   FileKeyValueStore.js             |    91.89 |    83.33 |      100 |    91.89 |          47,48,65 |
   NetworkConfig_1_0.js             |     98.3 |    87.08 |      100 |     98.3 |   381,395,428,429 |
   bccsp_pkcs11.js                  |    25.88 |    32.41 |     8.33 |    25.88 |... 1075,1079,1080 |
  fabric-client/lib/impl/aes        |    11.11 |        0 |        0 |    11.11 |                   |
   pkcs11_key.js                    |    11.11 |        0 |        0 |    11.11 |... 46,50,54,58,62 |
  fabric-client/lib/impl/ecdsa      |    49.29 |    31.25 |       45 |    49.29 |                   |
   key.js                           |    98.41 |    96.15 |      100 |    98.41 |               183 |
   pkcs11_key.js                    |     9.09 |        0 |        0 |     9.09 |... 71,175,176,179 |
  fabric-client/lib/msp             |    79.65 |    69.62 |    73.33 |    79.65 |                   |
   identity.js                      |    87.04 |    75.76 |    76.92 |    87.04 |... 97,107,203,230 |
   msp-manager.js                   |    86.54 |    77.27 |      100 |    86.54 |... 1,82,83,84,155 |
   msp.js                           |    68.18 |    54.17 |       50 |    68.18 |... 35,137,138,179 |
  fabric-client/lib/packager        |      100 |      100 |      100 |      100 |                   |
   BasePackager.js                  |      100 |      100 |      100 |      100 |                   |
   Car.js                           |      100 |      100 |      100 |      100 |                   |
   Golang.js                        |      100 |      100 |      100 |      100 |                   |
   Java.js                          |      100 |      100 |      100 |      100 |                   |
   Node.js                          |      100 |      100 |      100 |      100 |                   |
  fabric-client/lib/utils           |      100 |      100 |      100 |      100 |                   |
   ChannelHelper.js                 |      100 |      100 |      100 |      100 |                   |
  fabric-network/lib                |      100 |      100 |      100 |      100 |                   |
   contract.js                      |      100 |      100 |      100 |      100 |                   |
   gateway.js                       |      100 |      100 |      100 |      100 |                   |
   logger.js                        |      100 |      100 |      100 |      100 |                   |
   network.js                       |      100 |      100 |      100 |      100 |                   |
   transaction.js                   |      100 |      100 |      100 |      100 |                   |
  fabric-network/lib/api            |      100 |      100 |      100 |      100 |                   |
   queryhandler.js                  |      100 |      100 |      100 |      100 |                   |
   wallet.js                        |      100 |      100 |      100 |      100 |                   |
  fabric-network/lib/impl/event     |      100 |      100 |      100 |      100 |                   |
   abstracteventstrategy.js         |      100 |      100 |      100 |      100 |                   |
   allfortxstrategy.js              |      100 |      100 |      100 |      100 |                   |
   anyfortxstrategy.js              |      100 |      100 |      100 |      100 |                   |
   defaulteventhandlerstrategies.js |      100 |      100 |      100 |      100 |                   |
   eventhubfactory.js               |      100 |      100 |      100 |      100 |                   |
   transactioneventhandler.js       |      100 |      100 |      100 |      100 |                   |
  fabric-network/lib/impl/query     |      100 |      100 |      100 |      100 |                   |
   defaultqueryhandler.js           |      100 |      100 |      100 |      100 |                   |
  fabric-network/lib/impl/wallet    |      100 |      100 |      100 |      100 |                   |
   basewallet.js                    |      100 |      100 |      100 |      100 |                   |
   couchdbwallet.js                 |      100 |      100 |      100 |      100 |                   |
   filesystemwallet.js              |      100 |      100 |      100 |      100 |                   |
   hsmwalletmixin.js                |      100 |      100 |      100 |      100 |                   |
   inmemorywallet.js                |      100 |      100 |      100 |      100 |                   |
   x509walletmixin.js               |      100 |      100 |      100 |      100 |                   |
 -----------------------------------|----------|----------|----------|----------|-------------------|

 =============================== Coverage summary ===============================
 Statements   : 84.3% ( 6146/7291 )
 Branches     : 79.42% ( 2501/3149 )
 Functions    : 91.11% ( 871/956 )
 Lines        : 84.28% ( 6137/7282 )
 ================================================================================
 [18:35:46] 'test-headless' errored after 1.28 min
 [18:35:46] Error in plugin "gulp-shell"
 Message:
     Command `npx nyc gulp run-test-headless` failed with exit code 1
 npm ERR! Test failed.  See above for more details. 

Like Tom Brady, I hate to fail. 
What's going on, here?
Well the test has some thresholds for how much of the code in various places should be covered by the test. 
Scroll back up past those crazy columns of numbers and observe these error messages again::

  ERROR: Coverage for lines (84.36%) does not meet global threshold (85%)
  ERROR: Coverage for branches (79.52%) does not meet global threshold (80%)
  ERROR: Coverage for statements (84.38%) does not meet global threshold (85%)

Way back in Hyperledger Fabric 1.3 the coverage for branches was only 77.59% and the Fabric Node.js SDK developers got that coverage all the way up to 79.42%, and instead of being thanked for it, what do they get?
A big fat ERROR message.
They do and do and do for us and this is the thanks they get?
Sort of like your sales quota gets raised every year so you just can't quite reach it, isn't it?
Gentlemen, it's sorta' like the reaction you get from your wife when you buy her a vacuum cleaner for your anniversary.  
**ERROR**.  Even if it is a top-of-the-line Dyson.  Don't ask me how I know this.

**Step 6.2:** The prior test was quick.  The functionality it exercised did not actually require any chaincode Docker images or containers. This next test does. It sure does. Run this end-to-end test with the *gulp test* command.  While this command is running, a little bit of the output may look like errors, but some of the tests expect errors, so the real indicator is whether or not all tests passed. Enter this command and while you're waiting for the command to complete, scroll down in this lab and read the commentary after the command output. After you've done that and finished your lunch, scroll back up through the output as much as you can or feel like it, and see if you can find the snippets of output I've chosen to show here::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ gulp test
   .
   . (lots of output not shown here)
   . 
 
  188 passing (468ms)

 [18:29:23] Finished 'mocha-fabric-ca-client' after 1.63 s
 [18:29:23] Starting 'mocha-fabric-client'...

   .
   . (lots of output not shown here)
   . 
 

  1115 passing (2s)

 [18:29:28] Finished 'mocha-fabric-client' after 4.22 s
 [18:29:28] Starting 'mocha-fabric-network'...

   .
   . (lots of output not shown here)
   . 

  249 passing (2s)

 [18:29:31] Finished 'mocha-fabric-network' after 3.34 s
 [18:29:31] Finished 'run-test-mocha' after 9.19 s
 [18:29:31] Starting 'run-tape-unit'...

   .
   . (lots of output not shown here)
   . 

 1..685
 # tests 685
 # pass  685

 # ok

   .
   . (lots of output not shown here)
   . 

 1..1358
 # tests 1358
 # pass  1358

 # ok

   .
   . (lots of output not shown here)
   . 

 1..17
 # tests 17
 # pass  17

 # ok

   .
   . (lots of output not shown here)
   . 


 3 scenarios (3 passed)
 20 steps (20 passed)
 5m34.900s
 [18:56:30] Finished 'run-test-cucumber' after 5.6 min
 [18:56:30] Finished 'run-test' after 27 min
 -----------------------------------|----------|----------|----------|----------|-------------------|
 File                               |  % Stmts | % Branch |  % Funcs |  % Lines | Uncovered Line #s |
 -----------------------------------|----------|----------|----------|----------|-------------------|
 All files                          |    92.17 |     85.9 |    93.41 |    92.16 |                   |
  fabric-ca-client/lib              |      100 |    99.02 |      100 |      100 |                   |
   AffiliationService.js            |      100 |      100 |      100 |      100 |                   |
   IdentityService.js               |      100 |      100 |      100 |      100 |                   |
   helper.js                        |      100 |       95 |      100 |      100 |                67 |
  fabric-client/lib                 |    97.62 |    93.65 |    99.82 |    97.62 |                   |
   BaseClient.js                    |      100 |      100 |      100 |      100 |                   |
   BlockDecoder.js                  |      100 |      100 |      100 |      100 |                   |
   CertificateAuthority.js          |      100 |      100 |      100 |      100 |                   |
   Channel.js                       |    92.87 |    82.86 |    99.17 |    92.87 |... 4001,4002,4043 |
   ChannelEventHub.js               |      100 |      100 |      100 |      100 |                   |
   Client.js                        |      100 |    99.75 |      100 |      100 |               162 |
   Config.js                        |      100 |      100 |      100 |      100 |                   |
   Constants.js                     |      100 |      100 |      100 |      100 |                   |
   Orderer.js                       |      100 |      100 |      100 |      100 |                   |
   Organization.js                  |      100 |      100 |      100 |      100 |                   |
   Package.js                       |      100 |      100 |      100 |      100 |                   |
   Packager.js                      |      100 |      100 |      100 |      100 |                   |
   Peer.js                          |      100 |       95 |      100 |      100 |             74,79 |
   Policy.js                        |      100 |    93.88 |      100 |      100 |        81,171,191 |
   ProtoLoader.js                   |      100 |      100 |      100 |      100 |                   |
   Remote.js                        |      100 |      100 |      100 |      100 |                   |
   SideDB.js                        |      100 |      100 |      100 |      100 |                   |
   TransactionID.js                 |      100 |      100 |      100 |      100 |                   |
   User.js                          |      100 |    98.33 |      100 |      100 |                61 |
   api.js                           |      100 |      100 |      100 |      100 |                   |
   client-utils.js                  |      100 |      100 |      100 |      100 |                   |
   hash.js                          |      100 |      100 |      100 |      100 |                   |
   utils.js                         |    98.31 |    92.74 |      100 |    98.31 |   128,174,178,563 |
  fabric-client/lib/impl            |    72.21 |    67.45 |    70.99 |    72.18 |                   |
   BasicCommitHandler.js            |    88.33 |       85 |      100 |    88.33 |... 19,120,123,124 |
   CouchDBKeyValueStore.js          |    80.82 |    63.33 |    93.33 |    80.82 |... 50,152,166,167 |
   CryptoKeyStore.js                |      100 |     87.5 |      100 |      100 |             43,77 |
   CryptoSuite_ECDSA_AES.js         |    86.23 |    83.91 |    78.95 |    86.23 |... 78,307,324,330 |
   DiscoveryEndorsementHandler.js   |    88.24 |    78.05 |      100 |    88.24 |... 42,444,452,454 |
   FileKeyValueStore.js             |    91.89 |    83.33 |      100 |    91.89 |          47,48,65 |
   NetworkConfig_1_0.js             |     98.3 |    90.45 |      100 |     98.3 |   381,395,428,429 |
   bccsp_pkcs11.js                  |    25.88 |    32.41 |     8.33 |    25.88 |... 1075,1079,1080 |
  fabric-client/lib/impl/aes        |    11.11 |        0 |        0 |    11.11 |                   |
   pkcs11_key.js                    |    11.11 |        0 |        0 |    11.11 |... 46,50,54,58,62 |
  fabric-client/lib/impl/ecdsa      |    49.29 |    31.25 |       45 |    49.29 |                   |
   key.js                           |    98.41 |    96.15 |      100 |    98.41 |               183 |
   pkcs11_key.js                    |     9.09 |        0 |        0 |     9.09 |... 71,175,176,179 |
  fabric-client/lib/msp             |    80.81 |    72.15 |    76.67 |    80.81 |                   |
   identity.js                      |    90.74 |    78.79 |    84.62 |    90.74 |  82,94,97,107,203 |
   msp-manager.js                   |    86.54 |    81.82 |      100 |    86.54 |... 1,82,83,84,155 |
   msp.js                           |    68.18 |    54.17 |       50 |    68.18 |... 35,137,138,179 |
  fabric-client/lib/packager        |      100 |      100 |      100 |      100 |                   |
   BasePackager.js                  |      100 |      100 |      100 |      100 |                   |
   Car.js                           |      100 |      100 |      100 |      100 |                   |
   Golang.js                        |      100 |      100 |      100 |      100 |                   |
   Java.js                          |      100 |      100 |      100 |      100 |                   |
   Node.js                          |      100 |      100 |      100 |      100 |                   |
  fabric-client/lib/utils           |      100 |      100 |      100 |      100 |                   |
   ChannelHelper.js                 |      100 |      100 |      100 |      100 |                   |
  fabric-network/lib                |      100 |      100 |      100 |      100 |                   |
   contract.js                      |      100 |      100 |      100 |      100 |                   |
   gateway.js                       |      100 |      100 |      100 |      100 |                   |
   logger.js                        |      100 |      100 |      100 |      100 |                   |
   network.js                       |      100 |      100 |      100 |      100 |                   |
   transaction.js                   |      100 |      100 |      100 |      100 |                   |
  fabric-network/lib/api            |      100 |      100 |      100 |      100 |                   |
   queryhandler.js                  |      100 |      100 |      100 |      100 |                   |
   wallet.js                        |      100 |      100 |      100 |      100 |                   | 
  fabric-network/lib/impl/event     |      100 |      100 |      100 |      100 |                   |
   abstracteventstrategy.js         |      100 |      100 |      100 |      100 |                   |
   allfortxstrategy.js              |      100 |      100 |      100 |      100 |                   |
   anyfortxstrategy.js              |      100 |      100 |      100 |      100 |                   |
   defaulteventhandlerstrategies.js |      100 |      100 |      100 |      100 |                   |
   eventhubfactory.js               |      100 |      100 |      100 |      100 |                   |
   transactioneventhandler.js       |      100 |      100 |      100 |      100 |                   |
  fabric-network/lib/impl/query     |      100 |      100 |      100 |      100 |                   |
   defaultqueryhandler.js           |      100 |      100 |      100 |      100 |                   |
  fabric-network/lib/impl/wallet    |      100 |      100 |      100 |      100 |                   |
   basewallet.js                    |      100 |      100 |      100 |      100 |                   |
   couchdbwallet.js                 |      100 |      100 |      100 |      100 |                   |
   filesystemwallet.js              |      100 |      100 |      100 |      100 |                   |
   hsmwalletmixin.js                |      100 |      100 |      100 |      100 |                   |
   inmemorywallet.js                |      100 |      100 |      100 |      100 |                   |
   x509walletmixin.js               |      100 |      100 |      100 |      100 |                   |
 -----------------------------------|----------|----------|----------|----------|-------------------|

 =============================== Coverage summary ===============================
 Statements   : 92.17% ( 6720/7291 )
 Branches     : 85.9% ( 2705/3149 )
 Functions    : 93.41% ( 893/956 )
 Lines        : 92.16% ( 6711/7282 )
 ================================================================================
 [18:56:31] Finished 'test' after 27 min
 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$

*The commentary after the command output*: Now that you've read that, read the rest of the paragraph- the Fabric Node.js SDK developers, like all good developers, have been continuously adding functionality, and test cases to the test, throughout the life of the project.  Why, when I was a kid, this test only took eight minutes to run, and we liked it! Then it took about twelve minutes, then eighteen minutes, and now it takes about twenty-eight minutes!  (Your mileage may vary). 

Now is a good time to either go get your lunch, check your e-mail, visit your stockbroker, run to the Department of Motor Vehicles to renew your drivers license, or if you're rather technical, see if you can look into the *~/git/src/github.com/hyperledger/fabric-sdk-node/gulpfile.js* file- you'll have to open up another ssh or PuTTY session and log in, but you are rather technical aren't you- and see if you can figure out what the heck is going on in all these tests.  I can't, but you're smarter than me, right?  I can't prove it but you're sitting comfortably in your chair right now eating your lunch while I'm going around the room trying to help everybody else figure out why their lab is broken, and you took the last bag of chips. What, you didn't get your lunch yet?  You'd better hurry before all the chips are gone.

**STOP HERE AND DON'T READ FURTHER UNTIL THE TEST ENDS OR YOU WILL TURN INTO A PILLAR OF SALT**

You just couldn't resist, could you?  I lied about the pillar of salt bit.  And if I didn't, we'll all think of you fondly whenever we're eating our popcorn at the movies.

Did you notice that this test passed while the previous test didn't?  
For example in this test the *Branches* test coverage was 85.9% versus a paltry 79.52% in the prior test?  (Did I mention the Blockchain use case where NASA has proposed to use Hyperledger Fabric for air traffic management- I think 85% coverage is good enough for that, don't you, I mean, 85.9% would get you a rock solid B in most leading universities).  You think I'm making this up about the NASA use case but I'm not, I could give you a URL to prove it, but I don't recommend URLs without checking them out first, and I haven't read the paper yet, but I will on my next Greyhound bus trip from Maryland to Oregon. I didn't even know NASA was involved in air traffic management.  I guess they're getting ready for the future when Elon Musk has made rocket ships affordable to the masses-  by then test coverage should be up to 88% or so.

**Step 6.3:** Enter this command to see what Docker containers were created as part of the test::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ docker ps --all

**Step 5.4:** Enter this command to see that some Docker images for chaincode have been created as part of the test.  
These are the images that start with *dev-*::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ docker images dev-*
 
You may only see two Docker images. This test created a whole lot more but cleaned up most of them.  In earlier releases of Hyperledger Fabric all of the Docker images were left but the test has been modified to clean up most of them during the test.  Who says things aren't getting better?

**Step 5.5:** You will now clean up what little detritus remains. You will do this by running only the parts "hidden" within the *gulp test* command execution that do the initial cleanup::
 
 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ gulp clean-up docker-clean pre-test

This command will remove the Docker chaincode containers and images but the Docker containers for the Hyperledger Fabric network itself are still running.

**Step 5.6:** This command will show the Docker containers for the Hyperledger Fabric network::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ docker ps --all
 CONTAINER ID        IMAGE                                    COMMAND                  CREATED             STATUS              PORTS                                        NAMES
 eaf59e0106ef        hyperledger/fabric-peer:s390x-1.4.0      "peer node start"        About an hour ago   Up About an hour    0.0.0.0:8051->8051/tcp                       peer0.org2.example.com
 b45573f6d3d1        hyperledger/fabric-peer:s390x-1.4.0      "peer node start"        About an hour ago   Up About an hour    0.0.0.0:7051->7051/tcp                       peer0.org1.example.com
 0af56671c6fd        hyperledger/fabric-couchdb               "tini -- /docker-ent…"   About an hour ago   Up About an hour    4369/tcp, 9100/tcp, 0.0.0.0:5984->5984/tcp   couchdb.org1.example.com
 778a70d02db6        hyperledger/fabric-couchdb               "tini -- /docker-ent…"   About an hour ago   Up About an hour    4369/tcp, 9100/tcp, 0.0.0.0:6984->5984/tcp   couchdb.org2.example.com
 3da7a81000e7        hyperledger/fabric-orderer:s390x-1.4.0   "orderer"                About an hour ago   Up About an hour    0.0.0.0:7050->7050/tcp                       orderer.example.com
 99cfc0980995        hyperledger/fabric-ca:s390x-1.4.0        "sh -c 'fabric-ca-se…"   About an hour ago   Up About an hour    0.0.0.0:7054->7054/tcp                       ca0.example.com
 95c793434e52        hyperledger/fabric-ca:s390x-1.4.0        "sh -c 'fabric-ca-se…"   About an hour ago   Up About an hour    0.0.0.0:8054->7054/tcp                       ca1.example.com

**Step 5.7:** Run this set of commands in one fell swoop to stop the Hyperledger Fabric network and remove its containers::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ docker stop $(docker ps --quiet) && docker rm $(docker ps --all --quiet)
 eaf59e0106ef
 b45573f6d3d1
 0af56671c6fd
 778a70d02db6
 3da7a81000e7
 99cfc0980995
 95c793434e52
 eaf59e0106ef
 b45573f6d3d1
 0af56671c6fd
 778a70d02db6
 3da7a81000e7
 99cfc0980995
 95c793434e52

**Step 5.8:** Run this command to see that all your Docker containers are gone::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ docker ps --all
 CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

**Step 5.9:** Run this command to see that all your Docker chaincode images are gone::

  bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ docker images dev-*
 REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE

**Step 5.10:** The Docker chaincode images are gone but the Docker images for the Hyperledger Fabric components themselves still remain.  Prove it with this command::

 REPOSITORY                     TAG                            IMAGE ID            CREATED             SIZE
 hyperledger/fabric-ca          latest                         fb40d26bc7a1        5 hours ago         317MB
 hyperledger/fabric-ca          s390x-1.4.0                    fb40d26bc7a1        5 hours ago         317MB
 hyperledger/fabric-tools       latest                         36d8a7db8056        3 days ago          1.52GB
 hyperledger/fabric-tools       s390x-1.4.0-snapshot-d700b43   36d8a7db8056        3 days ago          1.52GB
 hyperledger/fabric-tools       s390x-latest                   36d8a7db8056        3 days ago          1.52GB
 <none>                         <none>                         10ea31c0bee7        3 days ago          1.58GB
 hyperledger/fabric-buildenv    latest                         172f9563573f        3 days ago          1.47GB
 hyperledger/fabric-buildenv    s390x-1.4.0-snapshot-d700b43   172f9563573f        3 days ago          1.47GB
 hyperledger/fabric-buildenv    s390x-latest                   172f9563573f        3 days ago          1.47GB
 hyperledger/fabric-ccenv       latest                         602af5999747        3 days ago          1.41GB
 hyperledger/fabric-ccenv       s390x-1.4.0-snapshot-d700b43   602af5999747        3 days ago          1.41GB
 hyperledger/fabric-ccenv       s390x-latest                   602af5999747        3 days ago          1.41GB
 hyperledger/fabric-orderer     latest                         63f428514b07        3 days ago          147MB
 hyperledger/fabric-orderer     s390x-1.4.0-snapshot-d700b43   63f428514b07        3 days ago          147MB
 hyperledger/fabric-orderer     s390x-latest                   63f428514b07        3 days ago          147MB
 hyperledger/fabric-peer        latest                         b415d479b21c        3 days ago          153MB
 hyperledger/fabric-peer        s390x-1.4.0-snapshot-d700b43   b415d479b21c        3 days ago          153MB
 hyperledger/fabric-peer        s390x-latest                   b415d479b21c        3 days ago          153MB
 hyperledger/fabric-orderer     s390x-1.4.0                    a8875e4d43b3        9 days ago          147MB
 hyperledger/fabric-peer        s390x-1.4.0                    598805b785db        9 days ago          153MB
 hyperledger/fabric-zookeeper   latest                         5db059b03239        3 months ago        1.42GB
 hyperledger/fabric-kafka       latest                         3bbd80f55946        3 months ago        1.43GB
 hyperledger/fabric-couchdb     latest                         7afa6ce179e6        3 months ago        1.55GB
 hyperledger/fabric-couchdb     s390x-0.4.14                   7afa6ce179e6        3 months ago        1.55GB
 hyperledger/fabric-baseimage   s390x-0.4.14                   6e4e09df1428        3 months ago        1.38GB
 hyperledger/fabric-baseos      s390x-0.4.14                   4834a1e3ce1c        3 months ago        120MB

**Step 5.11:** This command will remove all Docker images from your system. Run this if you want to clean up completely, but keep them around if you want to experiment on your own further or if your instructor asks you to stop here to leave these images intact on your system. **CAUTION:** These instructions are written for a lab environment and if you run the below command on a production system or even another development system with lots of other Docker images then you will need to either learn how to find another job or learn how to make withdrawals from your 401k plan in the very near future::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ docker images | awk '{print $1":"$2}' | xargs docker rmi

**Step 5.12:** Run this command again and you will see that this '<none>:<none>' stickler is still hanging around::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ docker images
 REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
 <none>              <none>              10ea31c0bee7        3 days ago          1.58GB
 
That troublesome image is some orphaned artifact that is a remnant of the *make* process from the *fabric* repo that the Fabric developers are too busy to cleanup, but you can do it manually in the next step

**Step 5.13:** This is the next step::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ docker rmi $(docker images --quiet)
 Deleted: sha256:10ea31c0bee798091fc311eebcbd2cde7bbd29030c81803dfa5c817ef2c97750
 Deleted: sha256:37cd100376dec6fee97b3145d36aad8b93f3b79ad23edfc88844dad6d83cc03a
 Deleted: sha256:ec2a09c857344d9c053346a88d69b9b6bb31b97304cec8fa0c2a830752e0b19f
 Deleted: sha256:4aae6fae286999b6d3ff0d970f0f0181959bacb8f3f9a2da82681051af6e42f5
 Deleted: sha256:67326600a867c380ea782ce36cfb13d1acc26d1f0aadf34eed46809e2b775ec2
 Deleted: sha256:a59823336df743dedd1391e0c33c1fcdafd4cebcdbca21f2a8a9332700ca7173
 Deleted: sha256:fca0a9f5fcb546e416436f2108e72cb8846b45895423173660b0d7f2659cfb70
 Deleted: sha256:1befa139c65f7b7491f0f8526c770dd704e98e25b1d90a1ed0ec885506336603
 Deleted: sha256:58b8d6c5fddfb50c4aa1f3c27e87d84741fc166ef2e23ab3d3f546d5b0ed2aec
 Deleted: sha256:6e4e09df14286dc6c7148f8e90e5a0c4ddf89e994de2954f07bc2b1ad09aeee8
 Deleted: sha256:1382bd5bc7bca39119d781098b5c4d0b30f19e72b709ffe1285bc685912e2fc0
 Deleted: sha256:13e30ed80dc9f7bda2e2854f87b6347e9529d23667ea2a6370966de8164e3bfc
 Deleted: sha256:8f0f56205c454bee3c78a30930b6e8bf6116efe68bba628c89a67f90a0502487
 Deleted: sha256:c07b03ff692551cb9124da9c7fd732abf5baecb05cfc4defd56b2957a1add578
 Deleted: sha256:edb964a46867e04b12fc5b9d2f0856a56cbd89fc2915212e7c46dad319c295c2
 Deleted: sha256:0ac5920f1daf502602a385b7878b587a9e37d89c2d3706c7e9f96e07849d2830
 Deleted: sha256:2f643c4c2c261dc7b616bfe820c537e702e7da928c99ade0c8610e9f8e344e5e

**Step 5.14:** Prove to yourself you're all out of Docker images::

 bcuser@ubuntu16045:~/git/src/github.com/hyperledger/fabric-sdk-node$ docker images
 REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
 
Don't worry, you can make more. But not in this lab.  This is the end.

**Recap:** In this section, you ran the Hyperledger Fabric Node.js SDK end-to-end tests and then you cleaned up its leftover artifacts afterward. This completes this lab.  You have downloaded and run a Hyperledger Fabric network and verified that the setup is correct by successfully running two end-to-end tests-  the *fabric-samples* "first network" test and the Node.js SDK end-to-end test- and the shorter Node.js SDK test.

If you really wanted to dig into the details of how Hyperledger Fabric works, you could do worse than to drill down into the details of each of these tests.  

*** End of Lab! ***
