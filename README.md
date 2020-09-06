# OCP4.5
Installation
https://docs.openshift.com/container-platform/4.5/installing/installing_bare_metal/installing-bare-metal.html

Host requirements
1) Bastion/Installer system (Openstack installer and oc)
2) Bootstrap (Temporary bootstrap system)
3) 3 master-nodes/ control plane
4) 2 worker/compute nodes.

Installation Node               - Bootstrap node - master-0
(Bastion/HA-proxy/DNS/HTTPD)                     - master-1
                                                 - master-2
                                                 - worker-0
                                                 - worker-1
                                   
                                  
Pre-requisites:
DNS server
DHCP/Static IP 
HTTPD
HA-proxy


1) Creating bastion/installer host:
  1.1) Decide the "Domain name" and "cluster name" and configure the DNS server as per it.
  1.2) Bastion host: Install RHEL system and configure DNS, HA-proxy and httpd server on it. (Refer dns.readme and haproxy.readme for installation and depoyment of services)

2) Fetch Openshift Installer from https://cloud.redhat.com/openshift/install/metal
   Pull secret
   Download OC

3) Copy and edit the install-config.yaml and put it in installation-directory

4) Create manifests (copy the install-config.yaml to installation directory)
./openshift-install create manifests --dir=<installation_directory> 

5) Create ingition files:
$ ./openshift-install create ignition-configs --dir=<installation_directory> 

6) Download coreos:
https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/4.5/4.5.6/
 6.1 rhcos-installer.x86_64.iso   -- Used to inital boot the Physical/Virtual machine.
 6.2 rhcos-metal.x86_64.raw.gz    -- Used for actual installation over network.

7) Copy ignition files and rhcos-metal.raw.gz file to /var/www/html/ directory.

Note: Before you perform the installation of the nodes, Fetch the static IP for the NIC and then add those entries in your DNS server and HA-Proxy server.

8) Boot the Physcal/Virtual machine with rhcos-installer.x86_64.iso (Dowloaded in 6.1)

Note: first dummy start the VM to check what is the static IP it gets, What is the disk name it gets, and NIC name. This will be used for actuall installation.

At the boot from ISO, press tab at install screen and add below additional parameters which will point towards the ignition file and metal.raw.gz
~~
coreos.inst=yes
coreos.inst.install_dev=sda 
coreos.inst.image_url=<image_URL> 
coreos.inst.ignition_url=http://example.com/config.ign 
ip=<dhcp or static IP address>   
bond=<bonded_interface> 
~~
  
Example file:
~~
coreos.inst.install_dev=sda 
coreos.inst.image_url=http://10.1.11.247:8080/metal.raw.gz 
coreos.inst.ignition_url=http://10.1.11.247:8080/master.ign
ip=10.1.11.219::10.1.11.254:255.255.252.0:bootstrap.kubevirt.kmash.com:ens3:none nameserver=10.1.11.247
~~

9) Test the cluster and deployment:
$ ./openshift-install --dir=<installation_directory> wait-for bootstrap-complete --log-level=info 
$ export KUBECONFIG=<installation_directory>/auth/kubeconfig 
$ oc whoami
$ oc get nodes
$ oc get clusteroperators


10) Manually approving the csr for worker nodes:
$ oc get nodes
$ oc get csr
$ oc adm certificate approve <csr_name> 
$ oc get csr -o go-template='{{range .items}}{{if not .status}}{{.metadata.name}}{{"\n"}}{{end}}{{end}}' | xargs oc adm certificate approve

Troubleshooting
$ oc get clusteroperator/console -o yaml
