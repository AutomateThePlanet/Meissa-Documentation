---
layout: default
title:  "Getting Started"
excerpt: "Getting Started"
date:   2018-02-20 04:50:17 +0200
---
Open CLI and execute
```
dotnet tool install --global meissaserver
```
This will install the MEISSA server as a global tool.
```
dotnet tool install --global meissa
```
This will install the MEISSA agent + runner as a global tool.

To update to the latest version execute:
```
dotnet tool update --global meissaserver
```
```
dotnet tool update --global meissa
```
**Note**: The new update feature is written in a way to preserve the data important for distributing your tests- like previous test execution times.

You need to choose which type of test execution you prefer:
- Parallel- single machine - multiple processes
- Distributed testing- multiple machines - single process
- Parallel distributed testing- multiple machines - multiple processes

For more information about the topic refer to the section- [**What is a Parallel Test Execution?**](what-is-parallel-test-execution.md)

Below you can find info about the different setup and examples how to start using MEISSA. However, for complete reference about all available options and their meanings refer to the [**How Do We Use It?**](how-do-we-use-it.md) section.

## Parallel- Single Machine - Multiple Processes ##

In this scenario, you will execute your test on a single machine.

- Start MEISSA in **server mode** on your machine. Execute in CLI.
```
meissaserver
```
- Start MEISSA in **test agent mode** on the same machine. Execute in CLI.
```
meissa testAgent --agentTag="APIAgent" --serverUrl="http://IPServerMachine:89"
```
- Run your tests with MEISSA **test runner mode** on the same machine. Execute in CLI.
```
meissa runner --resultsFilePath="pathToResults\result.trx"
--agentTag="APIAgent" --testTechnology="MSTest" 
--testLibraryPath="pathToBuildedFiles\SampleTestProj.dll" --serverUrl="http://IPServerMachine:89"
```

After the test execution, the runner will be closed. Do not close the server and the test agent they will be reused for the new runs.

## Distributed Testing- Multiple Machines - Single Process ##

In this scenario, you will execute your test on multiple machines.

- Start MEISSA in **server mode** on a dedicated machine. Depending on what resources does it have, you may consider the option to have a dedicated computer for the server or at least not start it on the same device where there is a test agent or runner. Please refer to the **requirements** section.
```
meissaserver
```
- Start MEISSA in **test agent mode** on all machines that you want to be agents. Depending on their resources and what will be executed, prefer scenarios where the test agent runs there in isolation. Make sure to set the correct test server URL. It is formed by the IP of the machine where you have started MEISSA in server mode.
```
meissa testAgent --agentTag="APIAgent" --serverUrl="http://IPServerMachine:89"
```
- Run your tests with MEISSA **test runner mode** on some of the machines or even better prefer starting it on a dedicated computer. Refer to the requirements section.
```
meissa runner --resultsFilePath="pathToResults\result.trx"
--agentTag="APIAgent" --testTechnology="MSTest" 
--testLibraryPath="pathToBuildedFiles\SampleTestProj.dll" --serverUrl="http://IPServerMachine:89"
```

After the test execution, the runner will be closed. Do not close the server and the test agent they will be reused for the new runs.

## Parallel Distributed Testing- Multiple Machines - Multiple Processes ##
All configurations are almost the same as the previous section except the running tests processes. 
To execute the tests in parallel, you have two options. If you use the parallel capabilities of the used native runner, then you don't have to do anything more. The tests will be executed in parallel on each test agent machine.
If you want to use MEISSA's parallel execution capabilities, you need to add an argument to the runner options.
```
--runInParallel
```
For more detailed information refer to the [**features**](features.md) and [**how it works internally**](how-does-it-work-internally.md) sections.