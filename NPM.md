---
title: NPM
lastmod: 2023-06-03T00:00:11-05:00
---
# NPM
## Upgrading
Somehow, updating to later versions of your packages is like fucking pulling teeth. Maybe one day I'll get it right.
```bash
# find out what's outdated
# this does not check max versions on dependencies
# just what versions are possible
npm outdated

# to simply update minor versions within your package.json specification
# this is in my experience, a useless waste of time
npm update

# to ACTUALLY accomplish what you want, use this tool
# it will actually force updates past what package.json specifies
npm i -g npm-check-updates
ncu -u

# examples of manually installing a newer version
# this is more work than just using the tool
npm i package@version
npm i gulp@latest
npm i @babel/eslint-parser@12.0.0

# when you can't install something because
# of a dependency, find out why it's installed
npm ls package
# example
npm ls color-string
```