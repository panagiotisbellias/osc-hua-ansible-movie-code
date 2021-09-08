# ansible-movie-code
Ansible Movie Code, an ansible project in the context of HUA DIT course 'Basic DevOps Concepts and Tools'

Content is coming very soon!

## Connectivity

* Create an inventory file (e.g. hosts.yml) that holds the remote hosts that ansible will handle. The file will have entries look like:
```vim
---
# that is a group entry
<group-name>:
    hosts:
        # this is a host entry
        <vm-name>:
            ansible_host: <DNS Name of VM or public IP>
            ansible_port: 22 # because 22 port is for SSH Connection
            ansible_ssh_user: <username in VM>

# in case you have already ssh connection in the VM using ~/.ssh/config no more info are needed like:
<group-name>:
    hosts:
        <vm-name> # The VM name must be the same as the HostName in the ~/.ssh/config
```
For testing you can use either [vagrant](https://www.vagrantup.com/) using [VirtualBox](https://www.virtualbox.org/) making locally VMs or create some extra VMs in some cloud provider like Gcloud, Azure or Okeanos.

* Run
```bash
ansible -m ping all
```
to check connectivity with declared hosts