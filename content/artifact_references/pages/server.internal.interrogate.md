---
title: Server.Internal.Interrogate
hidden: true
tags: [Server Event Artifact]
---

An internal artifact used track new client interrogations by the
Interrogation service.


```yaml
name: Server.Internal.Interrogate
description: |
  An internal artifact used track new client interrogations by the
  Interrogation service.

type: SERVER_EVENT

sources:
  - query: |
      SELECT * FROM foreach(
          row={
             SELECT ClientId, Flow, FlowId
             FROM watch_monitoring(artifact='System.Flow.Completion')
             WHERE Flow.artifacts_with_results =~ 'Generic.Client.Info'
          },
          query={
            SELECT * FROM switch(
              a={
                  SELECT ClientId,
                    FlowId,
                    Architecture,
                    BuildTime,
                    Fqdn,
                    Hostname,
                    KernelVersion,
                    Labels,
                    Name,
                    OS,
                    Platform,
                    PlatformVersion
                 FROM source(
                    client_id=ClientId,
                    flow_id=FlowId,
                    source="BasicInformation",
                    artifact="Custom.Generic.Client.Info")
               },
            b={
                SELECT ClientId,
                  FlowId,
                  Architecture,
                  BuildTime,
                  Fqdn,
                  Hostname,
                  KernelVersion,
                  Labels,
                  Name,
                  OS,
                  Platform,
                  PlatformVersion
               FROM source(
                  client_id=ClientId,
                  flow_id=FlowId,
                  source="BasicInformation",
                  artifact="Generic.Client.Info")
            })
          })

```
