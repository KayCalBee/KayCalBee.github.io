---
title: "OhSINT"
date: 2022-12-01
tags: ['tryhackme', 'OSINT', 'Writeup']
draft: false
---

# OhSINT

___

[OhSINT](http://tryhackme.com/room/ohsint), a room meant to test your skills using the OSINT framework.
<!--more-->

The room provides you with an image (pictured below), meant to be investigated with the OSINT framework to answer the questions. 

{{< figure src="windowsxp.jpg" attr="WindowsXP.jpg" >}}

___

We'll start with running the image through exiftool.

(The -k argument keeps the window open after completing the process)

``` bash
./exiftool -k ~/windowsxp.jpg
```

```  
ExifTool Version Number         : 12.52
File Name                       : windowsxp.jpg
Directory                       : /home/kcb
File Size                       : 234 kB
File Modification Date/Time     : 2022:12:31 17:35:07-08:00
File Access Date/Time           : 2022:12:31 17:35:07-08:00
File Inode Change Date/Time     : 2022:12:31 17:35:07-08:00
File Permissions                : -rwxr-xr-x
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
XMP Toolkit                     : Image::ExifTool 11.27
GPS Latitude                    : 54 deg 17' 41.27" N
GPS Longitude                   : 2 deg 15' 1.33" W
Copyright                       : OWoodflint
Image Width                     : 1920
Image Height                    : 1080
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
Image Size                      : 1920x1080
Megapixels                      : 2.1
GPS Latitude Ref                : North
GPS Longitude Ref               : West
GPS Position                    : 54 deg 17' 41.27" N, 2 deg 15' 1.33" W
-- press RETURN --
```

A quick Google search of the Copyright name gives us 3 accounts we can further investigate:  

- [Twitter](https://twitter.com/owoodflint)  
- [WordPress](https://oliverwoodflint.wordpress.com/author/owoodflint/)  
- [GitHub](https://github.com/OWoodfl1nt/people_finder)  

On the Twitter page we find a profile picture of a *cat*.  
We also find a post containing their BSSID  
{{< tweet user="OWoodflint" id="1102220421091463168" >}}  

If we input this BSSID into [Wigle](https://wigle.net) and zoom in as far as possible, we're awarded 
with the SSID of the WAP he connected to: *UnileverWiFi*.  

The *GitHub* page tells us he's from *London*, and that his personal email is *OWoodflint@gmail.com*.  

The Wordpress page is a bit more interesting.  
At first, it shows that he's gone to *New York* for vacation, but if we select all the text on the webpage
we'll also find the word *pennYDr0pper.!* written in white text on the white background.  

We can also find this password by looking through the page source, located just passed the paragraph tags.  
``` html
<article id="post-3" class="post-3 post type-post status-publish format-standard hentry category-uncategorised">
		<header class="entry-header">
		<h1 class="entry-title"><a href="https://oliverwoodflint.wordpress.com/2019/03/03/the-journey-begins/" rel="bookmark">Hey</a></h1>	</header>	
	<div class="entry-content">
		
<p>Im in New York right now, so I will update this site right away with new photos!</p>



<p style="color:#ffffff;" class="has-text-color">pennYDr0pper.!</p>
```

___

## Congrats!

And that's it! We've solved the tryhackme OhSINT room!
