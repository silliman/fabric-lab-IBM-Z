Section 1:  Lab Overview
========================
In this lab, you will start with a basic Ubuntu 16.04 server instance running on an IBM z13 Server that resides in the IBM Washington Systems Center in Herndon, Virginia.  During the installation process for this instance, no extra packages were selected for installation beyond the minimum offered by the installer.  Ubuntu updates were applied such that at this time the level of Ubuntu is *16.04.3 LTS* and the Linux kernel is at level *4.4.0-96*.

You will install the necessary software prerequisites to build and test a Hyperledger Fabric network, including

*	Docker and Docker Compose
*	Node.js and npm
*	Golang compiler
*	other necessary packages
During the lab, you will download three Hyperledger Fabric source code repositories, build the necessary artifacts from the source code, and run two comprehensive end-to-end tests to verify that your Hyperledger Fabric installation is in working order.

You will first download the *fabric* repository, which contains the source code for the core functionality of Hyperledger Fabric.  You will build from this source code Docker images that comprise the core Hyperledger Fabric services. Then you will run an end-to-end test that utilizes the Hyperledger Fabric Command Line Interface (CLI) to verify that everything is working correctly.

Then you will download two more Hyperledger Fabric source code repositories, *fabric-ca* and *fabric-sdk-node*, and build from this source the artifacts necessary to run a second, more comprehensive end-to-end test of the Fabric Node.js SDK.

 
Section 2: Install prerequisite software needed by Hyperledger Fabric
=====================================================================

In this section, you will install a subset of the software prerequisites mentioned in the Lab Overview section-  the major software prerequisites that will be necessary to run the end-to-end command line interface (CLI) test in Section 3. You will install:

*	Docker
*	PIP, a python installer program (needed to install Docker Compose)
*	curl (used to download PIP) 
*	Docker Compose
*	Golang compiler
**Step 2.1:** Log in to your assigned Ubuntu 16.04 Linux on IBM Z instance using the PuTTY icon on your workstation using the IP address assigned to your team.  This is an example command: ``ssh bcuser@192.168.22.225``. You will be greeted with a message like this::

    $ ssh bcuser@192.168.22.225   # substitute your team’s assigned IP address for 192.168.22.225
    bcuser@192.168.22.225's password: 
    Welcome to Ubuntu 16.04.2 LTS (GNU/Linux 4.4.0-83-generic s390x)
    
     * Documentation:  https://help.ubuntu.com
     * Management:     https://landscape.canonical.com
     * Support:        https://ubuntu.com/advantage
    
    0 packages can be updated.
    0 updates are security updates.
    
    
    Last login: Thu Jul 13 14:35:19 2017 from 192.168.215.24
    bcuser@ubuntu16042:~$

**Step 2.2:** This system is a relatively “clean” Ubuntu 16.04 instance- “clean” in the sense that after this instance was created,
no additional software was installed.  You will therefore need to install several software prerequisites.  The first thing you will 
install is Docker. Docker is a software container platform that is an integral part of Hyperledger Fabric.  *Smart contracts*, also 
known as *chaincode*, run as Docker containers.  In addition, you will run the Hyperledger Fabric processes themselves in Docker 
containers.  You can choose to run the Hyperledger Fabric processes natively on your host operating system, that is, *not* in Docker 
containers, but even if you did this you would still need Docker to run chaincode.  For this lab, you will use Docker containers for the chaincode and the Hyperledger Fabric processes.  

Issue this *which* command, which attempts to find the *docker* executable. The fact that it gives no response other than returning to 
the command prompt indicates that the program *docker* is not found::

    bcuser@ubuntu16042:~$ which docker
    bcuser@ubuntu16042:~$

**Note:** Some Linux distributions produce no output if the executable is not found, as in the above example.  Other Linux distributions
may produce an error message stating that the executable is not found.
   
**Step 2.3:** You will be using root authority for several commands throughout this lab.  You can tell when you have root authority by observing the command prompt-  it will end with a ‘#’ when you have root authority and it will end with a ‘$’ when you do not.  Use the sudo command to switch to the root user.  Observe the change in the command prompt after you enter this command::

  bcuser@ubuntu16042:~$ sudo su -
  root@ubuntu16042:~#

**Step 2.4:** Ubuntu provides a popular software package manager named *apt*, which stands for *Advanced Package Tool*. You will be 
using *apt* throughout this lab to install various software packages. The first thing you need to do is install the 
Docker Community Edition (CE).  This software is provided by Docker, so the next several steps will add a repository managed by Docker 
to your system’s list of repositories so that you can install Docker CE. Enter this command to update the *apt* package index::

 root@ubuntu16042:~# apt-get update
 Hit:1 http://us.ports.ubuntu.com/ubuntu-ports xenial InRelease
 Get:2 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates InRelease [102 kB]                    
 Get:3 http://us.ports.ubuntu.com/ubuntu-ports xenial-backports InRelease [102 kB]                             
 Get:4 http://ports.ubuntu.com/ubuntu-ports xenial-security InRelease [102 kB]    
 Get:5 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates/main s390x Packages [497 kB]   
 Get:6 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates/main Translation-en [250 kB]  
 Get:7 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates/universe s390x Packages [429 kB]   
 Get:8 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates/universe Translation-en [202 kB]
 Get:9 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates/multiverse s390x Packages [10.2 kB]     
 Get:10 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates/multiverse Translation-en [7540 B]
 Get:11 http://us.ports.ubuntu.com/ubuntu-ports xenial-backports/main s390x Packages [4696 B]           
 Get:12 http://us.ports.ubuntu.com/ubuntu-ports xenial-backports/universe s390x Packages [4844 B]
 Get:13 http://ports.ubuntu.com/ubuntu-ports xenial-security/main s390x Packages [268 kB]         
 Get:14 http://ports.ubuntu.com/ubuntu-ports xenial-security/main Translation-en [145 kB]
 Get:15 http://ports.ubuntu.com/ubuntu-ports xenial-security/universe s390x Packages [133 kB]
 Get:16 http://ports.ubuntu.com/ubuntu-ports xenial-security/universe Translation-en [79.8 kB]
 Fetched 2336 kB in 1s (1533 kB/s)                                  
 Reading package lists... Done
 
**Step 2.5:** Install packages to allow *apt* to use a repository over HTTPS::

 root@ubuntu16042:~# apt-get install apt-transport-https ca-certificates curl software-properties-common
 Reading package lists... Done
 Building dependency tree       
 Reading state information... Done
 ca-certificates is already the newest version (20160104ubuntu1).
 The following additional packages will be installed:
   python3-pycurl python3-software-properties unattended-upgrades xz-utils
 Suggested packages:
   libcurl4-gnutls-dev python-pycurl-doc python3-pycurl-dbg bsd-mailx mail-transport-agent
 The following NEW packages will be installed:
   curl python3-pycurl python3-software-properties software-properties-common unattended-upgrades xz-utils
 The following packages will be upgraded:
   apt-transport-https
 1 upgraded, 6 newly installed, 0 to remove and 33 not upgraded.
 Need to get 343 kB of archives.
 After this operation, 1552 kB of additional disk space will be used.
 Do you want to continue? [Y/n] Y
   .
   .  (remaining output not shown)
   .
 root@ubuntu16042:~#

**Step 2.6:**  Add Docker’s official GPG key::

 root@ubuntu16042:~# curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
 OK
 root@ubuntu16042:~#

**Step 2.7:** Verify that the key fingerprint is ``9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88``.::
 
 root@ubuntu16042:~# apt-key fingerprint 0EBFCD88
 pub   4096R/0EBFCD88 2017-02-22
       Key fingerprint = 9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
 uid                  Docker Release (CE deb) <docker@docker.com>
 sub   4096R/F273FCD8 2017-02-22
 
 root@ubuntu16042:~#

**Step 2.8:** Enter the following command to add the *stable* repository::

 root@ubuntu16042:~# add-apt-repository "deb [arch=s390x] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
 root@ubuntu16042:~#

**Step 2.9:** Update the *apt* package index again:: 

 root@ubuntu16042:~# apt-get update
 Hit:1 http://us.ports.ubuntu.com/ubuntu-ports xenial InRelease
 Hit:2 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates InRelease                             
 Hit:3 http://us.ports.ubuntu.com/ubuntu-ports xenial-backports InRelease                           
 Hit:4 http://ports.ubuntu.com/ubuntu-ports xenial-security InRelease         
 Get:5 https://download.docker.com/linux/ubuntu xenial InRelease [38.9 kB]
 Get:6 https://download.docker.com/linux/ubuntu xenial/stable s390x Packages [1075 B]
 Fetched 40.0 kB in 0s (139 kB/s)    
 Reading package lists... Done

**Step 2.10:** Enter this command to show some information about the Docker package.  This command won’t actually install anything::
 
 root@ubuntu16042:~# apt-cache policy docker-ce
 docker-ce:
   Installed: (none)
   Candidate: 17.09.1~ce-0~ubuntu
   Version table:
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

Some key takeaways from the command output:

*	Docker is not currently installed *(Installed: (none))*
*	*17.09.1~ce* is the candidate version to install- it is the latest version available
*	When you install the software, you will be going out to the Internet to the *download.docker.com* domain to get the software.

**Step 2.11:** Enter this *apt-get* command to install the software.  (Enter Y when prompted to continue)::

 root@ubuntu16042:~# apt-get install docker-ce
 Reading package lists... Done
 Building dependency tree       
 Reading state information... Done
 The following additional packages will be installed:
   aufs-tools cgroupfs-mount git git-man liberror-perl libltdl7 patch
 Suggested packages:
   mountall git-daemon-run | git-daemon-sysvinit git-doc git-el git-email git-gui gitk gitweb git-arch git-cvs git-mediawiki    git-svn diffutils-doc 
 The following NEW packages will be installed:
   aufs-tools cgroupfs-mount docker-ce git git-man liberror-perl libltdl7 patch
 0 upgraded, 8 newly installed, 0 to remove and 57 not upgraded.
 Need to get 23.3 MB of archives.
 After this operation, 128 MB of additional disk space will be used.
 Do you want to continue? [Y/n] Y
   .
   .   (remaining output not shown here)
   .

Observe that not only was Docker installed, but so were its prerequisites that were not already installed.

**Step 2.12:** Issue the *which* command again and this time it will tell you where it found the just-installed docker program::

 root@ubuntu16042:~# which docker
 /usr/bin/docker

**Step 2.13:** Enter the *docker version* command and you will see that version *17.09.1-ce* was installed, just as expected based on 
the output from your earlier *apt-cache* command::

 root@ubuntu16042:~# docker version
 Client:
  Version:      17.09.1-ce
  API version:  1.32
  Go version:   go1.8.3
  Git commit:   19e2cf6
  Built:        Thu Dec  7 22:24:08 2017
  OS/Arch:      linux/s390x

 Server:
  Version:      17.09.1-ce
  API version:  1.32 (minimum version 1.12)
  Go version:   go1.8.3
  Git commit:   19e2cf6
  Built:        Thu Dec  7 22:23:09 2017
  OS/Arch:      linux/s390x
  Experimental: false

**Step 2.14:** Enter *docker info* to see even more information about your Docker environment::

 root@ubuntu16042:~# docker info
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 0
 Server Version: 17.09.1-ce
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
 containerd version: 06b9cb35161009dcb7123345749fef02f7cea8e0
 runc version: 3f2f8b84a77f73d38244dd690525642a72156c64
 init version: 949e6fa
 Security Options:
  apparmor
 Kernel Version: 4.4.0-96-generic
 Operating System: Ubuntu 16.04.3 LTS
 OSType: linux
 Architecture: s390x
 CPUs: 2
 Total Memory: 1.717GiB
 Name: ubuntu16042
 ID: N457:47ES:GEC3:5N5M:P2OT:C4SB:PNSQ:N2TJ:XCNP:WEOT:NRWG:YQGF
 Docker Root Dir: /var/lib/docker
 Debug Mode (client): false
 Debug Mode (server): false
 Registry: https://index.docker.io/v1/
 Experimental: false
 Insecure Registries:
  127.0.0.0/8
 Live Restore Enabled: false

 WARNING: No swap limit support 

**Step 2.15:** After the Docker installation, non-root users cannot run Docker commands. One way to get around this for a non-root userid is to add that userid to a group named docker.  Enter this command to 
add the *bcuser* userid to the group *docker*::

 root@ubuntu16042:~# usermod -aG docker bcuser
 
*Note:* This method of authorizing a non-root userid to enter Docker commands, while suitable for a controlled sandbox environment, may not be suitable for a production environemnt due to security considerations. 

**Step 2.16:** Exit so that you are no longer running as root::

 root@ubuntu16042:~# exit
 logout
 bcuser@ubuntu16042:~$
 
**Step 2.17:** Even though *bcuser* was just added to the *docker* group, you will have to log out and then log back in again for this 
change to take effect.  To prove this, enter the *docker info* command before you log out and then again after you log in.  (You may 
need to start a new PuTTY session after you logged out so that you can get back in).

**Step 2.18:** You will need to get right back in as root to install *Docker Compose*.  Docker Compose is a tool provided by Docker to 
help make it easier to run an application that consists of multiple Docker containers.  On some platforms, it is installed along with 
the Docker package but on Linux on IBM Z it is installed separately.  It is written in Python and you will install it with a tool 
called Pip.  But first you will install Pip itself!  You will do this as root, so enter this again::

 bcuser@ubuntu16042:~$ sudo su -
 root@ubuntu16042:~#

**Step 2.19** Install the *python-pip* package which will provide a tool named *Pip* which is used to install Python packages from a public repository::

 root@ubuntu16042:~# apt-get -y install python-pip

This will bring in a lot of prerequisites and will produce a lot of output which is not shown here.

**Step 2.20:** Run this command just to verify that *docker-compose* is not currently available on the system::

 root@ubuntu16042:~# which docker-compose
 root@ubuntu16042:~# 

**Step 2.21:** Use Pip to install Docker Compose::

 root@ubuntu16042:~# pip install docker-compose
 
**Step 2.22:** There was a bunch of output from the prior step I didn’t show, but if your install works, you should feel pretty good about the output from this command::

 root@ubuntu16042:~# docker-compose --version
 docker-compose version 1.17.1, build 6d101fb
 
**Step 2.23:** Leave root behind and become a normal user again::

 root@ubuntu16042:~# exit
 logout
 bcuser@ubuntu16042:~$

**Step 2.24:** You won’t have to log out and log back in, like you did with Docker, in order to use Docker Compose, and to prove it, 
check for the version again now that you are no longer root::

 bcuser@ubuntu16042:~$ docker-compose --version
 docker-compose version 1.17.1, build 6d101fb

**Step 2.25:** The next thing you are going to install is the *Golang* programming language. You are going to install Golang version 
1.8.3.  Go to the /tmp directory::

 bcuser@ubuntu16042:~$ cd /tmp
 bcuser@ubuntu16042:/tmp$

**Step 2.26:** Use *wget* to get the compressed file that contains the Golang compiler and tools.  And now is a good time to tell you 
that from here on out I will just call Golang what everybody else usually calls it-  *Go*.  Go figure.
::
 bcuser@ubuntu16042:/tmp$ wget --no-check-certificate https://storage.googleapis.com/golang/go1.8.3.linux-s390x.tar.gz
 --2017-12-14 12:24:07--  https://storage.googleapis.com/golang/go1.8.3.linux-s390x.tar.gz
 Resolving storage.googleapis.com (storage.googleapis.com)... 173.194.219.128, 2607:f8b0:4002:c03::80
 Connecting to storage.googleapis.com (storage.googleapis.com)|173.194.219.128|:443... connected.
 HTTP request sent, awaiting response... 200 OK
 Length: 75341693 (72M) [application/x-gzip]
 Saving to: 'go1.8.3.linux-s390x.tar.gz'

 go1.8.3.linux-s390x.tar.gz               100%[===============================================================================>]  71.85M  10.2MB/s    in 6.3s    

 2017-12-14 12:24:14 (11.4 MB/s) - 'go1.8.3.linux-s390x.tar.gz' saved [75341693/75341693]

**Step 2.27:** Enter the following command which will extract the files into the /tmp directory, and provide lots and lots of output.
(It’s the *‘v’* in *-xvf* which got all chatty, or *verbose*, on you)::

 bcuser@ubuntu16042:/tmp$ tar -xvf go1.8.3.linux-s390x.tar.gz
   .
   .  (output not shown here)
   .

**Step 2.28:** You will move the extracted stuff, which is all under */tmp/go*, into */opt*, and for that you will need root authority.
Whereas before you were instructed to enter *sudo su* – which effectively logged you in as root until you exited, you can issue a 
single command with *sudo* which executes it as root and then returns control back to you in non-root mode.   Enter this command::

 bcuser@ubuntu16042:/tmp$ sudo mv -iv go /opt 
  'go' -> '/opt/go'

**Step 2.29:** You need to set a couple of Go-related environment variables.  First check to verify that they are not set already::

 bcuser@ubuntu16042:/tmp$ env | grep GO

That command, *grep*, is looking for any lines of input that contain the characters *GO*.  Its input is the output of the previous *env*
command, which prints all of your environment variables.

**Step 2.30:**  You will set these values now.  You will make these changes in a special hidden file named *.bashrc* in your home 
directory.  Change to your home directory::

 bcuser@ubuntu16042:/tmp$ cd ~  # that is a tilde ~ character I know it is hard to see 
 bcuser@ubuntu16042:~$

**Step 2.31:** There are six commands in this step- a *cp* command (to make a backup) and then five *echo* commands. Enter them exactly as shown.  It is critical that you use two ‘greater-than’ signs, i.e., ‘>>’, when you 
enter the commands.  This appends the arguments of the *echo* commands to the end of this file.  If you only enter one ‘>’ sign, you 
will overwrite the file’s contents.  I’d rather you not do that. Although the first command shown does create a backup copy of the file,
just in case::

 bcuser@ubuntu16042:~$ cp -ipv .bashrc .bashrc_orig
 '.bashrc' -> '.bashrc_orig'
 bcuser@ubuntu16042:~$ echo '' >> .bashrc   # that is two single quotes, not one double-quote
 bcuser@ubuntu16042:~$ echo export GOPATH=/home/bcuser/git >> .bashrc
 bcuser@ubuntu16042:~$ echo export GOROOT=/opt/go >> .bashrc
 bcuser@ubuntu16042:~$ echo export PATH=/opt/go/bin:/home/bcuser/bin:\$PATH >> .bashrc
 bcuser@ubuntu16042:~$ echo '' >> .bashrc  # that is two single quotes, not one double-quote

**Step 2.32:** Let’s see how you did.  Enter this command::

 bcuser@ubuntu16042:~$ head .bashrc
 # ~/.bashrc: executed by bash(1) for non-login shells.
 # see /usr/share/doc/bash/examples/startup-files (in the package bash-doc)
 # for examples
 
 # If not running interactively, don't do anything
 case $- in
     *i*) ;;
       *) return;;
 esac

If your output looked like the above, congratulations, you did not stomp all over your file. *head* prints the top of the file.  Had 
you made and mistake and used a single '>' instead two ‘>>’ like I told you, you would have whacked this stuff.  Your stuff is at the bottom.  If *head* 
prints the top of the file, guess what command prints the bottom of the file.

**Step 2.33:** Try this::

 bcuser@ubuntu16042:~$ tail -5 .bashrc
 
 export GOPATH=/home/bcuser/git
 export GOROOT=/opt/go
 export PATH=/opt/go/bin:$PATH

**Step 2.34:** These changes will take effect next time you log in, but you can make them take effect immediately by entering this::

 bcuser@ubuntu16042:~$ source .bashrc

**Step 2.35:** Try this to see if your changes took::

 bcuser@ubuntu16042:~$ env | grep GO
 GOROOT=/opt/go
 GOPATH=/home/bcuser/git

**Step 2.36:**  Then try this::

 bcuser@ubuntu16042:~$ go version
 go version go1.8.3 linux/s390x

**Recap:** Before you move on, here is a summary of the major tasks you performed in this section.

*	You installed Docker and added *bcuser* to the *docker* group so that *bcuser* can issue Docker commands
*	You installed Docker Compose (and Pip and curl, which were needed to install it)
*	You installed Go
*	You updated your *.bashrc* profile to make necessary environment changes

In the next section, you will download the Hyperledger Fabric source code, build it, and run a comprehensive verification test against 
the Hyperledger Fabric Command Line Interface, or CLI.
 
Section 3: Download, build and test the Hyperledger Fabric CLI
==============================================================

In this section, you will:

*	Install some support packages using the Ubuntu package manager, *apt-get*
*	Download the source code repository containing the core Hyperledger Fabric functionality
*	Use the source code to build Docker images that contain the core Hyperledger Fabric functionality
*	Test for success by running the comprehensive end-to-end CLI test.

**Step 3.1:** There are some software packages necessary to be able to successfully build the Hyperledger Fabric source code.  Install them with 
this command.  You have used the *apt-get* command earlier in the lab.  This time, use the *‘-y’* flag which will automatically 
reply ‘Y’ for you so that you will not be prompted to continue.  Observe the output, not shown here, to see the different packages 
installed::

 bcuser@ubuntu16042:~$ sudo apt-get install -y build-essential libltdl3-dev git
 
**Step 3.2:** Create the following directory path with this command.  Make sure you are in your home directory when you enter it::

 bcuser@ubuntu16042:~$ mkdir -p git/src/github.com/hyperledger
 bcuser@ubuntu16042:~$
 
**Step 3.3:** Navigate to the directory you just created::

 bcuser@ubuntu16042:~$ cd git/src/github.com/hyperledger/
 bcuser@ubuntu16042:~/git/src/github.com/hyperledger$
 
**Step 3.4:** Use the software tool *git* to download the source code of the Hyperledger Fabric core package from the official place 
where it lives::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger$ git clone https://gerrit.hyperledger.org/r/fabric
 Cloning into 'fabric'...
 remote: Counting objects: 4619, done
 remote: Finding sources: 100% (23/23)
 remote: Total 53642 (delta 2), reused 53631 (delta 2)
 Receiving objects: 100% (53642/53642), 66.32 MiB | 1.86 MiB/s, done.
 Resolving deltas: 100% (24828/24828), done.
 Checking connectivity... done.

*Note:* The numbers is the various output messages will almost certainly be different that what you see listed here since code is continuously being created or modified.  This will be the case anytime you do a *git clone* in the remainder of these labs.

**Step 3.5:** Switch to the *fabric* directory, which is the top directory of where the *git* command put the code it just downloaded::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger$ cd fabric
 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric$

**Step 3.6:** Enter this *git* command which will show the status of the code you just pulled down::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric$ git status
 On branch master
 Your branch is up-to-date with 'origin/master'.
 nothing to commit, working directory clean
 
You pulled down, by default, the master branch of the Hyperledger Fabric code.  The master branch is updated quite often still, so you will check out a version of the code that is a little bit less recent than the latest code.  You will be checking out a level of the code that has been verified to work for the activities in this lab. 

**Step 3.7:** Your instructor may provide you with a code level to checkout that would replace the level, *v1.0.5*, specified in 
this example.  If not, use the level from this example::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric$ git checkout v1.0.5
 Note: checking out 'v1.0.5'.

 You are in 'detached HEAD' state. You can look around, make experimental
 changes and commit them, and you can discard any commits you make in this
 state without impacting any branches by performing another checkout.

 If you want to create a new branch to retain commits you create, you may
 do so (now or later) by using -b with the checkout command again. Example:

   git checkout -b <new-branch-name>

 HEAD is now at 014d6be... [FAB-7281] Release Hyperledger Fabric v1.0.5

**Step 3.8:** You will use a program called *make*, which is used to build software projects, in order to build Docker images for Hyperledger Fabric.  But first, run this command to show that your system does not currently have any 
Docker images stored on it.  The only output you will see is the column headings::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric$ docker images
 REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE

**Step 3.9:** That will change in a few minutes.  Enter the following command, which will build the Hyperledger Fabric images.  You 
can ‘wrap’ the *make* command, which is what will do all the work, in a *time* command, which will give you a measure of the time, 
including ‘wall clock’ time, required to build the images::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric$ time make docker
   .
   .  (output not shown here)
   .
 real    6m42.737s
 user    0m7.088s
 sys     0m0.776s
 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric$

**Step 3.10:** Run *docker images* again and you will see several Docker images that were just created. You will notice that all but 
the last two of the Docker images were created in the last few minutes.  These were created by the *make docker* command.  Docker 
images can be built based on other Docker images.  The two Docker images listed at the bottom, that are a few months old, were used 
in the process, and were downloaded from the Hyperledger Fabric’s public DockerHub repository.  Your output should look similar to 
that shown here, although the tags will be different if your instructor gave you a different level to checkout, and your *image ids* 
will be different either way::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric$ docker images
 REPOSITORY                     TAG                 IMAGE ID            CREATED             SIZE
 hyperledger/fabric-tools       latest              e755ee5e303c        13 seconds ago      1.48GB
 hyperledger/fabric-tools       s390x-1.0.5         e755ee5e303c        13 seconds ago      1.48GB
 hyperledger/fabric-couchdb     latest              8336fcd3a2a8        44 seconds ago      1.69GB
 hyperledger/fabric-couchdb     s390x-1.0.5         8336fcd3a2a8        44 seconds ago      1.69GB
 hyperledger/fabric-kafka       latest              39b3bd557d14        2 minutes ago       1.43GB
 hyperledger/fabric-kafka       s390x-1.0.5         39b3bd557d14        2 minutes ago       1.43GB
 hyperledger/fabric-zookeeper   latest              45e273c535cc        2 minutes ago       1.46GB
 hyperledger/fabric-zookeeper   s390x-1.0.5         45e273c535cc        2 minutes ago       1.46GB
 hyperledger/fabric-testenv     latest              1a3386b80cd7        3 minutes ago       1.54GB
 hyperledger/fabric-testenv     s390x-1.0.5         1a3386b80cd7        3 minutes ago       1.54GB
 hyperledger/fabric-buildenv    latest              26c75aa3727e        3 minutes ago       1.46GB
 hyperledger/fabric-buildenv    s390x-1.0.5         26c75aa3727e        3 minutes ago       1.46GB
 hyperledger/fabric-orderer     latest              82252e81d84a        4 minutes ago       218MB
 hyperledger/fabric-orderer     s390x-1.0.5         82252e81d84a        4 minutes ago       218MB
 hyperledger/fabric-peer        latest              adcbfc3fbc82        4 minutes ago       221MB
 hyperledger/fabric-peer        s390x-1.0.5         adcbfc3fbc82        4 minutes ago       221MB
 hyperledger/fabric-javaenv     latest              df0d801d1b81        4 minutes ago       1.51GB
 hyperledger/fabric-javaenv     s390x-1.0.5         df0d801d1b81        4 minutes ago       1.51GB
 hyperledger/fabric-ccenv       latest              06a8d7072eb2        4 minutes ago       1.43GB
 hyperledger/fabric-ccenv       s390x-1.0.5         06a8d7072eb2        4 minutes ago       1.43GB
 hyperledger/fabric-baseimage   s390x-0.3.2         68622e5c9cba        3 months ago        1.4GB
 hyperledger/fabric-baseos      s390x-0.3.2         b02371d17ca4        3 months ago        194MB


**Step 3.11:** Navigate to the directory where the “end-to-end” test lives::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric$ cd examples/e2e_cli/
 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric/examples/e2e_cli$

**Step 3.12:** The end-to-end test that you are about to run will create several Docker containers.  A Docker container is what runs a 
process, and it is based on a Docker image.  Run this command, which shows all Docker containers, and right now there will be no 
output other than column headings, which indicates no Docker containers are currently running::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric/examples/e2e_cli$ docker ps -a
 CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

**Step 3.13:** Run the end-to-end test with this command::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric/examples/e2e_cli$ ./network_setup.sh up mychannel 10 couchdb
   .
   . (output not shown here)
   .
 ===================== Query on PEER3 on channel 'mychannel' is successful =====================
 
 ===================== All GOOD, End-2-End execution completed =====================
   .
   . (output not shown here)
   .

**Step 3.14:** Run the *docker ps* command to see the Docker containers that the test created::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric/examples/e2e_cli$ docker ps -a
 CONTAINER ID        IMAGE                                                                                                  COMMAND                  CREATED              STATUS                          PORTS                                                                       NAMES
 54b6a9fb94cf        dev-peer1.org2.example.com-mycc-1.0-26c2ef32838554aac4f7ad6f100aca865e87959c9a126e86d764c8d01f8346ab   "chaincode -peer.a..."   About a minute ago   Up About a minute                                                                                           dev-peer1.org2.example.com-mycc-1.0
 65484bd9cab4        dev-peer0.org1.example.com-mycc-1.0-384f11f484b9302df90b453200cfb25174305fce8f53f4e94d45ee3b6cab0ce9   "chaincode -peer.a..."   About a minute ago   Up About a minute                                                                                           dev-peer0.org1.example.com-mycc-1.0
 db10cb22dad0        dev-peer0.org2.example.com-mycc-1.0-15b571b3ce849066b7ec74497da3b27e54e0df1345daff3951b94245ce09c42b   "chaincode -peer.a..."   About a minute ago   Up About a minute                                                                                           dev-peer0.org2.example.com-mycc-1.0
 9466cd8fe887        hyperledger/fabric-tools                                                                               "/bin/bash -c './s..."   2 minutes ago        Exited (0) About a minute ago                                                                               cli
 91c48d53dbbb        hyperledger/fabric-peer                                                                                "peer node start"        2 minutes ago        Up 2 minutes                    0.0.0.0:9051->7051/tcp, 0.0.0.0:9052->7052/tcp, 0.0.0.0:9053->7053/tcp      peer0.org2.example.com
 8256b2779efc        hyperledger/fabric-peer                                                                                "peer node start"        2 minutes ago        Up 2 minutes                    0.0.0.0:8051->7051/tcp, 0.0.0.0:8052->7052/tcp, 0.0.0.0:8053->7053/tcp      peer1.org1.example.com
 fce9b16cb8c5        hyperledger/fabric-peer                                                                                "peer node start"        2 minutes ago        Up 2 minutes                    0.0.0.0:10051->7051/tcp, 0.0.0.0:10052->7052/tcp, 0.0.0.0:10053->7053/tcp   peer1.org2.example.com
 579ed02153b8        hyperledger/fabric-peer                                                                                "peer node start"        2 minutes ago        Up 2 minutes                    0.0.0.0:7051-7053->7051-7053/tcp                                            peer0.org1.example.com
 20e671772928        hyperledger/fabric-couchdb                                                                             "tini -- /docker-e..."   2 minutes ago        Up 2 minutes                    4369/tcp, 9100/tcp, 0.0.0.0:7984->5984/tcp                                  couchdb2
 e8dda81d6383        hyperledger/fabric-couchdb                                                                             "tini -- /docker-e..."   2 minutes ago        Up 2 minutes                    4369/tcp, 9100/tcp, 0.0.0.0:6984->5984/tcp                                  couchdb1
 3f71abaa0615        hyperledger/fabric-couchdb                                                                             "tini -- /docker-e..."   2 minutes ago        Up 2 minutes                    4369/tcp, 9100/tcp, 0.0.0.0:8984->5984/tcp                                  couchdb3
 27b5e176f63e        hyperledger/fabric-couchdb                                                                             "tini -- /docker-e..."   2 minutes ago        Up 2 minutes                    4369/tcp, 9100/tcp, 0.0.0.0:5984->5984/tcp                                  couchdb0
 dd25b3b4bd0d        hyperledger/fabric-orderer                                                                             "orderer"                2 minutes ago        Up 2 minutes                    0.0.0.0:7050->7050/tcp                                                      orderer.example.com

The first three Docker containers listed are chaincode containers-  The chaincode was run on three of the four peers, so they each 
had a Docker image and container created.  There were also four peer containers created, each with a couchdb container, and one 
orderer container. There was a container created to run the CLI itself, and that container stopped running ten seconds after the 
test ended.  (That was what the value *10* was for in the *./network_setup.sh* command you ran).

You have successfully run the CLI end-to-end test.  You will clean things up now.

**Step 3.15:** Run the *network_setup.sh* script with different arguments to bring the Docker containers down::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric/examples/e2e_cli$ ./network_setup.sh down

**Step 3.16:** Try the *docker ps* command again and you should see that there are no longer any Docker containers running::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric/examples/e2e_cli$ docker ps -a
 CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

**Recap:** In this section, you:

*	Downloaded the main Hyperledger Fabric source code repository
*	Installed prerequisite tools required to build the Hyperledger Fabric project
*	Ran *make* to build the project’s Docker images
*	Ran the Hyperledger Fabric command line interface (CLI) end-to-end test
*	Cleaned up afterwards
 
Section 4: Install the Hyperledger Fabric Certificate Authority
===============================================================

In the prior section, the end-to-end test that you ran supplied its own security-related material such as keys and certificates.  
Your next major goal is to run the Hyperledger Fabric Node.js SDK’s end-to-end test.  This test makes calls to the Hyperledger Fabric 
Certificate Authority (CA), therefore, you will get started by downloading and building the Hyperledger Fabric CA.

**Step 4.1:** Back up to the *hyperledger* directory::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric/examples/e2e_cli$ cd ../../..
 bcuser@ubuntu16042:~/git/src/github.com/hyperledger$

**Step 4.2:** Get the source code for the CA using *git*::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger$ git clone https://gerrit.hyperledger.org/r/fabric-ca
 Cloning into 'fabric-ca'...
 remote: Counting objects: 11, done
 remote: Total 9023 (delta 0), reused 9023 (delta 0)
 Receiving objects: 100% (9023/9023), 23.48 MiB | 2.41 MiB/s, done.
 Resolving deltas: 100% (3078/3078), done.
 Checking connectivity... done.

**Step 4.3:** Navigate to the *fabric-ca* directory, which is the top directory of where the *git* command put the code it just 
downloaded::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger$ cd fabric-ca
 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-ca$

**Step 4.4:** Enter this *git* command which will show the status of the code you just pulled down::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-ca$ git status
 On branch master
 Your branch is up-to-date with 'origin/master'.
 nothing to commit, working directory clean

You pulled down, by default, the master branch of the Hyperledger Fabric CA code.  The master branch is updated quite often still, so you will check out a version of the code that is a little bit less recent than the latest code.  You will be checking out a level of the code that has been verified to work for the activities in this lab. 

**Step 4.5:** Your instructor may provide you with a code level to checkout that would replace the level, *v1.0.5*, specified in this 
example.  If not, use the level from this example::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-ca$ git checkout v1.0.5
 Note: checking out 'v1.0.5'.

 You are in 'detached HEAD' state. You can look around, make experimental
 changes and commit them, and you can discard any commits you make in this
 state without impacting any branches by performing another checkout.

 If you want to create a new branch to retain commits you create, you may
 do so (now or later) by using -b with the checkout command again. Example:

   git checkout -b <new-branch-name>

 HEAD is now at 49a8d3b... [FAB-7282] Release Hyperledger Fabric CA v1.0.5

**Step 4.6:** Enter the following command, which will build the Hyperledger Fabric CA images.  You can ‘wrap’ the *make* command, which 
is what will do all the work, in a *time* command, which will give you a measure of the time, including ‘wall clock’ time,
required to build the images::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-ca $ time make docker
   .
   .  (output not shown here)
   .
 real    1m10.668s
 user    0m0.052s
 sys     0m0.062s
 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-ca$

**Step 4.7:** Enter the *docker images* command and you will see at the top of the output the Docker image that was just created for 
the Certificate Authority::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-ca$ docker images
 hyperledger/fabric-ca          latest              cdddb81e42ff        3 minutes ago       255MB
 hyperledger/fabric-ca          s390x-1.0.5         cdddb81e42ff        3 minutes ago       255MB
   .
   . (remaining output not shown here)
   .

I have shown two lines of output but implied that one Docker image is created for the CA.  What you have here is one image-  uniquely 
identified by its image ID, *cdddb81e42ff* in this example; yours will surely be different. An image can be given any number of names, 
or *tags*.  Think of these *tags* as nicknames, or aliases.  The *make* process first gave the Docker image it created a descriptive 
tag, *s390x-1.0.5*, and then it ‘tagged’ it with a new tag, *latest*.  It did that for a reason.  When you are working with Docker 
images, if you specify an image without specifying a tag, the tag defaults to the name *latest*. So, for this image, you can either 
refer to *hyperledger/fabric-ca*, *hyperledger/fabric-ca:latest*, or *hyperledger/fabric-ca:s390x-1.0.1*. 

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

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-ca$ cd /tmp
 bcuser@ubuntu16042:/tmp$

**Step 5.2:** Retrieve the *Node.js* package with this command::

 bcuser@ubuntu16042:/tmp$ wget https://nodejs.org/dist/v8.9.3/node-v8.9.3-linux-s390x.tar.xz
 --2017-12-14 15:46:45--  https://nodejs.org/dist/v8.9.3/node-v8.9.3-linux-s390x.tar.xz
 Resolving nodejs.org (nodejs.org)... 104.20.23.46, 104.20.22.46, 2400:cb00:2048:1::6814:162e, ...
 Connecting to nodejs.org (nodejs.org)|104.20.23.46|:443... connected.
 HTTP request sent, awaiting response... 200 OK
 Length: 10958756 (10M) [application/x-xz]
 Saving to: 'node-v8.9.3-linux-s390x.tar.xz'

 node-v8.9.3-linux-s390x.tar.xz                     100%[============================================================================================================>]  10.45M  9.52MB/s    in 1.1s    

 2017-12-14 15:46:46 (9.52 MB/s) - 'node-v8.9.3-linux-s390x.tar.xz' saved [10958756/10958756]

**Step 5.3:** Extract the package underneath your home directory, */home/bcuser*. This will cause the executables to be wind up in 
*/home/bcuser/bin*, which is in your path::

 bcuser@ubuntu16042:/tmp$ cd /home/bcuser && tar --strip-components=1 -xf /tmp/node-v8.9.3-linux-s390x.tar.xz

**Step 5.4:** In this step, you will issue some commands that will show you where *node* and *npm* reside, and what version of each is installed::

 bcuser@ubuntu16042:/tmp$ which node
 /home/bcuser/bin/node
 bcuser@ubuntu16042:/tmp $ which npm
 /home/bcuser/bin/npm
 bcuser@ubuntu16042:/tmp $ node --version
 v8.9.3
 bcuser@ubuntu16042:/tmp$ npm --version
 5.5.1

**Step 5.5:** Switch to the *~/git/src/github.com/hyperledger* directory::

 bcuser@ubuntu16042:/tmp$ cd ~/git/src/github.com/hyperledger/
 bcuser@ubuntu16042:~/git/src/github.com/hyperledger$

**Step 5.6:** Now you will download the Hyperledger Fabric Node SDK source code from its official repository::

 bcuser@ubuntu16042: ~/git/src/github.com/hyperledger $ git clone -b v1.0.2 https://github.com/hyperledger/fabric-sdk-node
 Cloning into 'fabric-sdk-node'...
 remote: Counting objects: 6428, done.
 remote: Compressing objects: 100% (16/16), done.
 remote: Total 6428 (delta 4), reused 12 (delta 0), pack-reused 6412
 Receiving objects: 100% (6428/6428), 2.28 MiB | 1.09 MiB/s, done.
 Resolving deltas: 100% (4164/4164), done.
 Checking connectivity... done.
 Note: checking out '6f7310ccda8648473d0f794e28c5b390ac030480'.

 You are in 'detached HEAD' state. You can look around, make experimental
 changes and commit them, and you can discard any commits you make in this
 state without impacting any branches by performing another checkout.

 If you want to create a new branch to retain commits you create, you may
 do so (now or later) by using -b with the checkout command again. Example:

   git checkout -b <new-branch-name>

*Note:* For the *fabric* and the *fabric-ca* repositories, you were instructed to download from *gerrit.hyperledger.org* and then switch to branch *v1.0.5*. Here, we are taking a slightly different approach.  You are downloading from *github.com/hyperledger* instead of *gerrit.hyperledger.org*, and, you are using branch *v1.0.2* instead of *v1.0.5*.  Why these differences?  I will try to explain the reasons.

The repositories on *github.com/hyperledger* for *fabric, fabric-ca* and *fabric-sdk-node* are all read-only mirrors of the repositories on *gerrit.hyperledger.org*. If you were a developer interested in contributing code to or modifying code within these projects, you would need to work with *gerrit.hyperledger.org* and submit your new or changed code to that repository.  But if you just wanted to *use* the code, even to build it from source, as you are doing in this lab, you could use either site- *gerrit.hyperledger.org* or *github.com/hyperledger*.  So in earlier sections in this lab when you did a **git clone** from *gerrit.hyperledger.org* for *fabric* and *fabric-ca*, I could just as easily have instructed you to get the code from *github.com/hyperledger*.  It doesn't matter which for the purposes of this lab.

The latest release of the Hyperledger Fabric v1.0.x code is *v1.0.5*.  The *fabric* and *fabric-ca* repositories tend to be synchronized closely, so usually when a new release is created, e.g. 1.0.3 or 1.0.4 or 1.0.5, a *fabric* and *fabric-ca* branch or tag of the same name is created at about the same time.  The SDKs, on the other hand, do not always follow in lockstep with *fabric* and *fabric-ca*. For example, *fabric-sdk-node* has a tag of *1.0.2* that works fine with Hyperledger Fabric v1.0.2, v1.0.3, v1.0.4 and v1.0.5.  That is why you were instructed to get *v1.0.2* for *fabric-sdk-node*.

The reason you were asked to use *github.com/hyperledger* instead of *gerrit.hyperledger.org* to get *fabric-sdk-node*, is that, for whatever reason, there is not a branch or tag for *v1.0.2* for *fabric-sdk-node* on *gerrit.hyperledger.org* whereas there is one on *github.com/hyperledger*.  This is probably just an accidental quirk- for instance, both sites do have a tag for *v1.0.1*.  There are ways to work around this with the code for *gerrit.hyperledger.org*, but for simplicity's sake I had you get the code from *github.com/hyperledger*.  

**Step 5.7:** And then change to the *fabric-sdk-node* directory which was just created::

 bcuser@ubuntu16042: ~/git/src/github.com/hyperledger $ cd fabric-sdk-node
 bcuser@ubuntu16042: ~/git/src/github.com/hyperledger/fabric-sdk-node$

**Step 5.8:** You are about to install the packages that the Hyperledger Fabric Node SDK would like to use. Before you start, 
run *npm list* to see that you are starting with a blank slate::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-sdk-node$ npm list
 ffabric-sdk-node@1.0.2 /home/bcuser/git/src/github.com/hyperledger/fabric-sdk-node
 `-- (empty)

 bcuser@ubuntu16042: ~/git/src/github.com/hyperledger/fabric-sdk-node$

**Step 5.9:** Run *npm install* to install the required packages.  This will take a few minutes and will produce a lot of output::

 bcuser@ubuntu16042: ~/git/src/github.com/hyperledger/fabric-sdk-node$ npm install
   .
   . (output not shown here)
   .
 npm notice created a lockfile as package-lock.json. You should commit this file.
 npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@1.1.3 (node_modules/fsevents):
 npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@1.1.3: wanted {"os":"darwin","arch":"any"} (current: {"os":"linux","arch":"s390x"})

 added 1020 packages in 57.095s


You may ignore the *WARN* messages at the end of the output.  If there is a serious error, the end of the output will leave little 
doubt about it.

**Step 5.10:** Repeat the *npm list* command.  The output, although not shown here, will be anything but empty.  This just proves what 
everyone suspected-  programmers would much rather use other peoples’ code than write their own.  Not that there’s anything wrong 
with that. You can even steal this lab if you want to.
::
 bcuser@ubuntu16042: ~/git/src/github.com/hyperledger/fabric-sdk-node$ npm list
   .
   . (output not shown here, but surely you will agree it is not empty)
   .
 bcuser@ubuntu16042: ~/git/src/github.com/hyperledger/fabric-sdk-node$

**Step 5.11:** Now you will install an automation tool named *gulp* at a global level, using the *-g* argument to the *npm install* 
command.  This makes the package installed available on a system-wide basis. Run the *which* command before and after the *npm install* 
command to verify success::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-sdk-node$ which gulp
 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-sdk-node$ npm install -g gulp
   .
   .  (output not shown here)
   .
 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-sdk-node$ which gulp
 /home/bcuser/bin/gulp
 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-sdk-node$

**Step 5.12:** Next you will install a code coverage testing tool named *istanbul*, also at a global level::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-sdk-node$ which istanbul
 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-sdk-node$ npm install -g istanbul
   .
   .  (output not shown here)
   .
 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-sdk-node$ which istanbul
 /home/bcuser/bin/istanbul
 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-sdk-node$

**Recap:** In this section, you:

*	Installed Node.js and npm
*	Downloaded the Hyperledger Fabric Node.js SDK
*	Installed the *npm* packages required by the Hyperledger Fabric Node.js SDK
*	Installed the *gulp* and *istanbul* packages to run the Hyperledger Fabric Node.js SDK end-to-end test (which you will run in the next section)
 
Section 6: Run the Hyperledger Fabric Node.js SDK end-to-end tests
==================================================================
In this section, you will run the two tests end-to-end tests provided by the Hyperledger Fabric Node.js SDK, verify their successful 
operation, and clean up afterwards.

The first test is a quick test that takes only a few seconds, and does not bring up any chaincode containers.  The second test is much more comprehensive, and will bring up several chaincode containers and will take a few minutes.  In between the first and second tests you will make one minor change to work around a quirk that I will explain at the right time in these instructions.

**Step 6.1:** The first test is very simple and can be run simply by running *npm test*::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-sdk-node$ npm test
   .
   . (initial output not shown)
   .
   1..623
 # tests 623
 # pass  623

 # ok

 ----------|----------|----------|----------|----------|----------------|
 File      |  % Stmts | % Branch |  % Funcs |  % Lines |Uncovered Lines |
 ----------|----------|----------|----------|----------|----------------|
 ----------|----------|----------|----------|----------|----------------|
 All files |      100 |      100 |      100 |      100 |                |
 ----------|----------|----------|----------|----------|----------------|


 =============================== Coverage summary ===============================
 Statements   : 100% ( 0/0 )
 Branches     : 100% ( 0/0 )
 Functions    : 100% ( 0/0 )
 Lines        : 100% ( 0/0 )
 ================================================================================
 [17:20:18] Finished 'test-headless' after 3.5 s

You may have seen some messages scroll by that looked like errors or exceptions, but chances are they were expected to occur within the test cases-  the key indicator of this is that of 623 tests, all of them passed.  

**Step 6.2:** Ideally we could proceed immediately to the second, more comprehensive test, but there is a quirk we need to deal with. Remember that in the previous section you used *v1.0.5* for *fabric* and *fabric-ca* but we are using *v1.0.2* for *fabric-sdk-node*.  Well, the configuration of the comprehensive second test is such that if left unchanged, it will look for Docker images tagged *s390x-1.0.2* but we want to run it with Docker images tagged *s390x-1.0.5*.  This occurs because of the following situation:

1. The tag for the Docker images to be used is specified in the file *test/fixtures/docker-compose.yaml* by an environment variable named DOCKER_IMG_TAG
2. This environment variable is constructed by the test in the file *build/tasks/test.js* 
3. Within *test.js* the end of the tag is based on the value of the *version* name within *fabric-client/package.json*, which happens to be *1.0.2*

*Note:* In all three items above, relative file paths are given that are relative to the *~/git/src/github.com/hyperledger/fabric-sdk-node* directory.  

So there are multiple ways to solve this problem. Any one of these will do the trick:

1. Remove references to the DOCKER_IMG_TAG environment variable within *test/fixtures/docker-compose.yaml*.  Without a tag, Docker will look for images with the tag *latest*, and our *v1.0.5* Hyperledger Fabric Docker images are also tagged with *latest*
2. Go into *build/tasks/test.js*, find where it looks for the *version* name, and add a line to hard-code the value *1.0.5*, either right after, or in place of, the original line.  **Hint:** Line 30 in *build/tasks/test.js* is where this magic happens
3. Go into *fabric-client/package.json* and change the value of the *version* name from *1.0.2* to *1.0.5* in line 3

I am going to guide you through the first option in the next three steps, using the command line.  If you want to go with option 2 or 3 or you want to use *vi* for option 1, go right ahead and then skip to *Step 6.6*.  Otherwise, let's continue.

**Step 6.3:** This step is read-only.  You will use *grep* to see what the lines in the *docker-compose.yaml* file look like before you change them::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-sdk-node$ grep image: test/fixtures/docker-compose.yaml
     image: hyperledger/fabric-ca${DOCKER_IMG_TAG}
     image: hyperledger/fabric-ca${DOCKER_IMG_TAG}
     image: hyperledger/fabric-orderer${DOCKER_IMG_TAG}
     image: hyperledger/fabric-peer${DOCKER_IMG_TAG}
     image: hyperledger/fabric-peer${DOCKER_IMG_TAG}
     image: hyperledger/fabric-couchdb${DOCKER_IMG_TAG}

**Step 6.4:** This step will use the *sed* command to change the file in place and it removes the string *${DOCKER_IMG_TAG}* from the file.  Enter this command exactly as shown (copy-and-paste is highly recommended)::

 [bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-sdk-node$ sed -i -e 's/\${DOCKER_IMG_TAG}//g' test/fixtures/docker-compose.yaml

**Step 6.5:** Repeat the same command you entered in *Step 6.4* and you will see that *${DOCKER_IMG_TAG}* is gone.  This will cause Docker to use the images tagged *latest*, which are the *v1.0.5* images::

 [bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-sdk-node$ grep image: test/fixtures/docker-compose.yaml
     image: hyperledger/fabric-ca
     image: hyperledger/fabric-ca
     image: hyperledger/fabric-orderer
     image: hyperledger/fabric-peer
     image: hyperledger/fabric-peer
     image: hyperledger/fabric-couchdb
 
**Step 6.6:** Run the end-to-end tests with the *gulp test* command.  While this command is running, a little bit of the output may look like errors, but some of the tests expect errors, so the real indicator is, again, like the first test, whether or not all tests passed::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-sdk-node$ gulp test
   .
   . (lots of output not shown here)
   . 
 
 1..950
 # tests 950
 # pass  950

 # ok
 
 ----------|----------|----------|----------|----------|----------------|
 File      |  % Stmts | % Branch |  % Funcs |  % Lines |Uncovered Lines |
 ----------|----------|----------|----------|----------|----------------|
 ----------|----------|----------|----------|----------|----------------|
 All files |      100 |      100 |      100 |      100 |                |
 ----------|----------|----------|----------|----------|----------------|


 =============================== Coverage summary ===============================
 Statements   : 100% ( 0/0 )
 Branches     : 100% ( 0/0 )
 Functions    : 100% ( 0/0 )
 Lines        : 100% ( 0/0 )
 ================================================================================
 [18:12:01] Finished 'test' after 3.02 min
 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-sdk-node$

**Step 6.7:** (Optional) What I really like about the second end-to-end test is that it cleans itself up really well at the beginning- that is, it will remove any artifacts left running at the end of the prior test, so if you wanted to, you could simply enter *gulp test* again if you'd to see this for yourself and have a few minutes to spare.  If you're pressed for time, skip this step::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-sdk-node$ gulp test
   .
   . (output not shown here)
   . 

**Step 6.8:** You will now clean up after the test completes. You will do this by running only the parts "hidden" within the *gulp test* command execution that did the initial cleanup. I have listed five commands in this step.  Only the middle one, the *gulp* command, does the actual cleanup.  The first two and last two *docker* commands are suggested so that you can see before and after the Docker containers and the chaincode Docker images are removed.  The output of these commands are not shown, but pay attention to them as you enter them::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-sdk-node$ docker ps -a
 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-sdk-node$ docker images
 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-sdk-node$ gulp clean-up pre-test docker-clean
 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-sdk-node$ docker ps -a
 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-sdk-node$ docker images

**Recap:** In this section, you ran the Hyperledger Fabric Node.js SDK end-to-end tests and then you cleaned up its leftover artifacts afterward.
This completes this lab.  You have downloaded and built a Hyperledger Fabric network and verified that the setup is correct by successfully running two end-to-end tests-  the CLI end-to-end test and the Node.js SDK end-to-end test- and the shorter Node.js SDK test.

If you really wanted to dig into the details of how the Hyperledger Fabric works, you could do worse than to drill down into the details of each of these tests.  

*** End of Lab! ***
