---
title: BloomX.app - stale prices
date: 2021-10-02
published: false
tags: post-mortem
---

Rates from Binance were 14 hours old before any action was taken

READMORE

# Date
2021-10-02

# Authors
Ramon Tayag

# Status
Complete

# Summary
On October 2, 2021, at about 5:00am, Justin David noticed that prices seed strange and turned off smaller capped cryptoassets and put the margin on some at 50%.

Upon investigation, I discovered that prices were stale -- the latest was around 15:30 from October 1.

# Impact
Since the app before the fix we deployed did not take into account the possibility that prices may be stale for whatever reason, it traded these prices with our customers, causing financial losses.

# Root causes
The root cause for the prices failing to update were not determined. When we deployed again, the prices began updating. However, there can be various reasons that prices don't get updated:

- Binance's price API goes down
- We hit our API limit (this has happened before) and get locked out of the API
- Our processes get busy and are unable to request for prices

# Resolution
Since there could always be reasons that the price may not get updated, the resolution of that resulted from this was to not continue if the rates are more than 2 minutes old. Currently, the app does not recover gracefully when this occurs (the users may see errors), but it is better than silently losing money.

# Detection
Justin David checked the prices at 5 a.m.

# Action Items

| Action Item                                                                  | Type | Owner | Bug                                                                             |
|------------------------------------------------------------------------------|------|-------|---------------------------------------------------------------------------------|
| [Expire binance prices](https://github.com/bloom-solutions/bloom21/pull/675) | code | RT    | [Basecamp](https://3.basecamp.com/4314124/buckets/24322653/messages/4211414035) |

# Lessons Learned

## What went well
- Notified early. Justin David had the presence of mind to check the prices early, having heard that the prices started to look off the day before.

## What went wrong
- The system assumed the price on the database was correct, even if it was stale.

## Where we got lucky
- ?

# Timeline
- 2021-10-02
  - ~05:00: Justin David (JD) checked the prices and they seemed off. He locked extra volatile coins and increased margins dramatically.
  - ~10:00: Ramon Tayag (RT) was aware of the situation
  - ~11:56: RT merges fix to expire prices when stale
