# Auditbeat Add-on for Splunk

## Release Notes

| Version               | Notes                                                                     |
|-----------------------|---------------------------------------------------------------------------|
| 1.0.0                 | Initial version                                                           |
|                       |                                                                           |
| 1.0.1                 | Minor fixes to some aliases and extractions                               |
|                       | Addition of the `ran-command` actions to the`elastic:auditbeat:processes` | 
|                       | sourcetype and corresponding CIM normalization                            |
|                       | Addition of the `elastic:auditbeat:packages` sourcetype and corresponding |
|                       | CIM normalization (Updates data model)                                    |

## Overview

### About the Auditbeat Add-on for Splunk

|                       |                                                                 |
|-----------------------|-----------------------------------------------------------------|
| Version               | 1.0.0                                                           |
| Vendor Products       | Elastic Auditbeat 8.3.x/8.4.x and above. Others are untested.   |
| Visible in Splunk Web | No.                                                             |

The **Auditbeat Add-on for Splunk** collects audit event data from Elastic Auditbeat. You can install the Add-on on a forwarder to send data from Auditbeat to a Splunk Enterprise indexer or group of indexers. You can also use the add-on to provide data for other apps, such as Splunk Enterprise Security or Splunk App for PCI Compliance.

The **Auditbeat Add-on for Splunk** collects the following data using file inputs:

- Audit events from Auditbeat to Splunk as they are written to the `file` output (see below).
- (Optionally) Changes to Auditbeat configuration file in at `/etc/auditbeat/auditbeat.yml` (Linux) or `C:\Program Files\Auditbeat\auditbeat.yml` (Windows).
- Contents of the Auditbeat.yml configuration file

### Source types for the Auditbeat Add-on for Splunk

The **Auditbeat Add-on for Splunk** provides index-time, search-time and CIM normalization for Auditbeat Audit events in the following formats:

| Source Type                | Description                                             | CIM Data Models    |
|----------------------------|---------------------------------------------------------|--------------------|
| elastic:auditbeat:log      | Main Auditbeat events                                   | n/a                |
| elastic:auditbeat:config   | Auditbeat configuration file contents                   | n/a                |


Sourcetype `elastic:auditbeat:log` is then sourcetyped further based on the following fields (triggers) to the following CIM data models:

| Trigger                     | Source Type                     | CIM Data Models                                  |
|-----------------------------|---------------------------------|--------------------------------------------------|
| "action":"process_started"  | elastic:auditbeat:processes     | Endpoint.Processes                               |
| "action":"process_stopped"  | elastic:auditbeat:processes     | Endpoint.Processes                               |
| "action":"ran-command"      | elastic:auditbeat:processes     | Endpoint.Processes                               |
| "action":"network_flow"     | elastic:auditbeat:network_flow  | Network_Traffic                                  |
| "action":"started-session"  | elastic:auditbeat:session_start | Network_Sessions.All_Sessions.Session_Start      |
| "action":"ended-session"    | elastic:auditbeat:session_end   | Network_Sessions.All_Sessions.Session_End        |
| "action":"started-service"  | elastic:auditbeat:services      | Endpoint.Services                                |
| "action":"stopped-service"  | elastic:auditbeat:services      | Endpoint.Services                                |
| "action":"user_login"       | elastic:auditbeat:user_login    | Authentication                                   |
| "action":"existing_package" | elastic:auditbeat:packages      | Updates                                          |
| "module":"file_integrity"   | elastic:auditbeat:fim           | Endpoint.Filesystem                              |

### Compatibility

This version of the add-on is compatible with the following platform, OS and CIM versions:

|                                  |                            |
|----------------------------------|----------------------------|
| Splunk Platform                  | 8.x and later              |
| CIM                              | 4.2 and later              |
| Supported OS for data collection | Any supported by Auditbeat |

## Installation

### Install the Auditbeat Add-on for Splunk

You can install the **Auditbeat Add-on for Splunk** with Splunk Web or from the command line. You can install the add-on onto any type of Splunk Enterprise instance (indexer, search head, or forwarder).

1. Download the add-on from [Github](https://github.com/ccl0utier/TA-auditbeat/releases) or alternatively clone the project using your `git` client.
2. Determine where and how to install this add-on in your deployment.
3. Perform any prerequisite steps before installing.
4. Complete your installation.

See Installing add-ons in Splunk Add-Ons for detailed instructions describing how to install a Splunk add-on in the following deployment scenarios:

- [Single-instance Splunk Enterprise](http://docs.splunk.com/Documentation/AddOns/released/Overview/Singleserverinstall)
- [Distributed Splunk Enterprise](http://docs.splunk.com/Documentation/AddOns/released/Overview/Distributedinstall)

### Distributed installation of this add-on

Use the tables below to determine where and how to install this add-on in a distributed deployment of Splunk Enterprise or any deployment for which you are using forwarders to get your data in. Depending on your environment, your preferences, and the requirements of the add-on, you may need to install the add-on in multiple places.

| Splunk instance type | Supported | Required    | Comments                                                                                            |
|----------------------|-----------|-------------|-----------------------------------------------------------------------------------------------------|
| Search Heads         | Yes       | Yes         | Install this add-on to all search heads where Auditbeat knowledge management is required.                 |
| Indexers             | Yes       | Yes         | Install this add-on to all indexers, it has index-time configurations.                              |
| Heavy Forwarders     | Yes       | Conditional | Required if you have Heavy Forwarders in your ingestion path or they perform data collection.       |
| Universal Forwarders | Yes       | Recommended | Install this add-on to the Universal Forwarders installed on your Auditbeat hosts to collect data.  |

### Distributed deployment compatibility

This table provides a quick reference for the compatibility of this add-on with Splunk distributed deployment features.

| Distributed deployment feature | Supported | Comments                                                         |
|--------------------------------|-----------|------------------------------------------------------------------|
| Search Head Clusters           | Yes       | N/A                                                              |
| Indexer Clusters               | Yes       | N/A                                                              |
| Deployment Server              | Yes       | Supported for deploying the configured add-on to multiple nodes. |


## Configuration

### Auditbeat Configuration
Configure your Auditbeat agent(s) as required. This add-on expects Auditbeat to log JSON events locally (`output.file`) so that events can then picked up with a Splunk Universal Forwarder. 

Example configurations (note: the `file_integrity` module is enabled by default):

```
auditbeat.modules:
- module: file_integrity
  paths:
  ...

### File outputs - Linux
output.file:
  enabled: true
  path: "/var/log/auditbeat/logs"
  filename: auditbeat

### File outputs - Windows
output.file:
  path: "C:/ProgramData/auditbeat/Logs"
  filename: auditbeat
```
### Enable data inputs for the Auditbeat Add-on for Splunk

After you have installed the **Auditbeat Add-on for Splunk**, you must enable the data and network inputs within the add-on so that it collects data from your Auditbeat environment.
You must enable the inputs using the configuration files.

> Note: When you configure data inputs using configuration files, copy only the input stanzas whose configurations you want to change. Do not copy the entire file, as those changes persist even after an upgrade.

1. Create `inputs.conf` in the `$SPLUNK_HOME/etc/apps/TA-auditbeat/local` directory.
2. Open `$SPLUNK_HOME/etc/apps/TA-auditbeat/local/inputs.conf` for editing.
3. Open `$SPLUNK_HOME/etc/apps/TA-auditbeat/default/inputs.conf` for editing.
4. Copy the input stanza text that you want to enable from the `$SPLUNK_HOME/etc/apps/TA-auditbeat/default/inputs.conf` file and paste them into the `$SPLUNK_HOME/etc/apps/TA-auditbeat/local/inputs.conf` file.
5. In the `$SPLUNK_HOME/etc/apps/TA-auditbeat/local/inputs.conf` file, enable the inputs that you want the add-on to monitor by setting the `disabled` attribute for each input stanza to `0` or alternatively removing it completely (the default is "enabled").
6. Save the `$SPLUNK_HOME/etc/apps/TA-auditbeat/local/inputs.conf` file.
7. Restart the instance.

The recommended approach is to deploy the **configured** add-on to the relevant nodes using a **Deployment Server**.  

### (Optional but recommended) Configure the Auditbeat Add-on for Splunk to send data to another index

You can (and likely should in most cases) send the collected data to a dedicated Splunk index.  
This can be achieved by creating the relevant index on your indexers and then adding:

```
index = myindexofchoice
```

... to the relevant input configuration stanza.  So if you wanted to send your Audit events data to an index named `auditbeat`, your configuration would look like this:

Linux
```
[monitor:///var/log/auditbeat/logs/auditbeat]
index=auditbeat
sourcetype=elastic:auditbeat:log
disabled=0
```

Windows
```
[monitor://C:\ProgramData\auditbeat\logs]
index=auditbeat
sourcetype=elastic:auditbeat:log
disabled=0
```

> Note: This works for all input stanzas, i.e. `monitor` inputs.

### Configuration Change Tracking

The provided inputs included can optionally enable `fschange` [monitoring](https://docs.splunk.com/Splexicon:Filesystemchangemonitor) on the Auditbeat config file to fulfill potential audit requirements (configuration tampering, malicious intent, etc.).  

This feature in Splunk is deprecated but still works as of writing this (Splunk 9.x). If this config conflicts with any of your existing `fschange` stanzas in other apps you will get errors such as change alerts that are invalid. If you don't need or like it - simply disable it and/or consider other options such as [AIDE](https://aide.github.io/) which comes with most Linux distributions.


## Troubleshooting

### General troubleshooting

For troubleshooting tips that you can apply to all add-ons, see [Troubleshoot add-ons](http://docs.splunk.com/Documentation/AddOns/released/Overview/Troubleshootadd-ons) in Splunk Add-ons.

## Credits, References & Notes

This add-on was loosely based on the original [Add-on](https://splunkbase.splunk.com/app/4436/) built by [Daniel Wilson](https://splunkbase.splunk.com/apps/#/author/daniel333). I've reworked the sourcetypes and Common Information Model mappings to extend them significantly.

Comments & suggestions for improvement are more than welcome!

 “Auditbeat” and "Beats" are trademarks of Elastic and they are in no way affiliated with this work.  Any rights, title and interest in these trademarks remains solely with them.

> Author: Christian Cloutier <ccloutier@splunk.com>
