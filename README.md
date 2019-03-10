# Hyperledger Fabric v1.4 labs for IBM Z

## March 9, 2019 status update
The Marbles labs have been updated to work with Hyperledger Fabric v1.4

These labs are designed to be run consecutively (lab1, lab2, lab3 and then lab4).

They were designed to run on Ubuntu 16.04 on IBM Z but should be transferable to Ubuntu 16.04 running on x86_64, as long as you can replace the specific commands in lab1 that get *s390x*-specific files for Docker, Node.js/npm and Golang.

You are reponsible for providing your own Ubuntu 16.04 instance running on an IBM Z mainframe server. These labs were originally written for a structured, classroom environment, where these instances were provided to the students.  Any statements in the labs implying that such systems are already available, are an artifact of these labs' original purpose. Similary, any statements that may imply instructor availability do not constitute a promise of support.

If you can translate the *apt* commands used on Ubuntu 16.04 to other Linux distros and can get at least Docker 1.13 or any version of Docker CE or EE on your distro then they might possibly work there as well. 

The Marbles Labs assume the existence of prerequisite software that can be installed by following the instructions in 
https://github.com/silliman/BlockchainImmersion/blob/master/SupplementalMaterial/FabricPreReqs.rst

**Note:** These labs are provided as-is, and are intended for educational purposes only.  Hyperledger Fabric and Hyperledger Composer are rapidly evolving and any parts or all of these labs can, and probably will, become obsolete or inoperable over time.  Theses labs will be kept up-to-date, improved or replaced on a best-effort basis only, at the author's sole discretion, with no guarantees to the timeliness or availability of such updates.
