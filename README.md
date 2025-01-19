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
       - The portal may produce an error initially, just return to the "Basics" tab then click "Review and create" again then the validation should be passed.
   - Once the validation has been passed, click "Create" 

  
  - OPTIONAL but RECOMMENDED VM hardening step: Once the VM has been successfully deployed, access its network settings and select the RDP rule within the VM's 
    network security group inbound port rules.
      - Then change the source IP addresses that are allowed to establish an RDP connection to the VM via port 3389 from any ip address on the internet to only known 
        addresses (either an individual IP address or a range of known IP addresses)
      - By restricting the allowed source IP addresses for RDP connections to only known IP addresses, this hardens the VM by significanly reducing its attack 
        surface and thus bolstering its security posture, as opposed to having this port exposed to the entire internet. 

     ![image](https://github.com/user-attachments/assets/d4df4d55-5f37-4196-aa4d-1c6641ff745f)

<h3> STEP 1.5: Set the (future) Domain Controller VM's private IP address assignment to be static</h3>

- Select the "Dom-Con" VM then under the "Networking" section select "Network settings" 
  - Select the virtual network interface card for the VM
    - Select "ipconfig1"
      - Change the Private IP address allocation to static and save the changes
    
    ![image](https://github.com/user-attachments/assets/55eed588-7256-4b5f-8ba6-1a0c558ebab5)
 


<h3> STEP 1.75: Access the created the VM via its public IP address and your preferred Remote Desktop Connection client. </h3>

- I used the Windows Remote Desktop Connection app for this lab.
- Use the admin credentials that you assigned to the VM during its creation to login to the Remote Desktop session.

![image](https://github.com/user-attachments/assets/e7fc2ba4-5652-4b19-9ddc-ea6385ad4e9a)

- Access the Windows Defender Firewall with Advanced Security. 
  - Type "wf.msc" in the search bar in the bottom left corner
 
-  Ensure that the "Core Networking Diagnostics - ICMP Echo Requests (ICMPv4-In)" inbound rule is enabled and the action is set to "Allow" for all three firewall 
    profiles (Private, Public, Domain)
   - Sorting by protocol and looking for "ICMPv4" can help make your search for this firewall rule more efficient.

 ![image](https://github.com/user-attachments/assets/ed1f7f90-8686-43ba-b07d-76ce55059ab4)

-  NOTE: By making this configuration change on the host-based firewall we enable ourselves to use the ping command line utility later in the lab to test the 
   connectivity of our client to the domain controller.  

---------------------------------------------------------------------------------------------------------------------------------------------------------------------

<h2> STEP 2: Create another Windows virtual machine that will eventually be joined to the Active Directory domain as a client.  </h2>

   - Please repeat the VM creatinon steps above (including the optional VM hardening step) to create a slightly different virtual machine.
     - Place it in the same resource group as the previous VM
     - Place it in the same region as well
     - VM OS image: Windows 10 Pro, version 22H2 -  x64 Gen2
     ![image](https://github.com/user-attachments/assets/b4a2d027-55f0-401d-b829-f323ccb68eb2)
     ![image](https://github.com/user-attachments/assets/43942c39-25b4-4c1b-bdc8-a31abc702389)

     - Ensure this VM is placed into the same Virtual network and subnet as the previous VM.
         - Again the default network configurations are satisfactory.
     
     ![image](https://github.com/user-attachments/assets/60c3cbb9-094e-419b-9dd9-46e4c25eba26)
     ![image](https://github.com/user-attachments/assets/43886d7f-8309-4bf4-8621-a8a93eb6d979)

<h3> STEP 2.5: Statically Assign the Dom-Con VM as the Dom-Client VM's DNS server </h3>

   - Access the Dom-Client VM's network settings again.
     - Select its virtual Network Interface Card
       - Select DNS servers settings
         - Change the DNS servers from "inherit from virtual network" to custom and enter the private IP address of the Dom-Con VM.
         - Save your changes.
        
       - (From the Azure portal), select the Dom-Client VM and restart it just to ensure that the DNS server change takes effect.    

![image](https://github.com/user-attachments/assets/61ba7180-f667-40b4-82a8-20c7df5c1a9e)


  - Establish a Remote Desktop connection to the Dom-Client VM using its public IP address, your preferred remote desktop connection client and the admin 
    credentials that you assigned to this VM during its creation.

    ![image](https://github.com/user-attachments/assets/24c16eca-732f-4a2f-bc31-d3f9e2775afb)

 - Open the command prompt (type "cmd.exe") in the windows search bar
   - Dom-Con should be on but just confirm that it is...never hurts to check the obvious in IT. 😊 
   - Attempt to ping Dom-Con's private IP address (should be 10.0.0.x) (mine was 10.0.0.4)
     - "ping (dom-con private ip address) eg. "ping 10.0.0.4" 

    ![image](https://github.com/user-attachments/assets/71e5fd4f-ef3f-4aa3-83f2-d62d87dd19bb)


   - The command should have sent four ICMP echo request (ping) packets and received four ICMP Echo Replies as well as some network diagnostic data such as TTL 
    (time to live) values and latency (round trip times).


  - Within the command prompt type one more command "ipconfig /all"
    - The DNS server configuration setting should match Dom-Con's private IP address.

![image](https://github.com/user-attachments/assets/326a170d-2267-431a-ab39-ebae8484de34)

-------------------------------------------------------------------------------------------------------------------------------------------------------------------- 

<h2> STEP 3: Return to the Dom-Con virtual machine (running Windows Server 2022) and Install Active Directory</h2>

- Open the "Server Manager" program if it does not automatically open.
  - Select "Add roles and features"
  - Installation Type: Role-based or feature-based installation
  - Server Selection: Select a server from the server pool
      - Select "Dom-Con"
   
  - Server Roles: [X] Active Directory Domain Services (AD DS)
      - Add features that are required for Active Directory Domain Services
   
  - Features: The default configurations can be left unchanged
     ![image](https://github.com/user-attachments/assets/0e4f03ac-8da4-4e63-b3ef-43ffb84ac58f)

  - Confirmation: [X] Restart the destination server automatically if required.

  - Let the installation process run its course within the "Add Roles and Features Wizard".
    - "Promote this server to a domain controller" when prompted.
      ![image](https://github.com/user-attachments/assets/2e9c8d44-80cd-424c-9a81-51c0cadb3921)

---------------------------------------------------------------------------------------------------------------------------------------------------------------------

<h2> STEP 4: Create a new Forest in active directory </h2>

- Within the "Active Directory Domain Services Configuration Wizard"
  - Under "Deployment Configuration"
    - Deployment operation: Add a new forest
    - Root domain name: (name it whatever you would like)...within the domain nomenclature rules
   ![image](https://github.com/user-attachments/assets/f4b67a88-a7c1-4316-8426-b930a32dfdda)

  - "Domain Controller Options"
    - The default configurations can be left unchanged
    - Assign the Directory Services Restore Mode (DSRM) password...hopefully you won't need to use this 😅
   
    ![image](https://github.com/user-attachments/assets/4db7d981-2fc5-4102-a7b0-939d66f38840)

    - "DNS Options": Leave "create DNS delegation" unchecked
  - Additional Options": verify the NetBIOS name
 
  ![image](https://github.com/user-attachments/assets/3ab63fbf-a82d-4b51-887b-3a3b15dc8a03)

   - "Paths": Leave default configurations
   - "Review Options"

  ![image](https://github.com/user-attachments/assets/b418bee9-5f33-4822-800c-490086f9996a)

- Perform the prerequisites check then click install
  - The VM will restart. Do not be alarmed if you are disconnected just reconnect via your RDP client.
 
- When reconnecting to the VM, slightly change your username credential by appending the name of your domain before the original username (eg. timsdomain.com\dom- 
  con-admin)
    - The password remains the same as the original.
 
  ![image](https://github.com/user-attachments/assets/9553e529-28bb-47a4-a208-396b68d85088)


  - Type "Active Directory Users and Computers" in the Windows search bar and open the program.
    - (Right click the the name of your created domain) and Create an organizational unit called "_EMPLOYEES"
      ![image](https://github.com/user-attachments/assets/a92ea68a-5170-4237-b32f-e89e28fd514c)
      ![image](https://github.com/user-attachments/assets/bd905b6b-9574-4eea-8cdb-6c06d6cd5774)

    - Create another organizational unit called "_ADMINS"
      ![image](https://github.com/user-attachments/assets/c17ced2b-4290-4e23-8baa-d1b41f64baa8)


    - Create a new user in the "_ADMINS" organizational unit (assign whatever username and password credentials you would like)
    ![image](https://github.com/user-attachments/assets/d023852e-08c4-4b4c-8d1d-f9a37eabfdc4)

     - Uncheck "User must change password at next logon"
    
    ![image](https://github.com/user-attachments/assets/8662eba3-2acc-4ed0-a605-90359439ecc9)


- Add the new user account you just created to the "Domain Admins" security group.
  - Right click the user (in the _ADMINS organizational unit) and select "Properties"
    - Then select the "Member Of" tab and add this user to the "Domain Admins" security group.

  ![image](https://github.com/user-attachments/assets/3673cecd-a452-4ea5-bdbc-86ca125efde9)

 - Apply the changes and select "OK"

<h3> NOTE: Adding this user to the "_ADMINS" organizational unit was nothing more than an arbitrary classification for the sake of simplicity in this
      lab. The organizational unit could have been called anything (even rocketship). 🚀
     
  - What actually granted this user Admin privileges was making them a member of the "Domain Admins" security group.
</h3>
