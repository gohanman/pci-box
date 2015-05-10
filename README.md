# pci-box
PCI in a Box!

This repo contains information and sample configurations for using
free, open source tools to comply with certain PCI-DSS requirements.
The PCI Box is assumed to be running Linux. I will be using CentOS
7 in the examples. Anyone with a strong alternate distro preference 
is assumed to be able to work out the differences on their own.

Example PCI Box specs:
ASUS J1900I-C Quad Core Celeron Mobo/CPU/VGA ITX combo
8GB (2x4GB) DDR3 1333
120GB SATA SSD
Habey EMC-800B Case with power supply

Cost of the PCI Box is approximately $250 US. An x86_64 CPU is
nice to have as occasionally 32-bit binaries are no longer 
available. Other choices can be summarized as "Eh, why not? It's
doesn't cost much."

### Openvas
