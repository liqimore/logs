---
layout: post
title: "How to completely uninstall/remove cygwin from windows10"
description: "remove cygwin from windows10"
date: 2021-08-17
tags: [cygwin, gcc]
categories: [notes]
comments: true
---

## Steps for removal

1. open powershell with administrator privilege.

2. run `takeown /f PATH_TO_CYGWIN /r /d y`, for me, PATH_TO_CYGWIN = C:\ENVs\MinGW64.

3. run `icacls PATH_TO_CYGWIN /t /grant everyone:F`

4. run `del PATH_TO_CYGWIN`.

5. check if the files are deleted and delete the residual files. 
