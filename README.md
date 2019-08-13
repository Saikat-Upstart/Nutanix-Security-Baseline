# Nutanix Security Baseline

A role to set and/or reset security settings on nutanix cvms for operation in a high governance area

Required Variables
ansible_user: nutanix           # An account with permissions to provision on the cluster
ansible_password: secret      # The password to your account, Note, you should not store this in the clear, use Ansible vault

ansible.cfg Changes
transfer_method = scp        # set transfer_method=scp in the ansible config file as the nutanix nodes have sftp turned off.

# tags
There are 5 tags in the playbook so you can run all or some of the settings, if you run 
ansible-playbook ntnx_sec.yml -t cvm,DODbanner,firewall 

This will set the firewall and secsettings on a nutanix cluster that is running ESXi, if you have an ahv cluster you do not need any tags.

cvm - Run CVM tasks

ahv - Run ahv related tasks for the hypervisor

syslog - syslog settings

firewall - set and apply firewall settings on cvms

DODbanner - set DODBanner on cvms


# Invetory Located in the hosts folder
This is where you will list the individual cvms for each node. I would suggest making the your cluster name the group name, for Example
[palo.nutanix.cluster]
palo-ntnx-cvm-a
palo-ntnx-cvm-b
palo-ntnx-cvm-c
palo-ntnx-cvm-d

# group_vars 
Located in the group_vars folder

Group vars will tell which firewall template to use if you have multiple cluster to manage. The group var must match group name for it to pick up the right firewall template file

# templates 
Located under roles/nutanix_baseline/templates

example._salt.conf.h2  # This is where your firewall rules replace example with your site name form the hosts file (inventory)
dodbanner.conf.j2  # The is the template for the banner that will be copied to each node in each cluster

# Security Settings 
This is what the settings will be after the playbook is run, if the setting is set it will skip it

ncli  cluster get-cvm-security-config

    Enable Aide               : true
    Enable Core               : false
    Enable High Strength P... : true
    Enable Banner             : true
    Enable SNMPv3 Only        : true
    Schedule                  : Daily

# Tasks
main.yml - main task List - Tasks can be comment out if not needed
 - checksecsettings.yml # checks to see what security settings are set
 - dodbanner.yml # imports custom DODbanner to all cvms
 - cvmsecsettings.yml # sets cvm security settings
 - ahvsecsettings.yml # sets the hypervisor security settings on the cvm (AHV only)
 - firewall.yml # copy firewall rules to all cvms
 - syslog.yml # Set the syslog
 - apply.yml apply firewall rules on all cvms
 
 # Example commands 
 This will run the playbook on a cluster that does not have AHV hypervisor
 
 ansible-playbook ntnx_sec.yml -l nutanix_cluster_name -t cvm,security,syslog,firewall  
 
 This will run the playbook on an AHV cluster
  ansible-playbook ntnx_sec.yml -l nutanix_cluster_name 
