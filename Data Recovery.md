---
title: Data Recovery
lastmod: 2025-12-12T15:54:29-05:00
---
# Data Recovery
This is a very underdeveloped page. I have a lot more to say but haven't written it yet.
## SMART Test
The easiest way to run do these tests is with a linux live environment. You should use one that has smartmontools already installed; I like Knoppix. Lean and convenient.
```bash
# if you need to install
sudo apt install smartmontools
# short test; defaults to background
sudo smartctl -t short /dev/sda
# short test in foreground (blocks until done)
sudo smartctl -t short -C /dev/sda
# long test on another drive
sudo smartctl -t long -C /dev/sdc
```
## Viewing SMART Data
```bash
# view data
sudo smartctl -a /dev/sda | less
# view even more data
sudo smartctl -x /dev/sda | less
```
## Recovery
If you believe your drive could be failing, __DON'T__ run tests on it. The more you use it, the more you push a precarious situation closer to failure. You should always immediately clone all data on the drive as fast as possible.
```bash
# example of cloning with dd, first with default 3 retries
# next example command of running dd with many retries to pick up last fragments
```
