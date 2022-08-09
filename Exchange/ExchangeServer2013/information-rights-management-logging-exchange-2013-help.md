---
title: 'Information Rights Management logging: Exchange 2013 Help'
TOCTitle: Information Rights Management logging
ms:assetid: e06f57f9-a9e2-43a2-b88c-288b324d71f0
ms:mtpsurl: https://technet.microsoft.com/library/Ff461940(v=EXCHG.150)
ms:contentKeyID: 49319937
ms.reviewer: 
manager: serdars
ms.author: serdars
author: msdmaguire
ms.topic: article
description: Information Rights Management logging in Exchange Server
f1.keywords:
- NOCSH
mtps_version: v=EXCHG.150
---

# Information Rights Management logging

_**Applies to:** Exchange Server 2013_

In Microsoft Exchange Server 2013, Information Rights Management (IRM) operations are logged in IRM logs. IRM logs help you monitor and troubleshoot interactions between the Rights Management Services (RMS) client on an Exchange 2013 server and the Active Directory Rights Management Services (AD RMS) cluster in your organization.

To learn about IRM, see [Information Rights Management](information-rights-management-exchange-2013-help.md).

Looking for management tasks related to IRM? See [Information Rights Management procedures](information-rights-management-procedures-exchange-2013-help.md).

## Structure of IRM logs

By default, IRM logs are located in C:\\Program Files\\Microsoft\\Exchange Server\\V14\\Logging\\IRMLogs.

The naming convention for IRM log files is \<_Process_\>\_\<_Process identifier_ or _IIS AppPool identifier_\>\_IRMLOG_yyyymmdd_-_nnnn_.log, where:

- \<_Process_\> = process that creates the log file. For example, on Transport service, this will be EdgeTransport.
- \<*Process identifier_ or _IIS AppPool identifier\>_ = numerical ID of the process.
- _yyyymmdd_ = Coordinated Universal Time (UTC) date when the log file was created.
- _nnnn_ = instance number, which starts at 1 for each day.

An example IRM log file name is EdgeTransport\_1056\_IRMLOG20101201-1.log.

The following table shows the logs generated on different server roles.

### Logs on server roles

|Server role|IRM log file name|Description|
|---|---|---|
|Transport service|EdgeTransport_\<_Process identifier_\>_IRMLOG_yyyymmdd_-_nnnn_.log|This log is used to record all RMS transactions made by the transport pipeline on Transport service (for example, transport protection rules and journal report decryption). The process identifier (PID) of the edgetransport.exe process is used to generate the log file name.|
|Mailbox|msftefd_\<_Process identifier_\>_IRMLOG_yyyymmdd_-_nnnn_.log|This log is used to record all RMS transactions that occur during search and index requests. Exchange 2013 Mailbox servers use the msftefd.exe process for content indexing. The PID of the msftefd.exe process is used to generate the log file name.|
|Client Access|w3wp_MSExchangeOWAAppOol_IRMLOG_yyyymmdd_-_nnnn_.log|This log is used to record all transactions for IRM in Microsoft Office Outlook Web App.|
|All Exchange 2013 server roles|w3wp_MSExchangePowerShellAppPool_IRMLOG_yyyymmdd_-_nnnn_.log|This log is used to record all IRM RMS transactions issued from Windows PowerShell, for example, when issuing the **Test-IRMConfiguration** cmdlet.|

## Logging process

Information is written to the log file until the file size reaches its maximum specified value. When the maximum size is reached, a log file that has an incremental instance number is created. This process is repeated throughout the day. Circular logging deletes the oldest log files when the IRM log directory reaches its maximum specified size or when a log file reaches the maximum age specified in the IRM logging configuration on each server.

## Information written to IRM logs

IRM log files are text files that contain data in comma-separated value (CSV) format. Each IRM log has a header that contains the following information:

- **\#Software**: Name of the software that created the IRM log file. Typically, the value is `Microsoft Exchange Server`.

- **\#Version**: Version number of the software that created the IRM log file.

- **\#Log-type**: Log type value, which is `Rms Client Manager Log`.

- **\#Date**: The UTC date and time when the log file was created. The UTC date and time is represented in the ISO 8601 date-time format: _yyyy_-_mm_-_dd_T_hh_:_mm_:_ss.fff_Z, where:

  - _yyyy_ = year
  - _mm_ = month
  - _dd_ = day
  - T = time designator used to show the start of the time component
  - _hh_ = hour
  - _mm_ = minute
  - _ss_ = second
  - _fff_ = fractions of a second
  - Z = Zulu, which is another way to denote UTC

- **\#Fields**: Comma-delimited field names used in IRM log files.

    The IRM log stores each RMS transaction event on a single line, organized in comma-separated fields. The following table lists the fields in IRM logs for all server roles that have IRM features enabled.

    |Field|Description|
    |---|---|
    |**Date-time**|Lists the UTC timestamp.|
    |**Feature**|Lists the RMS client feature used. Valid values include: <ul><li>`RacClc`</li><li>`Template`</li><li>`Prelicense`</li><li>`UseLicense`</li><li>`Signature verification`</li><li>`ServerInfo`</li></ul>|
    |**Event-Type**|Lists the event type. Valid values include: <ul><li>`Acquire`: An RMS license or template is requested.</li><li>`Success`: An RMS license or template is acquired successfully.</li><li>`Exception`: An error has occurred.</li><li>`Queued`: A request is pending.</li></ul>|
    |**Tenant-Id**|Reserved for internal Microsoft use.|
    |**Server-url**|Lists the RMS server URL accessed during the operation.|
    |**Context**|Used by the calling process to tie multiple RMS transactions together. Valid values include: <ul><li>`MessageID: \<Actual message ID>`</li><li>`MailboxGuid: \<Mailbox GUID>`</li><li>`AttachmentFileName: \<File name>`</li></ul>|
    |**Transaction-id**|Identifies a unique transaction. All events that occur during one transaction have the same transaction ID.|

## Managing IRM logs

On each server role that has IRM features enabled, IRM logging is enabled by default. For each server role, you can modify the following IRM log configuration by using the server role's corresponding **Set** cmdlet. For example, to configure IRM logging on a Mailbox server, you use the **Set-MailboxServer** cmdlet.

### Configuration parameters for IRM logs

|Parameter|Description|
|---|---|
|_IrmLogEnabled_|Enables logging of IRM transactions. IRM logging is enabled by default. To disable IRM logging for a server role, set the parameter to `$false`.|
|_IrmLogMaxAge_|Specifies the maximum age for an IRM log file. Files older than the specified age are deleted. The default value is `30.00:00:00` (30 days).|
|_IrmLogMaxDirectorySize_|Specifies the maximum size of all IRM logs in the connectivity log directory. When a directory reaches its maximum file size, the server deletes the oldest log files first. The default value is `250 MB`.|
|_IrmLogMaxFileSize_|Specifies the maximum file size for a single log file. When a file reaches the specified size, a log file is created, and the instance number is incremented. The default value is `10 MB`.|
|_IrmLogPath_|Specifies the IRM log location. The default path is %ExchangeInstallPath%Logging\IRMLogs.|

For detailed syntax and parameter information, see the following topics:

- [Set-MailboxServer](/powershell/module/exchange/Set-MailboxServer)
- [Set-ClientAccessServer](/powershell/module/exchange/Set-ClientAccessServer)
- [Set-TransportService](/powershell/module/exchange/Set-TransportService)
