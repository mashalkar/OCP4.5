# OCP4.5
Installation
https://docs.openshift.com/container-platform/4.5/installing/installing_bare_metal/installing-bare-metal.html

- Host requirements
1) Bastion/Installer system (Openstack installer and oc)
2) Bootstrap (Temporary bootstrap system)
3) 3 master-nodes/ control plane
4) 2 worker/compute nodes.

Installation Node               - Bootstrap node - master-0
(Bastion/HA-proxy/DNS/HTTPD)                     - master-1
                                                 - master-2
                                                 - worker-0
                                                 - worker-1
                                   
                                  
- Pre-requisites:
DNS server
DHCP/Static IP 
HTTPD
HA-proxy

- Openstack Installer
Bastion host: Install RHEL system and configure DNS, HA-proxy and httpd server on it.








Troubleshooting
$ oc get clusteroperator/console -o yaml
