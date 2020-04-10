- [Overview](#overview)
- [Windows Virtual Desktop general hierarchy](#Windows-Virtual-Desktop-general-hierarchy)
    - [Tenants ](#Tenants)
    - [Host pools ](#Host-pools )
    - [App groups ](#App-groups )
    - [End users ](#End-users )
- [Intro](#intro)
- [Prepare the virtual machine for Windows Virtual Desktop agent installations](#Prepare-the-virtual-machine-for-Windows-Virtual-Desktop-agent-installations)
  - [Install Powershell modules](#install-powershell-modules)
  - [Use your PowerShell client to create a host pool](#Use-your-PowerShell-client-to-create-a-host-pool)
  - [Register the virtual machines to the Windows Virtual Desktop host pool](Register-the-virtual-machines-to-the-Windows-Virtual-Desktop-host-pool)
  - [On the RDSH Server, Download and install the Windows Virtual Desktop Agent](#On-the-RDSH-Server,-Download-and-install-the-Windows-Virtual-Desktop-Agent) 
  - [Confirm host pool status is available](#Confirm-host-pool-status-is-available)
  -[Sign in to host pool interface ](#Sign-in-to-host-pool-interface)
- [RemoteApps ](#RemoteApps )
   - [Create a remote application group ](#Create-a-remote-application-group)
   - [Sign in to Remote App interface ](#Sign-in-to-Remote-App-interface)
- [Conclusion ](#Conclusion)



# Overview 

Windows Virtual Desktop is a multi-tenant service hosted by Microsoft that manages connections between RD clients and isolated Windows Virtual Desktop tenant environments. Each Windows Virtual Desktop tenant environment consists of one or more host pools. Each host pool consists of one or more identical session hosts. The session hosts are virtual machines (VMs) running Windows 10 Enterprise multi-session, Windows Server 2019, Windows Server 2016, Windows 7, and Windows 10 Enterprise.


Each host pool may have one or more app groups. There are two types of app groups: Remote Desktop and Remote App. Remote desktop app group offers access to a full desktop and provides immersive user experience and full interaction with the operating system running on the session host. A Remote App group publishes one or more Remote Apps that display on the Remote Desktop client as the application window on the local Remote Desktop client's desktop. 

# Windows Virtual Desktop general hierarchy 
## Tenants 

Windows Virtual Desktop Tenant is the entity that store your Windows Virtual Desktop configuration and service information. Each Windows Virtual Desktop tenant must be associated with the Azure Active Directory containing users. From the Windows Virtual Desktop tenant, you can begin creating host pools to run your users' workloads. 

## Host pools 

Host pool is a collection of Azure virtual machines that register to Windows Virtual Desktop as session hosts when you run the Windows Virtual Desktop agent. All session host in a host pool must be source from the same image and build on the same VM SU for a consistent user experience. 

A host pool can be one of two types: 
- Personal, where each session host is assigned to individual users. 

- Pooled, where session hosts can accept connections from any user authorized to an app group within the host pool.

You can set additional properties on the host pool to change its load balancing behavior, maximum user count, etc. You control the resources published to users through app groups. 

## App groups 

There are two type of app groups: 
- Remote App, where users access the Remote Apps you individually select and publish to the app group 
- Desktop, where users access the full desktop 

A Remote Desktop group (named "Desktop Application Group") is automatically created whenever you create a host pool. You can remove this app group at any time. To publish Remote Apps, you must create a Remote App group. You can create multiple Remote App groups to accommodate different worker scenarios. Different Remote App groups can also contain overlapping Remote Apps. 

To publish resources to users, you must assign them to app groups. When assigning users to app groups, consider the following things: 

- A user can't be assigned to both a desktop app group and a Remote App group in the same host pool. 
- A user can be assigned to multiple app groups within the same host pool, and their feed will be an accumulation of both app groups.

## End users 

After you've assigned users to their app groups, they can connect to a Windows Virtual Desktop deployment with any of the Windows Virtual Desktop clients. 

# Intro 

This workshop utilizes PowerShell and Azure. Basic familiarity with both is required to complete this workshop. There are several assumptions that are being made to facilitate this workshop 
- On premises active directory is already synched with Azure via AD connect. 

- Existing on premises session host have been already migrated to Azure via Azure Site Recovery 

- Migrated hosts are now VMs in Azure that have been joined to the on premises domain 

- Windows Virtual Desktop has been created 

The jumphost to the left of this window will be used to run the PowerShell commands . You will be launching a RDP session on the jumphost to connect to the RDSH 2019 Server that will be connected to the Windows Virtual Desktop environment. 

# Prepare the virtual machine for Windows Virtual Desktop agent installations 

You need to do the following things to prepare your virtual machines before you can install the Windows Virtual Desktop agents and register the virtual machine to your Windows Virtual Desktop host pool: 

- You must domain-join the machine. This allows incoming Windows Virtual Desktop users to be mapped from their Azure Active Directory account to their Active Directory account and be successfully allowed access to the virtual machine. 

- You must install the Remote Desktop Session Host (RDSH) role if the virtual machine is running a Windows Server OS. The RDSH role allows the Windows Virtual Desktop agents to install properly. 

**Note**: Because these steps require a reboot, the steps have already been performed for you, to save time. 

# Create a host pool with PowerShell 

Host pools are a collection of one or more identical virtual machines within 
Windows Virtual Desktop tenant environments. Each host pool can contain an app group that users can interact with as they would on a physical desktop. 

# Install PowerShell modules


First, download and import the Windows Virtual Desktop PowerShell module to use in your PowerShell session. 

The Windows Virtual Desktop module can be downloaded and installed from the **PowerShell Gallery** by using the PowerShell Get module.
   https://www.powershellgallery.com/packages/Microsoft.RDInfra.RDPowershell 
   
To quickly download and install the Windows Virtual Desktop PowerShell module, 

1.  Launch PowerShell ISE as an administrator and run the following command: 

      **Note**: to run PowerShell ISE as Administrator, **right click** the PowerShell ISE icon and click,**"run as administrator"**

    ```sql
       Install-Module -Name Microsoft.RDInfra.RDPowerShell
    ```

2.  When prompted to **install NuGet**, select **Yes**
      
3.  When prompted for **Untrusted repository**, select **Yes**
      
      
# Use your PowerShell client to create a host pool 
  
  In the same PowerShell ISE session, run the following commands to create a host pool 
  
  First let’s define some variables for the tenant and pool name, run the following commands to define the variables
  
  
 4. Replace  **"[DeploymentId]"** with **" UserId"**. 
    Note: UserId will be the xxxxx in odl_user_xxxxx@msazurelabs.onmicrosoft.com 

     ```sql
       $tenant = "wvdtrainingtenant[DeploymentId]" 
       $hostpoolname = "[hostpool1]"
     ```
  
 5. Run the following cmdlet to sign into the Windows Virtual Desktop  environment. 
       
       **Add-RdsAccount -DeploymentUrl "https://rdbroker.wvd.microsoft.com"**
       
 6. In the **Email, phone, or Skype box**, type  <inject key="AzureAdUserEmail" /> and click **Next**
       
 7. In the **Password** box, type <inject key="AzureAdUserPassword" /> and click **Sign in**Next 

 8.  Next, run this cmdlet to create a new host pool in your Windows Virtual Desktop tenant: 

      **New-RdsHostPool -TenantName $tenant -Name $hostpoolname** 

 9. Run the next cmdlet to create **aregistration token** to authorize a session host to join the host pool and save it to a new file on     your local computer. You can specify how long the registration token is valid by using the -ExpirationHours parameter. 
    
     ```sql

       New-RdsRegistrationInfo -TenantName $tenant -HostPoolName $hostpoolname -ExpirationHours 4 | Select-Object -ExpandProperty   Token  > "$env:PUBLIC\Desktop\token.txt"
       
       ```

       After that, run this cmdlet to add Azure Active Directory users to the default desktop app group for the host pool. 

10. Replace **"[userupn]"** with "**WVD User 1** from the environment details page 
       
     ```sql
       "Add-RdsAppGroupUser -TenantName $tenant -HostPoolName $hostpoolname -AppGroupName "Desktop Application Group" -UserPrincipalName "<userupn>"
       
      ```

       The Set-RdsRemoteDesktop cmdlet sets the properties for a published desktop. By changing the friendly name, you can set the name that appears to end-users for the published desktop in their Windows Virtual Desktop feed .
       
       
11. 
      Run the following powerShell command to set the friendly name to **WS 2019**
     ```sql
      Set-RdsRemoteDesktop -TenantName $tenant -HostPoolName $hostpoolname -AppGroupName "Desktop Application Group" -FriendlyName "WS 2019"
      ```

      # Register the virtual machines to the Windows Virtual Desktop host pool 

      Registering the virtual machines to a Windows Virtual Desktop host pool is as simple as installing the Windows Virtual Desktop agents. 

    To register the Windows Virtual Desktop agents, do the following on the RDSH windows server 
       
12. Go to the Environment details page and copy the value of "**RDSH VM PrivateIP**"

13. On the Jumpvm, search for Remote Desktop Connection in Windows Search and open that.

14. RDP to the Windows Server 2019 Server using the copied RDSH Private IP

15.  Provide the following values and connect to the VM: 

       Username: **rdshuser**

       Password: **demoPassword1!!** 
       

  # On the RDSH Server, Download and install the Windows Virtual Desktop Agent. 

16.  Open Internet Explorer 

15.  Download the **Windows Virtual Desktop Agent** 

       **https://query.prod.cms.rt.microsoft.com/cms/api/am/binary/RWrmXv**
       
16. Run the installer 

17.  When the installer asks you for the registration token, copy and paste in the value exported to the desktop file **token** in the previous step #10 

18.  Download and install the **Windows Virtual Desktop Agent Bootloader** 

   **https://query.prod.cms.rt.microsoft.com/cms/api/am/binary/RWrxrH**

19. Run the installer 

20. Minimize the **RDSHServer** RDP session 


 # Confirm host pool status is available 

21.  We can view details about our hostpool by running the following 

       **Get-RdsSessionHost -TenantName $tenant -HostPoolName $hostpoolname** 

       **Note**: Wait for agents to update  ~5 mins 

22. When the host pool **Status** is **Available**, continue to the next section 


  # Sign in to host pool interface 

23. Right click on **Edge** from the taskbar and select **New InPrivate Window** 

24. Go to **aka.ms/wvdweb**  

25. In the **Email, phone, or Skype** box, provide "WVD USER 1" email id and click **Next** 
      
26. In the Password box, type <inject key="AzureAdUserPassword" /> .1!! and click **Sign in** 

27.  Click on **WS 2019**, click **Allow**, and log in with: 

      Username: Enter "WVD USER 1" email id

      Password: <inject key="AzureAdUserPassword" /> .1!! 

28.  Once connected to the desktop, open task manager 

29. In task manager, click the More details button and then click on the Users tab, notice that your current user **WVD User 1** is connected to the RDSH server. 

      You have now logged in to the Windows Server 2019 RDSH session host.
       
       
  # RemoteApps 
       
   Now that you've made a host pool, you can populate it with RemoteApps. To learn more about how to manage apps in Windows Virtual        Desktop, see the Manage app groups tutorial. 
   
  **Manage app groups tutorial**
  
  **https://docs.microsoft.com/en-us/azure/virtual-desktop/manage-app-groups **
  
  
  # Create a remote application group 
  
  The default app group, Desktop Application Group, created automatically for a new host pool publishes the full desktop. We can also     create one or more RemoteApp app groups for the host pool which will allow users to connect to an application installed in our host     pool. 
  
  In this example, we will create a new remote application group that will publish the Wordpad Desktop application. 
  
  1.  Run the following PowerShell cmdlet to create a new empty RemoteApp group. 
  
       ```sql
        New-RdsAppGroup -TenantName $tenant -HostPoolName $hostpoolname -Name Wordpad -ResourceType 
       RemoteApp
       ```

2.  To see a list of available applications in the host pool run the following command. 

      ```sql
      Get-RdsStartMenuApp -TenantName $tenant -HostPoolName $hostpoolname -AppGroupName Wordpad
      ```
      
3.  We can search for specific apps, and in this exercise we want to search for and create a remote app for our Wordpad users. 

      ```sql
     Get-RdsStartMenuApp -TenantName $tenant -HostPoolName $hostpoolname -AppGroupName Wordpad | ?{$_.FriendlyName -match "Wordpad"}
      ```
      
4.  In the next step, we will use the FilePath, IconPath, and IconIndex fto create our Wordpad Desktop remote app 

5. To add the Wordpad application to the remote app group run the following cmdlet.

    ```sql
    
      New-RdsRemoteApp -TenantName $tenant -HostPoolName $hostpoolname -AppGroupName Wordpad -Name Wordpad -Filepath "C:\Program Files\Windows NT\Accessories\wordpad.exe" -IconPath "C:\Program Files\Windows NT\Accessories\wordpad.exe" 
      
    ```
       
6. Verify the app was published to the remote app group.

      ```sql
     Get-RdsRemoteApp -TenantName $tenant -HostPoolName $hostpoolname -AppGroupName Wordpad
     ```
     
     **Note**: You can add additional apps to this remote app group by repeating the above steps. 

7.  Now that we've created a remote app group and published the Wordpad app we can assign access to the group. 

8. Replace **"[userupn]"** with "**WVD User 2**" from the environment details page

     ```sql
      Add-RdsAppGroupUser -TenantName $tenant -HostPoolName $hostpoolname -AppGroupName Wordpad -UserPrincipalName [userupn]
     ```  
       
     Now that we've created a remote app and assigned access we can log in to the web client and open our published applications.
    
    
  # Sign in to Remote App interface 
  
 1. Close the current browser window 

2. Right click on Edge from the taskbar and select **New InPrivate Window**

3. Go to **aka.ms/wvdweb**  

4. In the **Email, phone, or Skype** box, provide email id of "**WVD User 2**" and click **Next** 

5.  In the **Password** box, type <inject key="AzureAdUserPassword"/> .1!! and click **Sign in** 

6. Click on **wordpad**, click **Allow**, and log in with: 

      Username: Provide email id of "**WVD User 2**"

      Password: <inject key="AzureAdUserPassword"/> .1!!
      
      
  # Conclusion 
  
  We've now reviewed how Windows Virtual Desktop allows us to quickly and easily virtualize and deploy modern and legacy desktop app experiences with unified management-without needing to host, install, configure and manage components such as diagnostics, networking, connection brokering, and gateway. 
  
  Windows Virtual Desktop is the only service to enable a multi-user Windows 10 experience, including compatibility with Microsoft Store and existing Windows line-of-business apps while delivering cost advantages previously only possible with server-based virtualization. 

 
    
    
   
  









