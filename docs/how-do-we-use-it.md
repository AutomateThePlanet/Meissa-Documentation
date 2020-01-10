---
layout: default
title:  "How Do We Use It?"
excerpt: "Provides detailed steps how to start using MEISSA. Find detailed explanations for all keywords, available arguments."
date:   2020-01-09 07:50:17 +0200
permalink: /how-do-we-use-it/
---
All commands are executed in command line interface CLI.

1. **Start MEISSA server**
```
meissaserver
```

To coordinate all agents and runners MEISSA has its own server with its own DB.


2. **Start MEISSA test agent**
```
meissa agent --tag="APIAgent" --server="http://IPServerMachine:89"
```

Start MEISSA in **test agent mode** on all machines that you want to be agents. Depending on their resources and what will be executed, prefer scenarios where the test agent runs there in isolation. 
```
--server="http://IPServerMachine:89"
```
Make sure to set the correct test server URL. It is formed by the IP of the machine where you have started MEISSA in server mode.
```
--tag="APIAgent"
```
You mark all of your test agents with specific tags. It is so because you can have multiple test agents connected to a single test server. However, you can run tests only on test agents with a specific tag (a machine that has a particular program installed, etc.). The computers where you execute your UI tests will be most probably stronger than the one that you run your API tests and has additional software installed.

Usually, you have one test agent per machine. So, the more machines you have, the faster your tests will be executed.

**Example:**
```
meissa agent --tag="API" --server="http://00.220.159.255:89"
```

3. **Start MEISSA tests runner**
```
meissa runner --results="pathToResults\result.trx" 
--tag="API" --testTechnology="MSTest" 
--library="pathToBuildedFiles\SampleTestProj.dll" --server="http://IPServerMachine:89"
```

Usually, you start the runner from CI job. The typical workflow will be: 
- Download the tests source code
- Build it
- Execute tests with MEISSA
- Publish the results produced by MEISSA

```
--results="pathToResults\result.trx
```
The path where the results produced by the runner will be saved. The file should be in the expected file format. **TRX** is the file format for .NET tests. In Java, the extension will be different.

```
--testTechnology="MSTest"
```
The testTechnology argument is used to point which native runner should MEISSA use. Right now you can choose between **MSTest** and **NUnit** (NodeJs Jasmine runner comming soon). Later, more options will be added. The parameter is required since depending on the technology used the localization of tests, merge of tests results, test executions are different. With the MSTest switch you can test .NET framework and .NET Core projects.
```
--library="pathToBuildedFiles\SampleTestProj.dll"
```

**Example:**
```
meissa runner --results="D:\MyRuns\result.trx"
--tag="API" --testTechnology="MSTest" 
--library="D:\MyRuns\BuildFiles\BellatrixTests.dll" --server="http://00.220.159.255:89"
```
### Advanced Parameters ###
```
--nativeRunnerArguments="--verbosity="detailed""
``` 

This argument will pass the value to the native tests runner. Here, we tell the .NET Core tests runner to produce a detailed log.

```
--retriedResults="pathToResults\retriedResult.trx" --retries=3 --threshold=90
```

With this one, we tell MEISSA to retry the failed tests three times if less than 5% of all tests have failed.
```
--filter="test.FullName != \"TestName\" AND !test.Categories.Contains(\"CI\")"
```

MEISSA has a built-in complex test filter parser, and we can write complex queries to filter the tests.
```
--customArguments="BuildNumber=42" 
```
**Note**: To use this feature make sure the runner and the agent are with **Administrative permissions**!

Sometimes, it is useful to pass data to tests from the runner. For example, you may need to pass the current build number, so you can create a folder in your tests. If you use this argument, each agent will create an environmental variable with the name BuildNumber, and it will assign the value 42. After that, it is easy to get the value from your tests.
```
--timeBased
```

Instructs MEISSA to balance the tests, not based on the count, but on the previous execution times of the tests. To use it, you need to execute all your tests at least one time. This is quite useful because some of your UI tests may execute for 1 minute or more but most of them for 30 seconds or less. As I mentioned, the whole tests execution time is equal to the slowest sub-run. This feature can sometimes drastically decrease the execution time. 
```
--parallelRun
```

Tells MEISSA to execute the tests on each agent in parallel. You can even specify how many processes to spawn. This is most useful for unit, API, or headless UI tests. 
```
--sameMachineByClass
```

The tests from a single class will be executed on the same machine.