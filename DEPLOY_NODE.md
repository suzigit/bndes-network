# Deploy a LACChain-BNDES node

* Below you will find instructions for the deployment of nodes using Ansible. This implies that it will be executed from a local machine on a remote server. The local machine and the remote server will communicate via ssh.

* The installation with ansible provided is compatible with **Ubuntu 18.04** and **Centos7**. If you want to deploy your node in a different operative system, you can go to the [documentation for Generic Onboarding](https://github.com/lacchain/bndes-network/blob/master/GENERIC_ONBOARDING.md).



## Minimum System Requirements

Recommended hardware features for the nodes in the test-net:

* **CPU**: 2 cores

* **RAM Memory**: 4 GB

* **Hard Disk**: 30 GB SSD

* **Operating System**: Ubuntu 16.04, Ubuntu 18.04, Centos7, always 64 bits

## Pre-requisites

### Install Ansible ###

For this installation we will use Ansible. It is necessary to install Ansible on a **local machine** that will perform the installation of the node on a **remote machine**.

Following the instructions to [install ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) in your local machine.

```shell
$ sudo apt-get update
$ sudo apt-get install software-properties-common
$ sudo apt-add-repository ppa:ansible/ansible
$ sudo apt-get update
$ sudo apt-get install ansible
```

### Clone Repository ####

To configure and install Pantheon and Orion, you must clone this git repository in your **local machine**.

```shell
$ git clone https://github.com/lacchain/bndes-network
$ cd bndes-network/
```

### Obtain SSH access to your remote machine ###

Make sure you have SSH access to the node you're setting up. This step will vary depending on your context (physical machine, cloud provider, etc.). This document assumes that you are able to log into your remote machine using the following command: `ssh remote_user@remote_host`.

### Prepare installation of Oracle Java 11 ###

* It is a requisite for Pantheon to install Java 11 in its LATEST version. Since Java cannot be downloaded directly, you must follow the next steps to install it:
	1.  Download the correspondent java tar.gz(for ubuntu) or java .rpm(for centos) file from https://www.oracle.com/technetwork/java/javase/downloads/jdk11-downloads-5066655.html. Oracle will request that you create an account before downloading the package.
	2.  Once the file is downloaded, send the Oracle java11 package to your remote machine by using SCP Linux command:
		```shell
		$ scp /your/local/path/to/downloaded/jdk-11.0.x_linux-x64_bin.tar.gz remote_user@remote_host:
		```
		If your VM is Centos7 then use:
		```shell
		$ scp /your/local/path/to/downloaded/jdk-11.0.x_linux-x64_bin.rpm remote_user@remote_host: 
		```
	3.  Log into your remote machine by using something like this:
		```shell
		$ ssh remote_user@remote_host
		```
	4.  On the remote machine, for Ubuntu VMs: Create the JDK folder and move the JDK to it:
		```shell
		$ sudo mkdir -p /var/cache/oracle-jdk11-installer-local
		$ sudo cp jdk-11.0.x_linux-x64_bin.tar.gz /var/cache/oracle-jdk11-installer-local/
		```
		If the VM is Centos7 then execute:
		```shell
		$ sudo rm -rf /usr/local/src/jdk*linux-x64_bin.rpm
		$ sudo cp jdk-11.0.x_linux-x64_bin.rpm  /usr/local/src
		```
	5.  Before leaving, it's a good idea to run an APT update:
		```shell
		$ sudo apt update
		```
		Or in Centos7 OS:
		```shell
		$ sudo yum update
		```

## Pantheon + Orion Installation ##

### Preparing installation of a new node ###

* There are three types of nodes (Bootnode / Validator / Writer) that can be created in the Pantheon network at this moment.

* After cloning the repository on the **local machine**, enter it and create a copy of the `inventory.example` file as `inventory`. Edit that file to add a line for the remote server where you are creating the new node. You can do it with a graphical tool or inside the shell:

    ```shell
    $ cd lacchain/
    $ cp inventory.example inventory
    $ vi inventory
    [writer] # or [validators] or [bootnodes] depending on its role
    192.168.10.72 node_ip=your.public.node.ip password=abc node_name=my_node_name node_email=your@email
    ```

Consider the following points:
- Place the new line in the section corresponding to your node's role: `[writer]`, `[validators]` or `[bootnodes]`.
- The first element on the new line is the IP or hostname where you can reach your remote machine from your local machine.
- The value of `password` is the password that will be used to set up Orion, for private transactions.
- The value of `node_name` is the name you want for your node in the network monitoring tool.
- The value of `node_email` is the email address you want to register for your node in the network monitoring tool. It's a good idea to provide the e-mail of the technical contact identified or to be identified in the registration form as part of the on-boarding process.

### Deploying the new node ###

* To deploy a **boot node** execute the following command in your **local machine**. If needed, don't forget to set the private key with option `--private-key` and the remote user with option `-u`:

	```shell
	$ ansible-playbook -i inventory --private-key=~/.ssh/id_rsa -u remote_user site-lacchain-bootnode.yml
	```

* To deploy a **validator node** execute the following command in your **local machine**. If needed, don't forget to set the private key with option `--private-key` and the remote user with option `-u`:

	```shell
	$ ansible-playbook -i inventory --private-key=~/.ssh/id_rsa -u remote_user site-lacchain-validator.yml
	```

* To deploy a **writer node** execute the following command in your **local machine**. If needed, don't forget to set the private key with option `--private-key` and the remote user with option `-u`:

	```shell
	$ ansible-playbook -i inventory --private-key=~/.ssh/id_rsa -u remote_user site-lacchain-writer.yml
	```

* At the end of the installation, if everything worked a PANTHEON service will be created in the case of a **validator node** managed by Systemctl with **stopped** status.

Don't forget to write down your node's "enode" from the log by locating the line that looks like this:
```
TASK [lacchain-validator-node : print enode key] ***********************************************
ok: [x.x.x.x] => {
    "msg": "enode://cb24877f329e0e3fff6c7d7b88d601b698a9df6efbe1d91ce77130f065342b523418b38cb3c92ea3bcca15344e68c7d85a696eb9f8c0152c51b9b7b74729064e@a.b.c.d:60606"
}
```

* If everything worked, an ORION service and a PANTHEON service managed by Systemctl will be created with **stopped** status.
* After installation has finished you will have nginx installed on your machine; it will be up and running and will allow secure and encrypted RPC connections (on the default 443 port). Certificates used to create the secure connections are self signed; it is up to you decide another way to secure RPC connections or continue using the provided  default service.
* In order to be permissioned, now you need to follow the [administrative steps of the permissioning process](https://github.com/lacchain/bndes-network/blob/master/PERMISSIONING_PROCESS.md).
* Once you are permissioned, you can verify that you are connected to other nodes in the network by following the steps detailed in [#issue33](https://github.com/lacchain/bndes-network/issues/33).

## Node Configuration

### Configuring the Besu node file ###

The default configuration should work for everyone. However, depending on your needs and technical knowledge you can modify your local node's settings in `/root/lacchain/config.toml`, e.g. for RPC access or authentication. Please refer to the [reference documentation](https://docs.pantheon.pegasys.tech/en/1.2.0/Configuring-Pantheon/Using-Configuration-File/).

### Start up your node ###

Once your node is ready, you can start it up with this command in **remote machine**:

```shell
<remote_machine>$ service orion start
<remote_machine>$ service pantheon start
```

### Node Operation ###

 * If you need to restart the services, you can execute the following commands:

```shell
<remote_machine>$ service orion restart
<remote_machine>$ service pantheon restart
```

### Updates ###
  * You can update **Besu**, by preparing your inventory with:
	```shell
	[writer] #here put the role you are gong to update
	35.193.123.227 
	```

	Optionally you can choose the sha_commit of the version you want to update refered to Orion; with Besu is is only neede to specify the version:
	```shell
	[writer] #here put the role you are gong to update
	35.193.123.227 besu_release_version='1.4.4' orion_release_version='1.5.2'
	```
	Current Besu versions obtained from: https://pegasys.tech/solutions/hyperledger-besu/
	Tested BESU versions: 
	1.4.4
	1.3.6

	Current orion commit sha versions obtained from: https://github.com/PegaSysEng/orion/releases
	Tested orion versions: 
	1.5.2
	1.3.2
	1.4.0

	Replace the ip address with your node ip address.

	Now according to the role your node has, type one of the following commands on your terminal:
	```shell
	$ ansible-playbook -i inventory --private-key=~/.ssh/id_ecdsa -u remote_user site-lacchain-update-writer.yml 
	```

	```shell
	$ ansible-playbook -i inventory --private-key=~/.ssh/id_ecdsa -u remote_user site-lacchain-update-bootnode.yml 
	```

	```shell
	$ ansible-playbook -i inventory --private-key=~/.ssh/id_ecdsa -u remote_user site-lacchain-update-validator.yml 
	```
	
## Checking your connection

Once you have been permissioned, you can check if your node is connected to the network properly.

Check that the node has stablished the connections with the peers:

```shell
$ sudo -i
$ curl -X POST --data '{"jsonrpc":"2.0","method":"net_peerCount","params":[],"id":1}' localhost:4545
```

You should get a result like this:

![Connections](/docs/images/log_connections.PNG)

Now you can check if the node is syncing blocks by getting the log of the last 100 blocks:

```shell
$ tail -100 /root/lacchain/logs/pantheon_info.log
```

You should get something like this:

![Log of latest blocks](/docs/images/log_blocks.PNG)

If any of these two checks doesn't work, try to restart the pantheon service:

```shell
$ service pantheon restart
```

If that doesn't solve the problem, contact us at info@lacchain.net.
	
## Deploying Dapps on LACCHAIN

For a quick overview of some mainstream tools that you can use to deploy Smart Contracts, connect external applications and broadcast transactions to the LACChain Besu Network, you can check our [Guide](https://github.com/lacchain/bndes-network/blob/master/DEPLOY_APPLICATIONS.md).

## Contact

For any issues, you can either go to [issues](https://github.com/lacchain/bndes-network/issues) or e-mail us at info@lacchain.net. Any feedback is more than welcome!

&nbsp;
&nbsp;

**LICENSE**

<center><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/88x31.png" /></center>

This work is licensed under a [license](http://creativecommons.org/licenses/by-nc/4.0/)

&nbsp;
&nbsp;

**Acknowledgement**

We acknowledge very much the contributions of [everis](https://www.everis.com/) to this development, leading the first deployment of the network and actively participating in the day-to-day.
