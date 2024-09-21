<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>On-premises Active Directory Deployed in the Cloud (Azure)</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.<br />

<!--
<h2>Video Demonstration</h2>

- ### [YouTube: How to Deploy on-premises Active Directory within Azure Compute](https://www.youtube.com)
-->

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 Pro (22H2)

<h2>High-Level Deployment and Configuration Steps</h2>

1. Create two *virtual machines* for the **client** and **domain controller**
2. Configure connectivity between the virtual machines
3. Install Active Directory on the domain controller
4. Create users in Active Directory
5. Join the client to the Active Directory domain
6. Allow remote desktop for normal users on the client VM
7. Create users with PowerShell

<h2>Deployment and Configuration Steps</h2>
<h3>Step 1 - Creating the virtual machines</h3>

From the Azure portal, we will create two virtual machines. One of these VMs will be the domain controller, which will be running Windows Server 2022. The other VM will be a client that will be used for testing AD, and will run Windows 10 Pro. The virtual machines will be created as follows:

<p float="left">
  <img src="images/DC_Creation.png" height="75%" width="75%" />
  <img src="images/Client_Creation.png" height="75%" width="75%" />
</p>

Note that when creating the client VM that you put it under the same virtual network as the domain controller!

<img src="images/Client_VirtualNetwork.png" height="75%" width="75%" />

<h3>Step 2 - Configuring Connectivity Between Client and Domain Controller</h3>

If we remote connect into the client and try pinging the domain controller, we'll notice we are unable to reach the domain controller.

<img src="images/Client_FailedPing.png" height="75%" width="75%" />

This is because the domain controller firewall is blocking ICMP traffic. We can log in to the domain controller and simply enable ICMPv4 in the firewall settings. Just navigate to `Control Panel > Windows Defender Firewall > Advanced Settings > Inbound Rules`, sort by *protocol*, and enable these two rules for ICMPv4.

<img src="images/DC_Firewall.png" />

Now, if we go back to our client and ping the domain controller, we should see that the ping succeeds.

<img src="images/Client_SuccessfulPing.png" height="75%" width="75%" />

<h3>Step 3 - Install Active Directory</h3>

In the domain controller, open server manager and click on `Add roles and features`

<img src="images/ServerManager_InstallAD.PNG" height="75%" width="75%" />

Choose the following options in the setup wizard

<img src="images/AD_Installation_1.PNG" height="75%" width="75%" />
<img src="images/AD_Installation_2.PNG" height="75%" width="75%" />

In the server roles step, select `Active Directory Domain Services`:

<img src="images/AD_Installation_3.PNG" height="75%" width="75%" />

After selecting this, a new windows will pop up. Select `Add Features` and hit next until you reach the confirmation step of the wizard.

<img src="images/AD_Installation_4.PNG" />

On the confirmation step, hit install:

<img src="images/AD_Installation_5.PNG" height="75%" width="75%" />

Once the installer has finished, we will need to promote the VM to an actual domain controller:

<img src="images/AD_Installation_6.PNG" height="75%" width="75%" />

A new wizard will open for setting up Actice Directory Domain Services. We will create a new forest, which we'll call `mydomain.com`:

<img src="images/AD_Installation_7.PNG" height="75%" width="75%" />

Set the options as follows and choose a strong a password for the DSRM:

<img src="images/AD_Installation_8.PNG" height="75%" width="75%" />

We will not create a DNS delegation, so uncheck this option:

<img src="images/AD_Installation_9.PNG" height="75%" width="75%" />

Continue through the wizard until the `Prerequisites check` and click install:

<img src="images/AD_Installation_10.PNG" height="75%" width="75%" />

After the installation wizard is finished, the machine will restart.

<h3>Step 4 - Creating Users in Active Directory</h3>

First open Windows search and click on `Active Directory Users and Computers`:

<img src="images/AD_Users_and_Computers_Search.png" height="75%" width="75%" />

Navigate to `mydomain.com`, which is our root domain name:

<img src="images/AD_Users_and_Computers_1.PNG" height="75%" width="75%" />

Here, we can organize users into `Organizational Units (OU)` . These can be thought of as folders/containers that hold users, groups, and computers. In our organization, we create OUs for two different types of users: **Employees** and **Admins**. To create these, simply right click on the domain name and go to `New -> Organizational Unit`:

<img src="images/AD_Users_and_Computers_2.png" height="75%" width="75%" />

We will create organizational units named `_EMPLOYEES` and `_ADMINS`. The initial underscore is a convention used to indicate that these were organizational units manually created rather than being a default created OU.

<p float="left">
  <img src="images/AD_Users_and_Computers_3.PNG" />
  <img src="images/AD_Users_and_Computers_4.PNG" />
</p>

Now, we can create a new admin user, who we'll call `jane_admin`. Right click on the `_ADMINS` OU, go to `New -> User`, and create the user as follows:

<p float="left">
  <img src="images/AD_Create_Admin_1.PNG" height="30%" width="30%" />
  <img src="images/AD_Create_Admin_2.PNG" height="30%" width="30%" />
  <img src="images/AD_Create_Admin_3.PNG" height="30%" width="30%" />
</p>

Next we add this new user to the `Domain Admins` security group. Note that this is a built-in security group. Go to the `_ADMINS` OU and go to the user's properties

<img src="images/AD_Create_Admin_4.png" height="60%" width="60%" />

Switch to the `Member of` tab and click `Add`

<img src="images/AD_Create_Admin_5.PNG" />

From here, we simply type `Domain Admins` groups and click `Check Names`.

<img src="images/AD_Create_Admin_6.PNG" />

We can confirm this all worked by logging out and logging back in to our domain controller VM with the username as `mydomain.com\jane_admin`.

<h3>Step 5 - Join the Client to the Domain</h3>

To join the client VM to the domain, we need to change its DNS server to that of the domain controller VM. That is, change it to the private IP address of the domain controller VM. To do so, navigate to the virtual machine in the Azure portal and go to network settings. From there, click on the network interface:

<img src="images/Client_NetworkSettings.png" height="75%" width="75%" />

From there, navigate to DNS settings, click custom, and enter the private IP address of the domain controller:

<img src="images/Client_DNS.png" height="75%" width="75%" />

After saving, restart the client VM:

<img src="images/Client_Restart.png" height="75%" width="75%" />

Now, we log into the client VM and join it to the domain. In file explorer, right click on `This PC` and click on `Properties`.

<img src="images/Client_JoinDomain_1.png" height="75%" width="75%" />

Then click on `Rename this PC` to open up the System Properties window.

<img src="images/Client_JoinDomain_2.png" height="75%" width="75%" />

Click on `Change`.

<img src="images/Client_JoinDomain_3.png" />

Under the `Member of` seciton, select `Domain` and enter our domain name in the field.

<img src="images/Client_JoinDomain_4.png" />

After applying this change, you will get a prompt to enter a domain admin's credentials. We can use our `jane_admin` user we created earlier.

<img src="images/Client_JoinDomain_5.png" />

Once you submit the credentials, you will get the following prompt confirming the machine is now part of the domain.

<img src="images/Client_JoinDomain_6.png" />

To finalize these changes, you must restart the client machine.

<img src="images/Client_JoinDomain_7.png" />

To confirm that the client is in the domain, we can check check the `Computers` container of AD in the domain controller VM.

<img src="images/AD_Computers_Container.PNG" />

Optionally, we can organize all our client machines into their own OU. Let's create an OU called `_CLIENTS`. Then we can drag `Client-1` into this OU.

<img src="images/AD_Clients_OU.PNG" />

<h3>Step 6 - Allow Remote Desktop</h3>

If we want to allow any non-administrative user to log into the client VM, we need to enable Remote Desktop in the client VM. Now that the client is part of the domain, we can access it by logging in as `mydomain.com\jane_admin`. After logging in, navigate to the system properties and click on `Remote desktop`

<img src="images/Client_EnableRemoteDesktop_1.png" height="75%" width="75%" />

Then we want to select which users can connect to this PC; that is, which users can log into the client with their credentials.

<img src="images/Client_EnableRemoteDesktop_2.png" height="75%" width="75%" />

We will allow `Domain Users` remote desktop access to the client.

<p float="left">
  <img src="images/Client_EnableRemoteDesktop_3.png" />
  <img src="images/Client_EnableRemoteDesktop_4.png" />
</p>

Now, we can log into our client as a normal user.

<h3>Step 7 - Creating Users</h3>

Let's head back to the domain controller as `jane_admin` to create users. We could manually create the users, but that would take too long. Instead we can use a PowerShell script. First, open `Windows PowerShell ISE` as administrator, click `New Script`, paste the code from `create_users.ps1`, and hit `F5` to run the script.

<img src="images/PowerShell_Script.PNG" height="75%" width="75%" />

This script will create 1000 users in the `_EMPLOYEES` OU with the password as `Password1`. Once the script has finished executing, we can refresh the OU and see the employees that were created. Of course, your created users will have different names as the names were created using a pseudo-random algorithm.

<img src="images/AD_Created_Users.PNG" />

Let's try to log in to the client with one of these users with the password as `Password1`. If all is successful, you should be logged in as the new user.

<img src="images/Client_RandomUserLogin.png" />

Congratulations! You have now configured Active Directory! As an admin, you can manage user permissions, reset passwords, disable accounts, add new users, delete a user, etc. As a user of the organization, you can log into any client VM that is part of the domain (in our case, we have just the Client-1 VM).