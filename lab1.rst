Section 1:  Lab Overview
========================
In this lab, you will start with a basic Ubuntu 16.04 server instance running on an IBM z13 Server that resides in the IBM Washington Systems Center in Herndon, Virginia.  During the installation process for this instance, no extra packages were selected for installation beyond the minimum offered by the installer.
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
**Step 1:** Log in to your assigned Ubuntu 16.04 Linux on IBM Z instance using the PuTTY icon on your workstation using the IP address assigned to your team.  This is an example command: ``ssh bcuser@192.168.22.225``. You will be greeted with a message like this::

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

**Step 2:** This system is a relatively “clean” Ubuntu 16.04 instance- “clean” in the sense that after this instance was created,
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
   
**Step 3:** You will be using root authority for several commands throughout this lab.  You can tell when you have root authority by observing the command prompt-  it will end with a ‘#’ when you have root authority and it will end with a ‘$’ when you do not.  Use the sudo command to switch to the root user.  Observe the change in the command prompt after you enter this command::

  bcuser@ubuntu16042:~$ sudo su -
  root@ubuntu16042:~#

**Step 4:** Ubuntu provides a popular software package manager named *apt*, which stands for *Advanced Package Tool*. You will be 
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
 
**Step 5:** Install packages to allow *apt* to use a repository over HTTPS::

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

**Step 6:**  Add Docker’s official GPG key::

 root@ubuntu16042:~# curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
 OK
 root@ubuntu16042:~#

**Step 7:** Verify that the key fingerprint is ``9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88``.::
 
 root@ubuntu16042:~# apt-key fingerprint 0EBFCD88
 pub   4096R/0EBFCD88 2017-02-22
       Key fingerprint = 9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
 uid                  Docker Release (CE deb) <docker@docker.com>
 sub   4096R/F273FCD8 2017-02-22
 
 root@ubuntu16042:~#

**Step 8:** Enter the following command to add the *stable* repository::

 root@ubuntu16042:~# add-apt-repository "deb [arch=s390x] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
 root@ubuntu16042:~#

**Step 9:** Update the *apt* package index again:: 

 root@ubuntu16042:~# apt-get update
 Hit:1 http://us.ports.ubuntu.com/ubuntu-ports xenial InRelease
 Hit:2 http://us.ports.ubuntu.com/ubuntu-ports xenial-updates InRelease                             
 Hit:3 http://us.ports.ubuntu.com/ubuntu-ports xenial-backports InRelease                           
 Hit:4 http://ports.ubuntu.com/ubuntu-ports xenial-security InRelease         
 Get:5 https://download.docker.com/linux/ubuntu xenial InRelease [38.9 kB]
 Get:6 https://download.docker.com/linux/ubuntu xenial/stable s390x Packages [1075 B]
 Fetched 40.0 kB in 0s (139 kB/s)    
 Reading package lists... Done

**Step 10:** Enter this command to show some information about the Docker package.  This command won’t actually install anything::
 
 root@ubuntu16042:~# apt-cache policy docker-ce
 docker-ce:
   Installed: (none)
   Candidate: 17.09.0~ce-0~ubuntu
   Version table:
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
*	*17.09.0~ce* is the candidate version to install- it is the latest version available
*	When you install the software, you will be going out to the Internet to the *download.docker.com* domain to get the software.

**Step 11:** However, you must install version *17.06.2*. Enter this *apt-get* command to install the software.  (Enter Y when prompted to continue)::

 root@ubuntu16042:~# apt-get install docker-ce=17.06.2~ce-0~ubuntu
 Reading package lists... Done
 Building dependency tree       
 Reading state information... Done
 The following additional packages will be installed:
   aufs-tools cgroupfs-mount git git-man liberror-perl libltdl7 patch
 Suggested packages:
   mountall git-daemon-run | git-daemon-sysvinit git-doc git-el git-email git-gui gitk gitweb git-arch git-cvs git-mediawiki
   git-svn diffutils-doc
 The following NEW packages will be installed:
   aufs-tools cgroupfs-mount docker-ce git git-man liberror-perl libltdl7 patch
 0 upgraded, 8 newly installed, 0 to remove and 33 not upgraded.
 Need to get 22.7 MB of archives.
 After this operation, 125 MB of additional disk space will be used.
 Do you want to continue? [Y/n] Y
   .
   .   (remaining output not shown here)
   .

Observe that not only was Docker installed, but so were its prerequisites that were not already installed.

**Step 12:** Issue the *which* command again and this time it will tell you where it found the just-installed docker program::

 root@ubuntu16042:~# which docker
 /usr/bin/docker

**Step 13:** Enter the *docker version* command and you will see that version *17.06.0-ce* was installed, just as expected based on 
the output from your earlier *apt-cache* command::

 root@ubuntu16042:~# docker version
 Client:
  Version:      17.06.2-ce
  API version:  1.30
  Go version:   go1.8.3
  Git commit:   cec0b72
  Built:        Tue Sep  5 20:02:38 2017
  OS/Arch:      linux/s390x

 Server:
  Version:      17.06.2-ce
  API version:  1.30 (minimum version 1.12)
  Go version:   go1.8.3
  Git commit:   cec0b72
  Built:        Tue Sep  5 20:00:51 2017
  OS/Arch:      linux/s390x
  Experimental: false

**Step 14:** Enter *docker info* to see even more information about your Docker environment::

 root@ubuntu16042:~# docker info
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 0
 Server Version: 17.06.2-ce
 Storage Driver: aufs
  Root Dir: /var/lib/docker/aufs
  Backing Filesystem: extfs
  Dirs: 0
  Dirperm1 Supported: true
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
 containerd version: 6e23458c129b551d5c9871e5174f6b1b7f6d1170
 runc version: 810190ceaa507aa2727d7ae6f4790c76ec150bd2
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
 ID: WPJE:WJSJ:LH57:MAC4:RT5Q:QDLF:G7QV:H6BD:637C:UQAA:ATF6:HVB2
 Docker Root Dir: /var/lib/docker
 Debug Mode (client): false
 Debug Mode (server): false
 Registry: https://index.docker.io/v1/
 Experimental: false
 Insecure Registries:
  127.0.0.0/8
 Live Restore Enabled: false
 
 WARNING: No swap limit support 

**Step 15:** For a non-root user to enter Docker commands, you must add that userid to a group named docker.  Enter this command to 
add the *bcuser* userid to the group *docker*::

 root@ubuntu16042:~# usermod -aG docker bcuser

**Step 16:** Exit so that you are no longer running as root::

 root@ubuntu16042:~# exit
 logout
 bcuser@ubuntu16042:~$
 
**Step 17:** Even though *bcuser* was just added to the *docker* group, you will have to log out and then log back in again for this 
change to take effect.  To prove this, enter the *docker info* command before you log out and then again after you log in.  (You may 
need to start a new PuTTY session after you logged out so that you can get back in).

**Step 18:** You will need to get right back in as root to install *Docker Compose*.  Docker Compose is a tool provided by Docker to 
help make it easier to run an application that consists of multiple Docker containers.  On some platforms, it is installed along with 
the Docker package but on Linux on IBM Z it is installed separately.  It is written in Python and you will install it with a tool 
called Pip.  But first you will install Pip itself!  You will do this as root, so enter this again::

 bcuser@ubuntu16042:~$ sudo su -
 root@ubuntu16042:~#
 
**Step 19:** Use the *curl* tool to download a program that will be used to install Pip::

 root@ubuntu16042:~# curl "https://bootstrap.pypa.io/get-pip.py" -o "get-pip.py"
   % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                  Dload  Upload   Total   Spent    Left  Speed
 100 1558k  100 1558k    0     0  2398k      0 --:--:-- --:--:-- --:--:-- 2400k
 
The output of the curl command is written to the file named *get-pip.py* (the value of the *-o* argument).

**Step 20:** Use *python3* to run *get-pip.py* which will get Pip and install it::

 root@ubuntu16042:~# python3 get-pip.py
 Collecting pip
   Downloading pip-9.0.1-py2.py3-none-any.whl (1.3MB)
     100% |################################| 1.3MB 1.0MB/s 
 Collecting setuptools
   Downloading setuptools-36.2.7-py2.py3-none-any.whl (477kB)
     100% |################################| 481kB 2.6MB/s 
 Collecting wheel
   Downloading wheel-0.29.0-py2.py3-none-any.whl (66kB)
     100% |################################| 71kB 11.7MB/s 
 Installing collected packages: pip, setuptools, wheel
 Successfully installed pip-9.0.1 setuptools-36.2.7 wheel-0.29.0
 
**Step 21:** Use Pip to install Docker Compose::

 root@ubuntu16042:~# pip install docker-compose
 
**Step 22:** There was a bunch of output from the prior step I didn’t show, but if your install works, you should feel the love from 
this command::

 root@ubuntu16042:~# docker-compose --version
 docker-compose version 1.15.0, build e12f3b9
 
**Step 23:** Leave root behind and become a normal user again::

 root@ubuntu16042:~# exit
 logout
 bcuser@ubuntu16042:~$

**Step 24:** You won’t have to log out and log back in, like you did with Docker, in order to use Docker Compose, and to prove it, 
check for the version again now that you are no longer root::

 bcuser@ubuntu16042:~$ docker-compose --version
 docker-compose version 1.15.0, build e12f3b9

**Step 25:** The next thing you are going to install is the *Golang* programming language. You are going to install Golang version 
1.7.5.  Go to the /tmp directory::

 bcuser@ubuntu16042:~$ cd /tmp
 bcuser@ubuntu16042:/tmp$

**Step 26:** Use *wget* to get the compressed file that contains the Golang compiler and tools.  And now is a good time to tell you 
that from here on out I will just call Golang what everybody else usually calls it-  *Go*.  Go figure.
::
 bcuser@ubuntu16042:/tmp$ wget --no-check-certificate https://storage.googleapis.com/golang/go1.7.5.linux-s390x.tar.gz
 --2017-08-16 16:31:09--  https://storage.googleapis.com/golang/go1.7.5.linux-s390x.tar.gz
 Resolving storage.googleapis.com (storage.googleapis.com)... 172.217.7.240, 2607:f8b0:4004:802::2010
 Connecting to storage.googleapis.com (storage.googleapis.com)|172.217.7.240|:443... connected.
 HTTP request sent, awaiting response... 200 OK
 Length: 70205840 (67M) [application/x-gzip]
 Saving to: 'go1.7.5.linux-s390x.tar.gz' 
 
 go1.7.5.linux-s390x.ta 100%[============================>]  66.95M  21.7MB/s    in 3.3s
 
 2017-08-16 16:31:15 (20.3 MB/s) - 'go1.7.5.linux-s390x.tar.gz' saved [70205840/70205840]

**Step 27:** Enter the following command which will extract the files into the /tmp directory, and provide lots and lots of output.
(It’s the *‘v’* in *-xvf* which got all chatty, or *verbose*, on you)::

 bcuser@ubuntu16042:/tmp$ tar -xvf go1.7.5.linux-s390x.tar.gz
   .
   .  (output not shown here)
   .

**Step 28:** You will move the extracted stuff, which is all under */tmp/go*, into */opt*, and for that you will need root authority.
Whereas before you were instructed to enter *sudo su* – which effectively logged you in as root until you exited, you can issue a 
single command with *sudo* which executes it as root and then returns control back to you in non-root mode.   Enter this command::

 bcuser@ubuntu16042:/tmp$ sudo mv go /opt
 bcuser@ubuntu16042:/tmp$
 
**Step 29:** You need to set a couple of Go-related environment variables.  First check to verify that they are not set already::

 bcuser@ubuntu16042:/tmp$ env | grep GO

That command, *grep*, is looking for any lines of input that contain the characters *GO*.  Its input is the output of the previous *env*
command, which prints all of your environment variables.

**Step 30:**  You will set these values now.  If you know how to use the *vi* editor, feel free to make the changes with that.  If not, 
you can use the *echo* command to make the changes.  You can make these changes in a special hidden file named *.bashrc* in your home 
directory.  Change to your home directory::

 bcuser@ubuntu16042:/tmp$ cd ~  # that is a tilde ~ character I know it is hard to see 
 bcuser@ubuntu16042:~$

**Step 31:** Enter the following commands exactly as shown.  It is critical that you use two ‘greater-than’ signs, i.e., ‘>>’, when you 
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

**Step 32:** Let’s see how you did.  Enter this command::

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
you screwed up and not used two ‘>>’ like I told you, you would have whacked this stuff.  Your stuff is at the bottom.  If *head* 
prints the top of the file, guess what command prints the bottom of the file.

**Step 33:** Try this::

 bcuser@ubuntu16042:~$ tail -5 .bashrc
 
 export GOPATH=/home/bcuser/git
 export GOROOT=/opt/go
 export PATH=/opt/go/bin:$PATH

**Step 34:** These changes will take effect next time you log in, but you can make them take effect immediately by entering this::

 bcuser@ubuntu16042:~$ source .bashrc

**Step 35:** Try this to see if your changes took::

 bcuser@ubuntu16042:~$ env | grep GO
 GOROOT=/opt/go
 GOPATH=/home/bcuser/git

**Step 36:**  Then try this::

 bcuser@ubuntu16042:~$ go version
 go version go1.7.5 linux/s390x

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

**Step 1:** There are some software packages necessary to be able to successfully build the Hyperledger Fabric source code.  Install them with 
this command.  You have used the *apt-get* command earlier in the lab.  This time, use the *‘-y’* flag which will automatically 
reply ‘Y’ for you so that you will not be prompted to continue.  Observe the output, not shown here, to see the different packages 
installed::

 bcuser@ubuntu16042:~$ sudo apt-get install -y build-essential libltdl3-dev git
 
**Step 2:** Create the following directory path with this command.  Make sure you are in your home directory when you enter it::

 bcuser@ubuntu16042:~$ mkdir -p git/src/github.com/hyperledger
 bcuser@ubuntu16042:~$
 
**Step 3:** Navigate to the directory you just created::

 bcuser@ubuntu16042:~$ cd git/src/github.com/hyperledger/
 bcuser@ubuntu16042:~/git/src/github.com/hyperledger$
 
**Step 4:** Use the software tool *git* to download the source code of the Hyperledger Fabric core package from the official place 
where it lives::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger$ git clone https://gerrit.hyperledger.org/r/fabric
 Cloning into 'fabric'...
 remote: Counting objects: 9, done
 remote: Total 45444 (delta 0), reused 45444 (delta 0)
 Receiving objects: 100% (45444/45444), 58.47 MiB | 4.28 MiB/s, done.
 Resolving deltas: 100% (21743/21743), done.
 Checking connectivity... done.

**Step 5:** Switch to the *fabric* directory, which is the top directory of where the *git* command put the code it just downloaded::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger$ cd fabric
 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric$

**Step 6:** Enter this *git* command which will show the status of the code you just pulled down::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric$ git status
 On branch master
 Your branch is up-to-date with 'origin/master'.
 nothing to commit, working directory clean
 
You pulled down, by default, the master branch of the Hyperledger Fabric code.  The master branch is updated quite often still, so you will check out a version of the code that is a little bit less recent than the latest code.  You will be checking out a level of the code that has been verified to work for the activities in this lab. 

**Step 7:** Your instructor may provide you with a code level to checkout that would replace the level, *v1.0.1*, specified in 
this example.  If not, use the level from this example::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric$ git checkout v1.0.1
 Note: checking out 'v1.0.1'.
 
 You are in 'detached HEAD' state. You can look around, make experimental
 changes and commit them, and you can discard any commits you make in this
 state without impacting any branches by performing another checkout. 

 If you want to create a new branch to retain commits you create, you may
 do so (now or later) by using -b with the checkout command again. Example:
 
   git checkout -b <new-branch-name>
 
 HEAD is now at e43b68f... FAB-5520 Release Hyperledger Fabric v1.0.1

**Step 8:** In mid-August 2017, one of the mirrors used during a Docker build began to cause consistent failures.  As a result, a 
change was introduced to use a more reliable mirror.  In order to add this “patch” to the 1.0.1 release, it is necessary to use *git* 
to “cherry-pick” the fix and apply it to v1.0.1. But in order for this to work, you first need to configure two git configuration variables- 
you can substitute the example values shown with more realistic values if you want to::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric$ git config --global user.email "you@example.com"
 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric$ git config --global user.name "Your Name"
 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric$ git fetch https://gerrit.hyperledger.org/r/fabric refs/changes/69/12369/1 && git cherry-pick FETCH_HEAD
 From https://gerrit.hyperledger.org/r/fabric
  * branch            refs/changes/69/12369/1 -> FETCH_HEAD
 [detached HEAD 17fa61a] FAB-5739 Update maven curl command
  Author: rameshthoomu <rameshbabu.thoomu@gmail.com>
  Date: Fri Aug 11 16:23:57 2017 -0400
  1 file changed, 1 insertion(+), 1 deletion(-)
 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric$

**Step 9:** One of the tools installed in step 1 was a program called *make*, which is used to build software projects.  You will use 
it to build Docker images for Hyperledger Fabric.  But first, run this command to show that your system does not currently have any 
Docker images stored on it.  The only output you will see is the column headings::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric$ docker images
 REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE

**Step 10:** That will change in a few minutes.  Enter the following command, which will build the Hyperledger Fabric images.  You 
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

**Step 11:** Run *docker images* again and you will see several Docker images that were just created. You will notice that all but 
the last two of the Docker images were created in the last few minutes.  These were created by the *make docker* command.  Docker 
images can be built based on other Docker images.  The two Docker images listed at the bottom, that are a few months old, were used 
in the process, and were downloaded from the Hyperledger Fabric’s public DockerHub repository.  Your output should look similar to 
that shown here, although the tags will be different if your instructor gave you a different level to checkout, and your *image ids* 
will be different either way::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric$ docker images
 REPOSITORY                     TAG                                   IMAGE ID            CREATED             SIZE
 hyperledger/fabric-tools       latest              5d841481c6b3        2 minutes ago       1.44GB
 hyperledger/fabric-tools       s390x-1.0.1         5d841481c6b3        2 minutes ago       1.44GB
 hyperledger/fabric-couchdb     latest              729eccd0e1d4        3 minutes ago       1.63GB
 hyperledger/fabric-couchdb     s390x-1.0.1         729eccd0e1d4        3 minutes ago       1.63GB
 hyperledger/fabric-kafka       latest              9d7161a4130f        4 minutes ago       1.4GB
 hyperledger/fabric-kafka       s390x-1.0.1         9d7161a4130f        4 minutes ago       1.4GB
 hyperledger/fabric-zookeeper   latest              8aeee7ddc8e5        4 minutes ago       1.41GB
 hyperledger/fabric-zookeeper   s390x-1.0.1         8aeee7ddc8e5        4 minutes ago       1.41GB
 hyperledger/fabric-testenv     latest              97f25587379c        5 minutes ago       1.5GB
 hyperledger/fabric-testenv     s390x-1.0.1         97f25587379c        5 minutes ago       1.5GB
 hyperledger/fabric-buildenv    latest              c3e57f5f5165        5 minutes ago       1.42GB
 hyperledger/fabric-buildenv    s390x-1.0.1         c3e57f5f5165        5 minutes ago       1.42GB
 hyperledger/fabric-orderer     latest              cc78bf4f171f        5 minutes ago       194MB
 hyperledger/fabric-orderer     s390x-1.0.1         cc78bf4f171f        5 minutes ago       194MB
 hyperledger/fabric-peer        latest              1081f30047d7        5 minutes ago       197MB
 hyperledger/fabric-peer        s390x-1.0.1         1081f30047d7        5 minutes ago       197MB
 hyperledger/fabric-javaenv     latest              a4d5b4ac736e        6 minutes ago       1.48GB
 hyperledger/fabric-javaenv     s390x-1.0.1         a4d5b4ac736e        6 minutes ago       1.48GB
 hyperledger/fabric-ccenv       latest              0109ec5a3d35        6 minutes ago       1.39GB
 hyperledger/fabric-ccenv       s390x-1.0.1         0109ec5a3d35        6 minutes ago       1.39GB
 hyperledger/fabric-baseimage   s390x-0.3.1         a165b6238eee        3 months ago        1.37GB
 hyperledger/fabric-baseos      s390x-0.3.1         2293f6d33733        3 months ago        171MB  

**Step 12:** Navigate to the directory where the “end-to-end” test lives::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric$ cd examples/e2e_cli/
 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric/examples/e2e_cli$

**Step 13:** The end-to-end test that you are about to run will create several Docker containers.  A Docker container is what runs a 
process, and it is based on a Docker image.  Run this command, which shows all Docker containers, and right now there will be no 
output other than column headings, which indicates no Docker containers are currently running::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric/examples/e2e_cli$ docker ps -a
 CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

**Step 14:** Run the end-to-end test with this command::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric/examples/e2e_cli$ ./network_setup.sh up mychannel 10 couchdb
   .
   . (output not shown here)
   .
 ===================== Query on PEER3 on channel 'mychannel' is successful =====================
 
 ===================== All GOOD, End-2-End execution completed =====================
   .
   . (output not shown here)
   .

**Step 15:** Run the *docker ps* command to see the Docker containers that the test created::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric/examples/e2e_cli$ docker ps -a
 CONTAINER ID        IMAGE                                 COMMAND                  CREATED             STATUS                         PORTS                                              NAMES
 c2007f7ca927        dev-peer1.org2.example.com-mycc-1.0   "chaincode -peer.addr"   23 seconds ago       Up 22 seconds                                                                  dev-peer1.org2.example.com-mycc-1.0
 c1d42da45625        dev-peer0.org1.example.com-mycc-1.0   "chaincode -peer.addr"   40 seconds ago       Up 39 seconds                                                                  dev-peer0.org1.example.com-mycc-1.0
 21376da7dda7        dev-peer0.org2.example.com-mycc-1.0   "chaincode -peer.addr"   57 seconds ago       Up 56 seconds                                                                  dev-peer0.org2.example.com-mycc-1.0
 0a74ddfde3d7        hyperledger/fabric-tools              "/bin/bash -c './scri"   About a minute ago   Exited (0) 12 seconds ago                                                      cli
 efc6886fa41a        hyperledger/fabric-peer               "peer node start"        About a minute ago   Up About a minute           0.0.0.0:7051->7051/tcp, 0.0.0.0:7053->7053/tcp     peer0.org1.example.com
 c2d3a3e6e0ac        hyperledger/fabric-peer               "peer node start"        About a minute ago   Up About a minute           0.0.0.0:10051->7051/tcp, 0.0.0.0:10053->7053/tcp   peer1.org2.example.com
 5a78d69ca6d1        hyperledger/fabric-peer               "peer node start"        About a minute ago   Up About a minute           0.0.0.0:8051->7051/tcp, 0.0.0.0:8053->7053/tcp     peer1.org1.example.com
 e89a42e8f0f8        hyperledger/fabric-peer               "peer node start"        About a minute ago   Up About a minute           0.0.0.0:9051->7051/tcp, 0.0.0.0:9053->7053/tcp     peer0.org2.example.com
 ac4a842439f4        hyperledger/fabric-couchdb            "tini -- /docker-entr"   About a minute ago   Up About a minute           4369/tcp, 9100/tcp, 0.0.0.0:5984->5984/tcp         couchdb0
 424fc9b9b954        hyperledger/fabric-couchdb            "tini -- /docker-entr"   About a minute ago   Up About a minute           4369/tcp, 9100/tcp, 0.0.0.0:6984->5984/tcp         couchdb1
 1d64e5419bae        hyperledger/fabric-orderer            "orderer"                About a minute ago   Up About a minute           0.0.0.0:7050->7050/tcp                             orderer.example.com
 3391d79d6fb7        hyperledger/fabric-couchdb            "tini -- /docker-entr"   About a minute ago   Up About a minute           4369/tcp, 9100/tcp, 0.0.0.0:8984->5984/tcp         couchdb3
 5626c8ed1760        hyperledger/fabric-couchdb            "tini -- /docker-entr"   About a minute ago   Up About a minute           4369/tcp, 9100/tcp, 0.0.0.0:7984->5984/tcp         couchdb2

The first three Docker containers listed are chaincode containers-  The chaincode was run on three of the four peers, so they each 
had a Docker image and container created.  There were also four peer containers created, each with a couchdb container, and one 
orderer container. There was a container created to run the CLI itself, and that container stopped running ten seconds after the 
test ended.  (That was what the value *10* was for in the *./network_setup.sh* command you ran).

You have successfully run the CLI end-to-end test.  You will clean things up now.

**Step 16:** Run the *network_setup.sh* script with different arguments to bring the Docker containers down::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric/examples/e2e_cli$ ./network_setup.sh down

**Step 17:** Try the *docker ps* command again and you should see that there are no longer any Docker containers running::

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

**Step 1:** Back up to the *hyperledger* directory::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric/examples/e2e_cli$ cd ../../..
 bcuser@ubuntu16042:~/git/src/github.com/hyperledger$

**Step 2:** Get the source code for the CA using *git*::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger$ git clone https://gerrit.hyperledger.org/r/fabric-ca
 Cloning into 'fabric-ca'...
 remote: Counting objects: 3536, done
 remote: Finding sources: 100% (90/90)
 remote: Total 7335 (delta 11), reused 7290 (delta 11)
 Receiving objects: 100% (7335/7335), 21.57 MiB | 3.00 MiB/s, done.
 Resolving deltas: 100% (2433/2433), done.Checking connectivity... done.

**Step 3:** Navigate to the *fabric-ca* directory, which is the top directory of where the *git* command put the code it just 
downloaded::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger$ cd fabric-ca
 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-ca$

**Step 4:** Enter this *git* command which will show the status of the code you just pulled down::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-ca$ git status
 On branch master
 Your branch is up-to-date with 'origin/master'.
 nothing to commit, working directory clean

You pulled down, by default, the master branch of the Hyperledger Fabric CA code.  The master branch is updated quite often still, so you will check out a version of the code that is a little bit less recent than the latest code.  You will be checking out a level of the code that has been verified to work for the activities in this lab. 

**Step 5:** Your instructor may provide you with a code level to checkout that would replace the level, *v1.0.1*, specified in this 
example.  If not, use the level from this example::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-ca$ git checkout v1.0.1
 Note: checking out 'v1.0.1'.
 
 You are in 'detached HEAD' state. You can look around, make experimental
 changes and commit them, and you can discard any commits you make in this
 state without impacting any branches by performing another checkout.
 
 If you want to create a new branch to retain commits you create, you may
 do so (now or later) by using -b with the checkout command again. Example:
 
   git checkout -b <new-branch-name>
 
 HEAD is now at 0262ccb... FAB-5067 Release Hyperledger Fabric CA v1.0.1

**Step 6:** Enter the following command, which will build the Hyperledger Fabric CA images.  You can ‘wrap’ the *make* command, which 
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

**Step 7:** Enter the *docker images* command and you will see at the top of the output the Docker image that was just created for 
the Certificate Authority::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-ca$ docker images
 REPOSITORY                     TAG                 IMAGE ID            CREATED             SIZE
 hyperledger/fabric-ca          latest              daaa35d81b43        17 seconds ago      255MB
 hyperledger/fabric-ca          s390x-1.0.1         daaa35d81b43        17 seconds ago      255MB
   .
   . (output not shown here)
   .

I have shown two lines of output but implied that one Docker image is created for the CA.  What you have here is one image-  uniquely 
identified by its image ID, *daaa35d81b43* in this example; yours will surely be different. An image can be given any number of names, 
or *tags*.  Think of these *tags* as nicknames, or aliases.  The *make* process first gave the Docker image it created a descriptive 
tag, *s390x-1.0.1*, and then it ‘tagged’ it with a new tag, *latest*.  It did that for a reason.  When you are working with Docker 
images, if you specify an image without specifying a tag, the tag defaults to the name *latest*. So, for this image, you can either 
refer to *hyperledger/fabric-ca*, *hyperledger/fabric-ca:latest*, or *hyperledger/fabric-ca:s390x-1.0.1*. 

**Recap:** In this section, you downloaded the source code for the Hyperledger Fabric Certificate Authority and built it.  That was easy.
 
Section 5: Install Hyperledger Fabric Node.js SDK and its prerequisite software
===============================================================================
The preferred way for an application to interact with a Hyperledger Fabric chaincode is through a Software Development Kit (SDK) that 
exposes APIs.  The Hyperledger Fabric Node.js SDK is very popular among developers, due to the popularity of JavaScript as a programming 
language for developing web applications and the popularity of Node.js as a runtime platform for running server-side JavaScript.

In this section, you will install and configure Node.js, which also includes a program called *npm*, which is the de facto Node.js 
package manager.  You will also install Python v2.7-  Python v2.x is required to build the packages required by the Hyperledger Fabric 
Node.js SDK.  Your system already has Python 3 installed on it-  you used that to install Pip earlier.  But some of the Node.js SDK 
prerequisites require Python v2.7.

Then you will download the Hyperledger Fabric Node.js SDK and install npm packages that it requires.

**Step 1:** Change to the */tmp* directory::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-ca$ cd /tmp
 bcuser@ubuntu16042:/tmp$

**Step 2:** Retrieve the *Node.js* package with this command::

 bcuser@ubuntu16042:/tmp$ wget  https://nodejs.org/dist/v6.10.3/node-v6.10.3-linux-s390x.tar.xz
 --2017-06-08 15:51:49--  https://nodejs.org/dist/v6.10.3/node-v6.10.3-linux-s390x.tar.xz
 Resolving nodejs.org (nodejs.org)... 104.20.22.46, 104.20.23.46, 2400:cb00:2048:1::6814:172e, ...
 Connecting to nodejs.org (nodejs.org)|104.20.22.46|:443... connected.
 HTTP request sent, awaiting response... 200 OK
 Length: 8959812 (8.5M) [application/x-xz]
 Saving to: 'node-v6.10.3-linux-s390x.tar.xz'
 
 node-v6.10.3-linux- 100%[===================>]   8.54M  20.4MB/s    in 0.4s
 
 2017-06-08 15:51:50 (20.4 MB/s) - 'node-v6.10.3-linux-s390x.tar.xz' saved [8959812/8959812]

**Step 3:** Extract the package underneath your home directory, */home/bcuser*. This will cause the executables to be wind up in 
*/home/bcuser/bin*, which is in your path::

 bcuser@ubuntu16042:/tmp$ cd /home/bcuser && tar --strip-components=1 -xf /tmp/node-v6.10.3-linux-s390x.tar.xz

**Step 4:** In this step, you will issue some commands that will show you where *node* and *npm* reside, and what version of each is 
installed::

 bcuser@ubuntu16042:/tmp$ which node
 /home/bcuser/bin/node
 bcuser@ubuntu16042:/tmp $ which npm
 /home/bcuser/bin/npm
 bcuser@ubuntu16042:/tmp $ node --version
 v6.10.3
 bcuser@ubuntu16042:/tmp$ npm --version
 3.10.10

**Step 5:** Use *apt-get* to install Python2.  I have tucked in a *which* command in front of it to try to convince you that python 
is not currently available::

 bcuser@ubuntu16042:/tmp$ which python
 bcuser@ubuntu16042:/tmp$ sudo apt-get install python-dev    # reply ‘Y’ when prompted to continue
   .
   . (output not shown here)
   .

**Step 6:** Now *which* will show you which directory python lives in, and then you’ll see that Python 2.7 was installed::

 bcuser@ubuntu16042:/tmp$ which python
 /usr/bin/python
 bcuser@ubuntu16042:/tmp$ python --version
 Python 2.7.12

**Step 7:** Switch to the *~/git/src/github.com/hyperledger* directory::

 bcuser@ubuntu16042:/tmp$ cd ~/git/src/github.com/hyperledger/
 bcuser@ubuntu16042:~/git/src/github.com/hyperledger$

**Step 8:** Now you will download the Hyperledger Fabric Node SDK source code from its official repository::

 bcuser@ubuntu16042: ~/git/src/github.com/hyperledger $ git clone https://gerrit.hyperledger.org/r/fabric-sdk-node
 Cloning into 'fabric-sdk-node'...
 remote: Counting objects: 7, done
 remote: Total 5459 (delta 0), reused 5459 (delta 0)
 Receiving objects: 100% (5459/5459), 2.75 MiB | 433.00 KiB/s, done.
 Resolving deltas: 100% (2772/2772), done. 
 Checking connectivity... done.

**Step 9:** And then change to the *fabric-sdk-node* directory which was just created::

 bcuser@ubuntu16042: ~/git/src/github.com/hyperledger $ cd fabric-sdk-node
 bcuser@ubuntu16042: ~/git/src/github.com/hyperledger/fabric-sdk-node$

**Step 10:** Enter this *git* command which will show the status of the code you just pulled down::

 bcuser@ubuntu16042: ~/git/src/github.com/hyperledger/fabric-sdk-node$ git status
 On branch master
 Your branch is up-to-date with 'origin/master'.
 nothing to commit, working directory clean

**Step 11:** This should be a familiar refrain- your instructor may provide you with a code level to checkout that would replace the 
level, *v1.0.1*, specified in this example.  If not, use the level from this example::

 bcuser@ubuntu16042: ~/git/src/github.com/hyperledger/fabric-sdk-node$ git checkout v1.0.1
 Note: checking out 'v1.0.1'.
 
 You are in 'detached HEAD' state. You can look around, make experimental
 changes and commit them, and you can discard any commits you make in this
 state without impacting any branches by performing another checkout.
 
 If you want to create a new branch to retain commits you create, you may
 do so (now or later) by using -b with the checkout command again. Example:
 
   git checkout -b <new-branch-name>

 HEAD is now at e7b80dc... FAB-5694 Release fabric-sdk-node v1.0.1

**Step 12:** You are about to install the packages that the Hyperledger Fabric Node SDK would like to use. Before you start, 
run *npm list* to see that you are starting with a blank slate::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-sdk-node$ npm list
 fabric-sdk-node@1.0.0-snapshot /home/bcuser/git/src/github.com/hyperledger/fabric-sdk-node
 `-- (empty)
 bcuser@ubuntu16042: ~/git/src/github.com/hyperledger/fabric-sdk-node$

**Step 13:** Run *npm install* to install the required packages.  This will take a few minutes and will produce a lot of output::

 bcuser@ubuntu16042: ~/git/src/github.com/hyperledger/fabric-sdk-node$ npm install
   .
   . (output not shown here)
   .
 npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@^1.0.0 (node_modules/chokidar/node_modules/fsevents):
 npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@1.1.1: wanted {"os":"darwin","arch":"any"} (current: {"os":"linux","arch":"s390x"})

You may ignore the *WARN* messages at the end of the output.  If there is a serious error, the end of the output will leave little 
doubt about it.

**Step 14:** Repeat the *npm list* command.  The output, although not shown here, will be anything but empty.  This just proves what 
everyone suspected-  programmers would much rather use other peoples’ code than write their own.  Not that there’s anything wrong 
with that.
::
 bcuser@ubuntu16042: ~/git/src/github.com/hyperledger/fabric-sdk-node$ npm list
   .
   . (output not shown here, but surely you will agree it is not empty)
   .
 bcuser@ubuntu16042: ~/git/src/github.com/hyperledger/fabric-sdk-node$

**Step 15:** Now you will install an automation tool named *gulp* at a global level, using the *-g* argument to the *npm install* 
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

**Step 16:** Next you will install a code coverage testing tool named *istanbul*, also at a global level::

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
*	Installed Python 2.7
*	Downloaded the Hyperledger Fabric Node.js SDK
*	Installed the *npm* packages required by the Hyperledger Fabric Node.js SDK
*	Installed the *gulp* and *istanbul* packages to run the Hyperledger Fabric Node.js SDK end-to-end test (which you will run in the next section)
 
Section 6: Run the Hyperledger Fabric Node.js SDK end-to-end tests
==================================================================
In this section, you will run the end-to-end tests provided by the Hyperledger Fabric Node.js SDK, verify their successful 
operation, and clean up afterwards.

**Step 1:** Switch to the *test/fixtures* directory::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-sdk-node$ cd test/fixtures
 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-sdk-node/test/fixtures$

**Step 2:** Start the Hyperledger Fabric network to be used by the tests with this Docker Compose command::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-sdk-node/test/fixtures$ docker-compose up -d
 Creating network "fixtures_default" with the default driver
 Creating ca_peerOrg2
 Creating ca_peerOrg1
 Creating orderer.example.com
 Creating couchdb
 Creating peer0.org2.example.com
 Creating peer0.org1.example.com

This command reads a file in the current directory named *docker-compose.yaml*.  (You can change which file it reads with the optional *-f* argument).  The *up* argument to the command says to bring up the services defined in this file.  Services are brought up as Docker containers, and each Docker container is based on a Docker image.  The *-d* argument says to run this command in *detached* mode- that is, the containers are run in the background; otherwise your SSH session would be flooded with output from the containers. In the order listed in its output in this example (the order listed in your output may differ), Docker creates

*	an internal Docker network that the Docker containers will use to communicate with each other.
*	A container for the Certificate Authority for *peerOrg2*
*	A container for the Certificate Authority for *peerOrg1*
*	A container for the orderer service
*	A container for CouchDB
*	A container for the peer for *peerOrg2*
*	A container for the peer for *peerOrg1*

**Step 3:** Run this command and ensure that all containers are in the *Up* state.  If any containers are *Exited*, that indicates a 
problem that will need to be corrected before you can proceed::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-sdk-node/test/fixtures$ docker ps -a
 CONTAINER ID        IMAGE                        COMMAND                  CREATED             STATUS              PORTS                                            NAMES
 a44206beb823        hyperledger/fabric-peer      "peer node start"        2 minutes ago       Up 2 minutes        0.0.0.0:7051->7051/tcp, 0.0.0.0:7053->7053/tcp   peer0.org1.example.com
 1fbab289836a        hyperledger/fabric-peer      "peer node start"        2 minutes ago       Up 2 minutes        0.0.0.0:8051->7051/tcp, 0.0.0.0:8053->7053/tcp   peer0.org2.example.com
 8cf5a47dbfd8        hyperledger/fabric-orderer   "orderer"                2 minutes ago       Up 2 minutes        0.0.0.0:7050->7050/tcp                           orderer.example.com
 438c333f5d0b        hyperledger/fabric-couchdb   "tini -- /docker-entr"   2 minutes ago       Up 2 minutes        4369/tcp, 9100/tcp, 0.0.0.0:5984->5984/tcp       couchdb
 91ac7853ecec        hyperledger/fabric-ca        "sh -c 'fabric-ca-ser"   2 minutes ago       Up 2 minutes        0.0.0.0:7054->7054/tcp                           ca_peerOrg1
 c65a9227eb54        hyperledger/fabric-ca        "sh -c 'fabric-ca-ser"   2 minutes ago       Up 2 minutes        0.0.0.0:8054->7054/tcp                           ca_peerOrg2

**Step 4:** Go back up two directory levels::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-sdk-node/test/fixtures$ cd ../..
 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-sdk-node$

**Step 5:** Run the end-to-end tests with the *gulp test* command.  While this command is running, read the commentary that
I have graciously provided below for your edification, in lieu of the actual output, which is voluminous::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-sdk-node$ gulp test
   .
   . lots and lots of output not shown; much of it will look like errors
   . but don’t panic – YET! – the tests are intentionally designed to introduce
   . errors so that the correct response to these errors can be detected
   . what you are really looking for, after the test ends, which will take
   . a few minutes, is something like this, where all tests have passed
   . 
 
 1..940
 # tests 940
 # pass  940 
 
 # ok
   .
   . you will have to scroll up a little bit to see the numbers above, and your 
   . actual number may vary, but the important thing is that all tests passed 
   . 
 =============================== Coverage summary ===============================
 Statements   : 82.01% ( 3441/4196 )
 Branches     : 69.52% ( 1184/1703 )
 Functions    : 77.33% ( 406/525 )
 Lines        : 82.24% ( 3426/4166 )
 ================================================================================
 [18:12:06] Finished 'test' after 3.05 min
 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-sdk-node$

**Step 6:** You will now clean up after the test completes.  Navigate back down to the *test/fixtures* directory::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-sdk-node$ cd test/fixtures
 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-sdk-node/test/fixtures$

**Step 7:** If *docker-compose up* brought up the Hyperledger Fabric network, figuring out what *docker-compose down* does to the 
Hyperledger Fabric network is left as an exercise for the reader::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-sdk-node/test/fixtures$ docker-compose down
 Stopping peer0.org1.example.com ... done
 Stopping peer0.org2.example.com ... done
 Stopping ca_peerOrg1 ... done
 Stopping orderer.example.com ... done
 Stopping couchdb ... done
 Stopping ca_peerOrg2 ... done
 Removing peer0.org1.example.com ... done
 Removing peer0.org2.example.com ... done
 Removing ca_peerOrg1 ... done
 Removing orderer.example.com ... done
 Removing couchdb ... done
 Removing ca_peerOrg2 ... done
 Removing network fixtures_default

**Step 8:** Here’s the thing.  The *docker-compose down* command stops *(Stopping …)* and then deletes *(Removing …)* each container 
that it created when you ran the *docker-compose up* command.  But as part of the end-to-end tests, Docker containers for the 
chaincode were also created.  These containers were not specified in the *docker-compose.yaml* file, so *docker-compose down* leaves 
them alone.  Those containers are brought down when the peer nodes are brought down, by design, but they have not been removed.  
Issue this *docker* command to show their status::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-sdk-node/test/fixtures$ docker ps -a
 CONTAINER ID        IMAGE                                                                     COMMAND                  CREATED             STATUS                      PORTS               NAMES
 4e00cf53e125        dev-peer0.org1.example.com-events_unit_test1497024802464-v1497024802464   "chaincode -peer.addr"   22 minutes ago      Exited (0) 11 minutes ago                       dev-peer0.org1.example.com-events_unit_test1497024802464-v1497024802464
 14900fd61036        dev-peer0.org2.example.com-end2endnodesdk-v1                              "chaincode -peer.addr"   23 minutes ago      Exited (0) 11 minutes ago                       dev-peer0.org2.example.com-end2endnodesdk-v1
 ef077d8ad12a        dev-peer0.org1.example.com-end2endnodesdk-v1                              "chaincode -peer.addr"   23 minutes ago      Exited (0) 11 minutes ago                       dev-peer0.org1.example.com-end2endnodesdk-v1
 1383ac1bfbd8        dev-peer0.org1.example.com-end2endnodesdk-v0                              "chaincode -peer.addr"   23 minutes ago      Exited (0) 11 minutes ago                       dev-peer0.org1.example.com-end2endnodesdk-v0
 4bf0b25d4f01        dev-peer0.org2.example.com-end2endnodesdk-v0                              "chaincode -peer.addr"   23 minutes ago      Exited (0) 11 minutes ago                       dev-peer0.org2.example.com-end2endnodesdk-v0

**Step 9:** There are several ways you can delete these containers.  This way is the surest way to impress your friends, as long as 
you do not try this in production::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-sdk-node/test/fixtures$ docker rm $(docker ps -aq)
 4e00cf53e125
 14900fd61036
 ef077d8ad12a
 1383ac1bfbd8
 4bf0b25d4f01

If you were in production, the safest, and most tedious way to delete these containers would be with this command::

 docker rmi 4e00cf53e125 14900fd61036 ef077d8ad12a 1383ac1bfbd8 4bf0b25d4f01

The first command would remove all exited containers, so if other applications were running on your system and had exited Docker 
containers, their containers would be deleted.  That is why I cautioned against running that command in production, at least not 
without thinking through its implications.

**Step 10:**  Rerun *docker ps -a* and you should not see any containers now, just column headings::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-sdk-node/test/fixtures$ docker ps -a
 CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

**Step 11:** The chaincode Docker images should also be deleted as part of cleanup.  You can display them by using the following 
command which will display only the Hyperledger Fabric Docker images which you want to delete::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-sdk-node/test/fixtures$ docker images dev-*
 REPOSITORY                                                                                                                                 TAG                 IMAGE ID            CREATED             SIZE
 dev-peer0.org1.example.com-end2endnodesdk-v3-78c36fcdd427a2cedc3441743b894733bfd2f1440c7ad6bd8c7b825981f2e5c9                              latest              0a3da9ff92b7        4 minutes ago       188MB
 dev-peer0.org2.example.com-end2endnodesdk-v3-d83d9a69fa471c4cd45b511a29301b5ed30b2a9f07847b9ae5a34f8f99c7f141                              latest              3debe05c63f2        4 minutes ago       188MB
 dev-peer0.org1.example.com-events_unit_test1502921343951-v1502921343951-b53d9027affbd20c306d0ec9cc998cc96d4dd1c92b4e88873832c9a52fcc0443   latest              012f23d10e32        5 minutes ago       188MB
 dev-peer0.org1.example.com-end2endnodesdk-v1-cb50140fc38a1dbff755119ff4f1af9c21ea51dd33e23af11035622c35921bd4                              latest              c76bd339aa57        5 minutes ago       188MB
 dev-peer0.org2.example.com-end2endnodesdk-v1-28ad5b85f1199c9112eb1ecc700a3d1df6e02826c37d26e0fa4b7435c6970156                              latest              c8d9f255e6c4        5 minutes ago       188MB
 dev-peer0.org1.example.com-end2endnodesdk-v0-2c1b3eb0a77138303953abb093fcd1df798601e1dc45c1d0d76fd23d671f44ad                              latest              c8cb3d82921d        6 minutes ago       188MB
 dev-peer0.org2.example.com-end2endnodesdk-v0-cffecf663c4cac97a99d46282042dc47e9b6b306eb3e1e3d271cf3b25f1e9958                              latest              cc447c1cad50        6 minutes ago       188MB

**Step 12:** Now add the *-q* (“quiet”) argument to the preceding command and you will see only the appropriate Image Ids::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-sdk-node/test/fixtures$ docker images -q dev-*
 0a3da9ff92b7
 3debe05c63f2
 012f23d10e32
 c76bd339aa57
 c8d9f255e6c4
 c8cb3d82921d
 cc447c1cad50

**Step 13:** Finally, wrap this command inside the *docker rmi* command and it will remove these Docker images::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-sdk-node/test/fixtures$ docker rmi $(docker images -q dev-*)
    .
    . (output not shown)
    .

**Step 14:** The chaincode Docker images are gone.  Do not take my word for it- prove it to yourself with these commands::

 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-sdk-node/test/fixtures$ docker images dev-*
 REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
 bcuser@ubuntu16042:~/git/src/github.com/hyperledger/fabric-sdk-node/test/fixtures$ docker images
 REPOSITORY                     TAG                                   IMAGE ID            CREATED             SIZE
 hyperledger/fabric-ca          latest              daaa35d81b43        20 minutes ago      255MB
 hyperledger/fabric-ca          s390x-1.0.1         daaa35d81b43        20 minutes ago      255MB
 hyperledger/fabric-tools       latest              5d841481c6b3        41 minutes ago      1.44GB
 hyperledger/fabric-tools       s390x-1.0.1         5d841481c6b3        41 minutes ago      1.44GB
 hyperledger/fabric-couchdb     latest              729eccd0e1d4        41 minutes ago      1.63GB
 hyperledger/fabric-couchdb     s390x-1.0.1         729eccd0e1d4        41 minutes ago      1.63GB
 hyperledger/fabric-kafka       latest              9d7161a4130f        43 minutes ago      1.4GB
 hyperledger/fabric-kafka       s390x-1.0.1         9d7161a4130f        43 minutes ago      1.4GB
 hyperledger/fabric-zookeeper   latest              8aeee7ddc8e5        43 minutes ago      1.41GB
 hyperledger/fabric-zookeeper   s390x-1.0.1         8aeee7ddc8e5        43 minutes ago      1.41GB
 hyperledger/fabric-testenv     latest              97f25587379c        43 minutes ago      1.5GB
 hyperledger/fabric-testenv     s390x-1.0.1         97f25587379c        43 minutes ago      1.5GB
 hyperledger/fabric-buildenv    latest              c3e57f5f5165        44 minutes ago      1.42GB
 hyperledger/fabric-buildenv    s390x-1.0.1         c3e57f5f5165        44 minutes ago      1.42GB
 hyperledger/fabric-orderer     latest              cc78bf4f171f        44 minutes ago      194MB
 hyperledger/fabric-orderer     s390x-1.0.1         cc78bf4f171f        44 minutes ago      194MB
 hyperledger/fabric-peer        latest              1081f30047d7        44 minutes ago      197MB
 hyperledger/fabric-peer        s390x-1.0.1         1081f30047d7        44 minutes ago      197MB
 hyperledger/fabric-javaenv     latest              a4d5b4ac736e        44 minutes ago      1.48GB
 hyperledger/fabric-javaenv     s390x-1.0.1         a4d5b4ac736e        44 minutes ago      1.48GB
 hyperledger/fabric-ccenv       latest              0109ec5a3d35        44 minutes ago      1.39GB
 hyperledger/fabric-ccenv       s390x-1.0.1         0109ec5a3d35        44 minutes ago      1.39GB
 hyperledger/fabric-baseimage   s390x-0.3.1         a165b6238eee        3 months ago        1.37GB
 hyperledger/fabric-baseos      s390x-0.3.1         2293f6d33733        3 months ago        171MB

In case you are wondering why you did not have to clean up the chaincode containers and images after you ran the CLI end-to-end test 
earlier in this lab, it is because this was done for you as part of the *./network_setup.sh down* command invocation.  Thereby 
depriving you of this wonderful opportunity to learn about these tricky Docker and Linux commands.  Shame on them!

**Recap:** In this section, you ran the Hyperledger Fabric Node.js SDK end-to-end test and then you cleaned up its leftover artifacts afterward.
This completes this lab.  You have downloaded and built a Hyperledger Fabric network and verified that the setup is correct by successfully running two end-to-end tests-  the CLI end-to-end test and the Node.js SDK end-to-end test.
If you really wanted to dig into the details of how the Hyperledger Fabric works, you could do worse than to drill down into the details of each of these tests.  

*** End of Lab! ***
