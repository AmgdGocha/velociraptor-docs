---
title: Windows.Events.ProcessCreation
hidden: true
tags: [Client Event Artifact]
---

Collect all process creation events.


```yaml
name: Windows.Events.ProcessCreation
description: |
  Collect all process creation events.

type: CLIENT_EVENT

parameters:
  # This query is fast but contains less data. If the process
  # terminates too quickly we miss its commandline.
  - name: eventQuery
    default: SELECT * FROM Win32_ProcessStartTrace

sources:
  - precondition:
      SELECT OS From info() where OS = 'windows'
    query: |
        // Get information about the process
        LET get_pid_query(Lpid) = SELECT Pid, Ppid, Name FROM if(condition=Lpid > 0,
        then={
            SELECT Pid, Ppid, Name FROM pslist(pid=Lpid)
        })

        // Build the call chain. Cache the results for a short time.
        LET pstree(LookupPid) = SELECT * FROM foreach(
          row=cache(func=get_pid_query(Lpid=LookupPid), key=str(str=LookupPid)),
          query={
              SELECT * FROM chain(
            a={
              SELECT Pid, Ppid, Name FROM scope()
            }, b={
              SELECT Pid, Ppid, Name FROM pstree(LookupPid=Ppid)
            })
        })

        LET call_chain(LookupPid) = SELECT Pid, Ppid, Name FROM pstree(LookupPid=LookupPid)

        // Convert the timestamp from WinFileTime to Epoch.
        SELECT timestamp(winfiletime=atoi(string=Parse.TIME_CREATED)) as Timestamp,
               Parse.ParentProcessID as PPID,
               Parse.ProcessID as PID,
               Parse.ProcessName as Name, {
                 SELECT CommandLine FROM pslist(pid=Parse.ProcessID)
               } AS CommandLine,
               {
                 SELECT CommandLine FROM pslist(pid=Parse.ParentProcessID)
               } AS ParentInfo,
               join(array=call_chain(LookupPid=Parse.ProcessID).Name, sep=" <- ") AS CallChain
        FROM wmi_events(
           query=eventQuery,
           wait=5000000,   // Do not time out.
           namespace="ROOT/CIMV2")

```
