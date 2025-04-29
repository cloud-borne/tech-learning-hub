---
title: How to use a Thread Dump Analyzer
subtitle:

# Summary for listings and search engines
summary:

# Link this post with a project
projects: []

# Date published
date: "2021-09-05T00:00:00Z"

toc: true

# Date updated
lastmod: "2021-09-05T00:00:00Z"

# Is this an unpublished draft?
draft: true

# Show this page in the Featured widget?
featured: false

# Featured image
# Place an image named `featured.jpg/png` in this page's folder and customize its options here.
image:
  caption:
  focal_point: "Center"
  placement: 1
  preview_only: false

authors:
- admin

tags:
- Monitoring

categories:

---

<!--more-->

### Overview

A common debugging technique for web application is to get a dump of all the JVM's threads at a point in time for analysis. The output of these
thread dumps is often a text file that is very large and can be cumbersome to analyze fully. To aid in this debugging effort, IBM has created a tool
that will read the thread dump text file and present the results to the user in an easy-to-consume format.

### Installation
IBM's Thread Dump Analyzer is a Java client contained in an executable JAR file. Installation, therefore, is just downloading and unzipping the
following file: https://public.dhe.ibm.com/software/websphere/appserv/support/tools/jca/jca469.jar

### Starting the Analyzer
To run the Analyzer, execute the JAR file. Note that this does require that Java is installed on your machine, and that java/bin is in your system or
user path. You can execute this command either from a cmd shell or as the Target of a shortcut.
```
  java -jar jca469.jar
```

### Usage
Full instructions are detailed within the readme in the zip file that you downloaded. In the below sections I will describe some common things to get you
started, but will not paraphrase all of the documentation already provided by IBM.

#### Opening a thread dump file
A good first step is to read in a thread dump file.
Select File --> Open, or use the Folder icon button to open and browse to a thread dump txt file

 {{% callout note %}}
 If you want to try out the tool but don't have a current thread dump, a sample one is available for use here:
 {{% /callout %}}

You should now see information about the JVM at the time the thread dump was taken. So, JVM args, build versions, and some summary stats
on threads. In the example thread dump file, look at the Thread Status Analysis table in the lower pane to see that 58% of the threads were
"Waiting" and 23% were "Blocked".

#### Viewing Thread Stacks
Our next step in debugging is to dive deeper into what the different threads were doing at that time.
1. After you have opened a thread dump file, it appears in the table at the top of the screen.
2. Right-click on a row in the table, and select "Thread Detail"

This should open up a view with three columns.

1. The left contains the name and status of each thread running in the JVM at that time. Select a thread for more details
2. The right contains the stack trace of selected thread when the dump was run
3. The middle contains information about other threads' dependencies on this thread. This pane only shows if dependencies are present for
the selected thread.

In our example thread dump file again, take a look at thread "Web Container : 4". This thread's stack trace indicates that it was waiting for data on
a socket opened by the Oracle database driver. Above the stack trace, you can see that this thread has a lock on two objects. Note that in the
middle pane, it shows that there are two threads (21, 42) that are waiting on this thread to release those locks.

Next, take a look at "Web Container : 21" which was reported blocked by 4. You can see that in the "Monitor" cell on the right that not only is this
thread waiting on the object locked by 4, but it also owns a lock. In the middle pane, then, you can see a large backup of other threads that
waiting for 21 to release its lock.

#### Comparing Monitors
As you can guess, it may take a while to find the root cause thread in circumstances. Instead of looking at thread detail, then, we want to compare
the threads and their locks against each other to discover these dependencies.
1. Back at the table of open thread dump files, right-click and select "Compare Monitors"
2. Select the java object name (not the thread name) to get more details on that thread's monitors

This view shows similar info as before, but is narrowed down to the monitor info. So, you should see less threads, because it only shows threads
that own locks. Using our example file again, you can see that 3 is blocked by 21, 21 is blocked by 4, and 4 is not blocked (thus a good candidate
for root cause).
