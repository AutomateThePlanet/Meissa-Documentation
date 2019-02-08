---
layout: default
title:  "How to Develop Plugins?"
excerpt: "See what type of plugins we support and how you can execute your code in Meissa's execution pipeline."
date:   2018-02-20 08:50:17 +0200
permalink: /how-to-develop-plugins/
anchors:
  test-technology-plugins: Test Technology Plugins
  extensibility-points-plugins: Extensibility  Points Plugins
  how-to-use-already-created-plugins: Already Created Plugins?
---
Meissa exposes two types of plugins.

- Test Runner Plugins
- Extensibility Points Plugins

## Test Technology Plugins ##
Test runner plugins are used for various tasks inside Meissa. For example- locating tests for a specific technology, reading the result file, merging files, getting passed/failed tests, etc. 
For each test technology that Meissa offers there is a separate test runner plugin.
If Meissa doesn't offer your test technology, you can build a new plugin quickly for it. The code of the plugins is in C#, but you can execute executables written in any language. If you don't know who to do it just contact us on Gitter, refer to the [**Get Involved**](get-involved.md) section.

### Steps to Write New Plugin ###
1. Create a new C# .NET Core/.NET Standard library project
2. Install **Meissa.Plugins.Contracts** NuGet Package
3. Implement the two test runner plugin interfaces- **INativeTestsRunnerTestCasesPluginService** and **INativeTestsRunnerPluginService**

You can find two already built plugins in the [GitHub repository](https://github.com/angelovstanton/Meissa)- **Meissa.Plugins.MSTest** and **Meissa.Plugins.NUnit**

## Extensibility Points Plugins ##
The extensibility points plugins offer way to plug in your own logic in various points of the execution pipeline of Meissa runner and test agents.
**Test Agent Points:**
- before test run
- after test run
- on test run abortion

**Test Runner Points:**
- before test run
- after test run

### Steps to Write New Plugin ###
1. Create a new C# .NET Core/.NET Standard library project
2. Install **Meissa.Plugins.Contracts** NuGet Package
3. Implement the two test runner plugin interfaces- **ITestAgentPluginService** and **ITestRunnerPluginService**
## How to Use Already Created Plugins? ##

Build your plugin project then copy the output files to the Meissa Plugins folder.

## Sample Plugin Implementation - MSTest ##
### NativeTestsRunnerPluginService ###
```csharp
[Export(typeof(INativeTestsRunnerPluginService))]
public class NativeTestsRunnerPluginService : INativeTestsRunnerPluginService
{
    public string Name => "MSTest";

    public string RunnerFile => "dotnet";

    public string ResultsFileExtension => "trx";

    public List<string> RunnerProcessesNamesToKill => new List<string> { "dotnet" };

//... rest of code
}
```
The first class that you need to create is **NativeTestsRunnerPluginService**. It needs to implement the **INativeTestsRunnerPluginService** interface. Also, don't forget to set the Export attribute.
Next, you need to set the Name for the plugin technology- MSTest, NUnit, Protractor, etc.
For runner file, you need to set a path to the runner that will execute the tests from CLI. In the case of .NET, we use the native dotnet command. If you use another downloadable runner, you need to place it inside the plugins project and make sure it is copied to the plugins folder or provide the full path to the runner.

Here is the example for NUnit:
```csharp
public string RunnerFile => Path.Combine(GetRunningAssemblyPath(), "Plugins\\nunit-native-runner\\nunit3-console.exe");
```
The **nunit-native-runner** folder is part of the **Meissa.Plugins.NUnit** project, and it is set to be always copied to the output folder. If you want to use the plugin later, you need to copy it to the Meissa Plugins folder.
```csharp
public string ResultsFileExtension => "xml";
```
For **ResultsFileExtension** you need to set the correct file extension for the results file produced by the native test runner.

The first method that you need to implement is **BuildNativeRunnerArguments**.
Since the tool starts multiple background threads in which it calls the native runner, we call the native runner from CLI (command line interface).
```csharp
public string BuildNativeRunnerArguments(string libraryName, string libraryPath, List<TestCase> testCasesToBeExecuted, string testResultsFilePath, string outputFilesDir, string nativeArguments)
{
    var testCaseFilter = CreateRunFilterArgument(testCasesToBeExecuted).FirstOrDefault();
    var testFilterArg = string.IsNullOrEmpty(testCaseFilter) ? string.Empty : $"--testcasefilter:\"{testCaseFilter}\"";
    var arguments = $"vstest \"{Path.Combine(outputFilesDir, libraryName)}\" --logger:trx;LogFileName=\"{testResultsFilePath}\" {nativeArguments} {testFilterArg}";

    return arguments;
}
```
The runner passes the following parameters to the method:
- libraryName - the name of the test project
- libraryPath - full path to the DLL folder
- testCasesToBeExecuted - a collection of C# object representing the test cases to be executed
- testResultsFilePath - the full path to the file where the test results should be saved
- outputFilesDir - the folder where all test DLLs are located
- nativeArguments - any additional arguments passed from CLI

Usually, in this method, we should build an argument/test list that we pass to the native test runner to tell which tests to be filtered, e.g. executed. For example, for MSTest we use the "--testcasefilter:" argument. For NUnit, we create a temp file containing the name of the tests that should be executed- tests list file.

```csharp
public object DeserializeTestResults(string originalRunTestResults) => Deserialize<TestRun>(originalRunTestResults);

public string SerializeTestResults(object testResults) => Serialize((TestRun)testResults);

private string Serialize<TEntity>(TEntity entityToBeSerialized)
{
    var settings = new XmlWriterSettings
    {
        Encoding = new UnicodeEncoding(false, false),
        Indent = false,
        OmitXmlDeclaration = false,
    };
    var xmlSerializer = new XmlSerializer(typeof(TEntity));
    string result;

    using (var stringWriter = new StringWriter())
    {
        using (var writer = XmlWriter.Create(stringWriter, settings))
        {
            xmlSerializer.Serialize(writer, entityToBeSerialized);
            result = stringWriter.ToString();
        }
    }

    result = result.Replace("xmlns:xsd=\"http://www.w3.org/2001/XMLSchema\"", string.Empty);
    result = result.Replace("xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\"", string.Empty);

    return result;
}

private TEntity Deserialize<TEntity>(string content)
{
    var serializer = new XmlSerializer(typeof(TEntity));
    var reader = new StringReader(content);
    var testRun = (TEntity)serializer.Deserialize(reader);
    reader.Close();

    return testRun;
}
```
Usually under a folder called Model you place the C# objects representation of your test results file. The runner will call Serialize and Deserialize methods to convert the object representation to file and vice verse. For XML we can use built-in .NET libraries for XML serialization for JSON files we can use NewtonSoft.Json NuGet.

```csharp
public List<TestCase> GetAllPassesTestCases(string testResultsFileContent)
{
    TestRun testRun = Deserialize<TestRun>(testResultsFileContent);
    var passedUnitTestResults = testRun.Results.Where(x => x.outcome.Equals("Passed")).ToList();
    var passedTestCases = ConvertUnitTestsResultsToTestCases(passedUnitTestResults, testRun);
    return passedTestCases;
}
```
First, we pass the full path to the test results file. After that, we deserialize the content to a C# object. After that in this method, we should filter only the passed test cases. After that, we need to convert the current test results C# object for test cases to a collection containing MEISSA TestCase objects.

```csharp
public object GetAllPassesTests(object testRunObj)
{
    var testRun = (TestRun)testRunObj;
    var results = testRun.Results.ToList().Where(x => x.outcome.Equals("Passed")).ToList();
    return results;
}
```
This method does the same but returns directly the current test results C# object for test cases without converting them to MEISSA test cases. Also, we pass test results C# object directly instead of providing the full path to the test results.
```csharp
public object GetAllPassesTests(string testRunContent)
{
    var testRun = Deserialize<TestRun>(testRunContent);
    var results = testRun.Results.ToList().Where(x => x.outcome.Equals("Passed")).ToList();
    var passedTestCases = ConvertUnitTestsResultsToTestCases(results, testRun);
    return passedTestCases;
}
```
The method does the same but here we pass the full path to the test results, and we need to serialize its content to C# object.
```csharp
public int GetAllPassesTestsCount(object testRunObj)
{
    var testRun = (TestRun)testRunObj;
    int count = testRun.Results.Count(x => x.outcome.Equals("Passed"));
    return count;
}
```
The runner will pass the C# object representation of the test results file. Here you need to return the number of passed tests.
```csharp
public int GetAllNotPassesTestsCount(object testRunObj)
{
    var testRun = (TestRun)testRunObj;
    int count = testRun.Results.Count(x => !x.outcome.Equals("Passed"));
    return count;
}
```
Return the count of all not passed tests.
```csharp
public string MergeTestResults(object testResultsToBeMergedObj)
{
    var result = string.Empty;
    var testRunsToBeMerged = (List<object>)testResultsToBeMergedObj;
    TestRun mergedTestRun = null;
    if (testRunsToBeMerged.Any())
    {
        mergedTestRun = (TestRun)testRunsToBeMerged.First();

        mergedTestRun.Times.finish = ((TestRun)testRunsToBeMerged.Last()).Times.finish;
        for (int i = 1; i < testRunsToBeMerged.Count; i++)
        {
            var msTestTestRun = (TestRun)testRunsToBeMerged[i];
            mergedTestRun.TestDefinitions = ConcatArrays(mergedTestRun.TestDefinitions, msTestTestRun.TestDefinitions);
            mergedTestRun.Results = ConcatArrays(mergedTestRun.Results, msTestTestRun.Results);
            mergedTestRun.TestEntries = ConcatArrays(mergedTestRun.TestEntries, msTestTestRun.TestEntries);
            if (mergedTestRun.ResultSummary.outcome == "Passed" &&
                mergedTestRun.ResultSummary.outcome != msTestTestRun.ResultSummary.outcome)
            {
                mergedTestRun.ResultSummary.outcome = msTestTestRun.ResultSummary.outcome;
            }

            mergedTestRun.ResultSummary.Counters.total += msTestTestRun.ResultSummary.Counters.total;
            mergedTestRun.ResultSummary.Counters.executed += msTestTestRun.ResultSummary.Counters.executed;
            mergedTestRun.ResultSummary.Counters.passed += msTestTestRun.ResultSummary.Counters.passed;
            mergedTestRun.ResultSummary.Counters.failed += msTestTestRun.ResultSummary.Counters.failed;
            mergedTestRun.ResultSummary.Counters.error += msTestTestRun.ResultSummary.Counters.error;
            mergedTestRun.ResultSummary.Counters.aborted += msTestTestRun.ResultSummary.Counters.aborted;
            mergedTestRun.ResultSummary.Counters.inconclusive +=
                msTestTestRun.ResultSummary.Counters.inconclusive;
            mergedTestRun.ResultSummary.Counters.passedButRunAborted +=
                msTestTestRun.ResultSummary.Counters.passedButRunAborted;
            mergedTestRun.ResultSummary.Counters.notRunnable +=
                msTestTestRun.ResultSummary.Counters.notRunnable;
            mergedTestRun.ResultSummary.Counters.notExecuted +=
                msTestTestRun.ResultSummary.Counters.notExecuted;
            mergedTestRun.ResultSummary.Counters.disconnected +=
                msTestTestRun.ResultSummary.Counters.disconnected;
            mergedTestRun.ResultSummary.Counters.warning += msTestTestRun.ResultSummary.Counters.warning;
            mergedTestRun.ResultSummary.Counters.completed += msTestTestRun.ResultSummary.Counters.completed;
            mergedTestRun.ResultSummary.Counters.inProgress += msTestTestRun.ResultSummary.Counters.inProgress;
            mergedTestRun.ResultSummary.Counters.pending += msTestTestRun.ResultSummary.Counters.pending;
        }
    }

    if (mergedTestRun != null)
    {
        result = Serialize(mergedTestRun);
    }

    return result;
}
```
One of the most important methods. The runner will pass a collection of test results C# objects. Here you need to merge them. The results are ordered by time of execution, meaning that the last one is the newest. You need to update the passed/failed tests and update all relevant properties like exceptions, stack traces, counts.
```csharp
public void UpdatePassedTests(object passedTests, object testRunObj)
{
    var passedTestCases = (List<TestCase>)passedTests;
    var testRun = (TestRun)testRunObj;
    var msTestTestCases = ConvertTestRunToMsTestCases(testRun);

    foreach (var currentTest in testRun.Results)
    {
        var currentTestFullName = msTestTestCases.FirstOrDefault(x => x.Id.Equals(currentTest.testId));
        if (passedTestCases.Count(x => x.FullName.Equals(currentTestFullName)) > 0)
        {
            currentTest.outcome = "Passed";
        }
    }
}
```
A method where we update the test results C# object. The first argument is a list of all newly passed test cases based on which you need to update the results object.
```csharp
public void UpdateResultsSummary(object testRunObj)
{
    TestRun testRun = (TestRun)testRunObj;
    testRun.ResultSummary.Counters.failed = (byte)testRun.Results.Count(x => x.outcome.Equals("Failed"));
    testRun.ResultSummary.Counters.passed = (byte)testRun.Results.Count(x => x.outcome.Equals("Passed"));
    if (testRun.ResultSummary.Counters.passed != testRun.Results.Length)
    {
        testRun.ResultSummary.outcome = "Failed";
    }
    else
    {
        testRun.ResultSummary.outcome = "Passed";
    }
}
```
Here the runner will pass the updated test results object. You need to check whether there are any failed tests if yes set the overall outcome to failed and vice versa.
```csharp
public List<TestCaseRun> UpdateTestCasesHistory(object testRunContent, string libraryName)
{
    var testRun = Deserialize<TestRun>((string)testRunContent);
    var allPassedTestsTestRun = (List<TestRunUnitTestResult>)GetAllPassesTests(testRun);
    var testCaseRuns = new List<TestCaseRun>();
    foreach (var currentUnitTestResult in allPassedTestsTestRun)
    {
        var unitTestId = currentUnitTestResult.testId;
        var duration = currentUnitTestResult.duration;
        var currentUnitTest = testRun.TestDefinitions.FirstOrDefault(x => x.id.Equals(unitTestId));

        var className = currentUnitTest?.TestMethod.className.Split(',').Last().Trim();
        var unitTestLibraryName = currentUnitTest?.TestMethod.codeBase;
        var name = currentUnitTest?.TestMethod.name;
        var fullName = $"{className}.{name}";
        if (unitTestLibraryName != null && unitTestLibraryName.ToLower().Contains(libraryName.ToLower()))
        {
            testCaseRuns.Add(new TestCaseRun(fullName, duration));
        }
    }

    return testCaseRuns;
}
```
MEISSA uses this method to update the tests statistics based on which can optimize the distribution of the tests across remote machines. Basically, we create a list of MEISSA TestRun objects that have two main properties test full name and duration. So, you need to get this information from the results C# object and build this collection and return it.
```csharp
public List<List<TestCase>> SplitTestCases(int availableCores, bool sameMachineByClass, List<TestCase> testCasesToBeDistributed)
{
    if (availableCores <= 0)
    {
        throw new ArgumentException("Test Agents Count Must be Greater Than 0.");
    }

    var orderedByClassTestCases = testCasesToBeDistributed.OrderBy(x => x.ClassName).ToList();
    var runFilterArgs = CreateRunFilterArgument(testCasesToBeDistributed);

    int totalPartitions = availableCores >= runFilterArgs.Count ? availableCores : runFilterArgs.Count;
    int numberOfTestsPerList = (int)Math.Ceiling(orderedByClassTestCases.Count / (double)totalPartitions);

    var distributedTestCases = new List<List<TestCase>>();
    if (numberOfTestsPerList > 0)
    {
        int distributedIndex = 0;
        int tempDistributedTestsCount = numberOfTestsPerList;
        string previousClass = null;
        for (int i = 0; i < orderedByClassTestCases.Count; i++)
        {
            bool shouldResetTestsPerList = ShouldResetTestsPerList(sameMachineByClass, orderedByClassTestCases[i].ClassName, previousClass);
            if (tempDistributedTestsCount <= 0 && shouldResetTestsPerList)
            {
                tempDistributedTestsCount = numberOfTestsPerList;
                distributedIndex++;
            }

            if (tempDistributedTestsCount == numberOfTestsPerList)
            {
                distributedTestCases.Add(new List<TestCase>());
            }

            distributedTestCases[distributedIndex].Add(orderedByClassTestCases[i]);
            previousClass = orderedByClassTestCases[i].ClassName;

            tempDistributedTestsCount--;
        }
    }
    else
    {
        distributedTestCases.Add(testCasesToBeDistributed);
    }

    return distributedTestCases;
}
private bool ShouldResetTestsPerList(bool sameMachineByClass, string currentClass, string previousClass)
    => !sameMachineByClass || previousClass != currentClass;
```
MEISSA uses this method to split the list of test cases to be executed once when need to split test cases across remote agents and the second time to divide them to multiple execution threads. If the **sameMachineByClass** is set to true this means that the test cases located in class should be executed on the same machine/thread. We respect it for NUnit right now. This is still not available as you can see for MSTest.
```csharp
public List<TestCase> GetAllNotPassedTests(string testResultsFileContent)
{
    TestRun testRun = Deserialize<TestRun>(testResultsFileContent);
    var notPassedUnitTestResults = testRun.Results.Where(x => !x.outcome.Equals("Passed")).ToList();
    var notPassedTestCases = ConvertUnitTestsResultsToTestCases(notPassedUnitTestResults, testRun);
    return notPassedTestCases;
}
```
The runner passes the full path to the test results file. Here, we need to deserialize it to a C# object. After that, we need to return a collection of MEISSA TestCase objects filtered from the test results C# object.
```csharp
public object GetRetriedTestResultsForTestRun(List<TestAgentRunDto> testAgentRuns)
{
    var msTestTestRuns = new List<object>();
    if (testAgentRuns.Any())
    {
        foreach (var testAgentRun in testAgentRuns)
        {
            if (!string.IsNullOrEmpty(testAgentRun.TestAgentRetriedRunResults))
            {
                var msTestTestRun = Deserialize<TestRun>(testAgentRun.TestAgentRetriedRunResults);
                msTestTestRuns.Add(msTestTestRun);
            }
        }
    }

    return msTestTestRuns;
}
```
The runner passes a collection of TestAgentRunDto. These are objects that contain information which tests should be retried based if the retry of failing tests is enabled. If there is something present in the string TestAgentRetriedRunResults property we deserialize it to test results C# object. We create here a collection containing all of these objects and return it.
```csharp
public object GetTestResultsForTestRun(List<TestAgentRunDto> testAgentRuns)
{
    var msTestTestRuns = new List<object>();
    if (testAgentRuns.Any())
    {
        foreach (var testAgentRun in testAgentRuns)
        {
            if (!string.IsNullOrEmpty(testAgentRun.TestAgentOriginalRunResults))
            {
                var msTestTestRun = Deserialize<TestRun>(testAgentRun.TestAgentOriginalRunResults);
                msTestTestRuns.Add(msTestTestRun);
            }
        }
    }

    return msTestTestRuns;
}
```
Similarly to the previous method we use this one to get the collection of TestRun objects for all test agents.

### NativeTestsRunnerTestCasesPluginService ###

The second class that you need to create is **NativeTestsRunnerTestCasesPluginService**. It needs to implement the **INativeTestsRunnerTestCasesPluginService** interface. Also, don't forget to set the **Export** attribute.
```csharp
[Export(typeof(INativeTestsRunnerTestCasesPluginService))]
public class NativeTestsRunnerTestCasesPluginService : INativeTestsRunnerTestCasesPluginService
{
    private const string MsTestCategoryAttributeName = "Microsoft.VisualStudio.TestTools.UnitTesting.TestCategoryAttribute";
    private const string MsTestClassAttributeName = "Microsoft.VisualStudio.TestTools.UnitTesting.TestClassAttribute";
    private const string MsTestTestAttributeName = "Microsoft.VisualStudio.TestTools.UnitTesting.TestMethodAttribute";
    private const string MsCodedUITestClassAttributeName = "Microsoft.VisualStudio.TestTools.UITesting.CodedUITestAttribute"; // search codded UI tests

    public string Name => "MSTest";

    public List<TestCase> ExtractAllTestCasesFromTestLibrary(string testLibraryPath)
    {
        var module = ModuleDefinition.ReadModule(testLibraryPath);
        var testCases = new List<TestCase>();

        foreach (var currentType in module.GetTypes())
        {
            if (currentType.CustomAttributes.Any(x => x.AttributeType.FullName.ToString().Contains(MsTestClassAttributeName) || x.AttributeType.FullName.ToString().Contains(MsCodedUITestClassAttributeName)))
            {
                // This is a Nunit test class - create new test suite for it.
                ////var currentTestSuite = CreateTestSuite(currentType);

                foreach (var currentMethod in currentType.GetMethods())
                {
                    if (currentMethod.CustomAttributes.Any(x => x.AttributeType.FullName.ToString().Equals(MsTestTestAttributeName)))
                    {
                        // This is a Nunit test - add it to the current test class list of tests.
                        var currentTestCase = CreateTestCase(currentMethod);
                        testCases.Add(currentTestCase);
                    }
                }
            }
        }

        return testCases;
    }

    private TestCase CreateTestCase(MethodDefinition testMethod)
    {
        var testCase = new TestCase
        {
            FullName = string.Concat(testMethod?.DeclaringType?.FullName, ".", testMethod.Name),
            ClassName = testMethod.DeclaringType.FullName,
        };
        var testCaseCategoryAttributes = testMethod.CustomAttributes.Where(x => x.AttributeType.FullName.ToString().Contains(MsTestCategoryAttributeName));
        testCase.Categories = GetCategoryNamesFromAttributes(testCaseCategoryAttributes);

        return testCase;
    }

    private List<string> GetCategoryNamesFromAttributes(IEnumerable<CustomAttribute> attributes)
    {
        var categoryNames = new List<string>();
        if (attributes != null)
        {
            foreach (var categoryAttribute in attributes)
            {
                if (categoryAttribute.HasConstructorArguments)
                {
                    foreach (var constructorArg in categoryAttribute.ConstructorArguments)
                    {
                        categoryNames.Add(constructorArg.Value.ToString());
                    }
                }
            }
        }

        return categoryNames;
    }
}
```
You need to implement only two things a property named **Name** and a method called **ExtractAllTestCasesFromTestLibrary**.
The Name property represents the name of your technology library- MSTest, NUnit, Protractor, TestNG, etc. Based on it the plugin is picked.
Then in the **ExtractAllTestCasesFromTestLibrary** the runner passes the path to the test file where your tests are- DLL, text file, etc. Here it is your job to find a way to extract all tests and return them as a collection of MEISSA TestCase objects. For MSTest and NUnit we use reflection API to get all test classes and then get all tests checking whether they have specific attributes or not.
If you want to write a plugin for another language like Java you can call a java app from CLI, extract the tests in it, save them as a serialized file somewhere and read the data in the method here and convert it to TestCase objects.
Or if you write a plugin for a language like JavaScript, you can just read your JS files and parse them and later based on some your logic convert them to TestCase objects.


