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

    -  Configure the VM's size to have at least 2 virtual CPU's and 8 GiB of memory (the more complex the VM specifications, the more money you will be charged to 
        have it provisioned and hosted in Azure)

    - Assign the credentials for the VM's administrator account (make a note of them)
    - Select TCP port 3389 as an allowed inbound port (this will facilitate Remote Desktop connection to this VM)
        - Ignore the security warning message for now, we will address it and harden this VM more later 😅  
    - Confirm your licensing to use the Windows Server operating system.
      - Simply check both boxes.

    ![image](https://github.com/user-attachments/assets/588d7e1a-fd24-47fe-93e4-ebd7b4a38dd3)

  - Navigate to the "Networking" tab and verify that a Virtual network and subnet have been created.
      - The default configurations do not need to be changed.

     ![image](https://github.com/user-attachments/assets/4d00bdc3-0f74-4af1-8585-6369e7e6e038)

   - Now simply select "Review + create"
       - Once the validation has been passed, click "Create" 

  
  - OPTIONAL but recommended VM hardening step: Once the VM has been successfully deployed, access its network settings and select the RDP rule within the VM's 
    network security group inbound port rules.
      - Then change the source IP addresses that are allowed to establish an RDP connection to the VM via port 3389 from any ip address on the internet to only known 
        addresses (either an individual IP address or a range of known IP addresses)
      - By restricting the allowed source IP addresses for RDP connections to only known IP addresses, this hardens the VM by significanly reducing its attack 
        surface and thus bolstering its security posture, as opposed to having this port exposed to the entire internet. 

     ![image](https://github.com/user-attachments/assets/d4df4d55-5f37-4196-aa4d-1c6641ff745f)


   - Please repeat the steps above (including the optional VM hardening step) to create a slightly different virtual machine.
     - Place it in the same resource group as the previous VM
     - Place it in the same region as well
     - VM OS image: Windows 10 Pro, version 22H2 -  x64 Gen2
     ![image](https://github.com/user-attachments/assets/b4a2d027-55f0-401d-b829-f323ccb68eb2)
     ![image](https://github.com/user-attachments/assets/43942c39-25b4-4c1b-bdc8-a31abc702389)

     - Ensure this VM is placed into the same Virtual network and subnet as the previous VM.
         - Again the default network configurations are satisfactory.
     
     ![image](https://github.com/user-attachments/assets/60c3cbb9-094e-419b-9dd9-46e4c25eba26)

            
