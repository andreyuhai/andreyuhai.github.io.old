---
layout: post
title: "Batch file in Windows Task Scheduler does not run"
excerpt: "Creating a scheduled task in Windows Task Scheduler to run a batch file is relatively straight forward, or so I thought."
modified: 2014-07-15 20:55:22 +0300
categories: articles
tags: [bat, batch, windows, windows task scheduler, schedule]
image:
  feature:
  credit:
  creditlink:
comments: true
share: true
published: true
aging: true
---

Creating a scheduled task in Windows Task Scheduler to run a batch file is relatively straight forward. Create a batch file, open task scheduler, create a new task, add an action and create a trigger. After running the task, nohting happened. Eventually it turned out to shcedule a `.bat` file to run, I had to specify the "Start in (optional)" path when choosing which program to run. It should point to the folder containing the `.bat` file. Otherwise the `.bat` file will not run.
