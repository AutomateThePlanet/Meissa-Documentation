# How Does It Work Internally? #

![High Overview](https://i.imgur.com/dqJlM0f.png)

We have many moving parts- server, test agents, runner and so on. All of them use single command-line-interface; there are no separate installers or executables.
First, we need to start the server. Its job is to synchronise the work between the runner and all agents. The first time we start it, a portable SQLite database is created. There are stored various kind of information, such as tests execution times, test output files, logs, exceptions, etc. The web service is self-hosted using the ASPNET.Core portable web server- Kestrel. All agents and runners communicate with the server via HTTP.
## Meissa Test Agent Mode ##
![Test Agent Internal](https://i.imgur.com/nmCzNwk.png)
The test agents do not have local databases. Because of that, they require the server to be up all the time; otherwise, they cannot function properly.
When you start a test agent, it registers itself as active. Then, it continuously asks the server whether there are scheduled test agent runs to be executed on the machine.
When it starts a run, it starts separating processes that send what messages should be printed by the runner to the client (messages from the native runners or another log info). Also, one more parallel process is started that pings the server every few seconds to verify the test agent’s health. If some of the agents don’t confirm its health on time, the test run is aborted. 
When the test execution is started, the agent creates a thread where it verifies its health. In parallel, another process checks the status of the runner. If it is not available, the test runner is aborted. Now, I will explain the whole test execution in more detail. 
First, the specific test technology plugins are loaded. Based on the parallel options, the agent creates multiple tests batches. Then it starts and waits to finish all the processes. While the tests are executed, separate parallel processes send messages to be printed to the runner via HTTP requests to the server. After all processes finish, the results files are merged. If the Meissa retry option is turned-on and there are any failed tests, the procedure is repeated for them. At the end of the retry cycle, the test results are updated if any of the tests succeeded. 
After this important step, the agent saves the merged results and completes the test agent run. After that, it waits for the test run to finish before starting to wait for new jobs.
There are more details and nitty-gritty details about the whole process which is quite complicated. However, these details are not so crucial for this particular talk.
## Meissa Test Runner Mode ##
![Test Runner Internal](https://i.imgur.com/O5h80ge.png)
The test runner doesn’t have a local database. Because of that, it requires the server to be up all the time; otherwise, it cannot function properly.
Once started, the runner first gets all active test agents from the API. Based on the specified test technology (MSTest, NUnit, etc.), it loads the specific plugins. Then it uses some logic from the plugins to extract and filter the test cases from the tests files. Based on the available test agents, it distributes the tests on each of them. It zips the test output files and sends them to the server, so each agent can download them before tests execution. The second part of the run is to wait for all test agents to finish. At the same time, a parallel process is started, where the runner verifies its health. Also, one more thread is triggered that continually checks whether there are new messages to be printed sent by the agents. At the end of the process, it merges all test results into a single file and completes the run.
