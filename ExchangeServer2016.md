# Exchange Server 2016
This page describes how to create Database Availability Group and add some additional monitor resources of EXPRESSCLUSTER X.

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
- Please check Microsoft web site (https://docs.microsoft.com/en-us/exchange/plan-and-deploy/prerequisites?view=exchserver-2016).
  - It is recommended to apply the latest updates with Windows Update.
  - KB2919355 is required to install .NET Framework 4.7.1. on Windows Server 2012 R2. For more detail, please check Microsoft web site (https://support.microsoft.com/en-us/help/2919355/windows-rt-8-1-windows-8-1-and-windows-server-2012-r2-update-april-201).

## Prerequisites for EXPRESSCLUSTER X
1. Partitions for data mirroring
   1. Cluster partition: This partition is needed to control data mirroring. 
      - EXPRESSCLUSTER X 3.x: 17 MB is required.
      - EXPRESSCLUSTER X 4.0: 1024 MB is required.
   1. Data partition: A mailbox database file and log must be saved on this partition.

For more information, please visit [NEC website](http://www.nec.com/en/global/prod/expresscluster/en/support/manuals.html?) and refer to **Getting Started Guide** and **Installation and Configuration Guide**.

## Install Exchange Server
1. Install Mailbox Role on each server.

## Setup Witness Server

## Create Database Availability Group

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
1. Install EXPRESSCLUSTER on both the primary and secondary node following **Installation and Configuration Guide**.
1. Register the licenses.
1. Restart both nodes.

## Save Script Files for Monitoring
1. 
1. Copy all script files to *bin* folder of EXPRESSCLUSTER (e.g. C:\Program Files\EXPRESSCLUSTER\bin).
1. Edit SetEnvironment.bat.

## Create Cluster and Add Monitor Resources
1. 
