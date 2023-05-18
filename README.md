# Single Node OpenShift on Single Node Azure Stack HCI 
Within this blog, I will provide the process for installing a Single Node Azure Stack HCI server, and using the OpenShift Assisted-Installer to install Single Node OpenShift (SNO) onto it.
If installing Red Hat OpenShift onto an existing Azure Stack HCI environment, you could skip directly to _STEP 7_.

The combination of Single Node OpenShift running on a Single Node Azure Stack HCI server provides a suitable platform for running edge workloads.  In addition, OpenShift running on Azure Stack HCI on-premises can be used along with OpenShift running in Microsoft Azure to provide a solution for hybrid cloud workloads.

## **SINGLE NODE OPENSHIFT REQUIREMENTS:**
Single-Node OpenShift has the following [minimum resource requirements](https://docs.openshift.com/container-platform/4.11/installing/installing_sno/install-sno-preparing-to-install-sno.html#install-sno-requirements-for-installing-on-a-single-node_install-sno-preparing):
 - **CPU**: 8 vCPU cores
 - **Memory**: 16 GB of RAM
 - **Storage**: 120 GB 

## **SINGLE NODE AZURE STACK HCI REQUIREMENTS:**
Make sure your system meets the [System requirements for Azure Stack HCI](https://learn.microsoft.com/en-us/azure-stack/hci/concepts/system-requirements?tabs=azure-public#server-requirements) for server and storage. 

The [physical networking requirements for Single Node Azure Stack HCI](https://learn.microsoft.com/en-us/azure-stack/hci/plan/single-server-deployment#physical-connectivity-components) can be summarized:
- **Management & compute**:  At least 1 Gbps; 10 Gbps recommended. Two teamed ports.
- **Storage**: (**OPTIONAL**, to allow for adding a second server). At least 1 Gbps; 10 Gbps recommended.
- **BMC**: Varies based on server hardware manufacturer; most are at least 1 Gbps.

In addition, the following diagram summarizes the [physical network connectivity](https://learn.microsoft.com/en-us/azure-stack/hci/plan/single-server-deployment#network-atc-intents) used for a Single Node Azure Stack HCI deployment.

<p align="center"><img width="450" alt="image" src="https://github.com/pmfarley/SNO-AzureStackHCI/assets/48925593/3b6235f0-7722-4f68-911a-b76f49b6207c"></p>


I installed Single-Node Azure Stack HCI onto a 1U rack server, with the following specs:
- CPU: 
  - Intel(R) Xeon(R) CPU E5-2680 v3 @ 2.50GHz (12 core)
- Memory: 384 GB
- Storage: 
  - (1) 120GB SATA SSD (boot/operating system)
  - (3) 400GB SAS SSD (storage pool)
- Network: 
  - Onboard dual-port 1GigE Intel I350 LOM
  - Mellanox ConnectX-4 LX dual-port 1/10/25/40/50 Gigabit Ethernet adapter 

You can learn more about the single-node Azure Stack HCI clusters on Microsoft Docs: 
 - https://learn.microsoft.com/en-us/azure-stack/hci/concepts/single-server-clusters
 - https://learn.microsoft.com/en-us/azure-stack/hci/deploy/single-server

<br></br>
## **STEP 1. INSTALL THE AZURE STACK HCI OS ON YOUR SERVER.**

<p align="center"><img width="800" alt="image" src="https://user-images.githubusercontent.com/48925593/192880327-f4fcc44c-dc0d-4c01-b6eb-df27718b2183.png"></p>


Perform the steps from the following URL, to download and install the Azure Stack HCI Operating System:
 - https://learn.microsoft.com/en-us/azure-stack/hci/deploy/operating-system

<br></br>
## **STEP 2. CONFIGURE THE SERVER UTILIZING THE SERVER CONFIGURATION TOOL (SCONFIG).**

<p align="center"><img width="800" alt="image" src="https://user-images.githubusercontent.com/48925593/192880152-8f8d247b-825e-438c-b622-35413f521440.png"></p>

Configure the server by performing these steps in SConfig:

**a. Select 8 to set the network addresses and DNS settings.**

**b. Select 2 & 3 to set the computername and join an Active Directory Domain.**

**c. Select 6 to install the latest updates.**

<br></br>
## **STEP 3. INSTALL THE REQUIRED ROLES AND FEATURES WITH POWERSHELL.**

Run this `Install-WindowsFeature` command to install the required roles and features from PowerShell:

   ```bash
   Install-WindowsFeature -Name "BitLocker", "Data-Center-Bridging", "Failover-Clustering", "FS-FileServer", "FS-Data-Deduplication", "Hyper-V", "Hyper-V-PowerShell", "RSAT-AD-Powershell", "RSAT-Clustering-PowerShell", "NetworkATC", "Storage-Replica" -IncludeAllSubFeature -IncludeManagementTools
   ```
<br></br>
## **STEP 4. CREATE AN AZURE STACK HCI CLUSTER WITH POWERSHELL.**

Run this `New-Cluster` command to create an Azure Stack HCI cluster from PowerShell:
   ```bash
   New-Cluster -Name <cluster-name> -Node <node-name> -NOSTORAGE -StaticAddress <ipaddress>
   ```

Here's the command that I ran from PowerShell:   
   ```bash
   New-Cluster -Name AZSHCI-cluster -Node AZSHCI -NOSTORAGE -StaticAddress 192.168.2.183
   ```
   
<br></br>
## **STEP 5. REGISTER THE CLUSTER WITH POWERSHELL [OR WINDOWS ADMIN CENTER].**

Run these `Install-Module` and `Register-AzStackHCI` commands from PowerShell:
   ```bash
   Install-Module -Name Az.StackHCI
   
   Register-AzStackHCI  -SubscriptionId "<subscription_ID>" -ResourceGroupName <resourcegroup>
   ```
<br></br>
## **STEP 6. CREATE VOLUMES WITH POWERSHELL.**

Run this `New-Volume` command to create a volume from PowerShell:
   ```bash
   New-Volume -FriendlyName "S2D on AZSHCI-cluster" -Size 1TB -ProvisioningType Thin
   ```
<br></br>
## **STEP 7. GENERATE DISCOVERY ISO FROM THE ASSISTED INSTALLER:**

**a. Open the OpenShift Assisted Installer from the [_Red Hat Hybrid Cloud Console_](https://console.redhat.com/openshift/assisted-installer/clusters/):** 
 - You will be prompted for your `Red Hat ID` and `password` to login.

**b. Select '_Create cluster_'.**

<p align="center"><img width="500" alt="image" src="https://user-images.githubusercontent.com/48925593/192834456-7c792bf6-270e-43bf-890a-0b08800c135c.png"></p>

**b. From the '_Cluster details_' step, enter the cluster name, the base domain; then select '_OpenShift 4.11.5_' and '_Install single node OpenShift (SNO)_', and click '_Next_'.**

<p align="center"><img width="800" alt="image" src="https://user-images.githubusercontent.com/48925593/192830931-2ffbf143-bd24-4207-a454-b654e5c61e72.png"></p>

**c. On the '_Operators_' step, click '_Next_'.**

<p align="center"><img width="800" alt="image" src="https://user-images.githubusercontent.com/48925593/192830345-95deccd9-a662-4cd2-ae5a-1c7174d1d50b.png"></p>

**d. On the '_Host discovery_' step, select '_Add host_'.**

<p align="center"><img width="800" alt="image" src="https://user-images.githubusercontent.com/48925593/192832043-bfb9dd47-4003-4a61-b87c-3525d0200edc.png"></p>

**e. Select '_Minimal Image File_' and '_Generate Discovery ISO_'.**

<p align="center"><img width="500" alt="image" src="https://user-images.githubusercontent.com/48925593/192832846-14bc0b90-9294-4e7d-adc1-b6745b4752e0.png"></p>

**f. Click on the '_Download Discovery ISO_' button.**

Save this ISO file for use in a later step, when creating the Virtual Machine for SNO.

<p align="center"><img width="500" alt="image" src="https://user-images.githubusercontent.com/48925593/192833572-c3976b7e-da62-430c-ad46-6ab635504e0e.png"></p>

**g. Click '_Close_' to return to the previous screen.**

<br></br>
## **STEP 8. FROM WINDOWS ADMIN CENTER, CREATE A VIRTUAL MACHINE FOR SINGLE NODE OPENSHIFT**

Install **Windows Admin Center**, which is the web-based management interface to manage Azure Stack HCI. You can install it onto a management PC, a Windows Server, or you may use it from the Azure Portal. For more information on installing and using Windows Admin Center in your environment, refer to the following URLs:

 - https://learn.microsoft.com/en-us/azure-stack/hci/get-started
 - https://learn.microsoft.com/en-us/windows-server/manage/windows-admin-center/azure/manage-hci-clusters
 - https://learn.microsoft.com/en-us/azure-stack/hci/manage/vm

**a. From Windows Admin Center, navigate to 'Virtual Machines', select '_Add_, _+New_'.**

<p align="center"><img width="800" alt="image" src="https://user-images.githubusercontent.com/48925593/192840528-9b103985-8105-4465-8650-95c3bd1b5ea1.png"></p>

**b. Enter the Virtual Machine Name, and the virtual processors, memory, and network settings.**

The minimum resource requirements for Single-Node OpenShift are **CPU**: 8 vCPUs, **Memory**: 16 GB, **Storage**: 120 GB 

<p align="center"><img width="500" alt="image" src="https://user-images.githubusercontent.com/48925593/192877497-2a7ba0f1-06da-461c-b1b9-93557bf6f212.png"></p>

**c. Continue, scrolling down to the Storage category, select '_+ Add_' and continuing below.**

<p align="center"><img width="500" alt="image" src="https://user-images.githubusercontent.com/48925593/192877175-16796a24-5726-4582-af33-f62ab6d07537.png"></p>

**d. Continue, in the '_Storage_' category, create an empty virtual hard disk of at least 120GB.**

**e. Continue, in the '_Operating System_' category, select 'Install an operating system from an image file (.iso)', and click on the '_Browse_' button to select the Discovery ISO file.**

**NOTE:** You will have to transfer the Discovery ISO file from where you downloaded it earlier, to the Azure Stack HCI server.

<p align="center"><img width="500" alt="image" src="https://user-images.githubusercontent.com/48925593/192877014-00efc4f4-dc20-4a14-b1b5-8df64fa2906e.png"></p>

**f. When complete, select '_Create_'.**

<p align="center"><img width="500" alt="image" src="https://user-images.githubusercontent.com/48925593/192847552-a68eaf2d-6a02-4f4b-8b94-c4ceb544a4fe.png"></p>

**g. Continue by editing the settings for the VM. Click on '_Settings_' (the gear icon), then under the _Security_ category, uncheck '_Enable Secure Boot_'. Select '_Save Security Settings_', then click '_Close_'.**

<p align="center"><img width="800" alt="image" src="https://user-images.githubusercontent.com/48925593/192849753-1f984931-1b11-4a58-a605-bc81477dbf67.png"></p>

This will allow you to boot from the Discovery ISO image, without it having a signed hash.  For more information see [_Generation 2 virtual machine security settings for Hyper-V_](https://learn.microsoft.com/en-us/windows-server/virtualization/hyper-v/learn-more/generation-2-virtual-machine-security-settings-for-hyper-v).

<br></br>
## **STEP 9. BOOT THE VIRTUAL MACHINE FROM THE DISCOVERY ISO:**

**a. From _Virtual Machines_, select the VM and then '_Power_, _Start_'.**

<p align="center"><img width="500" alt="image" src="https://user-images.githubusercontent.com/48925593/192850501-e5c64a10-8fb3-4b11-942e-d4da35f0b1f4.png"></p>

**b. To connect to the VM console to watch it during boot, select the VM and then '_Connect_, _Connect_'.**

<p align="center"><img width="500" alt="image" src="https://user-images.githubusercontent.com/48925593/192855462-8d72d335-a40d-4722-bbc3-a99415e3fdbb.png"></p>

<br></br>
## **STEP 10. RETURN TO THE ASSISTED INSTALLER TO FINISH THE INSTALLATION:**

Return to the OpenShift Assisted Installer.
 
 **a. You should see the SNO VM displayed in the list of discovered servers. 
      From the '_Host discovery_' menu, once the SNO VM is discovered, click '_Next_'.**
 
<p align="center"><img width="800" alt="image" src="https://user-images.githubusercontent.com/48925593/192861346-ad01d9db-9a08-4d2e-a569-e8f826215499.png"></p>

 **b. From the '_Storage_' menu, click on '_Next_' to proceed.**
 
<p align="center"><img width="800" alt="image" src="https://user-images.githubusercontent.com/48925593/192862082-54e1eee0-ec2e-4e61-acf6-728b74bd2a9f.png"></p>

 **c. From the '_Networking_' menu, confirm the discovered/selected 'machine network', and click on '_Next_' to proceed.**

<p align="center"><img width="800" alt="image" src="https://user-images.githubusercontent.com/48925593/192863902-ad33c29f-80c8-4ceb-925f-0f66d3bf5985.png"></p>

**d. Review the configuration, and click on '_Install Cluster_'.**

<p align="center"><img width="800" alt="image" src="https://user-images.githubusercontent.com/48925593/192865038-fff12a98-2c64-4f8f-a26a-e8d9f9fabb55.png"></p>

**e. Monitor the installation progress.**

<p align="center"><img width="800" alt="image" src="https://user-images.githubusercontent.com/48925593/192870231-ed154ce4-e90a-4acb-b21b-48896274144b.png"></p>

<p align="center"><img width="800" alt="image" src="https://user-images.githubusercontent.com/48925593/192866418-1ea22b75-6862-47c5-bdc8-504119a2b38a.png"></p>
 
<p align="center"><img width="800" alt="image" src="https://user-images.githubusercontent.com/48925593/192867012-a9efca61-ded2-414d-834f-2e43da334b6e.png"></p>

<p align="center"><img width="800" alt="image" src="https://user-images.githubusercontent.com/48925593/192869246-12d4d4e4-9434-489d-8e8c-e31eb01bd6cf.png"></p>

<p align="center"><img width="800" alt="image" src="https://user-images.githubusercontent.com/48925593/192872090-5b63d1ff-f1ff-44ac-8ffa-8e038dc8c8d6.png"></p>

**e. Installation Complete.**

Upon completion, you'll see the summary of the installation, and you'll be able to _download the kubeconfig file_, 
_copy the kubeadmin password_, and _launch the OpenShift Web console_.

<p align="center"><img width="800" alt="image" src="https://user-images.githubusercontent.com/48925593/192879833-50de1600-9bd6-4446-9d6b-bfe62953d754.png"></p>

<br></br>

## **CONCLUSTION**

