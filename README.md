# implement openstack 

## Host machines' requirements
The host machines must satisfy the following minimum requirements:

1. 2 network interfaces

2. 8GB main memory

3. 40GB disk space

## Install dependencies
Typically commands that use the system package manager in this section must be run with root privileges.

It is generally recommended to use a virtual environment to install Kolla Ansible and its dependencies, to avoid conflicts with the system site packages. Note that this is independent from the use of a virtual environment for remote execution, which is described in [Virtual Environments](https://docs.openstack.org/kolla-ansible/latest/user/virtual-environments.html).   
1. For Debian or Ubuntu, update the package index.

```
sudo apt update
```
2. Install Python build dependencies:
For Debian or Ubuntu, run:  
```
sudo apt install git python3-dev libffi-dev gcc libssl-dev -y 
```
## Install dependencies for the virtual environment

1. Install the virtual environment dependencies.

For Debian or Ubuntu, run:
```
sudo apt install python3-venv -y 
```

2. Create a virtual environment and activate it:

```
python3 -m venv ~/venv
source ~/venv/bin/activate 
```

The virtual environment should be activated before running any commands that depend on packages installed in it.

3. Ensure the latest version of pip is installed:

```
pip install -U pip
```
Install Ansible. Kolla Ansible requires at least Ansible 8 (or ansible-core 2.15) and supports up to 9 (or ansible-core 2.16).
```
pip install 'ansible-core>=2.15,<2.16.99'
```
## Install Kolla-ansible

1. Install kolla-ansible and its dependencies using pip.
```
pip install git+https://github.com/ArmanTaheriGhaleTaki/kolla-ansible@master
```  
2. Create the /etc/kolla directory.
```
sudo mkdir -p /etc/kolla
sudo chown $USER:$USER /etc/kolla
```
3. Copy globals.yml and passwords.yml to /etc/kolla directory.
```
cp -r ~/venv/share/kolla-ansible/etc_examples/kolla/* /etc/kolla
```
Copy  **multinode** inventory file to the current directory.
```
cp ~/venv/share/kolla-ansible/ansible/inventory/multinode .
```


## Install Ansible Galaxy requirements

Install Ansible Galaxy dependencies:
```
kolla-ansible install-deps
```
Prepare initial configuration

### Inventory
The next step is to prepare our inventory file. An inventory is an Ansible file where we specify hosts and the groups that they belong to. We can use this to define node roles and access credentials.

Kolla Ansible comes with **all-in-one** and **multinode** example inventory files. The difference between them is that the former is ready for deploying single node OpenStack on localhost. In this guide we will show the **multinode** installation.

### Kolla passwords
Passwords used in our deployment are stored in **/etc/kolla/passwords.yml** file. All passwords are blank in this file and have to be filled either manually or by running random password generator:
```
kolla-genpwd
```
### Kolla globals.yml

**globals.yml** is the main configuration file for Kolla Ansible and per default stored in /etc/kolla/globals.yml file. There are a few options that are required to deploy Kolla Ansible:

 - Networking

Kolla Ansible requires a few networking options to be set. We need to set network interfaces used by OpenStack.

First interface to set is “network_interface”. This is the default interface for multiple management-type networks.

    network_interface: "eth0"

Second interface required is dedicated for Neutron external (or public) networks, can be vlan or flat, depends on how the networks are created. This interface should be active without IP address. If not, instances won’t be able to access to the external networks.

    neutron_external_interface: "eth1"

To learn more about network configuration, refer [Network overview](https://docs.openstack.org/kolla-ansible/latest/admin/production-architecture-guide.html#network-configuration).

Next we need to provide floating IP for management traffic. This IP will be managed by keepalived to provide high availability, and should be set to be not used address in management network that is connected to our **network_interface**. If you use an existing OpenStack installation for your deployment, make sure the IP is allowed in the configuration of your VM.

    kolla_internal_vip_address: "10.1.0.250"

## Deployment
After configuration is set, we can proceed to the deployment phase. First we need to setup basic host-level dependencies, like docker.

Kolla Ansible provides a playbook that will install all required services in the correct versions.

first of all we need to identify hosts in **/etc/hosts**
```
10.10.10.1 control01
10.10.10.2 compute01
10.10.10.3 monitoring01
```
and add the following config  to ~/.ansible.cfg
```
[defaults]
host_key_checking = False
```
1. Bootstrap servers with kolla deploy dependencies:

    ```
    kolla-ansible -i ./multinode bootstrap-servers
    ```

2. Do pre-deployment checks for hosts:

   ```
    kolla-ansible -i ./multinode prechecks
    ```
3. Finally proceed to actual OpenStack deployment:

    ```
    kolla-ansible -i ./multinode deploy
    ```

## Using OpenStack

1. Install the OpenStack CLI client:

```
 pip install python-openstackclient -c https://releases.openstack.org/constraints/upper/master
```
2. OpenStack requires a  **clouds.yaml**  file where credentials for the admin user are set. To generate this file:
```
kolla-ansible post-deploy
```
3. Depending on how you installed Kolla Ansible, there is a script that will create example networks, images, and so on.
```
~/venv/share/kolla-ansible/init-runonce
```
at the end you can access the password of admin in  openstack panel in this file **/etc/kolla/clouds.yaml**
