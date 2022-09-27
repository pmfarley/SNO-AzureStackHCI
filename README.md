# SNO-AzureStackHCI
Installing Single Node OpenShift (SNO) on a Single Node Azure Stack HCI using the OpenShift Assisted-Installer.

REQUIREMENTS FOR INSTALLING ON A SINGLE NODE:  
 - https://docs.openshift.com/container-platform/4.9/installing/installing_sno/install-sno-preparing-to-install-sno.html

## **SINGLE NODE OPENSHIFT PREREQUISITES:**
Single-Node OpenShift requires the following minimum host resources: 
- CPU: 8 CPU cores
- Memory: 32GB of RAM
- Storage: 120 GB 

## **SINGLE NODE AZURE STACK HCI PREREQUISITES:**
Single-Node Azure Stack HCI requires the following minimum host resources: 
 - https://learn.microsoft.com/en-us/azure-stack/hci/concepts/system-requirements

I installed Azure Stack HCI on a Cisco C220M4 server, with the following
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

Perform these steps to install the Azure Stack HCI Operating System:
 - https://learn.microsoft.com/en-us/azure-stack/hci/deploy/operating-system

## **STEP 2. CONFIGURE THE SERVER UTILIZING THE SERVER CONFIGURATION TOOL (SCONFIG).**

![image](https://user-images.githubusercontent.com/48925593/192652884-f81af13c-c10b-4438-9903-5e4bc8746b94.png)

**a. Select 8 to set the network addresses and DNS settings.**

**b. Select 2 & 3 to set the computername and join an Active Directory Domain.**

**c. Select 6 to install the latest updates.**

## **STEP 3. INSTALL THE REQUIRED ROLES AND FEATURES WITH POWERSHELL.**

   ```bash
   Install-WindowsFeature -Name "BitLocker", "Data-Center-Bridging", "Failover-Clustering", "FS-FileServer", "FS-Data-Deduplication", "Hyper-V", "Hyper-V-PowerShell", "RSAT-AD-Powershell", "RSAT-Clustering-PowerShell", "NetworkATC", "Storage-Replica" -IncludeAllSubFeature -IncludeManagementTools
   ```

## **STEP 4. CREATE AN AZURE STACK HCI CLUSTER WITH POWERSHELL.**

   ```bash
   New-Cluster -Name <cluster-name> -Node <node-name> -NOSTORAGE -StaticAddress <ipaddress>
   ```
   
   ```bash
   New-Cluster -Name AZSHCI-cluster -Node AZSHCI -NOSTORAGE -StaticAddress 192.168.2.183
   ```
   

## **STEP 5. REGISTER THE CLUSTER WITH POWERSHELL [OR WINDOWS ADMIN CENTER].**

   ```bash
   Install-Module -Name Az.StackHCI
   
   Register-AzStackHCI  -SubscriptionId "<subscription_ID>" -ResourceGroupName <resourcegroup>
   ```

## **STEP 6. CREATE VOLUMES WITH POWERSHELL.**

   ```bash
   New-Volume -FriendlyName "S2D on AZSHCI-cluster" -Size 1TB -ProvisioningType Thin
   ```

## **STEP 7. GENERATE DISCOVERY ISO FROM THE ASSISTED INSTALLER:**

Open the OpenShift Assisted Installer website: 
 - https://console.redhat.com/openshift/assisted-installer/clusters/ 
You will be prompted for your Red Hat ID and password to login.

**a. Select 'Create cluster'.**

 ![image](https://user-images.githubusercontent.com/48925593/140575947-b4f8e666-637c-451f-b797-d30feff712d3.png)


**b. Enter the cluster name, and the base domain; then select 'Install single node OpenShift (SNO)' and 'OpenShift 4.11.4', and click 'Next'.**

 ![image](https://user-images.githubusercontent.com/48925593/143304033-01bd05b8-71ad-4a9d-94e5-e9ee4b38e729.png)



**c. Select 'Generate Discovery ISO'.**

 ![image](https://user-images.githubusercontent.com/48925593/143304631-df063601-d2a5-49d7-af05-db699fd9e01e.png)


**d. Select 'Minimal Image File' and 'Generate Discovery ISO'.**

 ![image](https://user-images.githubusercontent.com/48925593/140576887-3764d5fc-b271-4b7e-806a-f79ecde64be8.png)


**e. Click on the 'Download Discovery ISO' button.**

This file will be used in a later step from the SNO Virtual Machine.

 ![image](https://user-images.githubusercontent.com/48925593/143304940-fe1e3bbc-31c5-4127-8a09-f6885be762e0.png)



**e. Click 'Close' to return to the previous screen.**


## **STEP 8. FROM WINDOWS ADMIN CENTER, CREATE A VIRTUAL MACHINE FOR SINGLE NODE OPENSHIFT**

**a. From Virtual Machines, select "Add, +New".**

**b. Enter the Name, the number of virtual processors, memory, network.**

**c. Under the Storage category, select "+ Add" and create a new disk of at least 120GB.**

**d. Under the Operating System category, select "Install an operating system from an image file (.iso)".  
Click on the browse button to select the Discovery ISO file**

NOTE: You will have to transfer the Discovery ISO file to the Azure Stack HCI server.

**e. When complete, select "Create”.**

**f. Edit the settings for the VM. Select Settings, then under the Security category, uncheck “Enable Secure Boot”.**
This will allow you to boot from the Discovery ISO image, without it having a signed hash.  
For more information see: 
 - https://learn.microsoft.com/en-us/windows-server/virtualization/hyper-v/learn-more/generation-2-virtual-machine-security-settings-for-hyper-v

## **STEP 7. BOOT THE VIRTUAL MACHINE FROM THE DISCOVERY ISO:**

**a. From Virtual Machines, select the VM and then "Power, Start".**

**b. To watch it boot, select the VM and then "Connect, Connect".**


## **STEP 8. RETURN TO THE ASSISTED INSTALLER TO FINISH THE INSTALLATION:**

Return to the OpenShift Assisted Installer.
 
 **a. You should see the SNO instance displayed in the list of discovered servers. 
      From the _Host discovery_ menu, once the SNO instance is discovered, click 'Next'.**
 
  ![image](https://user-images.githubusercontent.com/48925593/143311949-ce94272a-0548-4b4e-9be2-9a76503617c2.png)

 
 **b. From the _Networking_ menu, select the discovered `network subnet`, and click on _Next_ to proceed.**

![image](https://user-images.githubusercontent.com/48925593/143313096-6ed9e605-50ee-43e1-8e05-9fd260c09d93.png)


![image](https://user-images.githubusercontent.com/48925593/143312805-d65410b7-0263-48bb-bbe1-56e885c99276.png)


**c. Review the configuration, and select _Install Cluster_.**

 ![image](https://user-images.githubusercontent.com/48925593/143313250-269de01d-5827-4001-bea4-69dde1982d2f.png)


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

