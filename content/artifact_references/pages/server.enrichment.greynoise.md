---
title: Server.Enrichment.GreyNoise
hidden: true
tags: [Server Artifact]
---

Submit an IP to the GreyNoise Community API.

https://developer.greynoise.io/reference/community-api

This is a rather simple artifact that can be called from within another artifact (such as one looking for network connections) to enrich the data made available by that artifact.

Ex.

  `SELECT * from Artifact.Server.Enrichment.GreyNoise(IP=$YOURIP)`


```yaml
name: Server.Enrichment.GreyNoise
author: Wes Lambert -- @therealwlambert
description: |
  Submit an IP to the GreyNoise Community API.

  https://developer.greynoise.io/reference/community-api

  This is a rather simple artifact that can be called from within another artifact (such as one looking for network connections) to enrich the data made available by that artifact.

  Ex.

    `SELECT * from Artifact.Server.Enrichment.GreyNoise(IP=$YOURIP)`


type: SERVER

parameters:
    - name: IP
      type: string
      description: The IP to submit to GreyNoise.
      default:

sources:
  - query: |
        LET URL <= 'https://api.greynoise.io/v3/community/'

        LET Data = SELECT parse_json(data=Content) AS GreyNoiseLookup
        FROM http_client(url=URL + IP,
                         headers=dict(`Accept`="application/json"),
                         method='GET')

        SELECT
            GreyNoiseLookup.ip AS IP,
            GreyNoiseLookup.classification AS Classification,
            GreyNoiseLookup.name AS Name,
            GreyNoiseLookup.riot AS Riot,
            GreyNoiseLookup.noise AS Noise,
            GreyNoiseLookup.last_seen AS LastSeen,
            GreyNoiseLookup.link AS Link,
            GreyNoiseLookup AS _GreyNoiseLookup
        FROM Data

```
