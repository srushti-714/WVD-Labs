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
  
  
 4. Replace the **Wvd tenant name** from the environment details page 
    
     **Note**: Wvd tenant name is like wvdtrainingtenantxxxxxx where xxxxxx is the deployment Id

    ```sql
       $tenant = "wvdtrainingtenant[DeploymentId]" 
       $hostpoolname = "hostpool1"
     ```
  
 5. Run the following cmdlet to sign into the Windows Virtual Desktop  environment. 
       
       **Add-RdsAccount -DeploymentUrl "https://rdbroker.wvd.microsoft.com"**
       
 6. In the **Email, phone, or Skype box**, type  <inject key="AzureAdUserEmail" /> and click **Next**
       
 7. In the **Password** box, type <inject key="AzureAdUserPassword" /> and click **Sign in**Next 

 8.  Next, run this cmdlet to create a new host pool in your Windows Virtual Desktop tenant: 

      **New-RdsHostPool -TenantName $tenant -Name $hostpoolname** 

 9. Run the next cmdlet to create **a registration token** to authorize a session host to join the host pool and save it to a new file on     your local computer. You can specify how long the registration token is valid by using the -ExpirationHours parameter. 
    
     ```sql

       New-RdsRegistrationInfo -TenantName $tenant -HostPoolName $hostpoolname -ExpirationHours 4 | Select-Object -ExpandProperty   Token  > "$env:PUBLIC\Desktop\token.txt"
       
       ```

       After that, run this cmdlet to add Azure Active Directory users to the default desktop app group for the host pool. 

10. Replace **"[userupn]"** with "**WVD User 1** from the environment details page 
       
     ```sql
       Add-RdsAppGroupUser -TenantName $tenant -HostPoolName $hostpoolname -AppGroupName "Desktop Application Group" -UserPrincipalName "<userupn>"
       
      ```

       The Set-RdsRemoteDesktop cmdlet sets the properties for a published desktop. By changing the friendly name, you can set the name that appears to end-users for the published desktop in their Windows Virtual Desktop feed .
       
       
11. 
      Run the following powerShell command to set the friendly name to **WS 2019**
     ```sql
      Set-RdsRemoteDesktop -TenantName $tenant -HostPoolName $hostpoolname -AppGroupName "Desktop Application Group" -FriendlyName "WS 2019"
      ```
