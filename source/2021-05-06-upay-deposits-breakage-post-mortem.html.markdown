---
title: UPAY deposits breakage post-mortem
date: 2021-05-06
tags: post-mortem
---

All UPAY deposits weren't automatically processed.

READMORE

# Date
2021-05-06

# Authors
Mark Chavez

# Status
Complete

# Summary
On May 6, 2021 in the morning, all customer deposits weren't getting approved
automatically due to a logic change that was made particularly in the admin
dashboard.

# Impact
All customers were waiting for the approval of deposits.

# Root causes
The logic change includes modifying the method `UserFiatDeposit#approvable?` to
also ensure that this deposit is not of type UPAY. It was able to accomplish
what was needed in the admin dashboard, but it also affected how UPAY deposits
were handled.

Before a deposit gets approved, we want to know if it's `approvable?` or not -- 
and so with the new code change, it always returned false which caused these 
deposits to stay in pending state.

# Resolution
Ramon T. reverted the deploy.

# Detection
Luis B. pinged the dev team on Slack.

# Lessons Learned

## What went wrong
The faulty change was deployed in the evening and so no one was monitoring
the deposits.

