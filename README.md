# pci-box
PCI in a Box!

This repo contains information and sample configurations for using
free, open source tools to comply with certain PCI-DSS requirements.
The PCI Box is assumed to be running Linux. I will be using CentOS
7 in the examples. Anyone with a strong alternate distro preference 
is assumed to be able to work out the differences on their own.

Example PCI Box specs:
* ASUS J1900I-C Quad Core Celeron Mobo/CPU/VGA ITX combo
* 8GB (2x4GB) DDR3 1333
* 120GB SATA SSD
* Habey EMC-800B Case with power supply

Cost of the PCI Box is approximately $250 US. An x86_64 CPU is
nice to have as occasionally 32-bit binaries are no longer 
available. Other choices can be summarized as "Eh, why not? It's
doesn't cost much."

# Pre-Reqs
I'll reference a couple diagnostics repeatedly. Linux veterans feel free to skip ahead.
## Is the process actually running?
```
$ ps -Af | grep <program name>
```
Where ```<program name>``` is the name of the program. Besides being a quick verfication that the program hasn't died, sometimes the arguments in the output help verify the program is running with the expected configuration.

## Is the network port open?
```
$ netstat -nltp
```
Show all network ports that are listening (-l) via TCP (-t). Display the numeric port rather than a procotol name (-n) and display the name of the process rather than its ID (-p). The -p option only works as root. Note that this command does not give any information about firewall rules; it merely lists what is listening inside the firewall.

# OpenVAS
* What is OpenVAS?
  * OpenVAS is a vulnerability scanner stemming from the now commercial, previously open source Nessus.
* How does OpenVAS help with PCI compliance?
  * OpenVAS can fulfill requirements related to internal vulnerability scans nicely. It's low cost, user friendly, and can be resident in the card data environment without introducing any external access.
  * OpenVAS can scan networks and verify no unexpected devices have been connected.
* What are the downsides of OpenVAS?
  * It's kind of a pain to install. Hence the guide.

## Installing OpenVAS
The [site](http://www.openvas.org/download.html) provides source code as well as 3rd party binaries. My luck with binaries has been mixed, at best, so we're going to build from source. Yes, this is a bit more work. The [source](http://www.openvas.org/install-source.html) version of OpenVAS-7 is comprised of several sub packages:
* [Libraries 7.0.10](http://wald.intevation.org/frs/download.php/2031/openvas-libraries-7.0.10.tar.gz)
* [Scanner 4.0.6](http://wald.intevation.org/frs/download.php/1959/openvas-scanner-4.0.6.tar.gz)
* [Manager 5.0.10](http://wald.intevation.org/frs/download.php/2035/openvas-manager-5.0.10.tar.gz)
* [GSA 5.0.7](http://wald.intevation.org/frs/download.php/2039/greenbone-security-assistant-5.0.7.tar.gz)
* [CLI 1.3.1](http://wald.intevation.org/frs/download.php/1803/openvas-cli-1.3.1.tar.gz)

They should be installed in that order. The build sequence is more or less the same for all of them. If you're used to classic autotools, the cmake step is much like ./configure and will generally note missing dependencies. For each component, the build+install steps will be:
```
$ wget <tar.gz URL>
$ tar xzf <tar.gz file>
$ cd <extracted directory>
$ make build
$ cd build
$ cmake ..
$ make
$ make install
```
Each subpackage contains an INSTALL file with information about build requirements. The cmake step will list information about any missing dependencies. I was able to install everything I needed via yum.

With defaults, this will install to /usr/local. After the first installation, libraries, you probably need to update PKG_CONFIG_PATH:
```
$ export PKG_CONFIG_PATH=/usr/local/lib64/pkg_config:$PKG_CONFIG_PATH
```

After installing, download the openvas-check-setup script which helpfully examines the system for problems and suggests solutions.
```
# wget "https://svn.wald.intevation.org/svn/openvas/trunk/tools/openvas-check-setup"
# chmod +x openvas-check-setup
# cp openvas-check-setup /usr/local/sbin
# openvas-check-setup --v7
```
You will need to get three services configured and running:
* The scanner service, openvassd, performs actual vulnerability scans. It listens on port 9391. The included init script does work even though systemd throws an error. Start up takes a while to load all the threat info but progress is shown in the process list.
* The manager service, openvasmd, starts/stops/schedules scans. It listens on port 9390. The included init script works fine in this case.
* The Greenbone Security Assistant, gsad, is a web-based interface to interact with the low level services using a GUI. It listens on port 9392. Like openvassd, systemd has some weird problem with the init script but does start the service.

Continue running openvas-check-setup and following its directions until it runs without any errors. Now point a browser at port 9392 (note: HTTPS) to access the scanner interface. Login with the user account you created during the openvas-check-setup process. You should see something like this:
[GSA Home Page](images/gsa.png)
