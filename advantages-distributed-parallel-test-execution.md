# Advantages Distributed Parallel Test Execution #

![Advantages Parallel Test Execution](https://i.imgur.com/L1fW4Lu.png)

The most obvious reason is the speed. Instead of executing the tests in 16 hours, they complete under 4. When all required tests can be run a couple of times in a business day, then you will be able to release your application every day (if we talk about web projects). Even if it is not web, you can improve the quality of your app and shorten the testing cycle by executing all of your tests as part of the CI process. This means that you will have higher coverage in short throughput time. As you know, each time your tests execute their ROI (return of investment) increases. Last but not least the more often you run all of your tests the probability to locate unstable/not-well-written tests raises.
## Why Do You Need to Distribute Your Tests? Horizontal vs Vertical Scaling ##

Why do you need to distribute your tests? In our observation, the optimal number of test run processes is 1.5-2.0 x total number of cores on your machine. Which means you are limited to the hardware you use. In the teams where I worked most of the virtual machines had up to 2 CPUs.
One "big" machine or many smaller? 
To answer to this question I need to explain the difference between horizontal and vertical scale. Horizontal scaling means that you scale by adding more machines to your pool of resources whereas vertical scaling means that you scale by adding more power (CPU, RAM) to an existing one. However, typically many smaller machines are cheaper than one big one. The VM clouds nowadays are the mainstream. Because of that, the prices differences are not so significant. 
