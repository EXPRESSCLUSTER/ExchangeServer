# Database Availability Group with EXPRESSCLUSTER X
This page describes how to create a Database Availability Group and add some additional EXPRESSCLUSTER X monitor resources.

## Evaluation Environment
```
+--------------------------+
| Active Directory         |
| Witness Server           |
| - Windows Server 2012 R2 |
+--------------------------+
 |
 |  +-----------------------------+
 +--| Cluster Node #1             |
 |  | - Windows Server 2012 R2    |
 |  | - Exchange Server 2016 CU11 |
 |  |   Mailbox role              |
 |  | - EXPRESSCLUSTER X          |
 |  +-----------------------------+
 |
 |  +-----------------------------+
 +--| Cluster Node #2             |
    | - Windows Server 2012 R2    |
    | - Exchange Server 2016 CU11 |
    |   Mailbox Role              |
    | - EXPRESSCLUSTER X          |
    +-----------------------------+
```

## Prerequisites for Exchange Server 2016
- Please check the Microsoft web site (https://docs.microsoft.com/en-us/exchange/plan-and-deploy/prerequisites?view=exchserver-2016).
  - It is recommended to apply the latest updates with Windows Update.
  - KB2919355 is required to install .NET Framework 4.7.1. on Windows Server 2012 R2. For more details, please check the Microsoft web site (https://support.microsoft.com/en-us/help/2919355/windows-rt-8-1-windows-8-1-and-windows-server-2012-r2-update-april-201).

## Prerequisites for EXPRESSCLUSTER X
1. Partitions for data mirroring
   1. Cluster partition: This partition is needed to control data mirroring. 
      - EXPRESSCLUSTER X 3.x: 17 MB is required.
      - EXPRESSCLUSTER X 4.0: 1024 MB is required.
   1. Data partition: A mailbox database file and log must be saved on this partition.

For more information, please visit [NEC website](https://www.nec.com/en/global/prod/expresscluster/en/doc/manual.html?) and refer to **Getting Started Guide** and **Installation and Configuration Guide**.

## Extend the Active Directory Schema
1. Run the following command on the first server.  
   ```bat
   <DVD drive>\Setup.exe /IAcceptExchangeServerLicenseTerms /PrepareSchema
   ```

## Prepare Active Directory
1. Run the following command on the first server.  
   ```bat
   <DVD drive>\Setup.exe /IAcceptExchangeServerLicenseTerms /PrepareAD /OrganizationName:"ExchOrg"
   ```

## Install Exchange Server
1. Install Mailbox Role on each server.

## Setup Witness Server
1. Log on the server (Active Directory server on the above configuration) will be witness server.
1. Add **Exchange Trusted Subsystem** group to Administrators.

## Create Database Availability Group
1. Start web browser on an Exchange server.
1. Access to http://localhost/ecp with Domain Administrator account.
1. Navigate to **servers** > **database availability groups**, click **+ (New)**, enter a name for the new DAG, and specify a witness server (and optionally a witness directory — leave blank to let Exchange choose one automatically). Click **Save** to create the DAG, then add both cluster nodes as members.
   - For the full, current procedure and options, see Microsoft's guide:
     [Manage database availability groups](https://learn.microsoft.com/en-us/exchange/high-availability/database-availability-groups/database-availability-groups).

## Edit PowerShell Files of Exchange Server
1. Move to the following directory.
   ```bat
   C:\Program Files\Microsoft\Exchange Server\V15\Bin
   ```
1. Copy the following file and rename it.
   ```
   PS> cp RemoteExchange.ps1 RemoteExchange-MAPI.ps1
   PS> cp RemoteExchange.ps1 RemoteExchange-SMTP.ps1
   ```
1. Edit RemoteExchange-MAPI.ps1 as below.
    ```
    (snip)
    ## now actually call the functions 
    
    #get-exbanner 
    #get-tip 
    
    #
    # TIP: You can create your own customizations and put them in My Documents\WindowsPowerShell\profile.ps1
    # Anything in profile.ps1 will then be run every time you start the shell. 
    #

    TestMAPI.ps1
    if($LASTEXITCODE)
    {
            exit $LASTEXITCODE
    }
    else
    {
            exit 0
    }
    (snip)
    ```
1. Edit RemoteExchange-SMTP.ps1 as below.
    ```
    (snip)
    ## now actually call the functions 
    
    #get-exbanner 
    #get-tip 
    
    #
    # TIP: You can create your own customizations and put them in My Documents\WindowsPowerShell\profile.ps1
    # Anything in profile.ps1 will then be run every time you start the shell. 
    #

    TestSMTP.ps1
    if($LASTEXITCODE)
    {
            exit $LASTEXITCODE
    }
    else
    {
            exit 0
    }
    (snip)
    ```

## Install EXPRESSCLUSTER
1. Install EXPRESSCLUSTER on both the primary and secondary nodes following the **Installation and Configuration Guide**.
1. Register the licenses.
1. Restart both nodes.

## Save Script Files for Monitoring
1. Download the script files from the Exchange Server section of the [NEC EXPRESSCLUSTER web site](http://www.nec.com/en/global/prod/expresscluster/en/support/Setup.html).    
   **Note**: This link no longer works (8/14/2026). There are links to Exchange Server 2010 and 2013 scripts on [NEC EXPRESSCLUSTER Documentation - Setup Guides](https://www.nec.com/en/global/prod/expresscluster/en/doc/guide.html?#:~:text=Application-,Exchange%20Server,-Material).
1. Copy all script files to the *bin* folder of EXPRESSCLUSTER (e.g. C:\Program Files\EXPRESSCLUSTER\bin).
1. Edit SetEnvironment.bat.

## Create Cluster and Add Monitor Resources
> **Note — incomplete section:** this section is not fully documented in the
> source repository; only the heading exists. Adding the failover group,
> mirror disk resource, and monitor resources for Exchange 2016 in the
> EXPRESSCLUSTER Cluster Manager follows the same general pattern as the
> application-resource setup documented for Exchange 2019 in
> [`ExchangeServer2019/Exchange2019.md`, section 2.4](../ExchangeServer2019/Exchange2019.md),
> but the exact steps and screens differ by EXPRESSCLUSTER/Cluster Manager
> version, so that procedure should not be copied as-is. This section needs
> to be completed by someone with access to the actual Cluster Manager
> version used in this environment before it's usable as a set of
> instructions.
