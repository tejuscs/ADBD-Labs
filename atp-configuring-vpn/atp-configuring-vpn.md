# Configuring VPN connectivity into your private ATP network

## Introduction

Oracle Autonomous AI Database on Dedicated Infrastructure is deployed in a private VCN in the Oracle Cloud Infrastructure (OCI) with no public IP address assigned. Hence to gain connectivity to the databases, a best practice approach is to use a VPN connection.

This lab walks you through the steps to deploy a VPN server in OCI and create an SSL VPN connection between a client machine (your desktop) and the dedicated Autonomous AI Database infrastructure. Once configured, a single VPN server can be shared among multiple users.

Estimated Time: 45 minutes

### Objectives
As a network or fleet admin:

1. Configure a VPN server in OCI based on OpenVPN software.
2. Configure your VPN client and connect to VPN Server.
3. Launch SQL Developer on client and connect to a dedicated Autonomous AI Database (ATP) instance.


### Required Artifacts

- An Oracle Cloud Infrastructure account with privileges to create compute instance and network resources
- A pre-provisioned dedicated Oracle Autonomous AI Database instance in a private network. Refer to the lab **Provisioning Databases** in the [Autonomous Database Dedicated for Developers and Database Users workshop](https://livelabs.oracle.com/ords/r/dbpm/livelabs/run-workshop?p210_wid=3197) on how to provision an ATP database.
- A pre-provisioned Virtual Cloud Network with public and private subnets setup with appropriate security lists. Refer to the earlier lab **Prepare Private Network for OCI Implementation** in this workshop.

The following illustration shows a network topology that can be used to provide secure access to your dedicated autonomous infrastructure.
    ![This image shows the result of performing the above step.](./images/highlevelssl.png " ")

- As shown above, Your OCI Virtual Cloud Network (VCN) has two subnets. A private subnet that hosts your exadata infrastructure and a public subnet that has public facing web and application servers and also the VPN Server.

- An internet gateway is attached to the public subnet to allow all resources within that subnet to be accessible over the internet.

- Security lists have been setup such that tcp traffic into the private exadata subnet is allowed only through hosts in the public subnet. This can be further tightened by allowing traffic from specific hosts and ports.

- For detailed instructions on network setup for an dedicated autonomous DB infrastructure, refer to the earlier lab **Prepare Private Network for OCI Implementation** in this workshop.


## Task 1: Launch a CentOS VM for the OpenVPN server

- Log in to the Oracle Cloud Infrastructure using your tenancy, userId and password. Once logged in, Click **Compute** and then click **Instances**.

    ![This image shows the result of performing the above step.](./images/createcompute1.png " ")

- Click **Create Instance**.
   
    ![This image shows the result of performing the above step.](./images/createcompute2.png " ")

- Provide Basic Information: 
    - Name: Enter a name for your instance.
    - Compartment: Select the compartment where you want to create the instance.
    - Placement: Choose the Availability Domain.

    ![This image shows the result of performing the above step.](./images/createcompute3.png " ")

- Image and Shape:
    - Image: Choose CentOS as the operating system image and select CentOS8 Stream as the image name.
    
    ![This image shows the result of performing the above step.](./images/createcompute4.png " ")

   - Shape: Click "Change shape" to select the instance type (e.g., Virtual Machine) and shape series (Intel).

    ![This image shows the result of performing the above step.](./images/createcompute5.png " ")

    ![This image shows the result of performing the above step.](./images/createcompute5a.png " ")

- You can optionally enable **Shielded instances**. 

    ![This image shows the result of performing the above step.](./images/createcompute6.png " ")

- Networking: Configure the Virtual Cloud Network (VCN) and subnet. Select "Assign a public IPv4 address" if you need internet access to the instance.
    
    ![This image shows the result of performing the above step.](./images/createcompute7.png " ")

    ![This image shows the result of performing the above step.](./images/createcompute7a.png " ")

- Add SSH Keys: For Linux instances, generate a new key pair or upload your public key (.pub file) to allow secure SSH access.

    ![This image shows the result of performing the above step.](./images/createcompute8.png " ")

- Boot Volume: Configure boot volume settings as shown below.

    ![This image shows the result of performing the above step.](./images/createcompute9.png " ")

    ![This image shows the result of performing the above step.](./images/createcompute9a.png " ")

- Review: Click Create after reviewing your settings.
   
    ![This image shows the result of performing the above step.](./images/createcompute10.png " ")

Within a few minutes your CentOS server will be ready with a public IP for ssh access.
 
   ![This image shows the result of performing the above step.](./images/createcompute11.png " ")

## Task 2: Install and configure OpenVPN Server

In this task, you will install OpenVPN Access Server (AS) on your CentOS VM, set an admin password, and configure routing/DNS so VPN clients can reach your private subnets (for example, Application and Exadata subnets) without forcing all internet traffic through the VPN.

- Prerequisites:
  * You have the public IP address of your CentOS VM.
  * You can SSH to the VM (security list/NSG allows TCP/22 from your source).
  * Your OpenVPN server VM can reach the internet to download packages.
  * You know the CIDR blocks for the private subnets you want to reach over VPN (for example: App subnet CIDR, Exadata subnet CIDR).

- SSH into the CentOS VM
From your terminal, connect to the VM using its public IP :

    ```
    <copy>
    $ ssh opc@O<public_ipAddress_of_your_centOS_VM>
    </copy>
    ```

    ![This image shows the result of performing the above step.](./images/install-epel-release.png " ")

- Download and Install the RPM package

    ![This image shows the result of performing the above step.](./images/install-openvpn.png " ")

    ![This image shows the result of performing the above step.](./images/install-openvpn-as.png " ")

- Set the OpenVPN admin password
    OpenVPN AS uses the Linux user openvpn for initial admin access. Set its password: 

    ![This image shows the result of performing the above step.](./images/openvpn-changepwd.png " ")

    Record the password securely—you’ll use it to log in to the Admin UI.

- Sign in to the OpenVPN Admin UI (port 943)

-    From your local browser, access the admin UI console of your VPN Server (*https://public_ipAddress_of_your_centOS_VM:943/admin*), using the password for OpenVPN user.
    ![This image shows the result of performing the above step.](./images/openvpn-login.png " ")

-   Once you are logged in, click **Network Settings** and replace the **Hostname** or **IP address** with the public IP of the OpenVPN Server Instance.
    ![This image shows the result of performing the above step.](./images/openvpn-network.png " ")

    Save your setting before advancing to the VPN settings page.

- Click **VPN settings** and scroll down to the section labeled **Routing**.

    Here you configure how traffic from your VPN client (that is, your personal laptop for example) should be NATed and how DNS resolution should occur.

    Configure this section as shown in the screenshot below.
    - Choose **Yes using NAT**.
    - Provide CIDR ranges for your application and exadata subnets.
    - Pick **No** for the question - **Should client internet traffic be routed through the VPN?**
        ![This image shows the result of performing the above step.](./images/vpn-nat.png " ")

    Scroll down and configure the DNS settings as shown below.
        ![This image shows the result of performing the above step.](./images/vpn-routing2.png " ")

-   In the **Advanced VPN** section, ensure that the option **Should clients be able to communicate with each other on the VPN IP Network?** is set to **Yes**.
    ![This image shows the result of performing the above step.](./images/openvpn-advancedvpn.png " ")

    **Note:** Once you have applied your changes, click **Save Settings** once again. Then, **Update Running Server** to push your new configuration to the OpenVPN server.

## Task 3: Install OpenVPN Client

-   Launch your OpenVPN Access Server Client UI at https://Your\_VPN\_Server\_Public\_IP:943 and download the OpenVPN client for your platforms.
    ![This image shows the result of performing the above step.](./images/openvpn-client.png " ")

-   Once the installation process has completed, you can see an OpenVPN icon in your OS taskbar. Right-click this icon to bring up the context menu to start your OpenVPN connection.
    ![This image shows the result of performing the above step.](./images/openvpn-conn.png " ")
    ![This image shows the result of performing the above step.](./images/openvpn-client-conn.png " ")

    **Note:** IP should be Public IP for OpenVPN Compute Instance.

-   Click **Connect**, which brings up a window asking for the OpenVPN username and password. Enter the credentials for your **openvpn** user and click **Connect** to establish a VPN tunnel.
    ![This image shows the result of performing the above step.](./images/openvpn-clientwindow.png " ")

You may also set up your VPN server with multiple users. Follow the OpenVPN configuration guide to set up additional users.


## Task 4: Connect SQL Developer to your dedicated ATP database

- Launch SQL Developer and connect using the downloaded credentials wallet as shown below.

    **Note:** Your SQL Developer version needs to be 18.3 or higher to connect to a cloud database using a wallet.
    ![This image shows the result of performing the above step.](./images/atpd-conn.png " ")

- To follow detailed instructions on downloading your database credentials wallet, refer to the lab **Configure a Development System** in the workshop **Introduction to ADB Dedicated for Developers and Database Users.**

- You may also connect to APEX directly from your local browser. Simply get the URL from the console and launch in a browser window.
    ![This image shows the result of performing the above step.](./images/atpd-application-apex.png " ")

You may now **proceed to the next lab**.

## Acknowledgements

*Congratulations! You just configured a secure VPN connection into your private autonomous exadata infrastructure.*

- **Author** - Tejus S. & Kris Bhanushali
- **Adapted by** -  Yaisah Granillo, Cloud Solution Engineer
- **Last Updated By/Date** - Yaisah Granillo, April 2022


## See an issue or have feedback?
Please submit feedback [here](https://apexapps.oracle.com/pls/apex/f?p=133:1:::::P1_FEEDBACK:1).   Select 'Autonomous DB on Dedicated Exadata' as workshop name, include Lab name and issue / feedback details. Thank you!
