---
layout: default
title:  "What is a Parallel Test Execution?"
excerpt: "Read more about the different types of parallel test execution."
date:   2018-02-20 11:50:17 +0200
permalink: /what-is-parallel-test-execution/
anchors:
  parallel-single-machine-multiple-processes: Parallel Testing
  distributed-testing-multiple-machines-single-process: Distributed Testing
  parallel-distributed-testing-multiple-machines: Distributed Parallel Testing
---
I can define at least three types of parallel test execution:

## Parallel- Single Machine - Multiple Processes ##
![Single Machine- Multiple Processes](https://github.com/AutomateThePlanet/Meissa-Documentation/blob/master/images/single-machine-multiple-processes.png)
Some unit test frameworks have native parallel execution support like NUnit. You have, for example, 100 tests. If you run them on a single machine with 5 CPU Cores, on each core, 20 tests will be executed simultaneously. However, not all test frameworks support this option, and there are some major problems related to it, depending on the type of tests you want to run.
## Distributed Testing- Multiple Machines - Single Process ##
![Multiple Machines- Single Process](https://github.com/AutomateThePlanet/Meissa-Documentation/blob/master/images/multiple-machines-single-processes.PNG)
Your second option is to run your tests at the same time on multiple machines and merge the results at the end. Usually, you need an additional complex tooling, for example Microsoft Test Controller/Agents setup.
## Parallel Distributed Testing- Multiple Machines - Multiple Processes ##
![Multiple Processes- Multiple Processes](https://github.com/AutomateThePlanet/Meissa-Documentation/blob/master/images/multiple-machines-multiple-processes.PNG)
You can mix both approaches. In this case, you will use the complex tooling and, at the same time, run the tests in parallel on each machine. 
