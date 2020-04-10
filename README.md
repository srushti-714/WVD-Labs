# WVD Labs

## Overview 

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
