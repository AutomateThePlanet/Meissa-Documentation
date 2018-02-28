---
layout: default
title:  "Requirements"
excerpt: "Requirements"
date:   2018-02-20 09:50:17 +0200
permalink: /requirements/
anchors:
  prerequisites: Prerequisites
---
**Supported Operating Systems**: Windows, Linux, MacOS

Depending on what you want to execute with the test runner the requirements will vary. 

**Optimal** Requirements for Test Agent (if nothing else is running)
2 GB RAM
2 CPU Cores
**Best**
4 GB RAM

**Optimal** Requirements for Test Server (if nothing else is running)
2 GB RAM
2 CPU Cores
**Best**
4 GB RAM

**Optimal** Requirements for Test Runner (if nothing else is running)
1 GB RAM
2 CPU Cores
**Best**
2 GB RAM

The typical usage of RAM is much lower < 500 MB. It depends on what type of tests you want to execute. If you need to run lots of UI tests, the RAM usage is much higher since Meissa starts in its process the UI frameworks if they have higher RAM usage like WebDriver than Meissa's RAM will increase as well. 
The same is valid for Meissa runner mode since internally Meissa uses native runners if they have higher RAM usage the overall Meissa RAM usage will increase.

## Prerequisites  ##
You don't need to install any additional software such as Java or .NET Core. The same may not be valid for some of the native runners you may need to use.
The only thing **you need to have installed is some software for unzipping the Meissa's files**.
