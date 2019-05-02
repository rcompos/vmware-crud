# VMware-CRUD

Create, read, update and destroy VMware infrastructure using Ansible.

## Description ##


Create, read, update and destroy VMware infrastructure using Ansible.

This project provides a very simple deployment process for managing VMware infrastructure.

## Requirements ##

General requirements:

* __Control Node:__ Where the Ansible commands are run (i.e. laptop or jump host).  MacOS High Sierra, RedHat 7, CentOS 7 or Ubuntu Xenial all tested. Python 2 is a requirement.

## Prepare Control Node ##

Prepare __control node__ where management tools are installed.  A laptop or desktop computer will be sufficient.  A jump host is fine too.


1. __Install Packages__ 

    a. Install Python 2 as requirement of Ansible.  

    _MacOS_: `$ brew install -vd python@2`  
    _RedHat 7_ or _CentOS 7_: `Python 2.7.5 installed by default`  
    _Ubuntu_: `$ apt install python2.7 python-pip`  

    b. Use Python package manager pip2 to install required packages on __control node__ including Ansible v2.4 (or newer) and python-netaddr.  

    `$ sudo -H pip2 install --updgrade pip`  
    `$ sudo -H pip2 install ansible pyvmomi`  

    c. _Debian_ or _Ubuntu_ control node also need:  

    `$ sudo apt-get install sshpass`. 

2. __Clone Repo__

    Clone vmware-crud repository.  

    `$ cd; git clone https://bitbucket.org/solidfire/vmware-crud.git`  

After a control node has been prepared with Python and Ansible, we are ready to create infrastructure.


## AdHoc ##

An ad-hoc VMware virtual machine can be rapidly deployed with the following steps.  See further sections for details of each step.

1. Edit file  
   Update configuration in adhoc ansible playbooks.  Provide VMware credentials and virtual machine info.
   
   \*\*\* For each adhoc playbook you want to use, you must update it with your own VMware credentials and VM specification.  \*\*\*
   
        $ cd ~/vmware-crud
        $ vi adhoc/vm-create.yml
        $ vi adhoc/vm-facts.yml
        $ vi adhoc/vm-create-range.yml


2. Create VM
   
        $ ansible-playbook adhoc/vm-create.yml -e vm_name='ci-myname-0' 

3. Get VM facts
   
        $ ansible-playbook adhoc/vm-facts.yml -e vm_name='ci-myname-0' 

4. Poweroff VM
   
        $ ansible-playbook adhoc/vm-create.yml -e vm_name='ci-myname-0' -e vm_state=poweredoff

5. Destroy VM
   
        $ ansible-playbook adhoc/vm-create.yml -e vm_name='ci-myname-0' -e vm_state=absent

6. Create a range of VM's  
   Range of VMs will be created:  ci-myname-1, ci-myname-2, ci-myname-3
   
        $ ansible-playbook adhoc/vm-create-range.yml -e vm_name='ci-myname-' -e end=3

7. Get VM facts for a range of VM's
   
        $ ansible-playbook adhoc/vm-facts-range.yml -e vm_name_base='ci-myname-' -e end=3

8. Poweroff a range of VM's
   
        $ ansible-playbook adhoc/vm-create-range.yml -e vm_name='ci-myname-' -e end=3 -e vm_state=poweredoff

9. Destroy a range of VM's
   
        $ ansible-playbook adhoc/vm-create-range.yml -e vm_name='ci-myname-' -e end=3 -e vm_state=absent


---

## Roles and Group Variables ##

An well organized VM infrastructure can be maintained as follows.  


1. Create host group 

  		Create a host group by copying an existing file.
  		
        $ cd ~/vmware-crud/hosts
        $ cp -a ci-k8s-ubu-base mygroup-myvms
        
        Change line:
       		[ci-k8s-ubu-base:children]
       	to:
       		[mygroup-myvms:children]

2. Create group vars
   
        $ cd group_vars
        $ cp -a ci-k8s-ubu-base mygroup-myvms
        $ cd mygroup-myvms
        
        Edit the vars.yml file to provide valid credentials and machine parameters.
        
3. Create encrypted file for VMware password

        $ rm vault.yml
        $ ansible-vault create vault.yml  
        
        Create a password when prompted.   Type or paste the following into the editor.  Substitute your actual Vcenter password.  
        
        vault_vcenter_password: <your_password>

4. Create VM
   
        $ cd ~/vmware-crud
        $ ansible-playbook -i hosts/mygroup-myvms playbooks/vm-create.yml -e vm_name=mygroup-myname --ask-vault-pass 
        
5. Get VM facts
   
        $ cd ~/vmware-crud
        $ ansible-playbook -i hosts/mygroup-myvms playbooks/vm-facts.yml -e vm_name=mygroup-myname --ask-vault-pass 

