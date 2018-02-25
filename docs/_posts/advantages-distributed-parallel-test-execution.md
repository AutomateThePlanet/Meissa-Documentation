---
layout: default
title:  "Advantages Distributed Parallel Test Execution"
excerpt: "We have many moving parts- server, test agents, runner and so on. All of them use single command-line-interface; there are no separate installers or executables."
date:   2018-02-20 00:50:17 +0200
---
# Advantages Distributed Parallel Test Execution #

![Advantages Parallel Test Execution](https://i.imgur.com/L1fW4Lu.png)

The most obvious reason is speed. Instead of executing the tests in 16 hours, they complete in under 4 hours. When all required tests can be run a couple of times in a business day, you will be able to release your application every day (if we talk about web projects). Even if it is not web, you can improve the quality of your app and shorten the testing cycle by executing all your tests as part of the CI process. This means you will have higher coverage in shorter throughput time. As you know, each time your tests execute, their ROI (return of investment) increases. Last, the more often you run all your tests, the probability of locating unstable/not-well-written tests rises.
## Why Do You Need to Distribute Your Tests? ##

In our observation, the optimal number of test run processes is 1.5-2.0 x total number of cores on your machine, which means you are limited to the hardware you use. In the teams where I worked, most of the VMs had up to 2 CPUs. 
One "big" machine or many smaller? 
To answer to this question, I need to explain the difference between horizontal and vertical scale. Horizontal scaling means you scale by adding more machines to your pool of resources, whereas vertical scaling means you scale by adding more power (CPU, RAM) to an existing one. However, typically, many smaller machines are cheaper than one big one. The VM clouds nowadays are the mainstream. Because of that, the price differences are insignificant. 