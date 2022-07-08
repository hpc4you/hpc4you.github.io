## hpc4you_toolkit
The _hpc4you toolkit_ is a simple but robust toolkit written by a computational chemist to set up a parallel computing cluster for scientific research. 

No computer skills or Linux knowledge are needed. Only **copy and paste the cmd from the screen then press the Enter key**.

If you have some knowledge of how parallel computing clusters work, and how to administrate Linux in the cmd line and configure Linux networking, then you can try out the [OpenHPC solution](https://github.com/openhpc/ohpc). 

Currently, the _hpc4you_toolkit_ supports: 
1. RHEL7, RHEL8, RHEL9 and their compatible operating systems, such as CentOS 7.x, 8.x, RockyLinux 8.x, AlmaLinux 8.x, AlmaLinux 9.x, CentOS Stream 9. 
2. Ubuntu 20.04/22.04 and their compatible operating systems. 
3. The maximum number of computing nodes depends on the capacity of your switch.

## Quick Start
1. Get the package. 
2. Edit **/etc/hosts** file. 
3. Upload the file **code** and the **package**. 
4. Run `source code`. 

All subsequent 5 commands will be automatically displayed in green. 

All you need to do is to copy and paste the green text according to the on-screen instructions. 

## Prerequisites
1. All servers running the same version of Linux. 
2. All servers are interconnected through a LAN, 1000 Mbps, or 10 Gbps ethernet, or may be InfiniBand. 
3. The yum/dnf/apt is working on all servers. 
4. All servers have a unique hostname. 
5. All servers share the same root password. 
6. The hostname of login node should only be **master** or the alias must be **master** if you have set the hostname to something else. 
7. The hostname of all computing nodes should only be **nodeXX** or the alias must be **nodeXX** if you have set the hostname to something else. 

## Get hpc4you_toolkit
Choose one machine as the management/master/login node, and run the following code, 
```
bash <(curl -k -Ss https://raw.githubusercontent.com/hpc4you/hpc/main/getInfo.sh)
```
Follow the on-screen prompts carefully. 

Remeber to copy and paste the **blue lines** into the body of your Email, please also attach the **XXX.dat** file with the Email.

## Cluster Architectures
### High-Performance Cluster
![High-Performance Cluster](/docs/multi-node-HPC.jpg)

### Intra-node Local-I/O Cluster
![Intra-node Local-I/O Cluster](/docs/intra-node-local-IO-Cluster.jpg)

### mini-HPC
![mini-HPC](/docs/mini-HPC.jpg)

## Declare Servers
On the login node, edit file **/etc/hosts**.  

### Scenario A
If you have configured the hostname for master/login node to **master**, and configured the hostname of computing/slave nodes to **nodeXX**, 
the content of the file **/etc/hosts** should be similar to the following.
```
### the IP and the corresponding real hostname 
192.168.1.100 master
192.168.1.101 node1
192.168.1.102 node2
192.168.1.112 node12
```

### Scenario B
If the hostname of the login/master node **is not** master, **and/or** the hostname of any computing/slave node **is not** nodeXX, 
the content of the file **/etc/hosts** should be similar to the following.
```
# the IP and real hostname
192.168.1.100 server0
192.168.1.101 server1
192.168.1.102 server2
192.168.1.112 server12

### the IP and the corresponding alias hostname 
192.168.1.100 master
192.168.1.101 node1
192.168.1.102 node2
192.168.1.112 node12
```
In this example, 
1. **server0** is the output of `hostname` on the login node. 
2. **server12**, is the output of `hostname` on the computing node which can be accessed via IP 192.168.1.112. 
3. The hostname of all compute nodes must be prefixed with **node**, and the the suffix numbers ('0', '1', '2', and '12' in current case) do not have to be consecutive. 
4. You can use `nmtui` to set hostname and configure the IP address. 

## Run hpc4you_toolkit
Put the file **code** and the package **hpc4you_toolkit-XXX.tgz** into the same folder on the login node. 

Within the above-mentioned folder, open terminal, run: 
```
source code
```
Follow the on-screen prompts carefully. 

You will be asked to copy and paste `./step1.sh` into the current terminal. Please just do it. 

Please wait a while (The waiting time depends on the network bandwidth), 
you would read on the screen, 

> Default root password for all slave nodes

Please type the root password, then press the Enter key. 

Nothing to do but wait ...

Copy and paste the green line, then press the Enter key, and wait ...

All servers will automatically reboot at least twice, and then the slurm scheduling cluster is ready for you. 

In particular, the restart operation of all compute nodes will be 1 minute lag behind the master node. 

Nothing else to do, just wait. 

By default, the **/opt** and **/home** from the master/login node are shared among all slave nodes. 
It is possible to add new share path by, 
1. Edit **/etc/exports** file on master/login node, 
2. Use 'setup_hpc --sync_do' to update the **/etc/fstab** file on all slave nodes. 

## User Management
### Add user
On login node, run 
```
useradd_hpc tom
```
this will add user **tom** to the default group **users**. 

```
useradd_hpc chem tom
```
this will add user **tom** to the group **chem**. 

Caution, you will be prompted to set a password for the new user. You will need to enter the password twice. 
But the screen does not show any asterisks. 

If you have slurm accounting enabled, you should also run, 
```
sacctmgr add user tom Account=hpc4you
```
in which, you will give user **tom** the default account **hpc4you**. Refer slurm manual for more details. 

### Delete user
On the login/master node, run, 
```
userdel_hpc tom
````
in this case, user **tom** is to be deleted. 

## Cluster Management
### Power on
Power on the master node and all switches first, and then power on all computing nodes. 

### Power off
On the login/master node, run, `poweroff_hpc`.

### Reboot
On the login/master node, run, `reboot_hpc`.

### Add new node
1. Clone the OS disk of any compute node. Use this hardware tool, [Offline Cloning Tool](https://item.jd.com/100014988528.html). 
2. Boot the new server with the cloned disk, modify hostname and configure network with command `nmtui`. 
3. Add the IP and hostname of the new server to **/etc/hosts** on the master/login node. 
4. On the master/login node, run `setup_hpc ‐‐sync_file /etc/hosts`. 
5. On the new server, run `slurmd ‐C | head -n 1`, please copy the output. 
6. On the master/login node, paste the coppied contents into last line of file **/etc/slurm/slurm.conf**. 
7. On the master/login node, run `setup_hpc ‐‐sync_do 'systemctl restart slurmd'; systemctl restart slurmctld`. 
8. Done. 

## Better Security
1. Disable internet connection on all slave/computing nodes. 
2. On login/master node, run `passwd` to change the root password, then run `setup_hpc --sync_user`. 
3. Disable password login for root user, only key authentication is allowed.
4. Or use ssh ProxyJump server or even hardware firewall. 
5. Or run `enhance_security.sh` to apply awesome security hardening configurations automatically. 

## Usage of setup_hpc
### sync file
On the login/master node, run
```
setup_hpc --sync_file /full/path/to/file
```
For example, `setup_hpc --sync_file /etc/hosts` will sync the hosts file on master node to all slave nodes. 

### run cmd
On the login/master node, run
```
setup_hpc --sync_do cmd
```
For example, 
```
setup_hpc --sync_do uptime
```
will print the uptime info for all slave nodes. 

```
setup_hpc --sync_do 'systemctl restart slurmd; utpdate -u 3.cn.pool.ntp.org'
```
will restart slurm client and sync time on all slave nodes. 

## hpc4you_toolkit solo
_hpc4you_toolkit solo_, is also available, which can deploy slurm to workstations by only run `source code`. 

## Pricing 
### *hpc4you_toolkit* features 

FEATURES | Basic | Adv | Pro
--- | --- | --- | ---
CPU Scheduling  | ✅ | ✅ | ✅ 
GPU Scheduling[^1]  | ✅   | ✅ | ✅
Job Log         | ❌ | ✅ | ✅
User Control[^2] | ❌  | ❌ | ✅
Monitoring Historical         | ❌   | ❌ | ✅
Monitoring Realtime           | ❌   | ❌ | ✅
Security & OS Optimize[^3]    | ❌   | ❌ | ✅
Pricing | 399 USD | Email ask@hpc4you.top | Email ask@hpc4you.top

### *hpc4you_toolkit solo* features

FEATURES | Basic | Adv 
--- | --- | --- 
CPU Scheduling  | ✅   | ✅ 
GPU Scheduling  | ✅   | ✅ 
Job Log         | ❌   | ✅
Pricing | 99 USD | 149 USD 

[^1]: SLURM natively supports GPU scheduling. However, the auto-detection mode often fails and needs to be configured manually once. If you can't do it, I can be a helping hand, but please pay the fee. 
[^2]: To prevent users from sshing into nodes that they do not have a running job on, and to track the ssh connection and any other spawned processes for accounting and to ensure complete job cleanup when the job is completed. [More info](https://slurm.schedmd.com/pam_slurm_adopt.html) 
[^3]: Reference, https://github.com/decalage2/awesome-security-hardening 

### Discount 
1. A huge discount of up to **75% OFF** is available if you choose the self-service mode.
2. The **self-service mode** refers to 'Read the manual, watch the video tutorials, and do it all by yourself. No support is available'.  
3. 中文用户🔗[点击这里](https://gitee.com/hpc4you/hpc). 

## Demo
### Live Demo
Real-time demo on how to set up HPC for scientific research with hpc4you_toolkit. Only copy and paste. No computer skills and Linux knowledge are required. 
1. All modules, 🔗[live-demo with Ubuntu Jammy](https://youtu.be/2nUUDSdy1Io)
2. All modules, 🔗[live-demo with Ubuntu Focal](https://youtu.be/ZRmjWn_sCWk)
3. All modules, 🔗[live-demo with CentOS 7.9](https://youtu.be/OhLdV_wd1n4)
4. All modules, 🔗[live-demo with RockyLinux 8](https://youtu.be/m0AhepMaoJA)

### Short Videos
To those who have no patience to watch real-time videos, 
1. copy+paste+Enter -> cluster ready, demonstration of **hpc4you_toolkit** 🔗[BV1GY411w7ZV](https://www.bilibili.com/video/BV1GY411w7ZV)
2. Deploy SLURM by only run `source code`, demonstration of **hpc4you_toolkit solo** 🔗[BV1Gg411R7tt](https://www.bilibili.com/video/BV1Gg411R7tt)

## Play with Linux
1. How to add new IP and hostname informations into file **/etc/hosts**? 🔗[bv19A4y1U7uX](https://www.bilibili.com/video/bv19A4y1U7uX)
2. How to upload file(s) to remote Linux server? 🔗[BV1fJ411n7uV](https://www.bilibili.com/video/BV1fJ411n7uV)
3. Change hostname 🔗[BV1yP4y1M77H](https://www.bilibili.com/video/BV1yP4y1M77H)
4. Configure network 🔗[BV1gP4y1u7Aw](https://www.bilibili.com/video/BV1gP4y1u7Aw)

## Terminology
In the field of parallel computing clusters, we refer to a server as a node. 

Usually, in small-scale clusters, the login server, storage node, and the contollor server can be merged into a single server, which is called as the login/master node. Accordingly, we refer to all compute nodes, as the slave nodes. 
