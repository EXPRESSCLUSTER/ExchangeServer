# Microsoft Exchange Server 2019 (CU9) on Windows Cluster ECX 4.3
This page describes how to create an Exchange Server 2019 cluster with EXPRESSCLUSTER X.

- For more information regarding EXPRESSCLUSTER X, please visit [this site](https://www.nec.com/en/global/prod/expresscluster/en/support/manuals.html).

## Exchange 2019 Prerequisites 

For more information on preparing Active Directory and Windows Server 2019 for Exchange Server 2019, please see Microsoft documentation at [this site](https://docs.microsoft.com/en-us/exchange/plan-and-deploy/prerequisites?view=exchserver-2019).

[Alternative Link](https://msexperttalk.com/part-2-install-and-configure-exchange-server-2019/)

## System Configuration
- Servers: 2 nodes with 1 Mirror Disk each
- OS: Windows Server 2019
- SW:
	- Exchange Server 2019 CU 9
	- EXPRESSCLUSTER X 4.0/4.1/4.2/4.3

```bat
<Public LAN>
 |
 | <Private LAN>
 |  |
 |  |  +--------------------------------+
 +-----| Primary Server                 |
 |  |  |  Windows Server 2019           |
 |  |  |  EXPRESSCLUSTER X 4.3          |
 |  +--|  Exchange Server 2019          |
 |  |  +--------------------------------+
 |  |
 |  |  +--------------------------------+
 +-----| Secondary Server               |
 |  |  |  Windows Server 2019           |
 |  |  |  EXPRESSCLUSTER X 4.3          |
 |  +--|  Exchange Server 2019          |
 |  |  +--------------------------------+
 |  |
 |  |
 |  |  +--------------------------------+
 |  +--| Client machine                 |
 |     +--------------------------------+
 |
[Gateway]
 :
```


### Requirements
- The Primary Server, Secondary Server and Client machine should be reachable via IP addresses.
- In order to use the fip resource, both servers should belong to the same nework.
	- If each server belongs to a different network, you can use a [ddns resource](https://www.manuals.nec.co.jp/contents/system/files/nec_manuals/node/539/W43_RG_EN/W_RG_03.html#understanding-dynamic-dns-resources) with [Dynamic DNS Server](https://github.com/EXPRESSCLUSTER/Tips/blob/master/ddnsPreparation.md) instead of an fip address.
- Ports which EXPRESSCLUSTER requires should be opened.
	- You can open ports by executing OpenPort.bat([X4.1](https://github.com/EXPRESSCLUSTER/Tools/blob/master/OpenPorts.bat)/[X4.2 and X4.3](https://github.com/EXPRESSCLUSTER/Tools/blob/master/OpenPorts_X42.bat)) on both servers.
- 2 partitions are required, one for Mirror Disk Data Partition and one for Cluster Partition.
	- Data Partition: Depends on mirrored data size (NTFS)
	- Cluster Partition: 1GB, RAW (do not format this partition)
	- **Note**
		- It is not supported to mirror the C: drive and please do NOT specify C: for the Data Partition.
		- Dynamic disk is not supported for Data Partition and Cluster Partition.
		- Data on the Secondary Server's Data Partition will be removed for initial Mirror Disk synchronization (Initial Recovery).
		
		
### Sample Configuration
- Primary/Secondary Server
	- OS: Windows Server 2019
	- EXPRESSCLUSTER X: 4.2 or 4.3
	- CPU: 2
	- Memory: 8GB
	- Disk
		- Disk0: System Drive
			- C:
		- Disk1: Mirror Disk
			- X:
			        Cluster Partition
				- Size: 1GB
				- File system: RAW (do NOT format)
			- E:
			        Data Partition
				- Size: Depending on data size
				- File system: NTFS
- Required Licenses
	- Core: For 4CPUs
	- Replicator Option: For 2 nodes
	- (Optional) Other Option licenses: For 2 nodes

- IP address  

  | |Public IP |Private IP |
  |-----------------|-----------------|-----------------|
  |Primary Server |10.0.7.11 |192.168.1.11 |
  |Secondary Server |10.0.7.12 |192.168.1.12 |
  |fip |10.0.7.21 |- |
  |Client |10.0.7.51 |- |
  |Gateway |10.0.7.1 |- |

## Cluster Configuration
- failover group
	- fip
	- vcom (optional)
	- md
		- Cluster Partition: X drive
		- Data Partition: E drive
	- application resource 1
	    - Exchange application resource to check if all Exchange services are running
	- application resource 2
	    - Exchange application resource to change AD parameters for a mailbox database
	- application resource 3
	    - Exchange application resource to control a Mailbox Database
		
	
## Setup
This section describes how to set up an Exchange Server with EXPRESSCLUSTER 4.3. 

### Set up a Basic Cluster
Please refer to [Basic Cluster Setup](https://github.com/EXPRESSCLUSTER/BasicCluster/blob/master/X41/Win/2nodesMirror.md). [vcom setup info here](https://www.manuals.nec.co.jp/contents/system/files/nec_manuals/node/539/W43_RG_EN/W_RG_03.html#understanding-virtual-computer-name-resources)

### Install Exchange 2019 Server

- For Exchange installation and configuration, please visit [this Microsoft site](https://docs.microsoft.com/en-us/exchange/plan-and-deploy/deploy-new-installations/deploy-new-installations?view=exchserver-2019).
- [Alternative link](https://msexperttalk.com/part-4-install-and-configure-exchange-server-2019/)

### 1.1 Modify the Powershell script execution policy to execute the script.
  
1. Launch **PowerShell** on the **Primary Server**.
 
2. Use _Get-ExecutionPolicy_ to check the current server's script execution policy.
 
3. Set the execution policy to **RemoteSigned** or **Unrestricted** using SetExecutionPolicy in order to run EC failover scripts.
         
		PS> Set-ExecutionPolicy RemoteSigned

- Repeat this process on the Standby Server.

### 1.2 Make a duplicate copy of RemoteExchange.ps1 and modify the copy as follows

- Navigate to the Exchange ‘Bin’ folder (e.g. C:\Program Files\Microsoft\Exchange Server\V15\Bin) on the Primary Server.
 
- Make a duplicate Copy of RemoteExchange.ps1 to the same folder and rename the copy to RemoteExchange-ECX.ps1.
 
- Edit RemoteExchange-ECX.ps1 by adding the line .\ControlMailboxDatabase.ps1 to the section where the functions are called. Comment out get-banner and get-tip in this section. Also add the error handling code as shown in the example below.
   
  now actually call the functions

      #get-exbanner
      #get-tip
      $ErrorControlMailboxDatabase = 90
      .\ControlMailboxDatabase.ps1
      $bRet = $?
      if ($bRet –eq $False)
      {
      exit $ErrorControlMailboxDatabase
      }

- Repeat this process on the Standby Server



## Cluster Setup 

- Confirm that the failover group is active on the primary server.

### 2.1 Check and stop service Microsoft Exchange Search Host Controller on both servers.

- Right-click Start and then click Run.(Machine 1)
- Type services.msc and click OK to open the Services management console.
- Right-click on the service Microsoft Exchange Search Host Controller and then
select Properties.
-  Set the Startup type to Disabled and then Stop the service.

- Repeat this process on the Standby Server (Machine 2)

### 2.2 Copy and configure failover scripts

- Download the script files from [NEC web site](http://www.nec.com/en/global/prod/expresscluster/en/support/Setup.html).

- Copy all script files to the EXPRESSCLUSTER bin folder (example. C:\Program
Files\EXPRESSCLUSTER\bin) on the Primary Server.

- Open SetEnvironment.bat with a text editor and change the parameters to match
your environment.
- Repeat the previous two steps on the Standby Server.

- **Note** - 
				
	  One of the scripts requires that the Active Directory module for Windows.PowerShell feature is installed. Verify this on both servers before continuing by accessing: Remote Server Administration Tools > Role Administration Tools > AD DS and AD LDS Tools > Active Directory Module for Windows PowerShell

### 2.3 Adding Application Resources in ECX cluster to Control a Exchnage Mailbox Database 
  
**STEP :- 1**

### Adding 1st application resource [example: appli-check-service]


- Start the ECX Cluster webui manager.

- In the Cluster Manager window, choose Config Mode.

- Right-click on the %failover group%, and then click Add Resource to add the first(1st) application resource
  
- From the drop down list, select application resource for Type, and give a name to
  the resource (example: appli-check-service). Click Next.

- Uncheck Follow the default dependency and click Next.

- Click Next if the default values are acceptable. Make changes to Retry Count or
  Failover Threshold first if necessary.

- Check Non-Resident and set the following parameter for Start Path.
  ```
  Start Path    : CheckExchangeServices01.bat
  Stop Path     : (NULL)
  Resident Type : Non-Resident
  ```

<p align="center">
<img src="Dpncy_appli_check_service.PNG")
</p>

- Click Tuning and set 0 for Normal Return Value and set a Timeout value of at least
  3600 for Start on the Parameter tab (see Note below). Click OK and then click Finish.
  
 **Note**
  ```
  The 1st application resource (example. appli-check-service) uses the following parameters in SetEnvironment.bat to wait for all Exchange services to be running.
        RetryCount : 30
        RetryInterval : 60
  By default, the application resource waits 1800 (= RetryCount x RetryInterval)seconds for all Exchange services to be running. If any services are not running, the
  application resource starts them and waits 1800 seconds for them to be running. Services can take up to 3600 seconds to start. It is recommended to set the Timeout value to 3600 or longer (= RetryCount x RetryInterval + some buffer).
  ```

	
**STEP :-2**
### Adding 2nd application resource [example: appli-control-AD]

- Right-click on the %failover group%, and then click Add Resource to add the
  second application resource.

- From the drop down list, select application resource for Type, and give a name to
  the resource (example: appli-control-AD). Click Next.

- Uncheck Follow the default dependency. Click the first application resource
  (example: appli-check-service) and click Add. Click Next.

 <p align="center">
<img src="Dpncy-appli-control-AD.PNG">
</p>   

- Click Next if the default values are acceptable. Make changes to Retry Count or
  Failover Threshold first if necessary.
  
- Check Non-Resident and set the following parameter for Start Path.
  ```
  Start Path : ControlActiveDirectory01.bat 
  Stop Path : (NULL)
  ```

<p align="center">
<img src="Details-appli-control-AD.PNG")
</p>   
	
- Click Tuning and set 0 for Normal Return Value of Start on the Parameter tab.

- In tunning page click on start tab 
             
		Option parameter : <Database name> ex:-<DB1>

<p align="center">
<img src="Start-Tunning-appli-control-AD.PNG")
</p>

- Click the Start tab and set the following parameters.
    ```  
    Domain   : your domain name
    Account  : a user belonging to the Schema Admins group
    Password : password for the above user
    ```
- Click OK and then click Finish.


**STEP :-3** 

### Adding 3rd application resource [example: appli-control-DB]

- Right-click on the %failover group%, and then click Add Resource to add the third
  application resource.

- From the drop down list, select application resource for Type, and give a name to
  the resource (example: appli-control-DB). Click Next
  
- Uncheck Follow the default dependency. Click the mirror disk resource and click
  Add. Click Next.

  <p align="center">
  <img src="Dpncy-appli-control-DB.PNG")
  </p>
  
  
- Click Next if the default values are acceptable. Make changes to Retry Count or
   Failover Threshold first if necessary.
   
- Check Non-Resident and set the following parameters for Start Path and Stop Path.
    ```          
    Start Path : ControlMailboxDatabase01.bat 
    Stop Path : ControlMailboxDatabase01.bat 
    ```
	
  <p align="center">
  <img src="Details-appli-control-DB.PNG")
  </p>		
				 			 
				 
				 
- Click Tuning and set 0 for Normal Return Value of both Start and Stop on the
  Parameter tab.

- Click the Start tab and set the following parameters.
    ```
    Option parameter: <Mailbox database name> Mount
    Domain          : your domain name
    Account         : a user belonging to the Organization Management group
    Password        : password of the above user
    ```
		
  <p align="center">
  <img src="Start-Tunning-appli-control-DB.PNG")
  </p>		
		 
- Click the Stop tab and set the following parameters.
    ```
    Option parameter: <Mailbox database name> Dismount
    Domain          : your domain name
    Account         : a user belonging to the Organization Management group
    Password        : password of the above user
    ```
    
  <p align="center">
  <img src="Stop-Tunning-appli-control-DB.PNG")
  </p>	
  


**Mirror Disk Resource  dependency**

- Click OK and then click Finish.

- Right-click the mirror disk resource (md) and click Properties.

- Select the Dependency tab and uncheck Follow the default dependency. 
add the only new created appli-control-AD and click Add.

Click OK.  
  
   <p align="center">
  <img src="Dpncy_Mirror_disk.PNG")
  </p>


###  Entire Dependency reference snippet.

Please refer the below image.

   <p align="center">
  <img src="Entire_Dependency.PNG")
  </p>

## 2.4 Upload the cluster configuration and start the cluster.


-  First dismount the mailbox database using Exchange Administrative Center or the
    following command in the Exchange Management Shell before starting the cluster.
        
	      Dismount-Database –Identity <Mailbox database name>

-  Then in the Cluster Manager window, click the File menu, and then Apply the Configuration File. Click OK. 

- Click OK.

- after the upload is complete, change to the Operation Mode.

- Right-click on the %failover_group% and select Start. Select the Primary Server to
start the group on and click OK. The mailbox database will mount on this server. If
the cluster is not running, click the Service menu, and then click Start Cluster. 
 
 - Click OK.


**Note**
There is no need to make changes to Microsoft Outlook or OWA.

## Testing Cluster Functionality
	   
-  Move the failover group to the Standby Server. Monitor the failover
   process in the Cluster Manager window. Verify that email clients are still able to
   connect to the mailbox database.
   
-  Move back the failover group to the Primary Server. Verify that email
   clients are still able to connect to the mailbox database.

	   
	   
 **Note:-**
	   
	 In the case that the appli-check-service fails, you will have to go to services.msc and ensure that all Exchange services are running automatically.
	 and you can also check scrpl0(C:\Program Files\EXPRESSCLUSTER\log) logs for help.


















































































		
