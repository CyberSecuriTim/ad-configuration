<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>Configuring and Deploying Active Directory in the Cloud (Azure)</h1>
This tutorial outlines the implementation and configuration of Active Directory within Azure Virtual Machines.<br />

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services (AD DS)
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)

<h2>High-Level Deployment and Configuration Steps</h2>

- Step 1: Create the Domain Controller VM
- Step 2: Create the Domain client VM
- Step 3: Install Active Directory
- Step 4: Create an Active Directory Forest/Domain
- Step 5: Join the Domain Client VM to the created Domain
- Step 6: Configure Remote Desktop Access for Non-administrative users on the Domain client VM

<h2>Deployment and Configuration Steps</h2>

<h2>
  
  STEP 1: Create a virtual machine within the [Azure Portal](https://portal.azure.com) that will be used as the Active Directory Domain Controller. 
   
  - NOTE: You will be required to create a new resource group, new virtual network (Vnet) and virtual subnet during the creation of your new VM.
</h2>

- (From the Azure portal's home page) select or search for "Virtual machines", then select Create
  - Select "Azure virtual machine"
 
-  Under the "Basics" tab place the VM within a subscription and resource group (create new if needed)
    -  Name the VM
    -  Place it in a region that is geographically close to your current location (for optimal performance and minimal latency as data is traversing the network)
    -  For the VM's operating system image select "Windows Server 2022 Datacenter: Azure Edition -x64 Gen2"
![image](https://github.com/user-attachments/assets/89350a68-6353-425b-8cbb-09ada17087e0)

-  

