# How Do We Use It? #

1. **Start Meissa server**
```
meissa.exe initServer
```

To coordinate all agents and runners Meissa has its own server with its own DB.

2. **Start Meissa test agent**
```
meissa.exe testAgent --testAgentTag="APIAgent" --testServerUrl="http://IPServerMachine:5000"
```

Start Meissa in **test agent mode** on all machines that you want to be agents. Depending on their resources and what will be executed, prefer scenarios where the test agent runs there in isolation. 
```
--testServerUrl="http://IPServerMachine:5000"
```
Make sure to set the correct test server URL. It is formed by the IP of the machine where you have started Meissa in server mode.
```
--testAgentTag="APIAgent"
```
You mark all of your test agents with specific tags. It is so because you can have multiple test agents connected to a single test server. However, you can run tests only on test agents with a specific tag (a machine that has a particular program installed, etc.). The computers where you execute your UI tests will be most probably stronger than the one that you run your API tests and has additional software installed.

Usually, you have one test agent per machine. So, more machines you have, faster your tests will be executed.

**Example:**
```
meissa.exe testAgent --testAgentTag="API" --testServerUrl="http://00.220.159.255:5000"
```

3. **Start Meissa tests runner**
```
meissa.exe runner --resultsFilePath="pathToResults\result.trx" --outputFilesLocation="pathToBuildedFiles" 
--agentTag="API" --testTechnology="MSTestCore" 
--testLibraryPath="pathToBuildedFiles\SampleTestProj.dll"
```

Usually, you start the runner from CI job. The typical workflow will be. Download the tests source code. Build it. Execute tests with Meissa. Publish the results produced by Meissa.

```
--resultsFilePath="pathToResults\result.trx
```
The path where the results produced by the runner will be saved. The file should be in the expected file format. **TRX** is the file format for .NET tests. In Java, the extension will be different.

```
--outputFilesLocation="pathToBuildedFiles"
```
Output files location is the place where the files of your built libraries are placed. They should be moved there after you build your project.

```
--testTechnology="MSTestCore"
```
The testTechnology argument is used to point which native runner should Meissa use. Right now you can choose between **MSTestCore**, **NUnit** and **MSTestFramework**. Later, more options will be added. The parameter is required since depending on the technology used the localization of tests, merge of tests results, test executions are different.
```
--testLibraryPath="pathToBuildedFiles\SampleTestProj.dll"
```
The path to the file where your tests are located, it is located in the same folder pointed with the **outputFilesLocation** argument.

**Example:**
```
meissa.exe runner --resultsFilePath="D:\MyRuns\result.trx" --outputFilesLocation="D:\MyRuns\BuildFiles\" 
--agentTag="API" --testTechnology="MSTestCore" 
--testLibraryPath="D:\MyRuns\BuildFiles\BellatrixTests.dll"
```
### Advanced Parameters ###
```
--nativeRunnerArguments="--verbosity="detailed""
``` 

This argument will pass the value to the native tests runner. For example, above we tell the .NET Core tests runner to produce detailed log.
```
--retriedResultsFilePath="pathToResults\retriedResult.trx" --retriesCount=3 --threshold=90
```

With this one we tell Meissa to retry the failed tests three times if less than 5% of all tests have failed.
```
--testsFilter="test.FullName != \"TestName\" AND !test.Categories.Contains(\"CI\")"
```

Meissa has built-in complex test filter parser and we can write complex queries to filter the tests.
```
--customArguments="BuildNumber=42" 
```

Sometimes it is useful to pass data to tests from the runner. For example, you may need to pass the current build number so that you can create a folder in your tests. If you use this argument each agent will create an environmental variable with the name BuildNumber and it will assign the value 42. After that, it is an easy job to get the value from your tests.
```
--timeBasedBalance
```

Instructs Meissa to balance the tests not based on the count but rather than on the previous execution times of the tests. To use it you need to execute all of your tests at least one time. This is quite useful because some of your UI tests may execute for 1 minute or more but most of them for 30 seconds or less. As I previously mentioned, the whole tests execution time is equal to the slowest sub-run. This feature can sometimes drastically decrease the execution time.
```
--runInParallel
```

Instructs Meissa to execute in parallel the tests on each agent. You can even specify how many processes to be spawn. This is most useful for unit, API or headless UI tests.