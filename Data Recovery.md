---
title: Data Recovery
lastmod: 2024-08-01T11:47:29-05:00
---
# Data Recovery
This is a very underdeveloped page. I have a lot more to say but haven't written it yet.
## S.M.A.R.T.
The easiest way to run do these tests is with a linux live environment. You should use one that has smartmontools already installed; I like Knoppix. Lean and convenient.
```bash
# example of triggering short test on drive
```
## Recovery
If you believe your drive could be failing, __DON'T__ run tests on it. The more you use it, the more you push a precarious situation closer to failure. You should always immediately clone all data on the drive as fast as possible.
```bash
# example of cloning with dd, first with default 3 retries
# next example command of running dd with many retries to pick up last fragments
```