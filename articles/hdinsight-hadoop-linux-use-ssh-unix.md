<properties
   pageTitle="Use SSH keys with Hadoop on Linux-based HDInsight from Linux, Unix, or OS X | Aure"
   description="Learn how to create and use SSH keys to authenticate to Linux-based HDInsight clusters."
   services="hdinsight"
   documentationCenter=""
   authors="Blackmist"
   manager="paulettm"
   editor="cgronlun"/>

<tags
   ms.service="hdinsight"
   ms.devlang=""
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="big-data"
   ms.date="02/18/2015"
   ms.author="larryfr"/>

##Use SSH with Linux-based Hadoop on HDInsight from Linux, Unix, or OS X (Preview)

Linux-based HDInsight clusters provide the option of securing SSH access using either a password or an SSH key. This document provides information on using SSH with HDInsight from Linux, Unix, or OS X clients.

> [AZURE.NOTE] The steps in this article assume you are using a Linux, Unix, or OS X client. While these steps may be performed on a Windows client if you have installed a package that provides `ssh` and `ssh-keygen` (such as Git for Windows,) we recommend that Windows clients follow the steps in [Use SSH with Linux-based HDInsight (Hadoop) from Windows](/documentation/articles/hdinsight-hadoop-linux-use-ssh-windows/).

##Prerequisites

* **ssh-keygen** and **ssh** for Linux, Unix, and OS X clients. This utility is usually provided with your operating system, or available through the package management system

* A modern web browser that supports HTML5

OR

* Azure Cross-Platform Command-line Tools

##What is SSH?

SSH is a utility for logging in to, and remotely executing, commands on a remote server. With Linux-based HDInsight, SSH establishes a secure connection to the cluster head node and provides a command-line that you use to type in commands. Commands are then executed directly on the server.

##Create an SSH key (optional)

When creating a Linux-based HDInsight cluster, you have the option of using a password or SSH key to authenticate to the server when using SSH. SSH keys are considered more secure, as they are certificate-based. Use the following information if you plan on using SSH keys with your cluster.

1. Open a terminal session and use the following command to see if you have any existing SSH keys

		ls -al ~/.ssh

	look for the following files in the directory listing. These are common names for public SSH keys.

	* id\_dsa.pub
	* id\_ecdsa.pub
	* id\_ed25519.pub
	* id\_rsa.pub

2. If you do not want to use an existing file, or have no existing SSH keys, use the following to generate a new file.

		ssh-keygen -t rsa

	You will be prompted for the following information:

	* The file location - defaults to ~/.ssh/id\_rsa.
	* A passphrase - you will be prompted to re-enter this
	
		> [AZURE.NOTE] We strongly recommend that you use a secure passphrase for the key. However, if you forget the passphrase, there is no way to recover it.

	After the command completes, you will have two new files, the private key (for example, **id\_rsa**,) and the public key (for example, **id\_rsa.pub**.)

##Create a Linux-based HDInsight cluster

When creating a Linux-based HDInsight cluster, you must provide the **public key** created previously. From Linux, Unix, or OS X clients, there are two ways to create an HDInsight cluster:

* **Azure Management Portal** - uses a web-based portal to create the cluster

* **Azure Cross-Platform Command-Line Interface (xplat-cli)** - uses command-line commands to create the cluster

Each of these methods will require either a **password** or a **public key**. For complete information on creating an Linux-based HDInsight cluster see <a href="/documentation/articles/hdinsight-hadoop-provision-linux-clusters/" target="_blank">Provision Linux-based HDInsight clusters</a>.

###Azure Management Portal

When using the portal to create a Linux-based HDInsight cluster, you must enter an **SSH User Name**, and select to enter a **Password** or **SSH Public Key**. If you select **SSH Public Key**, you must paste the public key (contained in the file with the **.pub** extension,) into the following form.

![Image of form asking for public key](./media/hdinsight-hadoop-linux-use-ssh-unix/ssh-key.png)

> [AZURE.NOTE] The key file is simply a text file. The contents should appear similar to the following.
> ```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCelfkjrpYHYiks4TM+r1LVsTYQ4jAXXGeOAF9Vv/KGz90pgMk3VRJk4PEUSELfXKxP3NtsVwLVPN1l09utI/tKHQ6WL3qy89WVVVLiwzL7tfJ2B08Gmcw8mC/YoieT/YG+4I4oAgPEmim+6/F9S0lU2I2CuFBX9JzauX8n1Y9kWzTARST+ERx2hysyA5ObLv97Xe4C2CQvGE01LGAXkw2ffP9vI+emUM+VeYrf0q3w/b1o/COKbFVZ2IpEcJ8G2SLlNsHWXofWhOKQRi64TMxT7LLoohD61q2aWNKdaE4oQdiuo8TGnt4zWLEPjzjIYIEIZGk00HiQD+KCB5pxoVtp user@system
> ```

This creates a login for the specified user, using the password or public key you provide.

###Azure Cross-Platform Command-Line Interface

You can use the <a href="../xplat-cli/" target="_brad">Azure Cross-Platform Command-Line Interface</a>, to create a new cluster using the `azure hdinsight cluster create` command.

For more information on using this command, see <a href="../hdinsight-hadoop-provision-linux-clusters/" target="_blank">Provision Hadoop Linux clusters in HDInsight using custom options</a>

##Connect to a Linux-based HDInsight cluster

From a terminal session, use the SSH command to connect to the cluster by providing the user name and cluster address.

* **SSH address** - the cluster name, followed by **-ssh.azurehdinsight.net**. For example, **mycluster-ssh.azurehdinsight.net**.

* **User name** - the SSH user name you provided when you created the cluster

The following example will connect to the cluster **mycluster** as the user **me**.

	ssh me@mycluster-ssh.azurehdinsight.net

If you used a **password** for the user account, you will be prompted to enter the password.

If you used an **SSH key** that is secured with a passphrase, you will be prompted to enter the passphrase; otherwise, SSH will attempt to automatically authenticate using one of the local private keys on your client.

> [AZURE.NOTE] If SSH does not automatically authenticate with the correct **private key**, use the **-i** parameter and specify the path to the private key. The following example will load the **private key** from `~/.ssh/id_rsa`.
> 
> `ssh -i ~/.ssh/id_rsa me@mycluster-ssh.azurehdinsight.net`

##Add additional accounts

2. Generate a new **public key** and **private key** for the new user account as described in the [Create an SSH key](#create) section.

	> [AZURE.NOTE] The private key should either be generated on a client that the user will use to connect to the cluster, or securely transfered to such the client after creation.

1. From an SSH session to the cluster, add the new user with the following command.

		sudo adduser --disabled-password <username> 

	This will create a new user account, but will disable password authentication.

2. Create the directory and files to hold the key using the following commands.

		sudo mkdir -p /home/<username>/.ssh
		sudo touch /home/<username>/.ssh/authorized_keys
		sudo nano /home/<username>/.ssh/authorized_keys

3. When the nano editor opens, copy and paste in the contents of the **public key** for the new user account. Finally, use **Ctrl-X** to save the file and exit the editor.

	![image of nano editor with example key](./media/hdinsight-hadoop-linux-use-ssh-unix/nano.png)

4. Use the following command to change ownership of the .ssh folder and contents to the new user account.

		sudo chown -hR <username>:<username> /home/<username>/.ssh

5. You should now be able to authenticate to the server with the new user account and **private key**.

##<a id="tunnel"></a>SSH Tunneling

SSH can also be used to tunnel local requests, such as web requests, to the HDInsight cluster. The request will then be routed to the requested resource as if it had originated on the HDInsight cluster head node.

This is most useful when accessing web-based services on the HDInsight cluster that use internal domain names for the head or worker nodes in the cluster. For example, some sections of the Ambari web page use internal domain names such as **headnode0.mycluster.d1.internal.cloudapp.net**. These names cannot be resolved from outside the cluster, however requests tunneled over SSH originate inside the cluster and will resolve correctly.

Use the following steps to create an SSH tunnel and configure your browser to use it to connect to the cluster.

1. The following command can be used to create an SSH tunnel to the cluster head node.

		ssh -C2qTnNf -D 9876 username@clustername-ssh.azurehdinsight.net

	This creates a connection that routes traffic to local port 9876 to the cluster over SSH. The options are:

	* **D 8080** - the local port that will route traffic through the tunnel

	* **C** - compress all data, because web traffic is mostly text

	* **2** - force SSH to try protocol version 2 only

	* **q** - quiet mode

	* **T** - disable pseudo-tty allocation, since we are just forwarding a port
	
	* **n** - prevents reading of STDIN, since we are just forwarding a port

	* **N** - do not execute a remote command, since we are just forwarding a port

	* **f** - run in the background

	If you configured the cluster with an **SSH key**, you may need use the `-i` parameter and specify the path to the private SSH key.

	Once the command completes, traffic sent to port 9876 on the local computer will be routed over SSL to the cluster head node and appear to originate there.

2. Configure the client program, such as Firefox, to use **localhost:9876** as a **SOCKS v5** proxy. Here's what the Firefox settings look like.

	![image of Firefox settings](./media/hdinsight-hadoop-linux-use-ssh-unix/socks.png)

	> [AZURE.NOTE] Selecting **Remote DNS** will resolve DNS requests using the HDInsight cluster. If unselected, DNS will be resolved locally.

	You can verify that traffic is being routed through the tunnel by vising a site such as <a href="http://www.whatismyip.com/" target="_blank">http://www.whatismyip.com/</a> with the proxy settings enabled and disabled in Firefox. While enabled, the IP address will be for a machine in the Microsoft Azure datacenter.

###Browser extensions

While configuring the browser to use the tunnel works, you don't usually want to route all traffic over the tunnel. Browser extensions such as <a href="http://getfoxyproxy.org/" target="_blank">FoxyProxy</a>  support pattern matching for URL requests (FoxyProxy Standard or Plus only,) so that only requests for specific URLs will be sent over the tunnel.

If you have installed **FoxyProxy Standard**, use the following steps to configure it to only forward traffic for HDInsight over the tunnel.

1. Open the FoxyProxy extension in your browser. For example, in Firefox, select the FoxyProxy icon next to the address field.

	![foxyproxy icon](./media/hdinsight-hadoop-linux-use-ssh-unix/foxyproxy.png)

2. Select **Add New Proxy**, the **General** tab, and then enter a proxy name of **HDInsight**.

	![foxyproxy general](./media/hdinsight-hadoop-linux-use-ssh-unix/foxygeneral.png)

3. Select the **Proxy Details tab** and populate the following fields.

	* **Host or IP Address** - localhost, since we are using an SSH tunnel on the local machine

	* **Port** - the port you used for the SSH tunnel

	* **SOCKS Proxy** - select this to enable the browser to use the tunnel as a proxy

	* **SOCKS v5** - Select this to set the required version for the proxy

	![foxyproxy proxy](./media/hdinsight-hadoop-linux-use-ssh-unix/foxyproxyproxy.png)

4. Select the **URL Patterns** tab, then select **Add New Pattern**. Use the following to define the pattern, then click **OK**.

	* **Pattern Name** - **headnode** - this is just a friendly name for the pattern

	* **URL pattern** - **\*headnode\*** - this defines a pattern that matches any URL with the word **headnode** in it.

	![foxyproxy pattern](./media/hdinsight-hadoop-linux-use-ssh-unix/foxypattern.png)

4. Select **Ok** to add the proxy and close the **Proxy Settings**

5. At the top of the FoxyProxy dialog, change **Select Mode** to **Use proxies based on their pre-defined patterns and priorities**, then select **Close**.

	![foxyproxy select mode](./media/hdinsight-hadoop-linux-use-ssh-unix/selectmode.png)

After following these steps, only requests for URLs that contain the string **headnode** will be routed over the SSL tunnel.

##Next steps

Now that you understand how to authenticate using an SSH key, learn how to use MapReduce with Hadoop on HDInsight.

* [Use Hive with HDInsight](../hdinsight-use-hive/)

* [Use Pig with HDInsight](../hdinsight-use-pig/)

* [Use MapReduce jobs with HDInsight](../hdinsight-use-mapreduce/)
 