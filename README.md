# WHIDS

Very flexible Host IDS designed for Windows. We are making
use of a previously developped rule engine [Gene](https://github.com/0xrawsec/gene)
designed to match Windows events according to custom rules. The
rules are simple to write and easy to understand so that everyone can
identify why a rule has triggered.

With the democratisation of Sysmon, this tools is perfect to quickly build
hunting rules or simply monitoring rules to screen things of interest happening on your
machine(s). With WHIDS you don't have to bother with an over
complicated Sysmon configuration which often turns to the nightmare when you want
to be very specific.The simplest thing is just to enable all the logging
capabilites of Sysmon and let WHIDS do his job, grab a coffee and wait
for the juicy stuff to happen. The tool has a low overhead for the system,
according to our current benchmarks.

This tool can be used on any Windows machine so you might install it either on
regular workstations or on Windows Event Collectors where you are receiving
all the logs of your infrastructure. The output format is nothing else than
JSON so it is very easy to handle the alerts generated by the HIDS in whatever
tool you want to use for this purpose like ELK, Splunk or simply your favourite
SIEM.

# Example

Here is an example of a rule designed to catch suspicious access to *lsass.exe*
as it is done by the well known Mimikatz credential dump tool.

```json
{
  "Name": "MaliciousLsassAccess",
  "Tags": ["Mimikatz", "Credentials", "Lsass"],
  "Meta": {
    "EventIDs": [10],
    "Channels": ["Microsoft-Windows-Sysmon/Operational"],
    "Computers": [],
    "Traces": [],
    "Criticality": 10,
    "Author": "0xrawsec"
  },
  "Matches": [
    "$ct: CallTrace ~= 'UNKNOWN'",
    "$lsass: TargetImage ~= '(?i:\\\\lsass\\.exe$)'"
  ],
  "Condition": "$lsass and $ct"
}
```

You can find a bunch of other rules as well as a quick introduction to the
syntax of the rules on the [Gene repository](https://github.com/0xrawsec/gene-rules).

# Demo

Running WHIDS with an already running Powershell Empire agent which invokes
Mimikatz module.

![WHIDS Mimikatz Demo](https://github.com/0xrawsec/whids/blob/master/demo/whids.gif)

Herafter is the kind of output returned by WHIDS. An additional section is added to the
JSON event where the criticality of the alert is reported along with the different signatures
which matched the event.

```json
{
  "Event": {
    "EventData": {
      "CallTrace": "C:\\Windows\\SYSTEM32\\ntdll.dll+4bf9a|C:\\Windows\\system32\\KERNELBASE.dll+189b7|UNKNOWN(00000000259123BC)",
      "GrantedAccess": "0x1410",
      "SourceImage": "C:\\Windows\\system32\\WindowsPowerShell\\v1.0\\powershell.exe",
      "SourceProcessGUID": "{49F1AF32-DD18-5A72-0000-0010042C0A00}",
      "SourceProcessId": "2248",
      "SourceThreadId": "3308",
      "TargetImage": "C:\\Windows\\system32\\lsass.exe",
      "TargetProcessGUID": "{49F1AF32-DB3B-5A72-0000-001013690000}",
      "TargetProcessId": "492",
      "UtcTime": "2018-02-01 11:24:53.331"
    },
    "GeneInfo": {
      "Criticality": 10,
      "Signature": [
        "MaliciousLsassAccess"
      ]
    },
    "System": {
        "Classical Windows Event System Section": "..."
    }
  }
}
```

# Documentation

Please visit: https://rawsec.lu/doc/gene/1.3/

# Changelog

## v1.3
  * **Event Hook** introduction
    * Can modify the events before going through detection engine
    * Created hooks to overcome domain name resolution issue
    * Implemented hooks to enrich Sysmon events 1, 6 and 7 with the size of the PE image
    * Implemented several other hooks
  * Can run in **service mode**:
    * restart in case of failure
    * log alerts to compressed file and rotate file automatically
    * log messages to a file
  * Installation script
    * creates a scheduled start running at boot to start Whids
    * agenerate an uninstall script dropped in the install folder
  * Number of new command lines arguments
    * **-hooks**: control event hook activation
    * **-protect**: dummy protection against crypto-locker (can be seen as a nice POC of event hooks)
    * **-all**: option to enable logging of **all** the events coming from the monitored channels
    should not be used in production, it is more for debugging purposes
    * ...
  * Some minor code refactoring

## v1.2
  * Log to Windows Application channel
  * Updated with latest version of gene so it benefits of its new features
    * "Match extracts" feature to match parts of event fields against containers (blacklist/whitelist)
  * New channel Alias to Microsoft-Windows-DNS-Client/Operational
  * Command line switch to enable DNS client logs (Microsoft-Windows-DNS-Client/Operational log channel)
