---
layout: default
title:  "Requirements"
excerpt: "Requirements"
date:   2020-01-07 09:50:17 +0200
permalink: /requirements/
anchors:
  prerequisites: Prerequisites
---
**Supported Operating Systems**: Windows, Linux, OSX

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

The typical usage of RAM is much lower < 500 MB. It depends on what type of tests you want to execute. If you need to run lots of UI tests, the RAM usage is much higher since MEISSA starts in its process the UI frameworks if they have higher RAM usage like WebDriver than MEISSA's RAM will increase as well. 
The same is valid for MEISSA runner mode since internally MEISSA uses native runners if they have higher RAM usage the overall MEISSA RAM usage will increase.

## Prerequisites  ##
You don't need to install any additional software such as Java or .NET Core. The same may not be valid for some of the native runners you may need to use.
The only thing **you need to have installed is some software for unzipping the MEISSA's files**.
