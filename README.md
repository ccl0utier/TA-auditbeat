# README.txt

## Auditbeat Add-on for Splunk

Author: Christian Cloutier <ccloutier@splunk.com>

This TA was built based on work from [Daniel Wilson's](https://splunkbase.splunk.com/apps/#/author/daniel333) version on [Splunkbase](https://splunkbase.splunk.com/app/4436/).
I've reworked the sourcetypes and Common Information Model mappings to extend them significantly.

## Install
1. Install auditbeat and configure it to log locally (see below)
2. Install Splunk Universal Forwarder, restart
3. Configure this add-on's inputs (see `default/inputs.conf`)
4. Push this app to the Universal Forwarder, restart
5. Push this app to the indexers and heavy forwarders, restart
6. Push this app to any search heads, restart
7. Craft alerts and dashboards or use the ES/PCI Premium App
8. Happy Splunking! 

*Auditbeat Add-on for Splunk version 1.0.0*

- Auditbeat documentation, see [here](https://www.elastic.co/guide/en/beats/auditbeat/current/index.html)
- FIM reading, see [here](https://isc.sans.edu/forums/diary/What+to+watch+with+your+FIM/20897/)

## PCI
Deploy Auditbeat with the file integrity monitoring module (which is enabled by default) to alert personnel to unauthorized modification of critical system files, configuration files or content files. Configure the software to perform critical file comparisons at least weekly.

## Auditbeat Configuration
Configure your Auditbeat agent(s) as required. This add-on expects Auditbeat to log JSON events locally and that events are then picked up with a Splunk Universal Forwarder. 

Example Configuration (Note: the `file_integrity` module is enabled by default):

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

## FSCHANGE
The provided inputs included enables `fschange` [monitoring](https://docs.splunk.com/Splexicon:Filesystemchangemonitor) on the Auditbeat config file to fulfill potential audit requirements (configuration tampering, malicious intent, etc.).  

The included `fschange` in inputs.conf is a feature in Splunk that is deprecated but still works as of writing this (Splunk 9.x). If this config conflicts with any of your existing `fschange` stanzas in other apps you will get errors such as change alerts that are invalid. If you don't need or like it - simply disable it. 

If you prefer not to use it but want to ensure you have cross monitoring in place consider AIDE which comes with most Linux distributions.

## Sourcetypes / CIM mapping

The main sourcetype for all Auditbeat events is `elastic:auditbeat:log`. 

Relevant evant are then further sourcetyped as follows:

```
"action":"process_started"  ->  elastic:auditbeat:processes     (CIM: Endpoint.Processes)
"action":"process_stopped"  ->  elastic:auditbeat:processes     (CIM: Endpoint.Processes)
"action":"network_flow"     ->  elastic:auditbeat:network_flow  (CIM: Network_Traffic)
"action":"started-session"  ->  elastic:auditbeat:session_start (CIM: Network_Sessions.All_Sessions.Session_Start)
"action":"ended-session"    ->  elastic:auditbeat:session_end   (CIM: Network_Sessions.All_Sessions.Session_End)
"action":"started-service"  ->  elastic:auditbeat:services      (CIM: Endpoint.Services)
"action":"stopped-service"  ->  elastic:auditbeat:services      (CIM: Endpoint.Services)
"action":"user_login"       ->  elastic:auditbeat:user_login    (CIM: Authentication)
"module":"file_integrity"   ->  elastic:auditbeat:fim           (CIM: Endpoint.Filesystem)
```