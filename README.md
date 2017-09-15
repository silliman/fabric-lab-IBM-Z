# Hyperledger Fabric v1.0.x labs for IBM Z

These labs are designed to be run consecutively (lab1, lab2, lab3 and then lab4). 

They were designed to run on Ubuntu 16.04 on IBM Z but should be transferable to Ubuntu 16.04 running on x86_64, as long as you can replace the specific commands in lab1 that get *s390x*-specific artifacts for Docker, Node.js/npm and Golang.

You are reponsible for providing your own Ubuntu 16.04 instance running on an IBM Z mainframe server. These labs were originally written for a structured, classroom environment, where these instances were provided to the students.  Any statements in the labs implying that such systems are already available, are an artifact of these labs' original purpose. Similary, any statements that may imply instructor availability do not constitute a promise of support.

If you can translate the *apt* commands used on Ubuntu 16.04 to other Linux distros and can get at least Docker 1.13 or any version of Docker CE or EE
on your distro then they might possibly work there as well. 

The initial state of your Ubuntu instance is assumed to be a minimal Ubuntu installation.   Lab 1 starts from this point.

Lab 2 and Lab 3 deal with the ubiquitous Marbles app.  

Lab 4 covers Hyperledger Composer. 

**Note:**  In Lab 2, Section 3, it mentions to extract the *zmarbles.tar* file.  That file is not provided here, but you can obtain it
from [another GitHub repo of mine](https://www.github.com/silliman/zmarbles).  (That site has *zmarbles.tar.gz* which is a compressed version of *zmarbles.tar*). Go to that page and follow the four steps in the Installation instructions section there instead of 
the instructions presented in Lab 2, Section 3 of the lab.  Do **not** run the *zmarbles-setup.sh* script per the instructions that *follow* the installation instructions over at https://www.github.com/silliman/zmarbles, as Lab 2 and Lab 3 here guide you step-by-step through what that single shell script does for you. So if you want to maximize your learning, I 
recommend that you do Lab 2 and Lab 3 rather than just running the shell-script *zmarbles-setup.sh*  

**Note:** These labs are provided as-is, and are intended for educational purposes only.  Hyperledger Fabric and Hyperledger Composer are rapidly evolving and any parts or all of these labs can, and probably will, become obsolete or inoperable over time.  Theses labs will be kept up-to-date, improved or replaced on a best-effort basis only, at the author's sole discretion, with no guarantees to the timeliness or availability of such updates.
