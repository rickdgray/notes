---
title: Privacy
lastmod: 2023-07-16T19:05:14-05:00
---
# Privacy
## Purge Personal Data
On Windows, select all files, right click, properties, details tab. Click the blue link that says "Remove Properties and Personal Information" and follow the prompts.
## Reset Date and Time Metadata
On Windows, this is difficult to do using higher level programming languages like Python, so it's easiest to use PowerShell.

```powershell
(Get-Item hello.txt).lastwritetime=$(Get-Date "8/28/2018 2:35 am")
(Get-Item goodbye.txt).creationtime=$(Get-Date "11/16/2077 9:34 pm")
(Get-Item *dds.txt).lastaccesstime=$(Get-Date "5/21/2006 7:15 am")
```
Last access time can get immediately updated, so careful with clicking on it after purging.
## Purge EXIF data from Images
Python is easiest using the piexif package
```
import piexif
from PIL import Image

# open the image
img = Image.open("image.jpg")

# remove the EXIF data
exif_dict = piexif.load(img.info["exif"])
exif_dict.clear()
exif_bytes = piexif.dump(exif_dict)
img.save("image_without_exif.jpg", exif=exif_bytes)
```
