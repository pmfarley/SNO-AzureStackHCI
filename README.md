# SNO-AzureStackHCI
Installing Single Node OpenShift (SNO) on a Single Node Azure Stack HCI using the OpenShift Assisted-Installer.

REQUIREMENTS FOR INSTALLING ON A SINGLE NODE:  
 - https://docs.openshift.com/container-platform/4.9/installing/installing_sno/install-sno-preparing-to-install-sno.html

## **SINGLE NODE OPENSHIFT PREREQUISITES:**
Single-Node OpenShift has the following minimum resource requirements: 
- CPU: 8 vCPU cores
- Memory: 16 GB of RAM
- Storage: 120 GB 

## **SINGLE NODE AZURE STACK HCI PREREQUISITES:**
Single-Node Azure Stack HCI requires the following minimum host resources: 
 - https://learn.microsoft.com/en-us/azure-stack/hci/concepts/system-requirements

I installed Single-Node Azure Stack HCI on a Cisco C220M4 server, with the following
- CPU: 
  - Intel(R) Xeon(R) CPU E5-2680 v3 @ 2.50GHz (12 core)
- Memory: 384 GB
- Storage: 
  - (1) 120GB SATA SSD (boot)
  - (3) 400GB SAS SSD (storage pool)
- Network: 
  - Onboard dual-port 1GigE I350 LOM
  - Mellanox ConnectX-4 LX dual-port 1/10/25/40/50 Gigabit Ethernet adapter 

You can learn more about the single-node Azure Stack HCI clusters on Microsoft Docs: 
 - https://learn.microsoft.com/en-us/azure-stack/hci/concepts/single-server-clusters
 - https://learn.microsoft.com/en-us/azure-stack/hci/deploy/single-server

## **STEP 1. INSTALL THE AZURE STACK HCI OS ON YOUR SERVER.**

Perform these steps to download and install the Azure Stack HCI Operating System:
 - https://learn.microsoft.com/en-us/azure-stack/hci/deploy/operating-system

## **STEP 2. CONFIGURE THE SERVER UTILIZING THE SERVER CONFIGURATION TOOL (SCONFIG).**

![image](https://user-images.githubusercontent.com/48925593/192652884-f81af13c-c10b-4438-9903-5e4bc8746b94.png)

Configure the server by performing these steps in SConfig:

**a. Select 8 to set the network addresses and DNS settings.**

**b. Select 2 & 3 to set the computername and join an Active Directory Domain.**

**c. Select 6 to install the latest updates.**

## **STEP 3. INSTALL THE REQUIRED ROLES AND FEATURES WITH POWERSHELL.**

   ```bash
   Install-WindowsFeature -Name "BitLocker", "Data-Center-Bridging", "Failover-Clustering", "FS-FileServer", "FS-Data-Deduplication", "Hyper-V", "Hyper-V-PowerShell", "RSAT-AD-Powershell", "RSAT-Clustering-PowerShell", "NetworkATC", "Storage-Replica" -IncludeAllSubFeature -IncludeManagementTools
   ```

## **STEP 4. CREATE AN AZURE STACK HCI CLUSTER WITH POWERSHELL.**

Run this command from PowerShell:
   ```bash
   New-Cluster -Name <cluster-name> -Node <node-name> -NOSTORAGE -StaticAddress <ipaddress>
   ```
   
   ```bash
   New-Cluster -Name AZSHCI-cluster -Node AZSHCI -NOSTORAGE -StaticAddress 192.168.2.183
   ```
   

## **STEP 5. REGISTER THE CLUSTER WITH POWERSHELL [OR WINDOWS ADMIN CENTER].**

Run this command from PowerShell:
   ```bash
   Install-Module -Name Az.StackHCI
   
   Register-AzStackHCI  -SubscriptionId "<subscription_ID>" -ResourceGroupName <resourcegroup>
   ```

## **STEP 6. CREATE VOLUMES WITH POWERSHELL.**

Run this command from PowerShell:
   ```bash
   New-Volume -FriendlyName "S2D on AZSHCI-cluster" -Size 1TB -ProvisioningType Thin
   ```

## **STEP 7. GENERATE DISCOVERY ISO FROM THE ASSISTED INSTALLER:**

**a. Open the OpenShift Assisted Installer website:** 
 - https://console.redhat.com/openshift/assisted-installer/clusters/ 
 - You will be prompted for your Red Hat ID and password to login.

**b. Select '_Create cluster_'.**

 ![image](https://user-images.githubusercontent.com/48925593/192834456-7c792bf6-270e-43bf-890a-0b08800c135c.png)



**b. From the '_Cluster details_' step, enter the cluster name, the base domain; then select 'OpenShift 4.11.5' and 'Install single node OpenShift (SNO)', and click '_Next_'.**

 ![image](https://user-images.githubusercontent.com/48925593/192830931-2ffbf143-bd24-4207-a454-b654e5c61e72.png)


**c. On the '_Operators_' step, click '_Next_'.**

![image](https://user-images.githubusercontent.com/48925593/192830345-95deccd9-a662-4cd2-ae5a-1c7174d1d50b.png)


**d. On the '_Host discovery_' step, select '_Add host_'.**

![image](https://user-images.githubusercontent.com/48925593/192832043-bfb9dd47-4003-4a61-b87c-3525d0200edc.png)


**e. Select '_Minimal Image File_' and '_Generate Discovery ISO_'.**

 ![image](https://user-images.githubusercontent.com/48925593/192832846-14bc0b90-9294-4e7d-adc1-b6745b4752e0.png)


**e. Click on the '_Download Discovery ISO_' button.**

Save this file for use in a later step, when creating the Virtual Machine for SNO.

 ![image](https://user-images.githubusercontent.com/48925593/192833572-c3976b7e-da62-430c-ad46-6ab635504e0e.png)


**e. Click '_Close_' to return to the previous screen.**


## **STEP 8. FROM WINDOWS ADMIN CENTER, CREATE A VIRTUAL MACHINE FOR SINGLE NODE OPENSHIFT**

To use Windows Admin Center, which is a web-based management interface used to manage Azure Stack HCI, you can install it on a management PC, a Windows Server, or use it from the Azure Portal. For more information, refer to:

 - https://learn.microsoft.com/en-us/azure-stack/hci/get-started
 - https://learn.microsoft.com/en-us/windows-server/manage/windows-admin-center/azure/manage-hci-clusters
 - https://learn.microsoft.com/en-us/azure-stack/hci/manage/vm


**a. From Windows Admin Center, navigate to 'Virtual Machines', select '_Add_, _+New_'.**

 ![image](https://user-images.githubusercontent.com/48925593/192840528-9b103985-8105-4465-8650-95c3bd1b5ea1.png)


**b. Enter the Virtual Machine Name, and the virtual processors, memory, and network settings.**

The minimum resource requirements for Single-Node OpenShift are **CPU**: 8 vCPUs, **Memory**: 16 GB, **Storage**: 120 GB 

 ![image](https://user-images.githubusercontent.com/48925593/192843243-63c3820d-18da-4ef3-816d-56ab2cffd82a.png)


**c. Continue, scrolling down to the Storage category, select "_+ Add_" and continuing below.**

 ![image](https://user-images.githubusercontent.com/48925593/192844003-c0fa927b-762b-41b5-82cb-011eddb5b058.png)

**d. Continue, in the '_Storage_' category, create an empty virtual hard disk of at least 120GB.**

**e. Continue, in the '_Operating System_' category, select 'Install an operating system from an image file (.iso)', and click on the '_Browse_' button to select the Discovery ISO file.**

**NOTE:** You will have to transfer the Discovery ISO file from where you downloaded it earlier, to the Azure Stack HCI server.

 ![image](https://user-images.githubusercontent.com/48925593/192845733-7089e538-1a15-464d-a32e-0ebef9ec9cca.png)


**f. When complete, select '_Create_'.**

 ![image](https://user-images.githubusercontent.com/48925593/192847552-a68eaf2d-6a02-4f4b-8b94-c4ceb544a4fe.png)


**g. Continue by editing the settings for the VM. Click on '_Settings_' (the gear icon), then under the Security category, uncheck '_Enable Secure Boot_'. Select '_Save Security Settings_', then click '_Close_'.**

 ![image](https://user-images.githubusercontent.com/48925593/192849753-1f984931-1b11-4a58-a605-bc81477dbf67.png)

This will allow you to boot from the Discovery ISO image, without it having a signed hash.  
For more information see: 
 - https://learn.microsoft.com/en-us/windows-server/virtualization/hyper-v/learn-more/generation-2-virtual-machine-security-settings-for-hyper-v


## **STEP 7. BOOT THE VIRTUAL MACHINE FROM THE DISCOVERY ISO:**

**a. From Virtual Machines, select the VM and then '_Power_, _Start_'.**

 ![image](https://user-images.githubusercontent.com/48925593/192850501-e5c64a10-8fb3-4b11-942e-d4da35f0b1f4.png)

**b. To watch it boot, select the VM and then '_Connect_, _Connect_'.**

 ![image](https://user-images.githubusercontent.com/48925593/192855462-8d72d335-a40d-4722-bbc3-a99415e3fdbb.png)


## **STEP 8. RETURN TO THE ASSISTED INSTALLER TO FINISH THE INSTALLATION:**

Return to the OpenShift Assisted Installer.
 
 **a. You should see the SNO VM displayed in the list of discovered servers. 
      From the '_Host discovery_' menu, once the SNO VM is discovered, click '_Next_'.**
 
  ![image](https://user-images.githubusercontent.com/48925593/192861346-ad01d9db-9a08-4d2e-a569-e8f826215499.png)

 **b. From the '_Storage_' menu, click on '_Next_' to proceed.**
 
  ![image](https://user-images.githubusercontent.com/48925593/192862082-54e1eee0-ec2e-4e61-acf6-728b74bd2a9f.png)

 **b. From the '_Networking_' menu, confirm the discovered/selected `machine network`, and click on '_Next_' to proceed.**

![image](https://user-images.githubusercontent.com/48925593/192863902-ad33c29f-80c8-4ceb-925f-0f66d3bf5985.png)


**c. Review the configuration, and select _Install Cluster_.**

 ![image](https://user-images.githubusercontent.com/48925593/192865038-fff12a98-2c64-4f8f-a26a-e8d9f9fabb55.png)


**d. Monitor the installation progress.**

 ![image](https://user-images.githubusercontent.com/48925593/143313390-7b40a8a7-381c-4b97-908e-b6f4d14d6a68.png)
 
 ![image](https://user-images.githubusercontent.com/48925593/143314810-30cfc435-b66a-4069-a8fa-157e7e84fa1b.png)

 ![image](https://user-images.githubusercontent.com/48925593/143314890-e5389a58-fe0e-47f7-b863-7f317de7a7bf.png)

 ![image](https://user-images.githubusercontent.com/48925593/143315164-77fea973-cfc1-4c1c-8980-2b5852c052ab.png)

 ![image](https://user-images.githubusercontent.com/48925593/143316128-d1a5f578-4234-4acd-8ec1-86f7b52c0c49.png)


**e. Installation Complete.**

Upon completion, you'll see the summary of the installation, and you'll be able to download the kubeconfig file, 
and retreive the kubeadmin password.

 ![image](https://user-images.githubusercontent.com/48925593/143317424-36b69123-21d1-4213-83ac-26cb980f1e4f.png)

