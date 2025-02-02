<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>Configuring and Deploying Active Directory in the Cloud (Azure)</h1>
This tutorial outlines the implementation and configuration of Active Directory within Azure Virtual Machines.<br />

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services (AD DS)

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (22H2)

<h2>High-Level Deployment and Configuration Steps</h2>

- Step 1: Create the Domain Controller VM
- Step 2: Create the Domain Client VM
- Step 3: Install Active Directory
- Step 4: Create an Active Directory Forest/Domain
- Step 5: Join the Domain Client VM to the created Domain

<h2>Deployment and Configuration Steps</h2>

<h2>
  
  STEP 1: Create a virtual machine using the [Azure Portal](https://portal.azure.com) that will be used as the Active Directory Domain Controller. 
   
  - NOTE: You will be required to create a new resource group, new virtual network (VNet) and virtual subnet during the creation of your new VM.
</h2>

- (From the Azure portal's home page) select or search for "Virtual machines", then select Create
  - Select "Azure virtual machine"
 
-  Under the "Basics" tab place the VM within a subscription and resource group (create new if needed)
    -  Name the VM
    -  Place it in a region that is geographically close to your current location (for optimal performance and minimal latency as data is 
       traversing the network).
    -  For the VM's operating system image, select "Windows Server 2022 Datacenter: Azure Edition -x64 Gen2"
![image](https://github.com/user-attachments/assets/89350a68-6353-425b-8cbb-09ada17087e0)

    -  Configure the VM's size to have at least 2 virtual CPU's and 8 GiB of memory (the more complex the VM specifications, the more money you 
       will be charged to have it provisioned and hosted in Azure)

    - Assign the credentials for the VM's administrator account (make a note of them)
    - Select TCP port 3389 as an allowed inbound port (this will facilitate Remote Desktop connection to this VM)
        - Ignore the security warning message for now, we will address it and harden this VM more later 😅  
    - Confirm your licenses to use the Windows Server operating system.
      - Simply check both boxes.

    ![image](https://github.com/user-attachments/assets/588d7e1a-fd24-47fe-93e4-ebd7b4a38dd3)

  - Navigate to the "Networking" tab and verify that a Virtual network and subnet have been created.
      - The default settings do not need to be changed.

     ![image](https://github.com/user-attachments/assets/4d00bdc3-0f74-4af1-8585-6369e7e6e038)

   - Now simply select "Review + create"
       - The portal may produce an error initially, simply return to the "Basics" tab (verify the configurations are correct) then click "Review 
         and create" again to resolve validation issues
   - Once the validation has been passed, click "Create" 

  
  - OPTIONAL but RECOMMENDED VM hardening step: Once the VM has been successfully deployed, access its network settings and select the RDP rule 
    within the VM's network security group inbound port rules.
      - Then change the source IP addresses that are allowed to establish an RDP connection to the VM via port 3389 from any ip address on the 
         internet to only known 
        addresses (either an individual IP address or a range of known IP addresses)
      - By restricting the allowed source IP addresses for RDP connections to only known IP addresses, this hardens the VM by significantly 
         reducing its attack 
        surface and thus bolstering its security posture, as opposed to having this port exposed to the entire internet. 

     ![image](https://github.com/user-attachments/assets/d4df4d55-5f37-4196-aa4d-1c6641ff745f)

<h3> STEP 1.5: Set the (future) Domain Controller VM's private IP address assignment to be static</h3>

- Select the "Dom-Con" VM then under the "Networking" section select "Network settings" 
  - Select the virtual network interface card for the VM
    - Select "ipconfig1"
      - Change the Private IP address allocation to static and save the changes
    
    ![image](https://github.com/user-attachments/assets/7ca7e5bb-3102-457d-845d-195faa54feec)

 


<h3> STEP 1.75: Access the created VM via its public IP address and your preferred Remote Desktop Connection client. </h3>

- I used the Windows Remote Desktop Connection app for this lab.
- Use the admin credentials that you assigned to the VM during its creation to login to the Remote Desktop session.

![image](https://github.com/user-attachments/assets/e7fc2ba4-5652-4b19-9ddc-ea6385ad4e9a)

- Access the Windows Defender Firewall with Advanced Security. 
  - Type "wf.msc" in the search bar in the bottom left corner
 
-  Ensure that the "Core Networking Diagnostics - ICMP Echo Requests (ICMPv4-In)" inbound rule is enabled and the action is set to "Allow" for 
   all three firewall profiles (Private, Public, Domain)
   - Sorting by protocol and looking for "ICMPv4" can help make your search for this firewall rule more efficient.

 ![image](https://github.com/user-attachments/assets/ed1f7f90-8686-43ba-b07d-76ce55059ab4)

-  NOTE: By making this configuration change on the host-based firewall we enable ourselves to use the ping command line utility later in the 
    lab to test the connectivity of our client to the domain controller.  

---------------------------------------------------------------------------------------------------------------------------------------------------------------------

<h2> STEP 2: Create another Windows virtual machine that will eventually be joined to the Active Directory domain as a client.  </h2>

   - Please repeat the VM creation steps above (including the optional VM hardening step) to create a slightly different virtual machine.
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
         - Change the DNS servers from "inherit from virtual network" to "custom" and enter the Dom-Con (Domain Controller) VM's private IP 
            address.
         - Save your changes.
        
       - (From the Azure portal), select the Dom-Client VM and restart it just to ensure that the DNS server change takes effect.    

![image](https://github.com/user-attachments/assets/ba615414-8674-43de-8db7-88e3215ca11f)




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
    - Assign the Directory Services Restore Mode (DSRM) password...hopefully you won't need to use this for any recovery scenarios😅
   
    ![image](https://github.com/user-attachments/assets/4db7d981-2fc5-4102-a7b0-939d66f38840)

    - "DNS Options": Leave "create DNS delegation" unchecked
  - "Additional Options": verify the NetBIOS name
 
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
      - I chose Mr. Robot (if you know, you know) 😉
    ![image](https://github.com/user-attachments/assets/d023852e-08c4-4b4c-8d1d-f9a37eabfdc4)

     - Uncheck "User must change password at next logon"
    
    ![image](https://github.com/user-attachments/assets/8662eba3-2acc-4ed0-a605-90359439ecc9)


- Add the new user account you just created to the "Domain Admins" security group.
  - Right click the user (in the _ADMINS organizational unit) and select "Properties"
    - Then select the "Member Of" tab and add the user to the "Domain Admins" security group.

  ![image](https://github.com/user-attachments/assets/3673cecd-a452-4ea5-bdbc-86ca125efde9)

 - Apply the changes and select "OK"

<h3> NOTE: Adding this user to the "_ADMINS" organizational unit was nothing more than an arbitrary classification for the sake of simplicity in this
      lab. The organizational unit could have been called anything (even rocketship). 🚀
     
  - What actually granted this user Admin privileges was making them a member of the "Domain Admins" security group.
</h3>

---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
<h2> STEP 4.5: Logout of the Domain Controller VM as the current Admin Account and Log in as the newly created Domain Admin Account to Verify the Admin Account's creation. </h2>

- I will be using this account for all administrative functions on the domain controller for the rest of the lab.
    - Feel free to do the same.
 
    - NOTE: You can use any of these variations to login as users in the domain
      - domainname\username: (eg. timsdomain.com\mr_robot)
      - username@domainname: (eg. mr_robot@timsdomain.com)
     
      ![image](https://github.com/user-attachments/assets/40512e61-1ba8-4a76-99c2-c8ec873ea17e) ![image](https://github.com/user-attachments/assets/cf6e844b-ced7-4c73-9d7b-51d3fe32d900)


- OPTIONAL STEP: You can also run the command "net user (created domain admin's username)" to verify the global group memberships for 
  this account.
    - It should be a member of "Domain Admins"

       ![image](https://github.com/user-attachments/assets/28f6581a-9738-4dc1-bba5-1c16347f7100)

---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

<h2> STEP 5: Join the Windows 10 VM to the Newly Created Domain as a Client </h2>

- Re-establish a remote desktop connection to the windows 10 VM in Azure if the previous connection was terminated.
  - Log in as the local admin account that was simultaneously created with the VM.  

![image](https://github.com/user-attachments/assets/d04d7eb5-8ea1-438c-8423-3a45420305fc)

- Right Click the Windows button in the bottom left corner of the VM's desktop
  - Select "System" to access the system settings.
  - Click "Rename this PC (advanced)"
  - Within the "System Properties window"
    - Click Change (to rename this computer or change its domain or workgroup)

![image](https://github.com/user-attachments/assets/1a51e78f-b9d7-42c3-9ed9-af3589f9bf7d)

   - Member of:
     - Domain: (name of your created domain)
   - Provide the credentials of an account with admin permissions to join the domain.

![image](https://github.com/user-attachments/assets/3467b2df-ad9a-44cf-916c-73f5c336415a) ![image](https://github.com/user-attachments/assets/25d6a44d-aeb9-4681-8fe2-4a5f8bf085ab)

<h4> NOTE: The arbitrarily created domain name (eg. timsdomain.com) was able to be properly resolved and point towards our domain controller's 
 IP address because remember we statically assigned the client VM's DNS server to be our Domain Controller VM which is obviously aware of its 
 created domain name and the appropriate IP address associated with it (i.e. its own IP address)...the wonders of DNS! </h4>

  
<h3> Welcome to the domain!!😃</h3>

![image](https://github.com/user-attachments/assets/b93d74b3-9d8b-4907-8a5c-a69a8c1855c1)

- Restart the VM to completely implement this change

<h2> STEP 5.5: Reconnect to the Domain Controller VM to verify that the other VM has successfully joined the domain.</h2>

- Login to the VM using either of the admin accounts that have been created so far (i.e. the local admin account or the recently created Domain 
 Admin account)
  - I will be using the latter (the mr_robot account).

![image](https://github.com/user-attachments/assets/f9663cc7-b513-4b9a-8380-0aeee66f36b7)


- Open ADUC (Active Directory Users and Computers) and verify that the Domain Client VM appears within the "Computers" container.

![image](https://github.com/user-attachments/assets/96d62a2c-208a-4c69-a608-b65742300403)


- Create a new organizational unit (OU) named "_CLIENTS".
 
![image](https://github.com/user-attachments/assets/be53e0ff-16f7-43a7-9b26-299dd5775db3)


- Add the Domain Client VM to this new OU.

![image](https://github.com/user-attachments/assets/363ffce8-8b89-4aa8-86d1-fd86de330e8f)  ![image](https://github.com/user-attachments/assets/e8f5fd3d-ba44-4d80-8c1e-6db01d77f090)

  - The tried and true "drag and drop" method also works here to move the Client VM 
-------------------------------------------------------------------------------------------------------------------------------------------------------------

<h2> Congratulations! You have successfully installed and configured Active Directory within your VM's and now have a domain with a Domain Controller and a 
  a Domain Client. 🎊</h2>

  <h5> If you would like to take a temporary break and want to reduce the cost of hosting your VM and computing resources in Azure then you can 
       Stop/Deallocate your VMs within the Azure portal. </h5>

  - From the Azure portal's home page, navigate to the "Virtual Machines" resources and highlight both your VM's then select "Stop"

![image](https://github.com/user-attachments/assets/bf7d9bf5-f6d4-49e3-af28-17edced16234)



