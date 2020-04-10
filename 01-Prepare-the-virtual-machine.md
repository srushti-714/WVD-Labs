
# Prepare the virtual machine for Windows Virtual Desktop agent installations 

You need to do the following things to prepare your virtual machines before you can install the Windows Virtual Desktop agents and register the virtual machine to your Windows Virtual Desktop host pool: 

- You must domain-join the machine. This allows incoming Windows Virtual Desktop users to be mapped from their Azure Active Directory account to their Active Directory account and be successfully allowed access to the virtual machine. 

- You must install the Remote Desktop Session Host (RDSH) role if the virtual machine is running a Windows Server OS. The RDSH role allows the Windows Virtual Desktop agents to install properly. 

**Note**: Because these steps require a reboot, the steps have already been performed for you, to save time. 
