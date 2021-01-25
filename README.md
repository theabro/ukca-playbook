# ukca-playbook

Ansible Playbooks for UKCA training based on the [Met Office Virtual Machine](https://github.com/metomi/metomi-vms).

**N. Luke Abraham, November 2020, National Centre for Atmospheric Science**

_With thanks to Richard Smith and Matt Prior, Centre for Environmental Data Archival_


This playbook is used to provision a VM on the JASMIN Unmanaged Cloud.
The playbook deploys a machine and installs X2Go server on the VM allowing the user to use applications requiring X windows. [X2Go Client](http://wiki.x2go.org/doku.php/download:start) will need to be installed on the local machine. It is based on original scripts written by Richard Smith of CEDA.

When the system is completed, the system of VMs and volumes will look like

![Diagram showing VM connections on the JASMIN Unmanaged Cloud](images/scheme.png?raw=true "VM layout on the JASMIN UMC")

with as many Student VMs provisioned as required.

Process
===

There are 5 steps that must be performed to use these playbooks to provision a working system:

1. Use the `main.yml` playbook to provision a working VM, and install a UM version (e.g. UMvn11.8), generate the KGO, etc., saving this information to an external volume in the UMC.
2. Mount this volume on a new VM and use this as an NFS server, using the `nfs.yml` playbook.
3. Make the login server using the `login.yml` playbook.
4. Make the students VMs via the `student.yml` playbook. These are provisioned concurrently through the login server, and they each will mount the volume via the NFS server.
5. Remove root access to the login server using the `login_noroot.yml` playbook (optional, or can be done manually).

While the _main_ VM will have a username of **vagrant**, each of the student VMs can have different usernames. The machines will also have a _vagrant_ user as the UM requires this when working on the VM, but each user can have their own username.

## Step 1 - Installing the UM

Before running the playbook against a cloud machine (main.yml)
===

Using [cloud.jasmin.ac.uk](https://cloud.jasmin.ac.uk) to preapre machine on jasmin for the playbook.

1. Click the **Volumes** tab and click **New volume**

2. Give a **Volume name** and a size of **25GB**

3.  Click **New machine**

4.  Enter a **Machine name** of your choice 

5.  Select the **Ubuntu-18.04** template (e.g. `ubuntu-1804-20200506`)

6. Select the size you need. The minimum size for a UM-capabil VM is **j4.small** (2 cpus, 8192MB of RAM), but a larger machine would allow you to install the KGO faster as you could run multiple tasks at once.

7.  Click **Create machine**

8.  When machine has been created, go to the **Volumes** tab and click the **Actions** drop-down menu and select **Attach volume to machine**, selecting the machine that you've just made from the drop-down menu.

9. Go back to the **Machines** tab and under the **Actions** tab for your machine, click **Attach external IP**, selecting one from the drop-down menu.

You may need to wait a few minutes before you will be able to access the VM over the internet once it's booted-up.

Running the playbook on the JASMIN unmanaged cloud
===

On macOS you may first need to

    export PATH="/Users/[YOUR USERNAME]/Library/Python/2.7/bin:$PATH"

The playbook is run using the following command from inside the repository::

    ansible-playbook main.yml -i main_inventory.ini

The playbook creates an ssh key and places it inside a folder called `ssh-keys`, named for the user. It will take around 20-30 minutes to provision this VM.

The public key is uploaded and added to the accepted ssh keys on the remote machine but it is up to the user to add the key
to their local environment.

To connect to the VM you can either use ssh over a terminal or MobaXTerm. For a graphical connection (recommended), you can use X2Go.

Using X2Go Client
===

Before you can connect to the VM you may need to install X2Go client.

* [https://wiki.x2go.org/doku.php/download:start](https://wiki.x2go.org/doku.php/download:start)

1. Fill out the settings details on new session popup as below.

2. Click on your new session on the right hand side of the x2goclient window.

3. It should automatically login, asking if you wish to allow connection on first run.

The advantage of using X2Go rather than a Terminal is that you can close the connection winow and re-open it later, leaving all your processes running as you left them.


**X2Go settings for single VM instance**

| Option | Setting |
| :--- | :--- |
| Session name | *e.g.* vagrant@vm |
| Host | *IP address for the VM, e.g.* 192.171.139.44 |
| Login | vagrant |
| Use RSA/DSA key for ssh connection | *The path to your* id\_rsa\_vagrant *key file (navigate via button)* |
| Session type | *Select* LXDE *from drop-down menu* |

UM Install Commands
===

**Note** that prior to UMvn11.1 the UM install won't work due to the `gfortran` compiler version used at Ubuntu 18.04.

**UKCA Tutorials at e.g. vn11.8**

The UKCA Tutorials at vn11.8 need some specific settings, particularly setting 

	grib_library: libgrib-api-dev 

in `group_vars/all.yml`. The current roles will perform the equivalent to `sudo install-um-extras`, but following that you will need to run the following commands in sequence:

    um-setup -s fcm:shumlib.x_tr@um11.8
    install-um-data
    install-ukca-data
    install-rose-meta
    cd src
    fcm checkout fcm:um.x_tr@vn11.8 UM11.8
    cd UM11.8
    rose stem --group=install,install_source -S CENTRAL_INSTALL=true -S UKCA=true
    rose stem --group=kgo,ukca -S GENERATE_KGO=true
    rose stem --group=fcm_make --name=vn11.8_prebuilds -S MAKE_PREBUILDS=true
    rose stem -O offline --group=fcm_make --name=vn11.8_offline_prebuilds -S MAKE_PREBUILDS=true

After the `um-setup` command you will need to close and re-open a terminal.

You may need to

    sudo chown nobody:nogroup -R .

the contents of the shared drive to allow the NFS mount to work correctly.

If you are using a `j4.medium` or higher you could use `-S LIMIT=2` with the rose-stem commands.

Creating the Tutorials
===

This is essentially a copy of what is described in the [current Tutorials](https://www.ukca.ac.uk/wiki/index.php/UKCA_Training_Overview) or [Abraham _et al._ (2018)](https://doi.org/10.5194/gmd-11-3647-2018) but at the version required (e.g. UMvn11.8). The UMvn11.8 tutorials can be found at

* [https://www.ukca.ac.uk/wiki/index.php/UKCA_Chemistry_and_Aerosol_Tutorials_at_vn11.8](https://www.ukca.ac.uk/wiki/index.php/UKCA_Chemistry_and_Aerosol_Tutorials_at_vn11.8)

and were created on a VM on the JASMIN Unmanaged Cloud using this playbook.

The material will need to be re-generated and the instructions determined by following the Tutorials at the new version. The UMvn11.8 output can be found at 

* [http://gws-access.jasmin.ac.uk/public/ukca/UKCA\_Tutorial\_vn118.tgz](http://gws-access.jasmin.ac.uk/public/ukca/UKCA_Tutorial_vn118.tgz)

and the scripts used are contained in their own repository at

* [https://github.com/theabro/ukca](https://github.com/theabro/ukca)

Note that the `/umshared` directory on the separate volume contains the following directories:

    cylc-run
    doc
    etc
    source
    src
    Tutorials
    umdir

and these are linked to `/home/vagrant`:

	cylc-run -> /umshared/cylc-run
	doc -> /umshared/doc
	source -> /umshared/source
	src -> /umshared/src
	Tutorials -> /umshared/Tutorials
	umdir -> /umshared/umdir
	
These directories are then symlinked on each of the student VMs.	
Relevant UMDPs and other documentation can be placed in the `doc/` directory.

## Step 2 - creating the NFS server

The NFS server is rather simple. On the JASMIN cloud portal you should shutdown the original UM VM, disconnect the volume, and detach the external IP. You then need to make a new VM, which doesn't need to be large, e.g. a `j3.small` (2x cpu, 4GB memory) or perhaps the slightly larger `j4.small` (2x cpu, 8GB memory). Then you need to assign your external IP address to this machine and then attach the volume.

_You will need to make a note of the **internal IP address** of this machine._ This IP address is needed later as the mount point.

You will need to edit your `~/.ssh/known_hosts` to delete the existing entries for the external IP. Once you have done that you can run the command

	ansible-playbook -v nfs.yml -i nfs_inventory.ini 
	
This will then install the required NFS software on the VM and allow the volume to be mounted by the student VMs. 

You may be asked if you want to continue connecting, if so, type `yes`.

Once the machine is made you can detach the external IP address, as you will need it for the login machine.

## Step 3 - creating the login server

Before you make the login server you may want to make all the Student VMs to get the internal IP addresses and assign the logins, although this is not essential. You should then make another `j3.small` (2x cpu, 4GB memory) or `j4.small` (2x cpu, 8GB memory) VM to be the login machine.

You will need to edit your `~/.ssh/known_hosts` to delete the existing entries for the external IP. Once you have done that you can run the command

	ansible-playbook -v login.yml -i login_inventory.ini 

It will take around 30 minutes to run the command.

You will not need to assign the external IP again after this login machine has been made, as it will be used when creating the Student VMs in the next step. This is useful as the logins at least need to have been created at this stage to create accounts on the machine.

The login server will have an `admin` account with sudo access created on it, that can be accessed via SSH.

## Step 4 - creating the Student VMs

You should create all the Student VMs via the [JASMIN Cloud Portal](https://cloud.jasmin.ac.uk) and fill-in the information into the `group_vars/all.yml` and `student_inventory.yml` files, e.g.

**student_inventory.yml**

	[deployment_servers]
	
	ukca-vm01 ansible_ssh_host=192.168.3.3 ansible_port=22 ansible_user=root username=ukcatr01 password=ukcaPass01
	ukca-vm02 ansible_ssh_host=192.168.3.11 ansible_port=22 ansible_user=root username=ukcatr02 password=ukcaPass02 
	
	[gatewayed]
	ukca-vm01 ansible_host=192.198.3.3
	ukca-vm02 ansible_host=192.168.3.11

**group_vars/all.yml**

	vmhosts:
	  ukca-vm01:
	    ip: 192.168.3.3
	    username: ukcatr01
	    password: ukcaPass01
	  ukca-vm02:
	    ip: 192.168.3.11
	    username: ukcatr02
	    password: ukcaPass02

etc. Multiple machines can be configured at once. Up to 19 have been used so far.

In order to complete the next step, SSH root access has not yet been disabled. 

X2Go Settings
===

When using

| Option | Setting |
| :--- | :--- |
| Session name | *e.g.* ukcatrXX@ukca-vmXX |
| Host | ukca-vmXX |
| Login | ukcatrXX |
| Use RSA/DSA key for ssh connection | *The path to your* id\_rsa\_ukcatrXX *key file (navigate via button)* |
| Use Proxy server for SSH connection | *Tick checkbox* |
| Proxy server – Host | 192.171.139.44 |
| Proxy server – same login as on X2Go server | *Tick checkbox* |
| Proxy server – same password as on X2Go server | *Tick checkbox* |
| Proxy server – RSA/DSA key | *The path to your* id\_rsa\_ukcatrXX *key file (navigate via button)* |
| Session type | *Select* LXDE *from drop-down menu* |


## Step 5 - remove root SSH access

Once all the VMs have been provisioned, you can use the 

	ansible-playbook -v login_noroot.yml -i login_inventory.ini 

command to disable root SSH access. The `admin` account can always be used to SSH into the machine again and enable it again in `/etc/ssh/sshd_config` by commenting out the line

	PermitRootLogin no

and then running the command

	sudo systemctl restart ssh

## Known Issues

### `rosie go`

The command

	rosie go

does not work on these VMs, unlike those provisioned with VirtualBox & Vagrant. However, the commands

	rosie checkout SUITE-ID[@REVISION]

or 

	rosie co SUITE-ID[@REVISION]

and

	rosie copy SUITE-ID[@REVISION]

can be used to checkout and copy suites respectively.