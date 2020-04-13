
# Introduction

This workshop utilizes PowerShell and Azure. Basic familiarity with both is required to complete this workshop. There are several assumptions that are being made to facilitate this workshop 
- On premises active directory is already synched with Azure via AD connect. 

- Existing on premises session host have been already migrated to Azure via Azure Site Recovery 

- Migrated hosts are now VMs in Azure that have been joined to the on premises domain 

- Windows Virtual Desktop has been created 

The jumphost to the left of this window will be used to run the PowerShell commands . You will be launching a RDP session on the jumphost to connect to the RDSH 2019 Server that will be connected to the Windows Virtual Desktop environment. 
