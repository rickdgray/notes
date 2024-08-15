---
title: NPM
lastmod: 2024-08-14T17:12:43-05:00
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
## Versioning
| Syntax | Details | Example |
|---|---|---|
| version | Must match version exactly | "foo": "1.0.0" |
| >version | Must be greater than version | "foo": ">1.0.0" |
| >=version | Greater than or equal to version | "foo": ">=1.0.0" |
| <version | Must be less than version | "foo": "<1.0.0" |
| <=version | Less than or equal to version | "foo": "<=1.0.0" |
| \~version | Allow new patch versions (1.0.1) | "foo": "\~1.0.0" |
| \^version | Allow new minor or patch versions (1.1.1) | "foo": "\^1.0.0" |
| 1.2.x | 1.2.0, 1.2.1, etc., but not 1.3.0 | "foo": "1.0.x" |
| http://... | URL to tarball dependency | "foo": "http://asdf.com/asdf.tar.gz" |
| * | Matches any version | "foo": "*" |
| "" | Same as * | "foo": "" |
| version1 - version2 |  | "foo": "1.0.0 - 2.0.0" |
| range1 \|\| range2 | Same as >=version1 <=version2 | "foo": "1.0.0 \|\| 2.0.0" |
| git://... | Git URL Dependency | "foo": "git://github.com/npm/cli.git" |
| user/repo | GitHub package Dependency | "foo": "expressjs/express" |
| path/path/path | local path | "foo": "file:../foo" |
