---
layout:     post
title:      PNGs on a diet with WebP
date:       2015-10-11 12:31:19
summary:    Reduce Android app size with WebP
categories: WebP Android
---

Recently I was working on a project with a large amount of png files. Our designers had created a bunch of custom assets and animations for the project. After resizing the assets for different screen densities the project was left bloated from all the png files. It was compiling (with progaurd enabled) to over 150mb. Even though Google recently raised the apk size limit from 50mb to 100mb, we were still way oversized.

I had read a while back about [Google's webp](https://developers.google.com/speed/webp/?hl=en) image format and figured it would be worth a shot to try it out. Boy was I shocked. By converting our images from png format to webp format I was able to reduce our apk size from over 150mb to 34mb.

<blockquote>
	  <p>
	  I was able to reduce our apk size from over 150mb to 34mb.
	  </p>
</blockquote>


#### But what about compatibity? 
Webp is natively compatible with Android 4.0 and higher. This makes it the perfect solution for most modern projects.

On Windows you can install [this codec](https://developers.google.com/speed/webp/docs/webp_codec) to enable you to view the images in any photo viewer. Unfortunately there is no such plugin from Google for Mac. I did find this [tool](https://github.com/emin/WebPQuickLook) for Mac, but I haven't tried it.


#### Let's do it!

First you need to download the compiled binaries from [here](https://developers.google.com/speed/webp/docs/precompiled).
To convert your png files you'll want to run the cwebp executable. The [docs](https://developers.google.com/speed/webp/docs/cwebp) tell you eveything you need to know.


#### Warning!

The Google Play store does not allow an apk to use .webp for the app icon. In in the Mac Bash script below I've set it to skip mipmap directories because that is where my app icons are. The Windows batch script does not check for this.

Also, these scripts will overwrite your png file. Make sure to back them up!


#### Scripts

Here's a bash script I wrote to easily convert the entire project to webp. Just replace the two comments with your path structure.

{% highlight batch lineanchors %}

@echo off
setlocal enableextensions enabledelayedexpansion
echo Converting to webp
echo script by Shmuel Rosansky

echo Warning, this will convert app icons also!

echo Listing all png files..
SET /A COUNT=0

cd [path to your res folder]

for /r . %%g in (*.png) do (

echo %%g
echo Old Size
echo   %%~zg

[path to webp.exe]\webp.exe %%g -o %%g -q 80 -alpha_cleanup -alpha_filter best -m 6 -mt -af -short

ren %%g *.webp

SET /A COUNT+=1
echo File # !COUNT!
echo ----

 )
 endlocal
{% endhighlight %}


If you perfer Mac here's a bash script.

{% highlight bash lineanchors %}
find ./ -type f -name "*.png" |while read line
do  

if [[ $line == *"mipmap"* ]]
then
  continue
fi
  	./cwebp $line -o $line -q 80 -alpha_cleanup -alpha_filter best -m 6 -mt -af -short || { echo 'Convert to webp failed' ; exit 1; }
  	mv "$line" "${line%.png}.webp"
done
{% endhighlight %}



You can change the quality level by raising or lowering 80. I found no visible loss of image quality at 80. Remember, your users will be on small phone screens and will not be able to analyze every pixel like you can on your desktop.


