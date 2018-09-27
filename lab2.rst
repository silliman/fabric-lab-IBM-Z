Section 1 - Overview of Hyperledger Fabric Smart Contract installation lab
==========================================================================
In this lab, you will work with "Marbles", a sample application provided by the Hyperledger Fabric project.
"Marbles" is the de facto "Hello World" application of Hyperledger Fabric.

You will perform the activities in this lab on an Ubuntu 16.04.5 KVM guest running on an IBM z13 server at the IBM Washington Systems Center in Herndon, Virginia, USA.

You will use Docker Compose to bring up a Fabric network in which two organizations will participate.
There will be one orderer service for the entire network, and each organization will use its own certificate authority service and have two peer nodes.
Each peer node will use CouchDB for its ledger's world state. 
Each peer node's CouchDB server will run in a separate Docker container.  
That makes eleven Docker containers, as follows:

*	1 orderer service Docker container
*	2 certificate authority (CA) Docker containers (one for each organization)
*	4 peer node Docker containers  (each of the two organizations has two peers)
*	4 CouchDB Docker containers (each Peer node has its own separate CouchDB ledger store)

You will also bring up a twelfth Docker container that we will call the *CLI* container.  
You will use it as a convenience to enter Hyperledger Fabric commands targeted to specific peers.  
You will see how this is done later in the lab.

The network you bring up will use Transport Layer Security (TLS) which provides secure, encrypted communication between the peer nodes and the orderer, just as most real-world implementations will require.

You will **install** the Marbles chaincode on the peer nodes, **instantiate** the chaincode, and **invoke** functions of the chaincode.  
I will explain later in the lab the difference between the install and instantiate actions and what each one does.

When you invoke functions of the chaincode, some of them will produce transactions on the ledger and some of them will not.   
*Spoiler alert*:  Functions that create, update or delete ledger data always produce a transaction, while functions that only query ledger data do not.  
 
Section 2	- Description of the subsequent sections in this lab
==============================================================
This section provides a brief description of the subsequent sections in the lab, where you will get hands-on experience with the Hyperledger Fabric command line interface.

1.	You will extract the artifacts necessary to run the lab in Section 3.  All the artifacts necessary for the lab are provided in a zip file.  
2.	You will use Docker Compose to bring up the twelve Docker containers that comprise the Hyperledger Fabric network in Section 4.  You will see that all twelve Docker containers mentioned in Section 1 are brought up with a single _docker-compose_ command, and I will explain some of the more interesting bits of what is going on under the covers.
3.	You will create a channel in the Hyperledger Fabric network in Section 5.  In Hyperledger Fabric, each channel is essentially its own ledger.  
4.	You will instruct each peer node to join the channel in Section 6.  We will join all four peer nodes to the channel.  Peer nodes can be members of more than one channel, but for our lab we are only creating one channel.
5.	You will define an “anchor” peer for each organization in the channel in Section 7.  An anchor peer for an organization is a peer that is known by all the other organizations in a channel.  
6.	You will install the chaincode on the peer nodes in Section 8. Installing chaincode simply puts the chaincode executable on the file system of the peer.  It is a necessary step before you execute that chaincode on the peer, but the next step is also required.
7.	You will instantiate the chaincode on the channel in Section 9.  This step is a prerequisite to being able to run chaincode on a channel.  It only needs to be performed on one peer that is a member of the channel.  This causes a transaction to be recorded on the channel’s ledger to indicate that the chaincode can be run on the channel.
8.	You will invoke functions on the chaincode that will create, read, update and delete (CRUD) data stored on the blockchain in Section 10. If you hear programmers use the word CRUD, unless they are talking about last night’s baseball game, they are probably talking about Creating, Reading, Updating, or Deleting data.   Blocks of transactions in a blockchain are always added (i.e., Created), and they can be Read, but they should never, ever, ever, in normal operations, be Updated or Deleted.   However, although the blocks in a blockchain are not updated or deleted, the transactions themselves operate on Key/Value pairs that can have all CRUD operations performed on them.  This collection of Key/Value pairs is often referred to as state data, or *world state*. 


 
Section 3 -	Extract the artifacts necessary to run the lab
==========================================================

**Step 3.1:**	Find the PuTTY icon on the desktop of your lab laptop.  Double-click it and you should see a session named *Blockchain*.  Double-click on this session name and you will be prompted to enter a userid. Enter the name **bcuser**.  If prompted for a password, enter the password provided by the instructor.  At this point you should be greeted with a message that looks like this, and you will be placed in your home directory with a command prompt::

 Welcome to Ubuntu 16.04.5 LTS (GNU/Linux 4.4.0-135-generic s390x)

  * Documentation:  https://help.ubuntu.com
  * Management:     https://landscape.canonical.com
  * Support:        https://ubuntu.com/advantage
 Last login: Sun Sep 23 11:47:48 2018 from 192.168.22.64
 bcuser@ubuntu16045:~$ 
 
**Step 3.2:** Retrieve the zmarbles compressed tarball prepared for this lab with the following command::

 bcuser@ubuntu16045:~$ wget https://raw.githubusercontent.com/silliman/ZHollywood2018/master/zmarbles.tar.gz
 --2018-09-24 15:33:39--  https://raw.githubusercontent.com/silliman/ZHollywood2018/master/zmarbles.tar.gz
 Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 151.101.200.133
 Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|151.101.200.133|:443... connected.
 HTTP request sent, awaiting response... 200 OK
 Length: 8884793 (8.5M) [application/octet-stream]
 Saving to: 'zmarbles.tar.gz'

 zmarbles.tar.gz                   100%[============================================================>]   8.47M  26.6MB/s    in 0.3s    

 2018-09-24 15:33:39 (26.6 MB/s) - 'zmarbles.tar.gz' saved [8884793/8884793]

**Step 3.3:**	List the *zmarbles* directory with this *ls* command::

 bcuser@ubuntu16045:~$ ls zmarbles     
 ls: cannot access 'zmarbles': No such file or directory
 
Don’t panic!  It wasn’t supposed to be there.  It will be after the next step.

**Step 3.4:**	Extract the *zmarbles.tar.gz* file which will create the missing directory (and lots of subdirectories).  
Enter the command below::

 bcuser@ubuntu16045:~$ tar -xzf zmarbles.tar.gz 
 
**Step 3.5:** List the *zmarbles* directory with this command::

 bcuser@ubuntu16045:~$ ls -l zmarbles
 total 64
 drwxr-xr-x  2 bcuser bcuser  4096 Sep 24 14:54 base
 drwxrwxr-x  2 bcuser bcuser  4096 Sep 24 15:01 bin
 drwxr-xr-x  2 bcuser bcuser  4096 Sep 24 14:53 channel-artifacts
 drwxrwxr-x  2 bcuser bcuser  4096 Jul  3 15:06 config
 -rw-r--r--  1 bcuser bcuser 12209 Jul 30 16:15 configtx.yaml
 -rw-r--r--  1 bcuser bcuser  4175 Jul 30 17:32 crypto-config.yaml
 -rw-r--r--  1 bcuser bcuser  6281 Sep 24 14:57 docker-compose-template.yaml
 drwxr-xr-x  3 bcuser bcuser  4096 Jun 18  2017 examples
 -rwxr-xr-x  1 bcuser bcuser  3587 Sep 24 13:53 generateArtifacts.sh
 drwxr-xr-x  2 bcuser bcuser  4096 Oct  1  2017 hostScripts
 drwxr-xr-x 12 bcuser bcuser  4096 Jul 31 11:54 marblesUI
 drwxr-xr-x  2 bcuser bcuser  4096 Sep  6  2017 scripts

An explanation of the purpose of each of these files and directories is given here:

The *base* directory contains Docker Compose files that are included in the *docker-compose-template.yaml* file with the *extends* directive.

The *bin* directory contains Hyperledger Fabric utility programs that will be used by the *generateArtifacts.sh* script to generate cryptographic material (*cryptogen*) and channel configuration transactions (*configtxgen).

The *channel-artifacts* directory is empty, but it must exist when the *generateArtifacts.sh* script, which you will run later, invokes the *configtxgen* utility which generates channel configuration transaction inputs.

The *configtx.yaml* file is input to the *configtxgen* utility

The *cryto-config.yaml* file is input to the *cryptogen* utiity, which is called by the *generateArtifacts.sh* script to create cryptographic material (in the form of X.509 certificates and public and private key pairs) used to identify peers, orderers, and administrative and regular users of a Hyperledger Fabric network.

The *docker-compose-template.yaml* file is used as a template file that the *generateArtifacts.sh* script will use to create the main Docker Compose template file, *docker-compose.yaml* that contains definitions for all of the Docker containers that you will need.

The *examples* directory contains the actual Marbles chaincode within its subdirectory structure.

The *generateArtifacts.sh* script is used to generate channel configuration transaction input and to generate cryptographic material and it also creates *docker-compose.yaml*, using *docker-compose-template.yaml* as input.

The *hostScripts* directory is not used in this lab.

The *marblesUI* directory is used in the next lab, in which you will be working with the web UI for Marbles.

The *scripts* directory contains a script named *setpeer* that you will be using throughout this lab from within the *cli* Docker container. This will be explained further in *Section 5*.
 
Section 4	- Bring up the twelve Docker containers that comprise the Hyperledger Fabric network
==============================================================================================

**Step 4.1:**	Change to the *zmarbles* directory with the *cd* command::

 bcuser@ubuntu16045:~$ cd zmarbles/ 
 bcuser@ubuntu16045:~/zmarbles$
 
**Step 4.2:**	You are going to run a script named *generateArtifacts.sh* that will create some configuration information that is necessary to get your Hyperledger Fabric network set up.  
There is one optional parameter you may pass to the script, and that is the name of the channel you will be creating.  
If you do not specify this parameter, the channel name defaults to *mychannel*. 
You may choose to specify your own channel name.  
E.g., if you wished to name your channel *tim*, then you would enter *source ./generateArtifacts.sh tim* instead of just *source ./generateArtifacts.sh* when directed below to enter the command.

**IMPORTANT: If you pick your own channel name, it must start with a lowercase character, and only contain lowercase characters, numbers, or the dash ('-') character.**

So, enter the command below, optionally specifying a custom channel name (not shown here) as the lone argument to the *generateArtifacts.sh* script::

 bcuser@ubuntu16045:~/zmarbles$ source ./generateArtifacts.sh    # specify a custom channel name or accept the default value of 'mychannel' 
 
 Using cryptogen -> /home/bcuser/zmarbles/bin/cryptogen

 ##########################################################
 ##### Generate certificates using cryptogen tool #########
 ##########################################################
 unitedmarbles.com
 marblesinc.com

 Using configtxgen -> /home/bcuser/zmarbles/bin/configtxgen
 ##########################################################
 #########  Generating Orderer Genesis block ##############
 ##########################################################
 2018-09-24 13:54:10.168 EDT [common/tools/configtxgen] main -> WARN 001 Omitting the channel ID for configtxgen is deprecated.  Explicitly passing the channel ID will be required in the future, defaulting to 'testchainid'.
 2018-09-24 13:54:10.168 EDT [common/tools/configtxgen] main -> INFO 002 Loading configuration
 2018-09-24 13:54:10.195 EDT [msp] getMspConfig -> INFO 003 Loading NodeOUs
 2018-09-24 13:54:10.196 EDT [msp] getMspConfig -> INFO 004 Loading NodeOUs
 2018-09-24 13:54:10.196 EDT [common/tools/configtxgen] doOutputBlock -> INFO 005 Generating genesis block
 2018-09-24 13:54:10.196 EDT [common/tools/configtxgen] doOutputBlock -> INFO 006 Writing genesis block

 #################################################################
 ### Generating channel configuration transaction 'channel.tx' ###
 #################################################################
 2018-09-24 13:54:10.270 EDT [common/tools/configtxgen] main -> INFO 001 Loading configuration
 2018-09-24 13:54:10.294 EDT [common/tools/configtxgen] doOutputChannelCreateTx -> INFO 002 Generating new channel configtx
 2018-09-24 13:54:10.295 EDT [msp] getMspConfig -> INFO 003 Loading NodeOUs
 2018-09-24 13:54:10.296 EDT [msp] getMspConfig -> INFO 004 Loading NodeOUs
 2018-09-24 13:54:10.297 EDT [common/tools/configtxgen] doOutputChannelCreateTx -> INFO 005 Writing new channel tx

 #################################################################
 #######    Generating anchor peer update for Org0MSP   ##########
 #################################################################
 2018-09-24 13:54:10.371 EDT [common/tools/configtxgen] main -> INFO 001 Loading configuration
 2018-09-24 13:54:10.396 EDT [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 002 Generating anchor peer update
 2018-09-24 13:54:10.396 EDT [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 003 Writing anchor peer update

 #################################################################
 #######    Generating anchor peer update for Org1MSP   ##########
 #################################################################
 2018-09-24 13:54:10.468 EDT [common/tools/configtxgen] main -> INFO 001 Loading configuration
 2018-09-24 13:54:10.493 EDT [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 002 Generating anchor peer update
 2018-09-24 13:54:10.493 EDT [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 003 Writing anchor peer update


By the way, if you enter a command and end it with #, everything after the # is considered a comment and is ignored by the shell.  
So, if you see me place comments after any commands you do not have to enter them but if you do, it will not hurt anything.  

This script calls two Hyperledger Fabric utilites- *cryptogen*, which creates security material (certificates and keys) 
and *configtxgen* (Configuration Transaction Generator), which is called four times, to create four things:

1.	An **orderer genesis block** – this will be the first block on the orderer’s system channel. The location of this block is specified to the Orderer when it is started up via the ORDERER_GENERAL_GENESISFILE environment variable.

2.	A **channel transaction** – later in the lab, this is sent to the orderer and will cause a new channel to be created when you run the **peer channel create** command.

3.	An **anchor peer update** for Org0MSP.  An anchor peer is a peer that is set up so that peers from other organizations may communicate with it.  The concept of anchor peers allows an organization to create multiple peers, perhaps to provide extra capacity or throughput or resilience (or all the above) but not have to advertise this to outside organizations.

4.	An anchor peer update for Org1MSP.   You will perform the anchor peer updates for both Org0MSP and Org1MSP later in the lab via **peer channel create** commands.

**Step 4.3:**	Issue the following command which will show you all files that were created by the *configtxgen* utility when it was called from inside *generateArtifacts.sh*::

 bcuser@ubuntu16045:~/zmarbles$ ls -ltr channel-artifacts
 total 28
 -rw-r--r-- 1 bcuser bcuser 12787 Sep 24 13:54 genesis.block
 -rw-r--r-- 1 bcuser bcuser   346 Sep 24 13:54 channel.tx
 -rw-r--r-- 1 bcuser bcuser   285 Sep 24 13:54 Org0MSPanchors.tx
 -rw-r--r-- 1 bcuser bcuser   282 Sep 24 13:54 Org1MSPanchors.tx

*genesis.block* will be passed to the *orderer* at startup, and will be used to configure the orderer's *system channel*.  This file contains the x.509 signing certificates for every organization defined within the consortia that were specified within the *configtx.yaml* file when *configtxgen* was run.  The *system channel* contains other values such as parameters defining when a block of transactions is cut- e.g., based on time, number of transactions, or block size- and these values serve as a template, that is, as defaults, for any additional channels that might be created, if a new channel creation request does not provide its own custom values.

*channel.tx* is the input for a configuration transaction that will create a channel.  You will use this as input to a *peer channel create* request in *Section 5*.

*Org0MSPanchors.tx* and *Org1MSPanchors.tx* are inputs for configuration transactions that will define an anchor peer for *Org0* and *Org1* respectively.  You will use these inputs in *Section 7*.

**Step 4.4:** Issue the following command which will show you all files that were created by the *cryptogen* utility when it was called from inside *generateArtifacts.sh*.  This command will show one screen at a time and pause-  press the *Enter* key to scroll to the end, that is, until you get your command prompt back::

 bcuser@ubuntu16045:~/zmarbles$ ls -ltrR crypto-config | more
   .
   .  (output not shown here)
   .
 
Actually, these files were created *before* the files listed in the prior step, *Step 4.3*, were created, because, among the many cryptographic artifacts created are the x.509 signing certificates for the organizations, which are baked into the *genesis.block* discussed in the prior step.

You can see that there is a dizzying set of directories and files, containing things like CA root certificates, signing certificates, TLS certificates, corresponding private keys, and public keys, for certificate authorities, organizations, administrative and general users.  A thorough discussion of them is beyond the scope of this lab, but at some point in a glorious future the author hopes to document, perhaps in an appendix somewhere, the purpose of each file. The author wants world peace, too.  Shall we proceed?


**Step 4.5:**	You are going to look inside the Docker Compose configuration file a little bit.   Enter the following command::

 bcuser@ubuntu16045:~/zmarbles$ vi -R docker-compose.yaml

You can enter ``Ctrl-f`` to scroll forward in the file and ``Ctrl-b`` to scroll back in the file.  The *-R* flag opens the file in 
read-only mode, so if you accidentally change something in the file, it’s okay.  It will not be saved.

The statements within *docker-compose.yaml* are in a markup language called *YAML*, which stands for *Y*\ et *A*\ nother *M*\ arkup *L*\ anguage.  
(Who says nerds do not have a sense of humor).  
We will go over some highlights here.

There are twelve “services”, or Docker containers, defined within this file.  
They all start in column 3 and have several statements to describe them.  
For example, the first service defined is **ca0**, and there are *image*, *environment*, *ports*, *command*, *volumes*, and 
*container_name* statements that describe it.  
If you scroll down in the file with ``Ctrl-f`` you will see all the services.  
Not every service has the same statements describing it.

The twelve services are:

**ca0** – The certificate authority service for “Organization 0” (unitedmarbles.com)

**ca1** – The certificate authority service for “Organization 1” (marblesinc.com)

**orderer.blockchain.com** – The single ordering service that both organizations will use

**peer0.unitedmarbles.com** – The first peer node for “Organization 0”	

**peer1.unitedmarbles.com** – The second peer node for “Organization 0”	

**peer0.marblesinc.com** – The first peer node for “Organization 1”	

**peer1.marblesinc.com** – The second peer node for “Organization 1”	

**couchdb0** – The CouchDB server for peer0.unitedmarbles.com  

**couchdb1** – The CouchDB server for peer1.unitedmarbles.com  

**couchdb2** – The CouchDB server for peer0.marblesinc.com

**couchdb3** – The CouchDB server for peer1.marblesinc.com

**cli** – The Docker container from which you will enter Hyperledger Fabric command line interface (CLI) commands targeted 
towards a peer node.

I will describe how several statements work within the file, but time does not permit me to address every single line in the file!

*image* statements define which Docker image file the Docker container will be created from.  Basically, the Docker image file is a 
static file that, once created, is read-only.  A Docker container is based on a Docker image, and any changes to the file system 
within a Docker container are stored within the container.  So, multiple Docker containers can be based on the same Docker image, 
and each Docker container keeps track of its own changes.  For example, the containers built for the **ca0** and **ca1** service will 
be based on the *hyperledger/fabric-ca:latest* Docker image because they both have this statement in their definition::

        image: hyperledger/fabric-ca    

*environment* statements define environment variables that are available to the Docker container.  The Hyperledger Fabric processes 
make ample use of environment variables.  In general, you will see that the certificate authority environment variables start with 
*FABRIC_CA*, the orderer’s environment variables start with *ORDERER_GENERAL*, and the peer node’s environment variables start with 
*CORE*.  These variables control behavior of the Hyperledger Fabric code, and in many cases, will override values that are specified 
in configuration files. Notice that all the peers and the orderer have an environment variable to specify that TLS is 
enabled-   *CORE_PEER_TLS_ENABLED=true* for the peers and *ORDERER_GENERAL_TLS_ENABLED=true* for the orderer.  You will notice there 
are other TLS-related variables to specify private keys, certificates and root certificates.

*ports* statements map ports on our Linux on IBM Z host to ports within the Docker container.  The syntax is *<host port>:<Docker 
container port>*.  For example, the service for **ca1** has this port statement::
 
     ports:
       - "8054:7054"

This says that port 7054 in the Docker container for the **ca1** node will be mapped to port 8054 on your Linux on IBM Z host.   This 
is how you can run two CA nodes in two Docker containers and four peer nodes in four Docker containers and keep things straight-  
within each CA node they are both using port 7054, and within each peer node Docker container, they are all using port 7051 for the 
same thing, but if you want to get to one of the peers from your host or even the outside world, you would target the appropriate 
host-mapped port. **Note:** To see the port mappings for the peers you have to look in *base/docker-compose.yaml*.  See if you can 
figure out why.

*container_name* statements are used to create hostnames that the Docker containers spun up by the docker-compose command use to 
communicate with each other.  A separate, private network will be created by Docker where the 12 Docker containers can communicate 
with each other via the names specified by *container_name*.  So, they do not need to worry about the port mappings from the *ports* 
statements-  those are used for trying to get to the Docker containers from outside the private network created by Docker.

*volumes* statements are used to map file systems on the host to file systems within the Docker container.  Just like with ports, the 
file system on the host system is on the left and the file system name mapped within the Docker container is on the right. For 
example, look at this statement from the **ca0** service::
 
     volumes:
       - ./crypto-config/peerOrganizations/unitedmarbles.com/ca/:/etc/hyperledger/fabric-ca-server-config

The security-related files that were created from the previous step where you ran *generateArtifacts.sh* were all within 
the *crypto-config* directory on your Linux on IBM Z host.  The prior *volumes* statement is how this stuff is made accessible to the 
**ca1** service that will run within the Docker container.   Similar magic is done for the other services as well, except for 
the CouchDB services.

*extends* statements are used by the peer nodes.  What this does is merge in other statements from another file.  For example, you 
may notice that the peer nodes do not contain an images statement.  How does Docker know what Docker image file to base the 
container on?  That is defined in the file, *base/peer-base.yaml*, specified in the *extends* section of *base/docker-compose.yaml*, 
which is specified in the *extends* section of *docker-compose.yaml* for the peer nodes.

*command* statements define what command is run when the Docker container is started.  
This is how the actual Hyperledger Fabric processes get started.  
You can define default commands when you create the Docker image.  
This is why you do not see *command* statements for the **cli** service or for the CouchDB services.   
For the peer nodes, the command statement is specified in the *base/peer-base.yaml* file.

*working_dir* statements define what directory the Docker container will be in when its startup commands are run.  
Again, defaults for this can be defined when the Docker image is created. 

When you are done reviewing the *docker-compose.yaml* file, exit the *vi* session by typing ``:q!``  (that’s “colon”, “q”, 
“exclamation point”) which will exit the file and discard any changes you may have accidentally made while browsing through the file.  
If ``:q!`` doesn’t work right away, you may have to hit the escape key first before trying it.  
If that still doesn’t work, ask an instructor for help-  *vi* can be tricky if you are not used to it.

If you would like to see what is in the *base/docker-compose-base.yaml* and *base/peer-base.yaml* files I mentioned, take a quick peek with ``vi -R base/docker-compose-base.yaml`` and ``vi -R base/peer-base.yaml`` and exit with the ``:q!`` key sequence when you have had enough.

**Step 4.6:**	Start the Hyperledger Fabric network by entering the command shown below::

 bcuser@ubuntu16045:~/zmarbles$ docker-compose up -d
 Creating network "zmarbles_default" with the default driver
    .
    .  You will see the required Docker images being pulled from the Hyperledger project's
    .  public Docker Hub repository for a few minutes before you see the following messages
    .
 Creating couchdb2                ... done
 Creating orderer.blockchain.com ... done
 Creating ca_Org1                ... done
 Creating couchdb0               ... done
 Creating ca_Org0                ... done
 Creating couchdb1                ... done
 Creating couchdb3                ... done
 Creating peer0.unitedmarbles.com ... done
 Creating peer1.marblesinc.com    ... done
 Creating peer0.marblesinc.com    ... done
 Creating peer1.unitedmarbles.com ... done
 Creating cli                     ... done

**Step 4.7:**	Verify that all twelve services are *Up* and none of them say *Exited*.  The *Exited* status means something went 
wrong, and you should check with an instructor for help if you see any of them in *Exited* status.

If, however, all twelve of your Docker containers are in *Up* status, as in the output below, you are ready to proceed to the next 
section::

 bcuser@ubuntu16045:~/zmarbles$ docker ps -a
 CONTAINER ID        IMAGE                               COMMAND                  CREATED             STATUS              PORTS                                                                       NAMES
 a37940d016a7        hyperledger/fabric-tools:1.2.0      "bash"                   About a minute ago   Up About a minute                                                                               cli
 a9045b1bae14        hyperledger/fabric-peer:1.2.0       "peer node start"        About a minute ago   Up About a minute   0.0.0.0:8051->7051/tcp, 0.0.0.0:8052->7052/tcp, 0.0.0.0:8053->7053/tcp      peer1.unitedmarbles.com
 f5ced05284a3        hyperledger/fabric-peer:1.2.0       "peer node start"        About a minute ago   Up About a minute   0.0.0.0:7051-7053->7051-7053/tcp                                            peer0.unitedmarbles.com
 1273c9653f24        hyperledger/fabric-peer:1.2.0       "peer node start"        About a minute ago   Up About a minute   0.0.0.0:10051->7051/tcp, 0.0.0.0:10052->7052/tcp, 0.0.0.0:10053->7053/tcp   peer1.marblesinc.com
 98c44b366a92        hyperledger/fabric-peer:1.2.0       "peer node start"        About a minute ago   Up About a minute   0.0.0.0:9051->7051/tcp, 0.0.0.0:9052->7052/tcp, 0.0.0.0:9053->7053/tcp      peer0.marblesinc.com
 c9965303275e        hyperledger/fabric-couchdb:0.4.10   "tini -- /docker-ent…"   About a minute ago   Up About a minute   4369/tcp, 9100/tcp, 0.0.0.0:6984->5984/tcp                                  couchdb1
 7db99161595b        hyperledger/fabric-ca:1.2.0         "sh -c 'fabric-ca-se…"   About a minute ago   Up About a minute   0.0.0.0:7054->7054/tcp                                                      ca_Org0
 9ef9b33a9490        hyperledger/fabric-orderer:1.2.0    "orderer"                About a minute ago   Up About a minute   0.0.0.0:7050->7050/tcp                                                      orderer.blockchain.com
 6ce17ab97f19        hyperledger/fabric-couchdb:0.4.10   "tini -- /docker-ent…"   About a minute ago   Up About a minute   4369/tcp, 9100/tcp, 0.0.0.0:5984->5984/tcp                                  couchdb0
 5c9909a6b9b5        hyperledger/fabric-couchdb:0.4.10   "tini -- /docker-ent…"   About a minute ago   Up About a minute   4369/tcp, 9100/tcp, 0.0.0.0:8984->5984/tcp                                  couchdb3
 f667587a207c        hyperledger/fabric-couchdb:0.4.10   "tini -- /docker-ent…"   About a minute ago   Up About a minute   4369/tcp, 9100/tcp, 0.0.0.0:7984->5984/tcp                                  couchdb2
 450ac4b68d59        hyperledger/fabric-ca:1.2.0         "sh -c 'fabric-ca-se…"   About a minute ago   Up About a minute   0.0.0.0:8054->7054/tcp                                                      ca_Org1
 bcuser@ubuntu16045:~/zmarbles$ 

Section 5	- Create a channel in the Hyperledger Fabric network
==============================================================
In a Hyperledger Fabric v1.2.0 network, multiple channels can be created.  
Each channel can have its own policies for things such as requirements for endorsement and what organizations may join the channel.  
This allows for a subset of network participants to participate in their own channel.  

Imagine a scenario where OrgA, OrgB and OrgC are three organizations participating in the network. 
You could set up a channel in which all three organizations participate.   
You could also set up a channel where only OrgA and OrgB participate.   
In this case, the peers in OrgC would not see the transactions occurring in that channel.    
OrgA could participate in another channel with only OrgC, in which case OrgB does not have visibility.  
And so on.  

You could create channels with the same participants, but have different policies.  
For example, perhaps one channel with OrgA, OrgB, and OrgC could require all three organizations to endorse a transaction proposal, but another channel with OrgA, OrgB and OrgC could require just two, or even just one, of the three organizations to endorse a transaction proposal.

The decision on how many channels to create and what policies they have will usually be driven by the requirements of the particular business problem being solved.

**Step 5.1:**	Access the *cli* Docker container::

 bcuser@ubuntu16045:~/zmarbles$ docker exec -it cli bash
 root@a37940d016a7:/opt/gopath/src/github.com/hyperledger/fabric/peer#ic/peer#

Observe that your command prompt changes when you enter the Docker container’s shell.

The *docker exec* command runs a command against an existing Docker container.  
The *-it* flags basically work together to say, “we want an interactive terminal session with this Docker container”.  
*cli* is the name of the Docker container (this came from the *container_name* statement in the *docker-compose.yaml* file for the *cli* service).  
*bash* is the name of the command you want to enter.   
In other words, you are entering a Bash shell within the *cli* Docker container.  
For most of the rest of the lab, you will be entering commands within this Bash shell.

Instead of working as user *bcuser* on the ubuntu16045 server in the *~/zmarbles* directory, you are now inside the Docker container with ID *f7bd77dd7124* (your ID will differ), working in the */opt/gopath/src/github.com/hyperledger/fabric/peer* directory.  
It is no coincidence that that directory is the value of the *working_dir* statement for the *cli* service in your *docker-compose.yaml* file.

**Step 5.2:** Read on to learn about a convenience script to point to a particular peer from the *cli* Docker container. 
A convenience script named *setpeer* is provided within the *cli* container that is in the *scripts* subdirectory of your current working directory. 
This script will set the environment variables to the values necessary to point to a particular peer.   
The script takes two arguments.  
This first argument is either 0 or 1 for Organization 0 or Organization 1 respectively, and the second argument is for 
either Peer 0 or Peer 1 of the organization selected by the first argument.   
Therefore, throughout the remainder of this lab, before sending commands to a peer, you will enter one of the following four valid combinations from within the *cli* Docker container, depending on which peer you want to run the command on:

*source scripts/setpeer 0 0*   # to target Org 0, peer 0  (peer0.unitedmarbles.com)

*source scripts/setpeer 0 1*   # to target Org 0, peer 1  (peer1.united marbles.com)

*source scripts/setpeer 1 0*   # to target Org 1, peer 0  (peer0.marblesinc.com)

*source scripts/setpeer 1 1*   # to target Org 1, peer 1  (peer1.marblesinc.com)

**Step 5.3:** Choose your favorite peer and use one of the four *source scripts/setpeer* commands listed in the prior step.   Although you are going to join all four peers to our channel, you only need to issue the channel creation command once.  
You can issue it from any of the four peers, so pick your favorite peer and issue the source command.  
In this screen snippet, I have chosen Org 1, peer 1.  
Issue the command below, leaving the arguments '1 1' as is, or change it to one of the other valid combinations as described in the previous step::

 root@a37940d016a7:/opt/gopath/src/github.com/hyperledger/fabric/peer# source scripts/setpeer 1 1
 CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/marblesinc.com/peers/peer1.marblesinc.com/tls/ca.crt
 CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/peers/peer0.unitedmarbles.com/tls/server.key
 CORE_PEER_LOCALMSPID=Org1MSP
 CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
 CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/peers/peer0.unitedmarbles.com/tls/server.crt
 CORE_PEER_TLS_ENABLED=true
 CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/marblesinc.com/users/Admin@marblesinc.com/msp
 CORE_PEER_ID=cli
 CORE_LOGGING_LEVEL=DEBUG
 CORE_PEER_ADDRESS=peer1.marblesinc.com:7051 
 root@a37940d016a7:/opt/gopath/src/github.com/hyperledger/fabric/peer#

The last environment variable listed, *CORE_PEER_ADDRESS*, determines to which peer your commands will be routed.  

**Step 5.4:**	The Hyperledger Fabric network is configured to require TLS, so when you enter your peer commands, you need to add a flag that indicates TLS is enabled, and you need to add an argument that points to the root signer certificate of the certificate authority for the orderer service.

Fortunately, an environment variable has been set for you within the CLI container that sets the flag (*--tls* argument) and points to the appropriate certificate (the *--cafile* argument) so that you can simply pass both arguments by specifying the single short environment variable name instead of having to enter the two arguments and the tediously long argument value for *--cafile*.

Enter this command now to see the value of this environment variable, and thank me later for setting this up for you::

 root@a37940d016a7:/opt/gopath/src/github.com/hyperledger/fabric/peer# echo $FABRIC_TLS 
 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/blockchain.com/orderers/orderer.blockchain.com/msp/cacerts/ca.blockchain.com-cert.pem

**Step 5.5:** Now enter this command::

 root@a37940d016a7:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer channel create -o orderer.blockchain.com:7050  -f channel-artifacts/channel.tx  $FABRIC_TLS -c $CHANNEL_NAME
 2018-09-24 19:26:28.221 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
 2018-09-24 19:26:28.250 UTC [cli/common] readBlock -> INFO 002 Got status: &{NOT_FOUND}
 2018-09-24 19:26:28.254 UTC [channelCmd] InitCmdFactory -> INFO 003 Endorser and orderer connections initialized
 2018-09-24 19:26:28.455 UTC [cli/common] readBlock -> INFO 004 Received block: 0

The last line before you get your command prompt back will contain the words "Received block: 0".
This indicates that your channel creation was successful, and the peer received the initial, or *genesis* block for the channel, with is block 0.

Proceed to the next section where you will join each peer to the channel.
 
Section 6	- Instruct each peer node to join the channel
=======================================================

In the last section, you issued the *peer channel create* command from one of the peers.   
Now any peer that you want to join the channel may join- you will issue the *peer channel join* command from each peer.

For a peer to be eligible to join a channel, it must be a member of an organization that is authorized to join the channel.  When you created your channel, you authorized *Org0MSP* and *Org1MSP* to join the channel.  
Each of your four peers belongs to one of those two organizations- two peers for each one- so they will be able to join successfully.   
If someone from an organization other than *Org0MSP* or *Org1MSP* attempted to join their peers to this channel, the attempt would fail.

You are going to repeat the following steps for each of the four peer nodes, in order to show that the peer successfully joined the channel:

1.	Use the *scripts/setpeer* script to point the CLI to the peer

2.	Use the *peer channel list* command to show that the peer is not joined to any channels

3.	Use the *peer channel join* command to join the peer to your channel

4.	Use the *peer channel list* command again to see that the peer has joined your channel

**Step 6.1:**	Point the *cli* to *peer0* for *Org0MSP*::

 root@a37940d016a7:/opt/gopath/src/github.com/hyperledger/fabric/peer# source scripts/setpeer 0 0
 CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/peers/peer0.unitedmarbles.com/tls/ca.crt
 CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/peers/peer0.unitedmarbles.com/tls/server.key
 CORE_PEER_LOCALMSPID=Org0MSP
 CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
 CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/peers/peer0.unitedmarbles.com/tls/server.crt
 CORE_PEER_TLS_ENABLED=true
 CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/users/Admin@unitedmarbles.com/msp
 CORE_PEER_ID=cli
 CORE_LOGGING_LEVEL=DEBUG
 CORE_PEER_ADDRESS=peer0.unitedmarbles.com:7051

**Step 6.2:** Enter *peer channel list* and observe that no channels are returned at the end of the output::

 root@a37940d016a7:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer channel list
 2018-09-24 19:29:36.558 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
 Channels peers has joined: 
 
You can observe that nothing is returned after the 'Channels peers has joined:' string-  this indicates that this peer has yet to join any channels. 
(You'll observe this same behavior on the other three peers as well so I will not repeat this commentary when you work with the other three peers).

**Step 6.3:** Issue *peer channel join -b $CHANNEL_NAME.block* to join the channel you set up when you ran *generateArtifacts.sh* a little while ago.  
Among the many things that script did, it exported an environment variable named $CHANNEL_NAME set to the channel name you specified (or *mychannel* if you did not specify your own name), and then the Docker Compose file for is set up to pass this environment variable to the *cli* container.  
If you are still on the happy path, the end of your output will look similar to this::

 root@a37940d016a7:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer channel join -b $CHANNEL_NAME.block 
 2018-09-24 19:30:28.143 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
 2018-09-24 19:30:28.209 UTC [channelCmd] executeJoin -> INFO 002 Successfully submitted proposal to join channel
 root@a37940d016a7:/opt/gopath/src/github.com/hyperledger/fabric/peer# 

**Step 6.4:**	Repeat the *peer channel list* command and now you should see your channel listed in the output::

 root@a37940d016a7:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer channel list
 2018-09-24 19:31:04.551 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
 Channels peers has joined: 
 mychannel

**Step 6.5:**	Point the *cli* to *peer1* for *Org0MSP*::

 root@a37940d016a7:/opt/gopath/src/github.com/hyperledger/fabric/peer# source scripts/setpeer 0 1
 CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/peers/peer1.unitedmarbles.com/tls/ca.crt
 CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/peers/peer0.unitedmarbles.com/tls/server.key
 CORE_PEER_LOCALMSPID=Org0MSP
 CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
 CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/peers/peer0.unitedmarbles.com/tls/server.crt
 CORE_PEER_TLS_ENABLED=true
 CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/users/Admin@unitedmarbles.com/msp
 CORE_PEER_ID=cli
 CORE_LOGGING_LEVEL=DEBUG
 CORE_PEER_ADDRESS=peer1.unitedmarbles.com:7051

**Step 6.6:** Enter *peer channel list* and observe that no channels are returned at the end of the output::

 root@a37940d016a7:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer channel list
 2018-09-24 19:32:31.342 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
 Channels peers has joined:  
 
**Step 6.7:**	Issue *peer channel join -b $CHANNEL_NAME.block* to join your channel. Your output should look 
similar to this::

 root@a37940d016a7:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer channel join -b $CHANNEL_NAME.block 
 2018-09-24 19:32:56.763 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
 2018-09-24 19:32:56.829 UTC [channelCmd] executeJoin -> INFO 002 Successfully submitted proposal to join channel
 root@a37940d016a7:/opt/gopath/src/github.com/hyperledger/fabric/peer#

**Step 6,8:** Repeat the *peer channel list* command and now you should see your channel listed::

 root@a37940d016a7:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer channel list
 2018-09-24 19:33:20.330 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
 Channels peers has joined: 
 mychannel

**Step 6.9:**	Point the *cli* to *peer0* for *Org1MSP*::

 root@a37940d016a7:/opt/gopath/src/github.com/hyperledger/fabric/peer# source scripts/setpeer 1 0
 CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/marblesinc.com/peers/peer0.marblesinc.com/tls/ca.crt
 CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/peers/peer0.unitedmarbles.com/tls/server.key
 CORE_PEER_LOCALMSPID=Org1MSP
 CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
 CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/peers/peer0.unitedmarbles.com/tls/server.crt
 CORE_PEER_TLS_ENABLED=true
 CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/marblesinc.com/users/Admin@marblesinc.com/msp
 CORE_PEER_ID=cli
 CORE_LOGGING_LEVEL=DEBUG
 CORE_PEER_ADDRESS=peer0.marblesinc.com:7051

**Step 6.10:** Enter *peer channel list* and observe that no channels are returned at the end of the output::

 root@a37940d016a7:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer channel list
 2018-09-24 19:34:27.456 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
 Channels peers has joined: 

**Step 6.11:** Issue *peer channel join -b $CHANNEL_NAME.block* to join your channel. Your output should look 
similar to this::

 root@a37940d016a7:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer channel join -b $CHANNEL_NAME.block
 2018-09-24 19:34:55.073 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
 2018-09-24 19:34:55.151 UTC [channelCmd] executeJoin -> INFO 002 Successfully submitted proposal to join channel
 root@a37940d016a7:/opt/gopath/src/github.com/hyperledger/fabric/peer# 

**Step 6.12:** Repeat the *peer channel list* command and now you should see your channel listed in the output::

 root@a37940d016a7:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer channel list
 2018-09-24 19:35:43.635 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
 Channels peers has joined: 
 mychannel
 
**Step 6.13:**	Point the *cli* to *peer1* for *Org1MSP*::

 root@a37940d016a7:/opt/gopath/src/github.com/hyperledger/fabric/peer# source scripts/setpeer 1 1
 CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/marblesinc.com/peers/peer1.marblesinc.com/tls/ca.crt
 CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/peers/peer0.unitedmarbles.com/tls/server.key
 CORE_PEER_LOCALMSPID=Org1MSP
 CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
 CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/peers/peer0.unitedmarbles.com/tls/server.crt
 CORE_PEER_TLS_ENABLED=true
 CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/marblesinc.com/users/Admin@marblesinc.com/msp
 CORE_PEER_ID=cli
 CORE_LOGGING_LEVEL=DEBUG
 CORE_PEER_ADDRESS=peer1.marblesinc.com:7051

The output from this should be familiar to you by now so from now on I will not bother showing it anymore in the remainder of these 
lab instructions.

**Step 6.14:** Enter *peer channel list* and observe that no channels are returned at the end of the output::

 root@a37940d016a7:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer channel list
 2018-09-24 19:36:32.033 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
 Channels peers has joined: 

**Step 6.15:** Issue *peer channel join -b $CHANNEL_NAME.block* to join your channel. 
(Am I being redundant? Am I repeating myself? Am I saying the same thing over and over again?) 
Your output should look similar to this::

 root@a37940d016a7:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer channel join -b $CHANNEL_NAME.block 
 2018-09-24 19:36:50.984 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
 2018-09-24 19:36:51.060 UTC [channelCmd] executeJoin -> INFO 002 Successfully submitted proposal to join channel
 root@a37940d016a7:/opt/gopath/src/github.com/hyperledger/fabric/peer#

**Step 6.16:**	Repeat the *peer channel list* command and now you should see your channel listed in the output::

 root@a37940d016a7:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer channel list
 2018-09-24 19:37:27.546 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
 Channels peers has joined: 
 mychannel
 
Section 7	- Define an “anchor” peer for each organization in the channel
========================================================================
An anchor peer for an organization is a peer that can be known by all the other organizations in a channel.  
Not all peers for an organization need to be defined as anchor peers.  
Peers from other organizations will reach out to anchor peers which can then make information about the other peers available.

In a production environment, an organization will typically define more than one peer as an anchor peer for availability and 
resilience. 
In our lab, we will just define one of the two peers for each organization as an anchor peer.

The definition of an anchor peer took place back in section 4 when you ran the *generateArtifacts.sh* script.  
Two of the output files from that step were *Org0MSPanchors.tx* and *Org1MSPanchors.tx.*  
These are input files to define the anchor peers for Org0MSP and Org1MSP respectively.  
After the channel is created, each organization needs to run this command.  
You will do that now-  this process is a little bit confusing in that the command to do this starts with *peer channel create …* but the command will actually *update* the existing channel with the information about the desired anchor peer.  
Think of *peer channel create* here as meaning, “create an update transaction for a channel”.

**Step 7.1:** Switch to *peer0* for *Org0MSP*::

 root@a37940d016a7:/opt/gopath/src/github.com/hyperledger/fabric/peer# source scripts/setpeer 0 0   # to switch to Peer 0 for Org0MSP
 CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/peers/peer0.unitedmarbles.com/tls/ca.crt
 CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/peers/peer0.unitedmarbles.com/tls/server.key
 CORE_PEER_LOCALMSPID=Org0MSP
 CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
 CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/peers/peer0.unitedmarbles.com/tls/server.crt
 CORE_PEER_TLS_ENABLED=true
 CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/users/Admin@unitedmarbles.com/msp
 CORE_PEER_ID=cli
 CORE_LOGGING_LEVEL=DEBUG
 CORE_PEER_ADDRESS=peer0.unitedmarbles.com:7051

**Step 7.2:** Issue this command to create the anchor peer for *Org0MSP*::

 root@a37940d016a7:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer channel create -o orderer.blockchain.com:7050 -f channel-artifacts/Org0MSPanchors.tx $FABRIC_TLS -c $CHANNEL_NAME 
 2018-09-24 19:38:38.023 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
 2018-09-24 19:38:38.034 UTC [cli/common] readBlock -> INFO 002 Received block: 0

**Step 7.3:** Switch to *peer0* for *Org1MSP*::

 root@a37940d016a7:/opt/gopath/src/github.com/hyperledger/fabric/peer# source scripts/setpeer 1 0
 CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/marblesinc.com/peers/peer0.marblesinc.com/tls/ca.crt
 CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/peers/peer0.unitedmarbles.com/tls/server.key
 CORE_PEER_LOCALMSPID=Org1MSP
 CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
 CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/peers/peer0.unitedmarbles.com/tls/server.crt
 CORE_PEER_TLS_ENABLED=true
 CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/marblesinc.com/users/Admin@marblesinc.com/msp
 CORE_PEER_ID=cli
 CORE_LOGGING_LEVEL=DEBUG
 CORE_PEER_ADDRESS=peer0.marblesinc.com:7051
 
**Step 7.4:** Issue this command to create the anchor peer for *Org1MSP*::

 root@a37940d016a7:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer channel create -o orderer.blockchain.com:7050 -f channel-artifacts/Org1MSPanchors.tx $FABRIC_TLS -c $CHANNEL_NAME
 2018-09-24 19:39:45.938 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
 2018-09-24 19:39:45.952 UTC [cli/common] readBlock -> INFO 002 Received block: 0

Section 8	- Install the chaincode on the peer nodes
===================================================

Installing chaincode on the peer nodes puts the chaincode binary executable on a peer node. 
If you want the peer to be an endorser on a channel for a chaincode, you must install the chaincode on that peer.  
If you only want the peer to be a committer on a channel for a chaincode, then you do not have to install the chaincode on that peer.  
In this section, you will install the chaincode on two of your peers.

**Step 8.1:** Switch to *peer0* in *Org0MSP*::

 root@a37940d016a7::/opt/gopath/src/github.com/hyperledger/fabric/peer#  source scripts/setpeer 0 0
 CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/peers/peer0.unitedmarbles.com/tls/ca.crt
 CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/peers/peer0.unitedmarbles.com/tls/server.key
 CORE_PEER_LOCALMSPID=Org0MSP
 CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
 CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/peers/peer0.unitedmarbles.com/tls/server.crt
 CORE_PEER_TLS_ENABLED=true
 CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/users/Admin@unitedmarbles.com/msp
 CORE_PEER_ID=cli
 CORE_LOGGING_LEVEL=DEBUG
 CORE_PEER_ADDRESS=peer0.unitedmarbles.com:7051
 
**Step 8.2:**	Install the marbles chaincode on Peer0 in Org0MSP::

 root@a37940d016a7:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer chaincode install -n marbles -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/marbles 
 2018-09-24 19:41:58.475 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
 2018-09-24 19:41:58.475 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
 2018-09-24 19:41:58.685 UTC [chaincodeCmd] install -> INFO 003 Installed remotely response:<status:200 payload:"OK" > 

**Step 8.3:** Switch to *peer0* in *Org1MSP*::

 root@a37940d016a7::/opt/gopath/src/github.com/hyperledger/fabric/peer#  source scripts/setpeer 1 0
 CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/marblesinc.com/peers/peer0.marblesinc.com/tls/ca.crt
 CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/peers/peer0.unitedmarbles.com/tls/server.key
 CORE_PEER_LOCALMSPID=Org1MSP
 CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
 CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/peers/peer0.unitedmarbles.com/tls/server.crt
 CORE_PEER_TLS_ENABLED=true
 CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/marblesinc.com/users/Admin@marblesinc.com/msp
 CORE_PEER_ID=cli
 CORE_LOGGING_LEVEL=DEBUG
 CORE_PEER_ADDRESS=peer0.marblesinc.com:7051

**Step 8.4:**	Install the marbles chaincode on Peer0 in Org1MSP::

 root@a37940d016a7:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer chaincode install -n marbles -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/marbles 
 2018-09-24 19:43:48.946 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
 2018-09-24 19:43:48.946 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
 2018-09-24 19:43:49.147 UTC [chaincodeCmd] install -> INFO 003 Installed remotely response:<status:200 payload:"OK" > 

An interesting thing to note is that for the *peer chaincode install* command you did not need to specify the $FABRIC_TLS environment variable.  
This is because this operation does not cause the peer to communicate with the orderer. 
Also, you did not need to specify the $CHANNEL_NAME environment variable.  
This is because the *peer chaincode install* command only installs the chaincode on the peer node.  
You only need to do this once per peer.  
That is, even if you wanted to install the same chaincode on multiple channels on a peer, you only install the chaincode once on that peer.

Installing chaincode on a peer is a necessary step, but not the only step needed, in order to execute chaincode on that peer.  
The chaincode must also be instantiated on a channel that the peer participates in.  
You will do that in the next section.
 
Section 9	- Instantiate the chaincode on the channel
====================================================

In the previous section, you installed chaincode on two of your four peers.  
Chaincode installation is a peer-level operation.  
Chaincode instantiation, however, is a channel-level operation.  
It only needs to be performed once on the channel, no matter how many peers have joined the channel.

Chaincode instantiation causes a transaction to occur on the channel, so even if a peer on the channel does not have the chaincode installed, it will be made aware of the instantiate transaction, and thus be aware that the chaincode exists and be able to commit transactions from the chaincode to the ledger-  it just would not be able to endorse a transaction on the chaincode.

**Step 9.1:**	You want to stay signed in to the *cli* Docker container, however, you will also want to issue some Docker commands from your Linux on IBM Z host, so at this time open up a second PuTTY session and sign in to your Linux on IBM Z guest, just as you did in *Step 3.1*.
For the remainder of this lab, I will refer to the session where you are in the *cli* Docker container as *PuTTY Session 1*, and this new session where you are at the Linux on IBM Z host as *PuTTY Session 2*.

**Step 9.2:**	You are going to confirm that you do not have any chaincode Docker images created, nor any Docker chaincode containers running currently. 
From PuTTY Session 2, enter this command and observe that all of your images begin with *hyperledger*::

 bcuser@ubuntu16045:~$ docker images
 REPOSITORY                   TAG                 IMAGE ID            CREATED             SIZE
 hyperledger/fabric-ca        1.2.0               7a9a8d2589c9        2 months ago        217MB
 hyperledger/fabric-tools     1.2.0               3264774e5306        2 months ago        1.5GB
 hyperledger/fabric-orderer   1.2.0               75b2abac5351        2 months ago        140MB
 hyperledger/fabric-peer      1.2.0               85d0c3e2c248        2 months ago        147MB
 hyperledger/fabric-couchdb   0.4.10              76a35badf382        2 months ago        1.76GB

You'll see soon that chaincode Docker images will start with the name *dev-peer\**.
But right now your output should look similar to what is shown above which is an indication that you have no chaincode Docker images yet, which is expected at this point in the lab.

**Step 9.3:** Now do essentially the same thing with *docker ps* and you should see all of the Docker containers for the 
Hyperledger Fabric processes and CouchDB, but no chaincode-related Docker containers (which would also start with *dev-peer\**::  

 bcuser@ubuntu16045:~$ docker ps -a
 CONTAINER ID        IMAGE                        COMMAND                  CREATED             STATUS              PORTS                                                                       NAMES
 a37940d016a7        hyperledger/fabric-tools:1.2.0      "bash"                   About an hour ago   Up About an hour                                                                                cli
 a9045b1bae14        hyperledger/fabric-peer:1.2.0       "peer node start"        About an hour ago   Up About an hour    0.0.0.0:8051->7051/tcp, 0.0.0.0:8052->7052/tcp, 0.0.0.0:8053->7053/tcp      peer1.unitedmarbles.com
 f5ced05284a3        hyperledger/fabric-peer:1.2.0       "peer node start"        About an hour ago   Up About an hour    0.0.0.0:7051-7053->7051-7053/tcp                                            peer0.unitedmarbles.com
 1273c9653f24        hyperledger/fabric-peer:1.2.0       "peer node start"        About an hour ago   Up About an hour    0.0.0.0:10051->7051/tcp, 0.0.0.0:10052->7052/tcp, 0.0.0.0:10053->7053/tcp   peer1.marblesinc.com
 98c44b366a92        hyperledger/fabric-peer:1.2.0       "peer node start"        About an hour ago   Up About an hour    0.0.0.0:9051->7051/tcp, 0.0.0.0:9052->7052/tcp, 0.0.0.0:9053->7053/tcp      peer0.marblesinc.com
 c9965303275e        hyperledger/fabric-couchdb:0.4.10   "tini -- /docker-ent…"   About an hour ago   Up About an hour    4369/tcp, 9100/tcp, 0.0.0.0:6984->5984/tcp                                  couchdb1
 7db99161595b        hyperledger/fabric-ca:1.2.0         "sh -c 'fabric-ca-se…"   About an hour ago   Up About an hour    0.0.0.0:7054->7054/tcp                                                      ca_Org0
 9ef9b33a9490        hyperledger/fabric-orderer:1.2.0    "orderer"                About an hour ago   Up About an hour    0.0.0.0:7050->7050/tcp                                                      orderer.blockchain.com
 6ce17ab97f19        hyperledger/fabric-couchdb:0.4.10   "tini -- /docker-ent…"   About an hour ago   Up About an hour    4369/tcp, 9100/tcp, 0.0.0.0:5984->5984/tcp                                  couchdb0
 5c9909a6b9b5        hyperledger/fabric-couchdb:0.4.10   "tini -- /docker-ent…"   About an hour ago   Up About an hour    4369/tcp, 9100/tcp, 0.0.0.0:8984->5984/tcp                                  couchdb3
 f667587a207c        hyperledger/fabric-couchdb:0.4.10   "tini -- /docker-ent…"   About an hour ago   Up About an hour    4369/tcp, 9100/tcp, 0.0.0.0:7984->5984/tcp                                  couchdb2
 450ac4b68d59        hyperledger/fabric-ca:1.2.0         "sh -c 'fabric-ca-se…"   About an hour ago   Up About an hour    0.0.0.0:8054->7054/tcp                                                      ca_Org1

**Step 9.4:** Entering this will make this fact stand out more as you should only see column headers in your output. 
(The *-v* flag for *grep* says “do not show me anything that contains the string “hyperledger”)::

 bcuser@ubuntu16045:~$ docker ps -a | grep -v hyperledger
 CONTAINER ID        IMAGE                        COMMAND                  CREATED             STATUS              PORTS                                                                       NAMES

Now that you have established that you have no chaincode-related Docker images or containers present, it is time to instantiate the chaincode.

**Step 9.5:**	On PuTTY Session 1, switch to Peer 0 of Org0MSP by entering::

 root@a37940d016a7:/opt/gopath/src/github.com/hyperledger/fabric/peer# source scripts/setpeer 0 0
 CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/peers/peer0.unitedmarbles.com/tls/ca.crt
 CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/peers/peer0.unitedmarbles.com/tls/server.key
 CORE_PEER_LOCALMSPID=Org0MSP
 CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
 CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/peers/peer0.unitedmarbles.com/tls/server.crt
 CORE_PEER_TLS_ENABLED=true
 CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/users/Admin@unitedmarbles.com/msp
 CORE_PEER_ID=cli
 CORE_LOGGING_LEVEL=DEBUG
 CORE_PEER_ADDRESS=peer0.unitedmarbles.com:7051

**Step 9.6:** On PuTTY Session 1, issue the command to instantiate the chaincode on the channel::

 root@a37940d016a7:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer chaincode instantiate -o orderer.blockchain.com:7050 -n marbles -v 1.0 -c '{"Args":["init","1"]}' -P "OR ('Org0MSP.member','Org1MSP.member')" $FABRIC_TLS -C $CHANNEL_NAME
 2018-09-24 20:19:37.884 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
 2018-09-24 20:19:37.885 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc

**Note:**  In your prior commands, when specifying the channel name, you used lowercase ‘c’ as the argument, e.g., *-c $CHANNEL_NAME*.  
In the *peer chaincode instantiate* command however, you use an uppercase ‘C’ as the argument to specify the channel name, e.g., *-C mychannel*, because -c is used to specify the arguments given to the chaincode.  
Why *c* for arguments you may ask?  
Well, the ‘*c*’ is short for ‘*ctor*’, which itself is an abbreviation for *constructor*, which is a fancy word object-oriented programmers use to refer to the initial arguments given when creating an object.  
Some people do not like being treated as objects, but evidently chaincode does not object to being objectified.

**Step 9.7:**	You may have noticed a longer than usual pause while that last command was being run.  
The reason for this is that as part of the instantiate, a Docker image for the chaincode is created and then a Docker container is started from the image.  
To prove this to yourself, on PuTTY Session 2, enter this to see the new Docker image::

 bcuser@ubuntu16045:~$ docker images dev-*
 REPOSITORY                                                                                                 TAG                 IMAGE ID            CREATED              SIZE
 dev-peer0.unitedmarbles.com-marbles-1.0-7e92f069adb7469939a96dcba723fa2019745461f05a562e81cec38e46424aa1   latest              9e2bc3cc323e        About a minute ago   135MB

**Step 9.8:** And enter this to see the Docker chaincode container created from the new Docker image::

 bcuser@ubuntu16045:~$ docker ps | grep -v hyperledger 
 CONTAINER ID        IMAGE                                                                                                      COMMAND                  CREATED             STATUS              PORTS                                                                       NAMES
 57b6108e3057        dev-peer0.unitedmarbles.com-marbles-1.0-7e92f069adb7469939a96dcba723fa2019745461f05a562e81cec38e46424aa1   "chaincode -peer.add…"   2 minutes ago       Up 2 minutes                                                                                    dev-peer0.unitedmarbles.com-marbles-1.0      
 bcuser@ubuntu16045:~$ 

The naming convention used by Hyperledger Fabric v1.2.0 for the Docker images it creates for chaincode is *HyperledgerFabricNetworkName-PeerName-ChaincodeName-ChaincodeVersion-SHA256Hash*. 
In our case of *dev-peer0.unitedmarbles.com-marbles-1.0-*, the default name of a Hyperledger Fabric network is *dev*, and you did not change it.  
*peer0.unitedmarbles.com* is the peer name of peer0 of Org0MSP, and you specified this via the CORE_PEER_ID environment variable in the Docker Compose YAML file. 
*marbles* is the name you gave this chaincode in the *-n* argument of the *peer chaincode install* command, and *1.0* is the version of the chaincode you used in the *-v* argument of the *peer chaincode install* command.

Note that a chaincode Docker container was only created for the peer on which you entered the *peer chaincode instantiate* command.  
Docker containers will not be created on the other peers until you run a *peer chaincode invoke* or *peer chaincode query* command on that peer.
 

Section 10 - Invoke chaincode functions
=======================================

You are now ready to invoke chaincode functions that will create, read, update and delete data in the ledger.

In this section, you will enter *scripts/setpeer* and *peer chaincode commands* in PuTTY session 1, while you will enter *docker ps* and *docker images* commands in PuTTY session 2.
 
**Step 10.1:** Switch to peer0 of Org0MSP::

 root@a37940d016a7:/opt/gopath/src/github.com/hyperledger/fabric/peer# source scripts/setpeer 0 0
 CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/peers/peer0.unitedmarbles.com/tls/ca.crt
 CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/peers/peer0.unitedmarbles.com/tls/server.key
 CORE_PEER_LOCALMSPID=Org0MSP
 CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
 CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/peers/peer0.unitedmarbles.com/tls/server.crt
 CORE_PEER_TLS_ENABLED=true
 CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/users/Admin@unitedmarbles.com/msp
 CORE_PEER_ID=cli
 CORE_LOGGING_LEVEL=DEBUG
 CORE_PEER_ADDRESS=peer0.unitedmarbles.com:7051

**Step 10.2:**	You will use the marbles chaincode to create a new Marbles owner named John.  
If you would like to use a different name than John, that is fine but there will be other places later where you will need to use your “custom” name instead of John.  
I will let you know when that is necessary.  
Enter this command in PuTTY session 1::

 root@a37940d016a7:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer chaincode invoke -n marbles -c '{"Args":["init_owner", "o0000000000001","John","Marbles Inc"]}' $FABRIC_TLS -C $CHANNEL_NAME
 2018-09-24 20:25:47.496 UTC [chaincodeCmd] InitCmdFactory -> INFO 001 Retrieved channel (mychannel) orderer endpoint: orderer.blockchain.com:7050
 2018-09-24 20:25:47.514 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 002 Chaincode invoke successful. result: status:200 
 
**Step 10.3:**	Let’s deconstruct the arguments to the chaincode::

 {“Args”:[“init_owner”, “o0000000000001”, “John”, “Marbles Inc”]}
 
This is in JSON format.  
JSON stands for JavaScript Object Notation, and is a very popular format for transmitting data in many 
languages, not just with JavaScript.  
What is shown above is a single name/value pair.  
The name is *Args* and the value is an array of four arguments.  
(The square brackets “[“ and “]” specify an array in JSON).

**Note:** In the formal JSON definition the term ‘*name/value*’ is used, but many programmers will also use the term ‘*key/value*’ instead.  
You can consider these two terms as synonymous.  
(Many people use the phrase “the same” instead of the word “synonymous”).

The *Args* name specifies the arguments passed to the chaincode invocation.  
There is an interface layer, also called a “shim”, that gains control before passing it along to user-written chaincode functions-  it expects this *Args* name/value pair.

The shim also expects the first array value to be the name of the user-written chaincode function that it will pass control to, and then all remaining array values are the arguments to pass, in order, to that user-written chaincode function.

So, in the command you just entered, the *init_owner* function is called, and it is passed three arguments, *o0000000000001*, *John*, and *Marbles Inc*. 

It is logic within the *init_owner* function that cause updates to the channel’s ledger- subject to the transaction flow in Hyperledger Fabric v1.2.0-  that is, chaincode execution causes proposed updates to the ledger, which are only committed at the end of the transaction flow if everything is validated properly.  
But it all starts with function calls inside the chaincode functions that ask for ledger state to be created or updated.

**Step 10.4:**	Go to PuTTY session 2, and enter this Docker command and you will observe that you still only have a Docker image and a Docker container for peer0 of Org0MSP::

 bcuser@ubuntu16045:~$ docker images dev-*
 REPOSITORY                                                                                                 TAG                 IMAGE ID            CREATED             SIZE
 dev-peer0.unitedmarbles.com-marbles-1.0-7e92f069adb7469939a96dcba723fa2019745461f05a562e81cec38e46424aa1   latest              9e2bc3cc323e        8 minutes ago       135MB

**Step 10.5:** Enter this command to see information about the chaincode container.  I introduce here the *--no-trunc* option, which stands for *no truncation*, so you can see more information about the container::

 bcuser@ubuntu16045:~$ docker ps --no-trunc | grep dev-
 57b6108e30571e3913b58956b172a7dcebef1520825f6e6a0acb29e28e118a98   dev-peer0.unitedmarbles.com-marbles-1.0-7e92f069adb7469939a96dcba723fa2019745461f05a562e81cec38e46424aa1   "chaincode -peer.address=peer0.unitedmarbles.com:7052"                                                                                                                                                                                                                9 minutes ago       Up 9 minutes                                                                                    dev-peer0.unitedmarbles.com-marbles-1.0

The takeaway is that the chaincode execution has only run on peer0 of Org0MSP so far, and this is also the peer on which you instantiated the chaincode, so the Docker image for the chaincode, and the corresponding Docker container based on the image, have been created for only this peer.  
You will see soon that other peers will have their own chaincode Docker image and Docker container built the first time they are needed.

**Step 10.6:**	You created a marble owner in the previous step, now create a marble belonging to this owner.   Perform this from peer0 of Org1, so from PuTTY session 1, switch to Peer0 of Org1MSP::

 root@a37940d016a7:/opt/gopath/src/github.com/hyperledger/fabric/peer# source scripts/setpeer 1 0
 CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/marblesinc.com/peers/peer0.marblesinc.com/tls/ca.crt
 CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/peers/peer0.unitedmarbles.com/tls/server.key
 CORE_PEER_LOCALMSPID=Org1MSP
 CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
 CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/peers/peer0.unitedmarbles.com/tls/server.crt
 CORE_PEER_TLS_ENABLED=true
 CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/marblesinc.com/users/Admin@marblesinc.com/msp
 CORE_PEER_ID=cli
 CORE_LOGGING_LEVEL=DEBUG
 CORE_PEER_ADDRESS=peer0.marblesinc.com:7051

**Step 10.7:** Now enter the command to create a new marble for John::

 root@a37940d016a7:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer chaincode invoke -n marbles -c '{"Args":["init_marble","m0000000000001","blue","35","o0000000000001","Marbles Inc"]}' $FABRIC_TLS -C $CHANNEL_NAME 
 2018-09-24 20:34:35.084 UTC [chaincodeCmd] InitCmdFactory -> INFO 001 Retrieved channel (mychannel) orderer endpoint: orderer.blockchain.com:7050
 2018-09-24 20:34:48.574 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 002 Chaincode invoke successful. result: status:200

This time you called the *init_marble* function.  Now you have created one owner, and one marble.

The owner is *John* (or your custom name) and his id is *o0000000000001*, and his marble has an id of *m0000000000001*.  
I cleverly decided that the letter ‘*o*’ stands for owner and the letter ‘*m*’ stands for marbles.  
I put 12 leading zeros in front of the number 1 in case you wanted to stay late and create trillions of marbles and owners.

**Step 10.8:**	In PuTTY session 2, issue the command to see that you have two Docker chaincode images::

 bcuser@ubuntu16045:~$ docker images dev-*
 REPOSITORY                                                                                                 TAG                 IMAGE ID            CREATED             SIZE
 dev-peer0.marblesinc.com-marbles-1.0-4077677f13838bacbfd8ff943e7348c00f3c4d6ca6e2838efd14204ca87ea12b      latest              4b98b899ca0b        About a minute ago   135MB
 dev-peer0.unitedmarbles.com-marbles-1.0-7e92f069adb7469939a96dcba723fa2019745461f05a562e81cec38e46424aa1   latest              9e2bc3cc323e        16 minutes ago       135MB


**Step 10.9:**	Also in PuTTY session 2, issue the command to see that you have two Docker chaincode containers::

 bcuser@ubuntu16045:~$ docker ps --no-trunc | grep dev-*
 78a3abbf69978b234ab43a3e1ef694e2332db920fdb7fcd10af2e8c563b9fe75   dev-peer0.marblesinc.com-marbles-1.0-4077677f13838bacbfd8ff943e7348c00f3c4d6ca6e2838efd14204ca87ea12b      "chaincode -peer.address=peer0.marblesinc.com:7052"                                                                                                                                                                                                                   About a minute ago   Up About a minute                                                                               dev-peer0.marblesinc.com-marbles-1.0
 57b6108e30571e3913b58956b172a7dcebef1520825f6e6a0acb29e28e118a98   dev-peer0.unitedmarbles.com-marbles-1.0-7e92f069adb7469939a96dcba723fa2019745461f05a562e81cec38e46424aa1   "chaincode -peer.address=peer0.unitedmarbles.com:7052"                                                                                                                                                                                                                16 minutes ago       Up 16 minutes                                                                                   dev-peer0.unitedmarbles.com-marbles-1.0
 bcuser@ubuntu16045:~$ 

**Step 10.10:**	You will create a new owner now.  Back on PuTTY session 1, try it on Peer 1 of Org0MSP::

 root@a37940d016a7:/opt/gopath/src/github.com/hyperledger/fabric/peer# source scripts/setpeer 0 1
 CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/peers/peer1.unitedmarbles.com/tls/ca.crt
 CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/peers/peer0.unitedmarbles.com/tls/server.key
 CORE_PEER_LOCALMSPID=Org0MSP
 CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
 CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/peers/peer0.unitedmarbles.com/tls/server.crt
 CORE_PEER_TLS_ENABLED=true
 CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/unitedmarbles.com/users/Admin@unitedmarbles.com/msp
 CORE_PEER_ID=cli
 CORE_LOGGING_LEVEL=DEBUG
 CORE_PEER_ADDRESS=peer1.unitedmarbles.com:7051

**Step 10.11:** Then run this command to try to create a new owner::

 root@a37940d016a7:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer chaincode invoke -n marbles -c '{"Args":["init_owner","o0000000000002","Barry","United Marbles"]}' $FABRIC_TLS -C $CHANNEL_NAME
 2018-09-24 20:38:05.165 UTC [chaincodeCmd] InitCmdFactory -> INFO 001 Retrieved channel (mychannel) orderer endpoint: orderer.blockchain.com:7050
 Error: endorsement failure during invoke. chaincode result: <nil>

Notice that an error occurred on this command.  
I meant for that to happen.  
(Otherwise you wouldn't be reading this, right?)

The *invoke* failed because you have not yet installed the chaincode on Peer 1 of Org0.  

You must first *install* chaincode on a peer not only before you can do an *instantiate* from that peer, but also before you can do an *invoke* or *query* from that peer.  
If you want a peer to perform the endorsing function for a transaction, the chaincode for that transaction must be installed on that peer.  
If that peer is a member of the channel on which the chaincode is instantiated, but has not had the chaincode installed on it, it will still perform the committer function and update its copy of the channel’s ledger when it receives valid transactions from the orderer, but it cannot endorse transaction proposals unless the chaincode has been installed on it.

**Step 10.12**:	Correct things by installing the chaincode on peer1 of Org0.  
In PuTTY session 1, enter this command, which should look familiar to you::

 root@a37940d016a7:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer chaincode install -n marbles -v1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/marbles
 2018-09-24 20:41:04.071 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
 2018-09-24 20:41:04.071 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
 2018-09-24 20:41:04.265 UTC [chaincodeCmd] install -> INFO 003 Installed remotely response:<status:200 payload:"OK" > 

**Step 10.13:**	Now, in PuTTY session 1, repeat the *peer chaincode invoke* command from *Step 10.11*.  It should work this time::

 root@a37940d016a7:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer chaincode invoke -n marbles -c '{"Args":["init_owner","o0000000000002","Barry","United Marbles"]}' $FABRIC_TLS -C $CHANNEL_NAME
 2018-09-24 20:42:13.399 UTC [chaincodeCmd] InitCmdFactory -> INFO 001 Retrieved channel (mychannel) orderer endpoint: orderer.blockchain.com:7050
 2018-09-24 20:42:27.139 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 002 Chaincode invoke successful. result: status:200 

**Step 10.14:**	Go back to PuTTY session 2 and enter the Docker command that will show you that you now have your third chaincode-related Docker image, the one just built for peer1 of Org0::

 bcuser@ubuntu16045:~$ docker images dev-*
 REPOSITORY                                                                                                 TAG                 IMAGE ID            CREATED             SIZE
 dev-peer1.unitedmarbles.com-marbles-1.0-dea1aa08dc7c6f282a31dd498670173c21d3e75ef0ef1d170b95e1212fbacb77   latest              8293766b88e8        46 seconds ago      135MB
 dev-peer0.marblesinc.com-marbles-1.0-4077677f13838bacbfd8ff943e7348c00f3c4d6ca6e2838efd14204ca87ea12b      latest              4b98b899ca0b        8 minutes ago       135MB
 dev-peer0.unitedmarbles.com-marbles-1.0-7e92f069adb7469939a96dcba723fa2019745461f05a562e81cec38e46424aa1   latest              9e2bc3cc323e        23 minutes ago      135MB
 
**Step 10.15:**	Enter the Docker commands that will show you that you now have your third chaincode-related Docker container, the one just built for peer1 of Org0::

 bcuser@ubuntu16045:~$ docker ps --no-trunc | grep dev-
 86a5e133e5a63a6398cad814177ad6e18627114a4d506815c1b3c5358b45b145   dev-peer1.unitedmarbles.com-marbles-1.0-dea1aa08dc7c6f282a31dd498670173c21d3e75ef0ef1d170b95e1212fbacb77   "chaincode -peer.address=peer1.unitedmarbles.com:7052"                                                                                                                                                                                                                About a minute ago   Up About a minute                                                                               dev-peer1.unitedmarbles.com-marbles-1.0
 78a3abbf69978b234ab43a3e1ef694e2332db920fdb7fcd10af2e8c563b9fe75   dev-peer0.marblesinc.com-marbles-1.0-4077677f13838bacbfd8ff943e7348c00f3c4d6ca6e2838efd14204ca87ea12b      "chaincode -peer.address=peer0.marblesinc.com:7052"                                                                                                                                                                                                                   8 minutes ago        Up 8 minutes                                                                                    dev-peer0.marblesinc.com-marbles-1.0
 57b6108e30571e3913b58956b172a7dcebef1520825f6e6a0acb29e28e118a98   dev-peer0.unitedmarbles.com-marbles-1.0-7e92f069adb7469939a96dcba723fa2019745461f05a562e81cec38e46424aa1   "chaincode -peer.address=peer0.unitedmarbles.com:7052"                                                                                                                                                                                                                23 minutes ago       Up 23 minutes                                                                                   dev-peer0.unitedmarbles.com-marbles-1.0
 bcuser@ubuntu16045:~$ 

**Step 10.16:**	Try some additional chaincode invocations. 
You have had enough experience switching between peers with  *source scripts/setpeer* and issuing the *peer chaincode invoke* command that I will not show the output, nor tell you from which peer you should enter your command.   
I will just list several more commands you can run against the marbles chaincode. 
Feel free to switch amongst the four peers as you see fit before you enter each command.  
Note however, that you have only installed the chaincode on three of the four peers, so if you choose that fourth peer, you will need to install the chaincode there first.   
I won’t tell you which peer does not currently have the chaincode installed, but if you need a hint, it is the one that does not have a Docker image built yet for its chaincode.  
(Note that checking for the absence of a Docker image for a peer is not, by itself, proof that you have not installed the chaincode on that peer- the Docker image is not built until you first invoke a function against the chaincode on that peer).

If you are ambitious and want to install the chaincode on that fourth peer, try the useful Docker commands I have shown you from PuTTY session 2 to see that the chaincode's Docker image and Docker containerare created when you invoke a transaction on that fourth peer.

Try some or all of these commands from PuTTY session 1:

Create a marble for Barry, i.e., owner o0000000000002::

 peer chaincode invoke -n marbles -c '{"Args":["init_marble","m0000000000002","green","50","o0000000000002","United Marbles"]}' $FABRIC_TLS -C $CHANNEL_NAME

Obtain all marble information-  marbles and owners::

 peer chaincode invoke -n marbles -c '{"Args":["read_everything"]}' $FABRIC_TLS -C $CHANNEL_NAME

Change marble ownership-  ‘Barry’ is giving his marble to ‘John’::

 peer chaincode invoke -n marbles -c '{"Args":["set_owner","m0000000000002","o0000000000001","United Marbles"]}' $FABRIC_TLS -C $CHANNEL_NAME

Get the history of marble ‘m0000000000002’::

 peer chaincode invoke -n marbles -c '{"Args":["getHistory","m0000000000002"]}' $FABRIC_TLS -C $CHANNEL_NAME

Delete marble ‘m0000000000002’::

 peer chaincode invoke -n marbles -c '{"Args":["delete_marble","m0000000000002","Marbles Inc"]}' $FABRIC_TLS -C $CHANNEL_NAME

Try again to get the history of marble ‘m0000000000002’ after you just deleted it::

 peer chaincode invoke -n marbles -c '{"Args":["getHistory","m0000000000002"]}' $FABRIC_TLS -C $CHANNEL_NAME

Obtain all marble information again.  See if it matches your expectations based on the commands you have entered::

 peer chaincode invoke -n marbles -c '{"Args":["read_everything"]}' $FABRIC_TLS -C $CHANNEL_NAME
 
**Step 10.17:** Exit the *cli* Docker container from PuTTY session 1.  
Your command prompt should change to reflect that you are now back at your Linux on IBM Z host prompt and no longer in the Docker container::

 root@a37940d016a7:/opt/gopath/src/github.com/hyperledger/fabric/peer# exit
 exit
 bcuser@ubuntu16045:~/zmarbles$ 


**Step 10.18:**	Congratulations!! Congratulations on your fortitude and perseverance.  Leave your Hyperledger Fabric network and all the chaincode Docker containers up and running-  you will use what you created here in part 2 of this lab where you will install a front-end Web application that will interact with the marbles chaincode that you have installed here.


