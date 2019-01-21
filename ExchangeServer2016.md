# Exchange Server 2016
This page describes how to create Database Availability Group and add some additional monitor resources of EXPRESSCLUSTER X.

## Evaluation Environment
```
+--------------------------+
| Active Directory         |
| - Windows Server 2012 R2 |
+--------------------------+
 |
 |  +--------------------------+
 +--| Cluster Node #1          |
 |  | - Windows Server 2012 R2 |
 |  | - Exchange Server 2016   |
 |  |   Mailbox                |
 |  | - EXPRESSCLUSTER X       |
 |  +--------------------------+
 |
 |  +--------------------------+
 +--| Cluster Node #2          |
 |  | - Windows Server 2012 R2 |
 |  | - Exchange Server 2016   |
 |  |   Mailbox                |
 |  | - EXPRESSCLUSTER X       |
 |  +--------------------------+
 |
 |  +--------------------------+
 +--| Client Machine           |
    | - Windows Server 2012 R2 |
    | - Office 2013            |
    +--------------------------+
```

## Prerequisites for Exchange Server 2016

## Prerequisites for EXPRESSCLUSTER X
1. Partitions for data mirroring
   1. Cluster partition: This partition is needed to control data mirroring. 
      - EXPRESSCLUSTER X 3.x: 17 MB is required.
      - EXPRESSCLUSTER X 4.0: 1024 MB is required.
   1. Data partition: A mailbox database file and log must be saved on this partition.

For more information, please visit [NEC website](http://www.nec.com/en/global/prod/expresscluster/en/support/manuals.html?) and refer to **Getting Started Guide** and **Installation and Configuration Guide**.

## Install Exchange Server

## Create Database Availability Group

## Install EXPRESSCLUSTER
1. Install EXPRESSCLUSTER on both the primary and secondary node following **Installation and Configuration Guide**.
1. Register the licenses.
1. Restart both nodes.

## Create Cluster and Add Monitor Resources

