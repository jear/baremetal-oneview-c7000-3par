![HPE Logo](https://github.com/hpe-design/logos/blob/master/HPE%20Element%20-%20PNG/hpe-element-color.png?raw=true)

Join the discussion: [![Slack Status](https://dashif-slack.azurewebsites.net/badge.svg)](https://hpe-incubationgroup.slack.com)

Create BareMetal servers like VMs with  OneView API and 3par API
================================================================


Synergy version
---------------

    https://github.com/tdovan/telco-caas-on-baremetal
    https://github.com/tdovan/OBS-NGP-POC


Prerequisites
-------------

#### Prepare your virtualenv Python (here with Miniconda3)

    # Install Miniconda3 then :
    conda create --name baremetal_python36 python=3.6
    conda activate baremetal_python36
    
#### Get 3PAR and OneView python Ansible libraries
    pip install hpeOneView==5.5.0
    pip install hpe3par-sdk
    pip install ansible==2.9.17

    mkdir -p /etc/ansible-hpe/
    cd /etc/ansible-hpe/
    git clone https://github.com/HewlettPackard/oneview-ansible.git

    cd /etc/ansible-hpe/
    git clone https://github.com/HewlettPackard/hpe3par_ansible_module

#### Add Ansible modules/utils in  /etc/ansible/ansible.cfg  in [default] section
    [defaults]
    library         = /etc/ansible-hpe/oneview-ansible/library:/etc/ansible-hpe/hpe3par_ansible_module
    module_utils    = /etc/ansible-hpe/oneview-ansible/library/module_utils:/etc/ansible-hpe/hpe3par_ansible_module/Modules/:/root/miniconda3/envs/baremetal_python36/lib/python3.6/site-packages/


#### Get OneView API version

    ansible-playbook -i inventory/localhost playbooks/oneview_version_facts.yml


Quick start 
-------------

#### Obviously you have manually installed an OS in a volume, this is your Golden volume which will be cloned by 3PAR and booted from SAN by Oneview server profiles
#### Clone 3PAR Boot Volume and Create OneView Server Profile based on a template, so giving WWPN from Oneview profile to 3PAR 
    ansible-playbook -i inventory/baremetal-hosts playbooks/assign-bluedata-worker-Gen8-bfs-0-addon-3par-disks-large.yml -e ansible_python_interpreter=/root/miniconda3/envs/baremetal_python36/bin/python  --limit workers 


#### Get IP assign to the Server and add it to the DNS
    yum install perl-libwww-perl

    wget https://download-ib01.fedoraproject.org/pub/epel/7/x86_64/Packages/a/arp-scan-1.9.7-3.el7.x86_64.rpm
    rpm -Uvh arp-scan-1.9.7-3.el7.x86_64.rpm


    ansible-playbook -i inventory/baremetal-hosts playbooks/get_dhcp_ip_arp-scan-serial.yml -e ansible_python_interpreter=/root/miniconda3/envs/baremetal_python36/bin/python --limit workers 


#### Configure hostname
    ANSIBLE_HOST_KEY_CHECKING=false ansible-playbook -i inventory/baremetal-hosts playbooks/set-hosts.yml --limit masters,workers



#### delete cluster, cleanup Oneview, 3PAR and DNS
    ansible-playbook -i inventory/$INVENTORY -e ansible_python_interpreter=/root/miniconda3/envs/baremetal_python36/bin/python playbooks/ov_server_delete.yml
    ansible-playbook -i inventory/$INVENTORY -e ansible_python_interpreter=/root/miniconda3/envs/baremetal_python36/bin/python playbooks/clean-3par-worker.yml
    ansible-playbook -i inventory/$INVENTORY -e ansible_python_interpreter=/root/miniconda3/envs/baremetal_python36/bin/python playbooks/manage_dns_rm.yml








