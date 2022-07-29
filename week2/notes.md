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
    - The tricky part is dividing up the tasks so that we get the same result in the end.