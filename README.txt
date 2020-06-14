VMM Workload Automation is about the automation of network configuration
in Cisco's Nexus switches for workloads spawned in a VMWare environment.

Automatic Installation of Dependancies:
----------------------------------------

During installation, the following dependent python packages are installed.

a) pyvim

b) pyvmomi

c) PyYAML

d) Flask

If the above packages has dependancies on different packages, they also get installed.

Functionality:
--------------
When the script is started, it module scans all the vCenters specified in the conf.yml file
and collects the following information for each vCenter:

a) List of DVS and the DV-PG configured in all DC's.

b) List of PG's configured in every host in all DC's.

c) For every DV-PG or PG specified in the conf file, it finds the
configured VLAN's and the directly connected neighbour switches along with
its interface information.

d) For every DV-PG or PG specified in the config file, it gets the associated
network mapping in DCNM.

Then, it merges all the information and calls the DCNM API's to attach the
networks to all the switches that are discovered as neighbours in one of the
steps above.


REST API:
----------

This script also provides the following REST API's

1)curl -XPOST http://127.0.0.1:{port}/workload_auto/refresh

This API is used to tell the script that the CSV file has changed and to
re-read the file and apply any new configuration if needed.

2)curl -XPOST http://127.0.0.1:{port}/workload_auto/resync

This API is used to tell the script to re-discover the DVS-PG, PG, Vlan or
neighbour switches in case it has changed. If there are any changes found,
the configuration is re-appliied accordingly.

3)curl -XPOST http://127.0.0.1:{port}/workload_auto/clean

This API is used to clean-up or delete the network attachments that was done.

Configuration Files:
--------------------
This python code uses the following config files:

conf.yml:
-----------

This file specifies the IP address, username and password of DCNM. 
And, for each DCNM, the list of vCenter's information like the IP address,
username, and password of the vCenters are also specified. There can be 
multiple DCNM's that can be specified in this conf.yml. For every DCNM 
instance, there's an associated csv file.

The location of this file depends on the installation method. Please refer the
installation section. This file comes up with some example entries. Modify
this file to suit your environment. This file has the following entries.

LogFile: This specifies the name of the logfile including the absolute path.
Make sure the directory has write permission for creating the log file.
e.g. /tmp/workloadauto.log

ListenPort: This specifies the port that the python script will use to listen
for the REST API's. e.g. 9590. Make sure that this port is not used by any other
application. One way to find this information is by issuing 
"sudo netstat -tulpn".

AutoDeploy: This value indicates whether the script should automatically
deploy the configuration in the switches after doing the attachment. By
default, it's set to False so that the user can review the config and deploy
it himself in the DCNM.

NwkMgr: This is the top level section under that indicates the Network Manager 
section a.k.a DCNM. For multiple-DCNM's, the information underneath this will
repeat with the appropriate values. Look at the file conf_multiple_dcnm.yml
for a sample yml file that handles multiple DCNM's.

Ip: This specifies the iManagement IP address of the DCNM. e.g. 172.28.10.156. In case of a HA
setup, specify the Active DCNM node's IP address.

User: This specifies the username used to login to DCNM. e.g. admin

Password: This specifies the password of DCNM.

CsvFile: This specifies the absolute path of the location of the CSV file
for this DCNM. e.g. /etc/vmm_workload_auto/sample.csv.

ServerCntrlr: This section has information for the server controller,
a.k.a vCenter/vSphere. For multiple vCenters that fall under this DCNM, this
section will repeat. Please refer the file conf_multiple_vcenter.yml for a
sample file for multiple vCenters under a DCNM.

Ip: This specifies the IP address of the vCenter.

Type: This is the type of the server controller. Leave the default as
vCenter.

User: This refers to the username used to login to the vCenter.
e.g. administrator@vsphere.local

Password: This refers to the password for the vCenter.

CSV File:
----------

This file holds the mapping of the network object in vCenter to the network
created in DCNM. This file has the following entries in CSV format,
i.e.. comma separated entries.

vCenter - This refers to the IP address of vCenter

Dvs - This refers to the name of the DVS.

Dvs_pg - This refers to the DVS PG in the DVS

Host - This refers to the ESXi Host/Server (IP address)

Host_pg - This refers to the port-group in the host.

Fabric - This refers to the fabric in DCNM.

Network - This refers to the name of the network already created in DCNM.

The network object is identified by a unique pair of either <DVS, DVS_PG> or
<Host, Host PG>. If there's a value specified for DVS, DVS_PG, then the values
for <Host, Host_PG> will be blank. In other words, <DVS, DVS_PG> and 
<Host, Host_PG) is mutually exclusive. Consider this example entry:

172.28.10.184,DSwitchPad,DSPad-PG2,,,DEF,MyNetwork_30000

The above line in the CSV file specifies the IP address of vCenter as
172.28.10.184 and the <DVS, DVS PG> values are DSwitchPad, DSPad-PG2
respectively. Since the values for DVS, DVS-PG is specified, the values for
Host, Host-PG will be blank as seen in the above example. The Fabric name is
DEF and the network in DCNM is MyNetwork_30000.

Consider another example:

172.28.10.184,,,172.28.11.33,Pad_Workload_Auto_Nwk,DEF,MyNetwork_60000

In this example, the values for <DVS, DVS-PG> is left blank and the values for
<Host, Host_PG> is specified as 172.28.11.33 and Pad_Workload_Auto_Nwk
respectively. The fabric in DCNM is DEF and the network name in DCNM is
MyNetwork_60000. 

Installation:
--------------

There are couple of ways to install and use this script.

Option 1:
----------
This is for users who are familiar with doing pip install and know how to
setup the proxy or handle cases when there's a conflict in the python packages.

Step 1: Decide whether you want to run it run this in a virtual env or just
normally. If you decide to run this normally, ensure that the user has write permission for doing pip install.

Step 2: Setup the http_proxy, https_proxy and no_proxy appropriately.

For e.g: export http_proxy=http://proxy.esl.cisco.com:80

export https_proxy=https://proxy.esl.cisco.com:80

export no_proxy=127.0.0.1,172.28.10.0/24

In the above example, no_proxy is the DCNM's management subnet.

Step 3:Download and install it from https://test.pypi.org/.

pip3 install -i https://test.pypi.org/simple/ --extra-index-url https://pypi.org/simple vmm-workload-auto

[Please note the above extra-index-url is preceded by two dashes.]

Step 4:
By default, the installation will happen in the following directories unless
the user overrides by giving options in pip command.

The package will be installed under /usr/local/lib/python3.7/site-packages/vmm_workload_auto-0.1.1.dist-info

The config files will be installed under /usr/local/lib/python3.7/site-packages/etc/vmm_workload_auto

The source code will be placed under /usr/local/lib/python3.7/site-packages/workload_auto

The config files will be placed under 
/usr/local/lib/python3.7/site-packages/etc/vmm_workload_auto

Step 5:
Edit the config files in 
/usr/local/lib/python3.7/site-packages/etc/vmm_workload_auto as explained
under the Configuration Section.

Make sure that the path of the CSV file specified in the conf.yml file is correct.

Step 6 (Running the script):
The entry point for the script will be /usr/local/bin/vmm_workload_auto.
Either run it as /usr/local/bin/vmm_workload_auto or just vmm_workload_auto,
if '/usr/local/bin/ is already in $PATH. Provide the config file as a command 
line option.

e.g
"/usr/local/bin/vmm_workload_auto --config=/usr/local/lib/python3.7/site-packages/etc/vmm_workload_auto/conf.yml"

[Please note the above config is preceded by two dashes.]

Step 7:
Finally, to uninstall the script, do "pip3 uninstall vmm-workload-auto"



Option 2:
----------

This is for users who wants the script to do much of the work and not worry
about the nitty gritty details of pip install or setup.

Step 1:
Goto https://test.pypi.org/project/vmm-workload-auto/ and 
download the latest .tar.gz file

Step 2:
Untar it. e.g.

tar -xvf vmm_workload_auto-0.1.0.tar.gz.

Step 3:
Modify the config/conf.yml and config/sample.csv according to your environment.

Step 4:
Run the setup script as "source setup.sh"

Step 5:
This script will initially prompt the user to edit the conf.yaml and .csv file.
Once that is done, the script will prompt the user for proxy and other details.
Once all that is done, the script will install the python packages and start
the script.

Post Install:
-------------

After running the script, go to the DCNM Network page and look for the Network
attachments done by the script. Review the configs and deploy it, if auto_deploy
is set to false.
