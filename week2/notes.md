# slowness
CPU, Memory, Sick, Network

### why is my computer slow ?
Few of the many possible reasons could be:
- Process needs more CPU time
- Time spent in reading data from the disk 
- waiting for data transmitted over the network
- moving data from disk to RAM
- Or some other resource that's limiting the overall performance

**Solutions:**
- Close unused applications. This will free up computer resources.
- Get better hardware. We are running an advanced app on an old hardware.
- **`Linux`**:
    - We can use programs like `top, iotop, iftop` to collect some computer statistics
- **`MacOS`**:
    - Activity Monitor tool helps us collect the relevant information
- **`Windows`**:
    - Resource Monitor
    - Performance Monitor
- Sometimes, we need to figure out what the software is doing wrong and where it's spending most of its time to understand how to make it run faster. We need to really study each problem to get to the root cause of the slowness.

## How computer access resources ?
- cpu memory < ram < disk < network.
    - cpu internal memory is the fastest for accessing the data
- Cache:
    - Stores data in a form that's faster to access than its original form
    - *`webproxy`* : It is a form of cache. It stores websites, images or videos that are accessed often by users behind the proxy. So, they don't need to be downloaded from internet every time.
    - *`dns`*: DNS services usually implement a local cache for the websites they resolve. So they don't need to query from the Internet every time someone asks for their IP address
- RAM outage: What happens when you run out of RAM?
    - At first, the OS will just remove from RAM anything that's cached, but not strictly necessary.
    - If there's still not enough RAM after that, the operating system will put the parts of the memory that aren't currently in use onto the hard drive in a space called *`swap`*
    - *`swap`*: Reading and writing from disk is much slower than reading and writing from RAM. So when the swapped out memory is requested by an application, it will take a while to load it back. This is normal operation, and most of the time, we don't notice it.
        - But is the available memory is significantly less than what the running applications need, the OS will have to keep swapping out the data that's not in use right now to move the data currently in use to RAM, and as we called out, our computer can switch between applications very quickly, which means that the data currently in use can also change very quickly.  The computer will start spending a lot of time writing to disc to make some space in RAM and then reading from disk to put other things in RAM. This can be super slow.
    - So what do you do if you find that your machine is slow because it's spending a lot of time swapping?

## Machine is slow because of swapping ?
- If there are too many open applications and some can be closed
    - Close the ones that are not needed
- Available memory is too small for the amount computer is using ?
    - Add more RAM to the computer
- One of the running programs may have a memory leak ?
    - *`Memory Leak `*: Memory which is no longer needed is not getting released.
    - Lets just say, if a program is using a lot of memory and this stops when you restart the program, it's probably because of a memory leak.

## Possible causes of computer slowness
When trying to diagnose, why a computer is slow, we should use the process of elimination that we looked at earlier. We first look for the simplest explanations that are the easiest to check. And after eliminating a possible root cause, we go back to the problem and come up with the next possible cause to check.

- When the computer is slow ?
    - If it's slow when starting up, it's probably a sign that there are too many applications configured to start on boot.
- In case if computer becomes sluggish after days of running just fine, and the problem goes away with a reboot.
    - it means that there's a program that's keeping some state while running that's causing the computer to slow down.
    - **For ex:** this can happen if a program stores some data in memory and the data keeps growing over time, without deleting old values. If a program like this stays running for many days, the data might grow so much that reading it becomes slow and the computer runs out of RAM. This is almost certainly a bug in the program.
        - Ideal solution is change the code so that it frees up some of the memory used.
        - If you don't have access to the code, another option is to schedule a regular restart to mitigate both the slow program and your computer running out of RAM.
    - **For ex:** file that an application is reading have grown too large. Example, we keep adding new testcases and hence the jenkins log has reached to MB's. Now whenever we try to open the jenkins log it is very very large in size.
        - this generally points to a bug in the way the program was designed because it didn't expect the files to grow so large. The best solution in this case is to fix the bug.
        - But what can you do if you can't modify the code of the program? You can try to reduce the size of the files involved. If the file is a log file, you can use a program like **logrotate** to do this for you.
- Whether the slowness is happening for all users of the application or just a subset of them.
    -  If only some users are affected, we'll want to know if there's something that's configured differently on those computers that might be triggering the slowness.
    - **For ex:** many operating systems include a feature that tracks the files in our computer so it's easy and fast to search for them.
        - This feature can be really useful when looking for something on a computer, but can get in the way of everyday use if we have tons of files and not the most powerful hardware.
    - **For ex:**  It's common for computers in an office network to use a file system that's mounted over the network so they can share files across computers.
        - This normally works just fine, but can make some programs really slow if they're doing a lot of reads and writes on this network-mounted file system.
        - To fix this, we'll need to make sure that the directory used by the program to read and write most of its data is a directory local to the computer.
- Hardware failures can also cause our computer to become slow ?
    - If your hard drive has errors, the computer might still be able to apply error correction to get the data that it needs, but it will affect the overall performance.
    - And once a hard drive starts having errors, it's only a matter of time until they're bad enough that data starts getting lost.
        - we can use some of the OS utilities that diagnose problems on hard drives or on RAM, and check if there's anything that could be causing problems
- Another source of slowness is malicious software
    - we can feel the effects of malicious software even if they aren't installed
        - **For example:** you might have come across a website that includes scripts, either in the website's content or the ads displayed, that use our processor to mine for cryptocurrency.
        - Malicious browser extensions also fall into this category.

# Practical Case : Webserver is slow
A user has alerted us that a particular webserver in our company is being slow.
- try to login in the page yourself. Page loads but seems to be a bit slow.
- Use `ab` (apache benchmark) tool to understand how slow it is.
```shell
$ ab -n 500 site.example.com    # to get the average timing of 500 requests
```
- Mean time per request is 155 ms, which is slow for such a simple website. It means sth is going on in the webserver and we need to investigate further.
- Lets connect to webserver
```shell
$ ssh webserver
$ top
$ load average : 30.38
```
- Start by `top` and see if there is anything suspicious there.
- We noticed that there are bunch of `ffmppeg` processes that are running, which are basically using all the available CPU. Load number of `30` is not normal in modern systems.
- Load Average on Linux shows how much time the processor is busy in a given minute, with `1` meaning it was busy for the whole minute.
- During each minute, there were more processes waiting for the processor time than the processor had to give.
- This ``ffmpeg` program is used for video transcoding which means converting files from one video format to another. This is a CPU intensive process and looks like the culprit for our server being overloaded.
- We can try to change the job priorities so that the webserver takes the precedence. For processor priorities in linux the lower the number the higher the process priority. **Default priority for all process is 0**. Typical priority range is from `0-19`
### **Change the process priorities using `nice` and `renice` commands**
```shell
for pid in $(pidof ffmpeg); do renice 19 $pid; done
```
- `nice` is used to start a process with a different priority
- `renice` is used to change the priority of an already running process
    - syntax : renice(<new_priority\> \<pid>)
- `pidof` command receives the process name and returns all the process IDs that have that name.
### ** For this scanerio **`renice`** does not helped.
-  Reason is OS is still giving the ffmpeg processes way too much processor time.
- These conversion processes is CPU intensive and running them in parallel is overloading the computer.

**Solution** : Modify whatever's triggering them to run them one after the other instead of all at the same time. To do that, we'll need to find out how these processes got started
- To do this we call `ps ax` command which shows all the running processes on the computer, and connect the output to less, to be able to scroll through it.
```shell
$ ps ax | less
# Search for ffmpeg process in less window
$ /ffmpeg
# we got the details of ffmpeg processes, lets try to locate from where they are getting triggered.
$ locate static/001.webm
/src/deploy_videos/static/001.webm  # is the o/p of locate
$ cd /src/deploy_videos/
$ ls -l 
# Now try to see if any of the files contains a call to ffmpeg
$ grep ffmpeg *
$ vim deploy.sh
# It contains some line as:
daemonize -c $pwd /usr/bin/ffmpeg ...
```
- Inside the `.sh` we noticed that the script is using `daemonize` program to run each program separately as if it were a daemon.
- It is okay, if we are converting 1 video. But launching one separate process for each of the video is overloading out server. So, we want to change this one to only one video conversion process at a time.
- We simply achieve this by removing the `daemonize` part in the process.
- But this won't change the processes that are already running.
- So, lets stop the already running processes, but not cancel them.
- `killall -STOP` signal will stop the processes
- `CONT` signal will make them re-run
- while is used to send the CONT signal and wait until the process is done.
- Below while loop remains true as long as the process exists and fails once the process goes away.
```shell
$ killall -STOP ffmpeg
$ for pid in $(pidof ffmpeg); do while kill -CONT $pid; do sleep 1; done; done
$ $ ab -n 500 site.example.com    # to get the average timing of 500 
# Now we get a lower mean time for page loading
```
## Conclusion
- We tried multiple approaches to bring the consumption down.
- Check out the following links for more information:
    - https://docs.microsoft.com/en-us/sysinternals/downloads/procmon 
    - http://www.brendangregg.com/linuxperf.html
    - http://brendangregg.com/usemethod.html
    - [Activity Monitor in Mac](https://support.apple.com/en-gb/guide/activity-monitor/welcome/mac)
    - [Performance Monitor on Windows](https://www.windowscentral.com/how-use-performance-monitor-windows-10)
    - https://www.digitalcitizen.life/how-use-resource-monitor-windows-7
    - https://docs.microsoft.com/en-us/sysinternals/downloads/process-explorer
    - https://en.wikipedia.org/wiki/Cache_(computing)
    - https://www.reddit.com/r/linux/comments/d7hx2c/why_nice_levels_are_a_placebo_and_have_been_for_a/

# Slow Code
We should always start by writing clear code that does what it should, and only try to make it faster if we realize that it's not fast enough.
### **Rule of thumb for writing code**
- Write code that is readable
- Easy to maintain
- Easy to understand

**If we have to optimize our code, then we can think based on below lines:**
- storing data that was already calculated to avoid calculating it again
- using right data structures
- reorganizing the code, so that the computer can stay busy while waiting for information from slow sources like (disk or network)
- **` To know what sources of slowness we need to address, we have to figure out where our code is spending most of its time. There are bunch of tools that can help us with this called ` profilers**
## Profilers:
- A tool that measures the resources that our code is using, giving us a better understanding of what's going on.
- In particular, they help us see how memory is allocated and how the time is spent.
- Profilers are specific to each programming language.
    - `gprof` to analyze a c program
    - `c-Profile` module to analyze a python program
- Using these tools we can see which functions are called by our program, how many times each function was called and how much time program spent on each of them.
- To optimize code, we need to restructure it to avoid repeating expensive actions.
- **Expensive operations include:**
    - Parsing a file
    - Reading data over a network
    - Iterating through a whole list
    - Creating copies of the structures that we have in memory
        - If these structures are big, it can be pretty expensive to create those copies. **DO IT ONLY IF UNAVOIDABLE**
    - **DO NOT PERFORM EXPENSIVE ACTIONS INSIDE LOOP**
    - What if parsing a file/downloading data is taking a lot of time:
        - Create local copies/ Create Caches
        - Cache saves us time and make our programs run faster.
        - Cache Decisions:
            - How often we need to update the cache
            - What to do if the data is stale

### Time command
- Running a script with time command produces 3 values
- **Real**:
    - The amount of actual time that it took to execute the command. This value is sometimes called Wall-clock time.
- **User**:
    - The time spent in operations in the user space.
- **Sys**
    - The time spent in doing system-level operations.

### Using Profilers
There are different profilers available for python that works for different use-cases.
- Here we are going to demo pprofile3
- -f flag tells which file format to use
- -o flag tells where to store the output
- We can open the generated file usinf `kcachegrind` tool
- Check the graph to see which function is taking most of the time
- Then navigate to the function, to understand why it is consuming more time
```python
$ pprofile3 -f callgrind -o profile.out ./myscript.py
$ kcachegrind profile.out
```
## Conclusion
- [More About Improving Our Code](https://en.wikipedia.org/wiki/Profiling_(computer_programming))

<br>

# Parallelizing Operations
-  Reading information from disk or transferring it over the network is a slow operation.
- In typical scripts while this operation is going on, nothing else happens. The script is blocked, waiting for input or output while the CPU sits idle.
- **One way we can make this better is to do operations in parallel.**
    - So that, while the computer is waiting for the slow IO, other work can take place. 
    - The tricky part is dividing up the tasks so that we get the same result in the end. There's actually a whole field of computer science called `concurrency`, dedicated to how we write programs that do operations in parallel. 
- **Lets understand what OS already does for us:**
    - If a computer has more than one core, the operating system can decide which processes get executed on which core. Processes running on different cores always execute in parallel.Each of them has its own memory allocation and does its own IO calls. So, **`Easiest way to run operations in parallel is  just to split them across different processes, calling your script many times each with a different input set, and just let the operating system handle the concurrency`**
    - **Another easy way is to have a good balance of different workloads that you run on your computer.** If you have a process that's using a lot of CPU while a different process is using a lot of network IO and another process is using a lot of disk IO, these can all run in parallel without interfering with each other.
    - When using the OS to split the work and the processes, these processes *`don't share any memory, and sometimes we might need to have some shared data.`* In that case, we'd use **threads.**

### **Threads:**
Threads let us run parallel tasks inside a process. This allows threads to share some of the memory with other threads in the same process. Since this isn't handled by the OS, we'll need to modify our code to create and handle the threats.
- Threading Implementation is language specific.
- In Python, we can use the Threading or AsyncIO modules to do this.
- These modules let us specify which parts of the code we want to run in separate threads or as separate asynchronous events, and how we want the results of each to be combined in the end. <br>
- **NOTE:**
    - depending on the actual threading implementation for the language you're using, it might happen that all threads get executed in the same CPU processor.  In that case, if you want to use more processors, you'll need to split the code into fully separate processes.

### **Multiprocessing**
- If your script is mostly just waiting on input or output, also known as I/O bound, it might matter if it's executed on one processor or eight.
- If your script is CPU bound,  you'll definitely want to split your execution across processors.
- **NOTE:**
    - If we're trying to read a bunch of files from disk and do too many operations in parallel, the disk might end up spending more time going from one position to another then actually retrieving the data
    - If we're doing a ton of operations that use a lot of CPU, the OS could spend more time switching between them than actually making progress in the calculations we're trying to do.
    - **TIP:** When doing operations in parallel, we need to find the right balance of simultaneous actions that let our computers stay busy without starving our system for resources.

# Slowly growing in complexity
Let's say you're writing a secret Santa script where each person gives a secret gift to one other randomly assigned person. The script randomly selects pairs of people and then sends an email to the gift-giver telling them who they're buying a present for. 

- If you're doing this for the people working on your floor, `you might just store the list of names and emails in a CSV file`
    - Company keep hiring more and more people and now parsing CSV is taking a lot of time
    - This is where you might want to consider a different technology
- `You could decide to store your data in a SQLite file`. This is a lightweight database system that lets you query the information stored in the file without needing to run a database server.
    - But imagine that you have keep on adding more and more features to the service.
        - It now includes a way to create a wish list
        - a machine learning algorithm that suggests possible gifts
        - a tracker that keeps a history of each present given.
    - And since people at your company love the program so much, you've made it an external service available to anybody.
        - Keeping all the data in one file would be too slow. 
        - So you'll need to move to a different solution.
- Use a full fledged database server
    - And run it on a separate machine than the once running secret santa service.
    - If the service becomes really really popular, 
        - you might notice that your database isn't fast enough to serve all the queries
- `you can add a caching service like memcached` which keeps the most commonly used results in RAM to avoid querying the database unnecessarily.

*`So we've gone from hosting the data in a CSV file to having it in a SQLite file then moving it to a database server and finally using a dynamic casher in front of the database server to make it run even faster.`*

**TIP :** You need to pay attention to how the service is growing to know when you need to take the next step to make it work best for the current use case.

# Dealing with Complex Slow Systems:
What to do if your complex system is slow ?
- You want to find the bottleneck that is causing your system to under perform
    - Is it the generation of **dynamic pages** on the webserver
    - Is it the queries to the **database**
    - Is it due to calculations going on in **fulfillment process**
- one key thing is to have a **good monitoring infrastructure** that lets you know where the system is spending the most time.

Suppose, you notice that website is loading slowly. 
- When you check the web server, you noticed that it is not overloaded.
- Most of the time is spent waiting on network calls
- When looking at your database server
    - you find that it's spending a lot of time on Disk I/O
    - This shows that there's a problem with how the data is being accessed in the database
- Now, check if the indexing is happening properly ?
    - When a database server needs to find data, it can do it much faster if there's an index on the field that you're querying for.
    - On the flip side, if the database has too many indexes, adding or modifying entries can become really slow because all of the indexes need updating.
    - **We need to look for a good balance of having indexes for the fields that are actually going to be used.**
- If the problem is not solved by indexing and there are too many queries for the server to reply to all of them on time ?
    - You might need to look into either caching the queries or distributing the data to separate database servers.
- Suppose, service is slow, you see that the CPU on the web serving machine is saturated
    - The first step is to check if the code of the service can be improved
    - If it's a dynamic website, we might try adding caching on top of it.
    - But if the code is fine and cache does not help because the problem is that there are just too many requests coming in for 1 machine to answer them all

- Too many requests for a single machine
    - Distribute the load across more computers
        - You might need to reorganize the code so that it's capable of running in distributed system instead of single computer.
        - This might take some work, but once you have done it, you can easily scale your application to as many requests as needed.
# Using Threads to improve the run time
### Problem Statement:
We need to create thumbnail images from the full-size images. But total number of images is very-very huge in count. Write a script to convert the images ?

**Approach:**
- Try to come-up with a naive script which get the work done
- Try to measure the efficiency of naive script
    - Take some small sample of images from total images and run your script against the test sample. Measure the time taken using the `time` command.
- **Lets try to make the script faster by processing images in parallel.**
```python
### Thread based model
from concurrent import futures

def process_file(root, basename):
    pass

if __name__ == "__main__":
    executor = futures.ThreadPoolExecutor()
    executor.submit(process_file, root, basename)
    print("waiting for all threads to finish")
    executor.shutdown()
```
```python
### Process based model
from concurrent import futures

def process_file(root, basename):
    pass

if __name__ == "__main__":
    executor = futures.ProcessPoolExecutor()
    executor.submit(process_file, root, basename)
    print("waiting for all threads to finish")
    executor.shutdown()
```
- First we need to create an **Executor**
    - The process that's in charge of distributing teh work among the different workers.
- **Futures Module**
    - Provides a couple of different executors, one for using threads and another for using processes

## More About Complex Slow Systems
- https://realpython.com/python-concurrency/
- https://hackernoon.com/threaded-asynchronous-magic-and-how-to-wield-it-bba9ed602c32

