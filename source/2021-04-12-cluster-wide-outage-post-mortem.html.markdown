---
title: Cluster-wide outage post-mortem
date: 2021-04-12
tags: post-mortem
---

All services hosted on our Kubernetes cluster on Google Cloud became inaccessible.

READMORE

# Date
2021-04-14

# Authors
Ramon Tayag

# Status
Complete, with pending action items

# Summary
On April 14, 2021, at about 9:40am, all services hosted on our Kubernetes cluster on Google Cloud became inaccessible. Because no direct action on the engineering's end triggered it, we had a difficult time pinpointing the issue and deploy a fix.

After ruling out different guesses, we arrived at the conclusion that the issue most likely lay in our load balancer that directs traffic to their respective apps (i.e. if someone visits bloomremit.net, the load balancer sends that connection over to the BloomRemit application; if someone visits trade.bloom.solutions, the load balancer sends that connection over to the BloomTrade app).

Instead of spending time figuring out why things broke without us changing anything, we decided to upgrade our load balancer software. This worked -- at 12:10pm, all services that were inaccessible due to this outage came back online.

# Impact
Trading and remittance activity halted, possibly no revenue lost because these were just delayed

# Root causes
We did not figure out the root cause (exactly why the issue started without us changing anything, and why updating Traefik, our load balancer, fixed the issue). Before upgrading, Traefik was at version 1.7.9. We updated it to the latest 1.7 version, 1.7.30. Looking at the the logs, we saw many of [these errors](https://console.cloud.google.com/logs/query;cursorTimestamp=2021-04-14T03:12:54.520817Z;query=resource.type%3D%22k8s_container%22%20resource.labels.cluster_name%3D%22bloom-general%22%20resource.labels.namespace_name%3D%22kube-system%22%20resource.labels.container_name%3D%22lb-traefik%22%0Atimestamp%3D%222021-04-14T03:12:53.515318Z%22%0AinsertId%3D%224gesh6fv4xtw1%22;timeRange=2021-04-14T03:12:57.842Z%2F2021-04-14T03:12:57.842Z--PT1H?project=bloom-general):

```
Failed to list *v1.Service: v1.ServiceList.Items: []v1.Service: v1.Service.ObjectMeta: v1.ObjectMeta.readObjectFieldAsBytes: expect : after object field, but found p, error found in #10 byte of
```

RT intuits that newly deployed Kubernetes services used some new API format that Traefik could not read. We did not, however, change the Kubernetes cluster version.

# Resolution
Upgrading Traefik to 1.7.30

# Detection
Monitoring service pinged Slack and RT's phone (registered as a device in Google Cloud Monitoring)

# Action Items
| Action Item                                 | Type    | Owner | Bug |
|---------------------------------------------|---------|-------|-----|
| Update SOP: keep server software up to date | process | RT    | n/a |

# Lessons Learned
## What went well
- Notified early: we got notified ahead of time that something was wrong when BXA staging broke. We had time to rule other things out and detect patterns.

## What went wrong
- Didn't test if problem was system-wide: we should have deployed something else when we first saw the issue to see if it would affect all services once restarted

## Where we got lucky
- Traefik v1.7.30 fixed the issue

# Timeline
- 2021-04-13
  - 4:16: Cecille Manalang (CM) deploys bloom21/BXA staging and sees 404 when visiting the BXA staging website
  - 4:38: CM and RT go on a call to figure out the issue
  - ~18:30: CM and RT end the call and decide to figure things out the next day
- 2021-04-14
  - 8:30: RT resumes diagnosing the issue
  - ~9:10: CM joins RT
  - 9:50: all services restart (probably due to automated maintenance), causing all of them to be undiscoverable just like BXA staging
  - 9:54: LV joins CM and RT in a Zoom call
  - 12:09: Traefik 1.7.30 deployed, resolving issue
