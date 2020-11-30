# ukca-playbook

Ansible Playbooks for UKCA training based on the [Met Office Virtual Machine](https://github.com/metomi/metomi-vms).

**Luke Abraham, November 2020**

_With thanks to Richard Smith and Matt Prior_


This playbook is used to provision a VM on the JASMIN Unmanaged Cloud.
The playbook deploys a machine and installs X2Go server on the VM allowing the user to use applications requiring X windows. The
[x2goclient](http://wiki.x2go.org/doku.php/download:start) will need to be installed on the local machine.

When the system is completed, the system of VMs will look like

![Diagram showing VM connections on the JASMIN Unmanaged Cloud](images/scheme.png?raw=true "VM layout on the JASMIN UMC")

Process
----------------------

There are 5 steps that must be performed to use these playbooks to provision a working system:

1. Use the `main.yml` playbook to provision a working VM, and install a UM version (e.g. UMvn11.8), generate the KGO, etc., saving this information to an external volume in the UMC.
2. Mount this volume on a new VM and use this as an NFS server, using the `nfs.yml` playbook.
3. Make the login server using the `login.yml` playbook.
4. Make the students VMs via the `student.yml` playbook. These are provisioned concurrently through the login server, and they each will mount the volume via the NFS server.
5. Remove root access to the login server using the `login_noroot.yml` playbook.

While the _main_ VM will have a username of **vagrant**, each of the student VMs can have different usernames. The machines will also have a _vagrant_ user as the UM requires this when working on the VM, but each user can have their own username.

Before running the playbook against a cloud machine (main.yml)
---------------------------------------------------

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

Running the playbook
--------------------

Running the playbook on unmanaged cloud
=======================================


The playbook is run using the following command from inside the repository::

    ansible-playbook main.yml -i inventory.ini

On macOS you may first need to

    export PATH="/Users/[YOUR USERNAME]/Library/Python/2.7/bin:$PATH"

The playbook creates an ssh key and places it inside a folder called ssh-keys inside a folder labelled by the hostname.
The public key is uploaded and added to the accepted ssh keys on the remote machine but it is up to the user to add the key
to their local environment.

Using x2goclient
----------------

Before you can connect to the VM you may need to install X2Go client.

* [https://wiki.x2go.org/doku.php/download:start](https://wiki.x2go.org/doku.php/download:start)

1. Fill out the settings details on new session popup as below.

2. Click on your new session on the right hand side of the x2goclient window.

3. It should automatically login, asking if you wish to allow connection on first run.


**Settings**

    Session name: e.g. vagrant@vm
    Host: IP address for the VM
    Login: vagrant

    Select the ssh key file from the ssh_keys directory in the Use RSA/DSA key
    
    Check Try auto login (via SSH Agent...)
    
    Session type: LXDE

UM Install Commands
-------------------

**UKCA Tutorials at vn11.8**

The UKCA Tutorials at vn11.8 need some specific settings, particularly setting `grib_library: libgrib-api-dev` in `group_vars/all.yml`. The current roles will perform the equivalent to `sudo install-um-extras`, but following that you will need to run the following commands in sequence:

    um-setup -s fcm:shumlib.x_tr@um11.8
    install-um-data
    install-ukca-data
    install-rose-meta
    cd src
    fcm checkout fcm:um.x_tr@vn11.7 UM11.8
    cd UM11.8
    rose stem --group=install,install_source -S CENTRAL_INSTALL=true -S UKCA=true
    rose stem --group=kgo,ukca -S GENERATE_KGO=true
    rose stem --group=fcm_make --name=vn11.8_prebuilds -S MAKE_PREBUILDS=true
    rose stem -O offline --group=fcm_make --name=vn11.8_offline_prebuilds -S MAKE_PREBUILDS=true

You may need to

    sudo chown nobody:nogroup -R .

the contents of the shared drive to allow the NFS mount to work correctly.
