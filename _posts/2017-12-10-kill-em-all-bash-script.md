---
layout: post
title: "Kill 'em all! - A bash script to kill all sub-tree of a parent process"
date: 2017-12-10
---

I spent 2 hours yesterday searching for a quick way to kill a parent process and all its children 
and grand-children processes, i.e the whole process sub-tree rooted at the process. To my surprise, 
I couldn't find a straight forward way to achieve this.

# Killing The Parent

Child processes are not necessarily killed when the parent is killed. In my case, the direct child 
processes were killed after killing the parent. However, grand-children processes were not killed 
and were adopted by the init process. Killing child processes depends whether the parent process 
propagates the killing signal received to its children or not.

# Using Group PID `GPID`

One elegant way I found on Stack Over Flow was this nice command:

```
kill -- -$(ps -o pgid= $PID | grep -o '[0-9]*')
```

It first finds the id `GPID` of the process group that the given pid belongs to. Then, it 
sends the default `TERM` signal to all the processes within that group `GPID`. Nice, short and 
clean, isn't it? However, it didn't work for me.

I was calling this command from a bash script to kill a background process it creates earlier, the 
bash script had the same `GPID` as that process. So it ended up killing itself not only the process
sub-tree.

This command doesn't only kill the child processes but also kills the parent processes if they 
belong to the same group.

# Ugly But Working

Eventually, I came across this 10 years old [post](https://deadlockprocess.wordpress.com/2008/01/23/how-to-using-bash-to-kill-a-parent-process-and-all-spawned-child-processes-2/). The guy introduced a long script that finds all the direct children processes of the given `PID`s then their children and
 so on, till no further processes found. Then kills all the found `PID`s. It worked like a charm!

I tweaked the script a little bit, I used `grep` instead of `precgrep` and made other minor changes. 
The script can be found below:

<script src="https://gist.github.com/HazemSamir/70505f4ba5efd02c62009ce3b1cd46d7.js"></script>

I also added an alias for it to make it easy to call from anywhere after that. I added this 
line to `~/.bashrc`:

```
alias killem="bash ~/killem.sh"
```
