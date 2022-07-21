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
# Tips:
- Always read the log in bottom-up fashion.