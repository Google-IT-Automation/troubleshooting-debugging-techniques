## Troubleshooting: 
The process of identifying, analyzing and solving problems. We can refer the term troubleshooting to refer to solving any kind of problem.
## Debugging:
The process of identifying, analyzing and removing bugs in a system
- We generally use the term troubleshooting and debugging interchangeably.
- But generally we say, troubleshooting when we're fixing problems in the system running the application, and debugging when we're fixing the bugs in the actual code of the application.
## Debuggers:
Let us follow the code line by line, inspect changes in variable assignments, interrupt the program when a specific condition is met and more..
# Problem Solving Steps:
### Gathering Information:
Problems could be hardware issue, software issue, configuration issue etc..
- What the issue is ?
- Since when we are facing ?
- What are the consequences ?
- When the issue happened ?
- What was the user doing when the issue happened ?
- How often the issue is coming ?
### Reproduction Case:
- A clear description of how and when the problem appears
### Finding Root Cause:
- It is most difficult step.
### Performing the necessary remediation
- Depending on the problem, provide immediate fix
- Medium term or long term fix to prevent system to fall into same state again
### Document solution Or Steps
- Throughout the whole process its important that we document the steps that we performed to solve the problem.
- We should note down the info that we get
- Different things that we tested to try
- Write down the root cause
- Finally the steps that we took to fix the issue.

These steps of problem solving don't always happen sequentially. It often happens, that while performing root cause analysis, we realized that we need more information about the current state. So, we go back to information gathering phase. <br>
Or we can understand the problem just enough to create a workaround to unblock the users. But we still need more time to root cause the problem and prevent the problem from happening again. <br>
Documentation is really time saviour, when we encounter the same issue next time.
## Tools:
There are bunch of tools that can help us understand what is going on with the system and our applications. With the help of these tools, we can extend our knowledge of a particular problem view the actions of the program from a different point of view and get the info we need
### strace:
- lets us look more deeply at what the program is doing. It will trace a system calls made by the program and tell us what the result of each of these calls was.
### ltrace:
- ltrace tool is used to look at library calls made by the software.
### Network Traffic
-  software tools are used to analyze network traffic to isolate problems are wireshark, tcpdump
- The tcpdump tool is a powerful command-line analyzer that captures or "sniffs" TCP/IP packets.
- Wireshark is an open source tool for profiling network traffic and analyzing TCP/IP packets.
### system calls:
- Calls that the programs running on our computer make to the running kernel.
- -1 after equal sign shows that the system call did not finish correctly.
- O flag in openat syscall says that the system is trying to open the path as directory.
```shell
$ strace ./test.py                 # will invoke strace on our script
$ strace -o out.log ./test.py      # store output in a out.log file
```
### Common questions to ask the user ?
- what were you trying to do ?
- what steps did you follow ?
- what was the expected result ?
- what was the actual result ?
### Ways to solve the issues ?
- First ask the questions to user to gather more information
- Second try to reproduce the issue at your end
    - This way you eliminated that it is user specific
- Start simple checks first, these again helps in reducing the problem scope
    - By looking at possible simple explanations first, you avoid losing time chasing the wrong problem.
### How to come up with reproduction cases ?
- It can sometimes be complex to create the reproduction case.
- Step1 : Read the logs available to you.
    - Linux 
        - we read system logs like : /var/log/syslog
        - user specific log like : ~/.xsession-errors
    - MacOs
        - /Library/Logs
    - Windows
        - We use event viewer tool to go through the event logs
- Step2 : If error message is not helpful, then try below steps
    - Try to isolate the conditions that trigger the issue
    - Do other user in the same office also experience the problem ?
    - Does the same thing happens if the same user logs into a different computer ?
    - Does the problem happens if the application config directory is moved away ?
### Root Cause
- Understanding the root cause is essential for performing the long-term remediation
- Step1: Have an easy and clear reproduction case
- Step2:
    - Whenever possible, we should check our hypothesis in a test environment, instead of the production environment that our users are working with.
- Step3:
    - Too much disk input and output:
        - Use iotop command to check which processes uses more i/p and o/p
        - Use ```iostat``` and vmstat to collect io statistics
        - Use ```ionice``` command to let applications reduce its priority to access the disk and let the other services use it too.
        - Use ```iftop``` to check current traffic on the network interfaces
        - rsync command contains an option ```-bwlimit``` to limit the bandwidth consumption in a process
        - Otherwise use a program ```Trickle``` to limit the bandwidth use
### Dealing with Intermittent Issues
- Understand what is going on
    - So, you understand when the issue happens and when it doesn't
- Enable debugging flag to collect more logs, to be able to see the issue in the logs, whenever the next time it happens
- Otherwise, resort to the environment when the issue triggers.
- If we could not able to root cause the issue, then scheduling a restart of the system could also help.

### Apply Binary Search in Troubleshooting
- We can apply this idea when we need to go through and test a long list of hypothesis.
- When doing this the list of elements contains all possible causes of the problem. We keep reducing the problem by half until only one option is left.
- With each iteration we cut the problem in half. This approach is sometimes called ```Bisecting```
- **Example:**
    - New version of program failed to start when ond configuration directory was present.
    - If the directory contained a bunch of different files in it, we could identify the one causing the failure by bisecting the list of files.
    - Assuming dir contains 12 different config files.
    - Then lets keep only 6 config files and then try to run the software. If works, that means the issue is in other 6 files.
    - Now take another 3 files and keep it in the dir, if software works then it means the issue is in remaining 3 files and so on ..
- We can do it in similar fashion for a single file as well. Cut the contents of single file(commenting half of the code) and then try running the software. We repeat it until we find the specific portion of the file that is causing the problem.
- ***Possible use case problems could be:***
    - which extensions is causing the browser to crash
    - which plug-in in desktop environment is causing the computer to run out of memory
    - which entry in database is causing the program to raise an exception
    - find the bug that was introduced in a recent version
        - Git provides this facility using a command called ```git bisect```
- **Hands-on:**
    - [find_corrput_data_in_file](https://www.coursera.org/learn/troubleshooting-debugging-techniques/lecture/Zf9Ob/finding-invalid-data)
    - importing of csv file is failing
    - Always test things on test server
```shell
$ cat contacts.csv | ./script.py --server test
$ wc -l contacts.csv    # count lines in csv  assume lines=100
### Now use bisecting method to get the faulty data
$ head -50 contacts.csv |  ./script.py --server test   # failure -> means problem in first half
$ head -50 contacts.csv | head -25  ./script.py --server test # success -> means problem in second half
$ head -50 contacts.csv | tail -25  ./script.py --server test # failure -> means problem in this half only
$ head -50 contacts.csv | tail -25 | head -13 ./script.py --server test # success
$ head -50 contacts.csv | tail -25 | tail -12 | head -6 ./script.py --server test # failure
$ head -50 contacts.csv | tail -25 | tail -12 | head -6 | head -3 ./script.py --server test # failure
## means now we have 3 entries, out of which 1 is faulty
$ head -50 contacts.csv | tail -25 | tail -12 | head -6 | head -3  # check it and fix it
```


# Tips:
- Always read the log in bottom-up fashion.
- Make sure you understand the issue correctly, before moving on to troubleshooting.
