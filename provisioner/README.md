# NFS Provisioner
It is used for provisioning persistent volumes for persistent volume claims.
If you want to launch nfs provisioner you should first create nfs server and also install nfs client on each node.

### NFS server installation guide:
#### On Ubuntu:
Originally developed by Sun’s Microsystems, NFS is an acronym for Network File System. It is a distributed protocol that allows a user on a client PC to access shared files from a remote server much the same way they would access files sitting locally on their PC. The NFS protocol provides a convenient way of sharing files across a Local Area Network (LAN). In this guide, we will walk you through the installation of the NFS Server on Ubuntu 20.04 LTS (Focal Fossa). We will then demonstrate how you can access files on the server from a client system.

##### Lab setup
```
NFS Server          IP:  192.168.2.103       Ubuntu 20.04
Client System       IP:  192.168.2.105       Ubuntu 20.04
```
###### Step 1) Install the NFS kernel Server package
To get started we are going to install the NFS kernel server package on Ubuntu which will, in effect, turn it into an NFS server. But first, let’s update the package list as shown.
```
$ sudo apt update
```
Thereafter, run the following command to install the NFS kernel server package.

```
$ sudo apt install nfs-kernel-server
```
This installs additional packages such as keyutils, nfs-common, rpcbind, and other dependencies required for the NFS server to function as expected.

You can verify if the nfs-server service is running as shown
```
$ sudo systemctl status nfs-server
NFS-Server-Status-Ubuntu-20-04
```
###### Step 2) Create an NFS directory share
The next step will be to create an NFS directory share. This is the directory in which we will place files to be shared across the local area network. We will create it in the /mnt/ directory as shown below. Here, our NFS share directory is called /my_shares. Feel free to assign any name to your directory.
```
$ sudo mkdir /mnt/my_shares
```
Since we want all the files accessible to all clients, we will assign the following directory ownership and permissions.
```
$ sudo chown nobody:nogroup /mnt/my_shares
$ sudo chmod -R 777 /mnt/my_shares
```
These permissions are recursive and will apply to all the files and sub-directories that you will create.

###### Step 3) Grant the NFS Server access to clients
After creating the NFS directory share and assigning the required permissions and ownership, we need to allow client systems access to the NFS server. We will achieve this by editing the /etc/exports file which was created during the installation of the nfs-kernel-server package.

So, open the /etc/exports file.
```
$ sudo vi /etc/exports
```
To allow access to a single client, add the line below and replace the client-IP parameter with the client’s actual IP.
```
/mnt/my_shares client-IP(rw,sync,no_subtree_check)
```
To add more clients to the list, simply specify more lines as shown:
```
/mnt/my_shares client-IP-1(rw,sync,no_subtree_check)
/mnt/my_shares client-IP-2(rw,sync,no_subtree_check)
/mnt/my_shares client-IP-3(rw,sync,no_subtree_check)
```
Additionally, you can specify an entire subnet a shown.
```
/mnt/my_shares 192.168.0.0/24 (rw,sync,no_subtree_check)
```
This allows all clients in the 192.168.0.0 subnet access to the server. In our case, we will grant all clients access to the NFS server as shown
```
/mnt/my_shares 192.168.2.0/24(rw,sync,no_subtree_check)
```
NFS-Server-Exports-file-Ubuntu-20-04

Let’s briefly brush through the permissions and what they stand for.

rw  (Read and Write )
sync  (Write changes to disk before applying them)
no_subtree_check  (Avoid subtree checking )
###### Step 4 ) Export the shared directory
To export the directory and make it available, invoke the command:
```
$ sudo exportfs -a
```
###### Step 5) Configure the firewall rule for NFS Server
If you are behind a UFW firewall, you need to allow NFS traffic across the firewall using the syntax shown.
```
$ sudo ufw allow from [client-IP or client-Subnet-IP] to any port nfs
```
In our case, the command will appear as follows:
```
$ sudo ufw allow from 192.168.2.0/24 to any port nfs
(Firewall-Rule-NFS-Server-Ubuntu-20-04)
```
We are all good now with configuring the NFS Server. The next step is to configure the client and test if your configuration works. So, let’s proceed and configure the client.

###### Step 5) Configure the Client system
Now log in to the client system and update the package index as shown.
```
$ sudo apt update
```
Next, install the nfs-common package as shown.
```
$ sudo apt install nfs-common
(Install-NFS-Common-Client-Package)
```

Then create a directory in the /mnt folder on which you will mount the NFS share from the server.
```
$ sudo mkdir -p /mnt/client_shared_folder
(Finally, mount the remote NFS share directory to the client directory as follows.)
```
```
$ sudo mount 192.168.2.103:/mnt/my_shares /mnt/client_shared_folder
(Mount-NFS-share-on-client-machine)
```
###### Step 6) Testing the NFS Share setup

To test if our configuration is working, we are going to create a test file in the NFS directory as shown
```
$ cd /mnt/my_shares
$ touch nfs_share.txt
(Test-NFS-Server-Ubuntu-20-04)
```
Now, let’s get back to our client and see if we can see the file in our mounted directory
```
$ ls /mnt/client_shared_folder/
```
And voila! There goes our file as shown in the snippet below. This is confirmation that our setup was successful.
~> List-NFS-Files-Client-Ubuntu
