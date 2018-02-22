# Common Challenges Running Tests in Parallel #

There are many benefits in running tests in parallel; however, you need to know there are many challenges to overcome before you are ready to speed up your test runs this way. Now, I will share some of the issues we faced testing our product and add some from my previous experience and the Automate The Planet's survey. I will try to give you sample solutions to those problems.

## Preparing Test Environment and Managing Tests Data ##
One of the most significant challenges in writing maintainable automated tests has been making the tests atomic, not depending on each other. 
An example from last year that I saw on a team I worked with was that they ran their tests sequentially because each test used a single user with a specific setup. They had to automate an e-commerce platform and test what the clients will see in their accounts after the purchases are completed. Instead of creating a client with separate unique orders, they used already created one manually with the needed orders. Some of the tests at the end modified the data. If you run those tests sequentially, everything is OK, but if you run them in parallel, the data messed up, since test A deleted the data for test B.
Also, there is always the probability that someone will to go and modify this client’s settings, which means all tests will fail.
Something we did in a team that I was part of was the following. Each test used a separate new client. Since the processes of creating a new client could take up to 30 seconds, we couldn't do it in each test. So, we built a small tool that regularly ran in Jenkins job. It ensured we always have at least 5000 new clean test client accounts. Then in our tests, we had a logic to get a client from the pool. The solution was quite fast. 
### Solution ###
Since, we needed to have separate specific orders, we used internal APIs to create them. The challenge was that we needed to set up the test products that we want to buy. So, we created a separate tool that always ran before the tests to create the range of products the tests require. 
With this setup, we managed to run our tests in parallel.
So, my suggestion is, if possible, execute the expensive setup, cleanup before and after the whole run, and make sure each test uses its own unique set of data.

## Internal APIs Load ##
If you use internal APIs to setup/clean your test data, you need to make sure your web server can handle the load if you run your tests in parallel. Even if the tests pass, this can slow down their execution.
### Solution ###
Make sure the required services are hosted on the appropriate machines and are load balanced, so they can handle the load.

## Clear Browsers State ##
As promised, now I will share how we make sure the test agents' browsers are clean after each run.
### Solution ###
Meissa supports writing custom plugins and executing their logic at different points of the tests execution, for example, before the whole test run, after it, or when performing some logic on the test agent’s machines. As a part of custom plugin, we added logic for killing all WebDriver related processes and browsers at the end of each test agent run. Below you can find the code.
```
public static class DisposeDriverService
{
    private static readonly List<string> _processesToCheck = new List<string>
    { "chromedriver", "opera", "chrome", "firefox", "ie", "gecko", "phantomjs", "edge", "microsoftwebdriver", "webdriver" };

    public static DateTime? TestRunStartTime { get; set; }

    public static void Dispose()
    {
        var processes = Process.GetProcesses();
        foreach (var process in processes)
        {
            try
            {
                Debug.WriteLine(process.ProcessName);
                var shouldKill = false;
                foreach (var processName in _processesToCheck)
                {
                    if (process.ProcessName.ToLower().Contains(processName))
                    {
                        shouldKill = true;
                        break;
                    }
                }

                if (shouldKill)
                {
                    process.Kill();
                    process.WaitForExit();
                }
            }
            catch (Exception e)
            {
                Debug.WriteLine(e);
            }
        }
    }
}
```
 
## Video Recording and Screenshots ##
Another UI tests related challenge was that, if you run multiple browsers on a single machine, the video recording of the tests become useless. We have such capability in our automation framework for locating problems easier.
The same is valid for taking screenshots for some drivers that take screenshots of the entire desktop instead of the browser tab.
### Solution ###
In my opinion, running multiple browsers on single machine brings more problems than benefits. So, in our company, we prefer to execute the tests on more agents on one browser at a time.
Also, instead of making screenshots with WebDriver, we use JavaScript to create full page screenshots, which means, even if we use multiple browsers on a machine, we would be able to make them.

## Change Port on WebDriver Initialization ##
Another problem we faced trying to run our UI tests in parallel on a single machine was that not all drivers get a free port on initialization. If you attempt to create the driver a second time before disposing of the first one, it tells you the port is already in use. The same may happen if you use proxies.
### Solution ###
The solution here, at least in .NET, is simple. We created a little C# code snippet for getting the first free port.
```
private static int GetFreeTcpPort()
{
    Thread.Sleep(100);
    var tcpListener = new TcpListener(IPAddress.Loopback, 0);
    tcpListener.Start();
    int port = ((IPEndPoint)tcpListener.LocalEndpoint).Port;
    tcpListener.Stop();
    return port;
}
```
## Edge Driver Cannot Start Multiple Instances ##
In our company, we run our CI tests using the Edge browser, since the tests on it execute faster, even compared to the headless browsers. However, it has some drawbacks. Once the driver is disposed, it closes all currently opened browsers (even not opened by WebDriver), which makes the execution in parallel on a single machine impossible. Moreover, it is designed in a way that doesn't support multiple WebDriver instances, even if you assign a different port on initialize.
### Solution ### 
The solution is executing the tests on more agents on one browser at a time.
## Usage of Static in Tests ##
If you are leveraging the native test framework support for parallel execution, the use of static data may get you lots of trouble. There might be race conditions where one test can change the data on some other test, which is similar to the first example I gave you.
### Solution ###
Meissa runs tests in parallel, but in isolation, which solves the problem.
