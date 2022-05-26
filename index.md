## hpc4you_toolkit
The _hpc4you toolkit_ is a simple but robust toolkit to setup a parallel computing cluster with slurm as workload manager by only run copy and paste. 

Currently, the _hpc4you_toolkit_ supports: 
1. RHEL7 and RHEL8 and their compatible operating systems, such as CentOS 7.x, 8.x, RockyLinux 8.x.
2. Ubuntu 20.04/22.04 and their compatible operating systems. 

### Prerequisites
1. All servers running the same version of Linux, 
2. All servers are interconnected through a LAN, 1000 Mbps, or 10 Gbps ethernets, or may be InfiniBand. 
3. The yum/dnf/apt is working on all servers. 
4. All servers have a unique hostname. 
5. All servers share the same root password. 
6. The hostname of login node should only be **master** or the alias must be **master** if you have set the hostname to something else. 
7. The hostname of all computing nodes should only be **nodeXX** or the alias must be **nodeXX** if you have set the hostname to something else. 

### Get hpc4you_toolkit
Choose one machine as the management/master/login node, and run the following code, 
```
bash <(curl -k -Ss https://raw.githubusercontent.com/hpc4you/hpc/main/getInfo.sh)
```
Follow the on-screen prompts carefully. 

Remeber to copy and paste the blue lines into the body of you mail, also attach the **XXX.dat** file with the Email.

### Declare servers
On the login node, edit file /etc/hosts. The content of the file should be similar to the following. 
```
# the IP and real hostname
192.168.1.100 server0
192.168.1.101 server1
192.168.1.102 server2
192.168.1.112 server12

### for the cluster
192.168.1.100 master
192.168.1.101 node1
192.168.1.102 node2
192.168.1.112 node12
```
In this example, 
1. **server0** is the output of `hostname` on the login node. 
2. **server12**, is the output of `hostname` on the computing node which can be accessed via IP 192.168.1.112. 
3. The hostname of all compute nodes must be prefixed with **node**, and the numbers (XX) after **node** do not have to be consecutive. 

### Run hpc4you_toolkit to setup Cluster
Put the file code and the package hpc4you_toolkit-XXX.tgz into the same folder on the login node. 

Within the above-mentioned folder, open terminal, run: 
```
source code
```
Follow the on-screen prompts carefully. 

You will be asked to copy and paste `./step1.sh` into the current terminal. Please just do it. 

Please wait a while (The waiting time depends on the network bandwidth), 
you would read on the screen, 

> The default root password please ...

Please type the root password, then press enter. 

For the rest of the questions, just press the Enter key. 

Nothing to do but wait ...

Copy and paste the green line, then press the Enter key, and wait ...

All servers will automatically reboot at least twice, and then the slurm scheduling cluster is readly for you. 

In particular, the restart operation of all compute nodes will take about 1 minute after the master node. You do not need to do anything, just wait. 

### Change root password
1. Disable internet connection on all slave/computing nodes. 
2. On login/master node, run `passwd` to change the root password. 

### User Admin
#### Add user to this cluster
On login node, run 
```
useradd_hpc tom
```
this will add user **tom** to the default group **users**. 

```
useradd_hpc chem tom
```
this will add user **tom** to the group **chem**. 

Caution, you will be prompted to set a password for the new user. You will need to enter the password twice. But the screen does not show any asterisks. 

If you have slurm accounting enabled, you should also run, 
```
sacctmgr add user tom Account=hpc4you
```
in which, you will give user **tom** the default account **hpc4you**. Refer slurm manual for more details. 

#### Delete user from this cluster
On the login/master node, run, 
```
userdel_hpc tom
````
in this case, user **tom** is to be deleted. 

### Poweroff the cluster
On the login/master node, run, 
```
poweroff_hpc
````

### Reboot the cluster
On the login/master node, run, 
```
reboot_hpc
````

### Poweron the cluster
Power on the master node and all switches first, and then power on all computing nodes. 

### Add new computhing node
1. Clone any of the current compute node OS disks
2. Boot the new server with the cloned OS disk, edite hostname, configre network. 
3. Add the IP and hostname of the new server to /etc/hosts on the master/login node. 
4. On the master/login node, run `setup_hpc ‐‐sync_file /etc/hosts`
5. On the new server, run `slurmd ‐C | head -n 1`, please copy the output contents. 
6. On the master/login node, paste the coppied contents into last line of file /etc/slurm/slurm.conf. 
7. On the master/login node, run `setup_hpc ‐‐sync_do 'systemctl restart slurmd'; systemctl restart slurmctld`. 
