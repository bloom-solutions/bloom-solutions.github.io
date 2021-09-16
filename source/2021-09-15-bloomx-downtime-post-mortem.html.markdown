---
title: BloomX.app downtime
date: 2021-09-15
tags: post-mortem
---

bloomx.app went down for about 40 minutes

READMORE

# Date
2021-09-15

# Authors
Ramon Tayag

# Status
Complete

# Summary
On September 15, 2021, at ~5:08pm, bloomx.app went down. This occurred about ~30 minutes after a deployment that introduced the use of websockets.

# Impact
All activity on bloomx.app halted.

# Root causes
To isolate the issue, the app was rolled back to a previous deploy and it ran fine for a period of time. Not sure what the cause was, I deployed the version with websockets again, and this time, about 10 minutes later, the website went down. I quickly rolled back, and the site went back up after ~1 minute. This confirmed my suspicion that Heroku doesn't handle long-living connections such as websockets properly.

# Resolution
Undoing the websockets changes

# Detection
BetterUpdate monitoring service pinged Slack and sent emails.

# Action Items

| Action Item                                                                        | Type | Owner | Bug |
|------------------------------------------------------------------------------------|------|-------|-----|
| [Rollback websockets changes](https://github.com/bloom-solutions/bloom21/pull/577) | code | RT    | n/a |

# Lessons Learned

## What went well
- The code change that I had to do was simple; no migrations. Rolling back was simple.

## What went wrong
- We tested on staging, but because staging's setup is different from bloomx.app production, we are less likely to catch errors like these.

## Where we got lucky
N/A

# Timeline
- 2021-09-15
  - 17:08: bloomx.app goes down
  - 17:45: RT is aware, and rolls back
  - 17:50: bloomx.app back up
  - ~18:10: RT deploys websockets version again
  - 18:20: bloomx.app goes down
  - 18:22: RT rolls back
  - 18:25 bloomx.app back up
