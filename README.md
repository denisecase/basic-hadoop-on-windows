# Basic Hadoop on Windows

> How to install Apache Hadoop (HDFS, YARN, MapReduce) on Windows 10.

---

## Recommended Prerequisites

- Be able to open PowerShell as administrator and run commands
- Be able to stop a PowerShell process with CTRL+C
- Be able to close PowerShell with ALT+SPACE C
- Chocolatey, the Windows package manager
- OpenJDK (install or upgrade to latest using Chocolatey)
- Apache Maven, Java build tool (install or upgrade)
- VS Code for editing configuration files (install or upgrade)

```PowerShell
choco install openjdk -y
choco install maven -y
choco install vscode -y
choco list -local
```

---

## Hadoop Needs Java with No Spaces (and Yarn needs JDK 8)

Choco installs OpenJDK to C:\Program Files\OpenJDK\jdk-14.0.1.

- This works fine for most programs, but not Hadoop. 
- Hadoop cannot have spaces in the path. 
- Yarn cannot have a version greater than 8.
- Keep Windows Environment Variables at the original Chocolatey path. (Windows 1 below.)
- Use a special "no spaces in the path" JDK 8 version for Hadoop (set in hadoop-env.cmd in Windows 2 below.)

An easy way to get this version:

- Go to [https://adoptopenjdk.net/upstream.html](https://adoptopenjdk.net/upstream.html)
- Download OpenJDK 8 (LTS) for Windows x64 JDK 103 MB as a zip file to C:\.
- Right-click / Extract all / Extract. 
- Move the openjdk-8u252-b09 folder from C:\OpenJDK8U-jdk_x64_windows_8u252b09 to C:\.

Now we have JDK 8 with no spaces available for Hadoop. 

---

## Hadoop 1 - Get Hadoop

- Go to <https://hadoop.apache.org/releases.html>.
- Download most current binary to C:\

In C:\, open PowerShell as Administrator and run:

```PowerShell
tar xzvf hadoop-3.2.1.tar.gz
```

This creates C:\hadoop-3.2.1. Explore the subdirectories. Find jars, webapps, and more.

## Hadoop 2 - Add winutils

- Go to [https://github.com/cdarlint/winutils](https://github.com/cdarlint/winutils).
- Find the your version of Hadoop. 
- Download the correct version of winutils.exe to C:\hadoop-3.2.1\bin.
- Download the correct version of hadoop.dll to C:\hadoop-3.2.1\bin.

## Hadoop 3 - Update jar

- Rename your C:\hadoop-3.2.1\share\hadoop\hdfs\hadoop-hdfs-3.2.1.jar to hadoop-hdfs-3.2.1.bk (5820 KB)
- Save the [share-hadoop-hdfs/hadoop-hdfs-3.2.1.jar](./share-hadoop-hdfs/hadoop-hdfs-3.2.1.jar) in this repo (5780 KB) to your C:\hadoop-3.2.1\share\hadoop\hdfs\ folder.
- Read more at <https://kontext.tech/column/hadoop/379/fix-for-hadoop-321-namenode-format-issue-on-windows-10>
- Source: [https://github.com/FahaoTang/big-data/blob/master/hadoop-hdfs-3.2.1.jar](https://github.com/FahaoTang/big-data/blob/master/hadoop-hdfs-3.2.1.jar)

---

## Windows 1 - Configure System Environment Variables

Edit the system environment variables. All paths must reflect the installation location on your machine. These paths reflect the locations on my machine.

- Open with Windows key + "Edit the system environment variables" / Environment variables / System variables (bottom half of the window).
- Set HADOOP_HOME to C:\hadoop-3.2.1
- Verify JAVA_HOME is C:\Program Files\OpenJDK\jdk-14.0.1 OR C:\Program Files\Java\jdk1.8.0_211 (or similar if you want JDK 8)
- Verify M2_HOME C:\ProgramData\chocolatey\lib\maven\apache-maven-3.6.3

Edit Path - use Edit or New entry to add these locations to your path. Note the \bin at the end. Spelling and capitalization are critical.

- Verify: C:\Program Files\OpenJDK\jdk-14.0.1\bin OR %JAVA_HOME%\bin (cannot have both)
- Verify: %M2_HOME%\bin
- New: %HADOOP_HOME%\bin

Caution: You may get errors if you have multiple Java options in your path. I delete Oracle JDK and use only the most current OpenJDK.

## Windows 2 - Edit Hadoop Files

Open C:\hadoop-3.2.1\etc\hadoop in VS Code to edit these files as shown in [./etc-hadoop](./etc-hadoop).

- core-site.xml
- hadoop-env.cmd (set JAVA_HOME=C:\openjdk-8u252-b09 - no spaces, must be version 8)
- hdfs-site.xml (path values must be formatted as shown in the example)
- mapred-site.xml
- yarn-site.xml

## Windows 3 - Format new namenode

Create and format a new namenode (at the path specified in hdfs-site.xml). In PowerShell as Adminstrator, run:

```PowerShell
hdfs namenode -format
```

Verify it runs without errors. The process will shutdown the new namenode after formatting. 

Explore the new folder created at C:\hadoop-3.2.1\nodes\namenode (or the path given in hdfs-site.xml).

---

## Running A Psuedo-Distributed Hadoop Cluster on Windows

Start the namenode first. From your desktop, run PowerShell as Administrator and start the namenode service. Leave the window open to keep the process running.

```PowerShell
hdfs namenode
```

Then, start the datanode. Important: Make sure the datanode folder does NOT exist. If it does exist, stop the process if necessary and delete the directory. Once gone, from your desktop, run PowerShell as Administrator and start the datanode service. Leave the window open to keep the process running.

```PowerShell
hdfs datanode
```

Notes:

- These locations are set in hdfs-site.xml. The datanode directory must NOT EXIST before running.
- Open these directories when running to see what's happening on disk.
- Exit gracefully when done. If you leave processes running, you may need to close them manually with CTRL+ALT+DELETE / Task Manager. Find the process, right-click / End task.

## Monitoring Hadoop

Open a browser to <http://localhost:9870/>. Explore.

## Interacting with HDFS

HDFS is a powerful, fast file system for managing large files with commodity hardware. It breaks each file added into blocks and replicates them across different machines (and different racks) on the cluster. By default, it creates 3 copies. It does this automatically. 

We can list the contents, get help, make directories in HDFS to organize our work, put files into HDFS directories, and get files out of HDFS.

Make sure your nodes are running. From your desktop, open a third PowerShell as Administrator and use hadoop fs to access the file system. Try these [Frequently Used HDFS Shell Commands](https://stepupanalytics.com/frequently-used-hdfs-shell-commands/)

```PowerShell
hadoop fs -ls /
hadoop fs -df hdfs:/
hadoop fs -mkdir /wordcount
hadoop fs -put C:/romeoandjuliet.txt /wordcount
hadoop fs -ls /
hadoop fs -ls /wordcount
hadoop fs -help
```

## Start up YARN

Open PowerShell as an Administrator in sbin and run (no spaces): 

```PowerShell
.\start-yarn.cmd
```

This will start the Resource Manager and Node Manager services.

Open a browser to <http://localhost:8088/cluster/nodes> to see your managed cluster. 

## References

- [Romeo and Juliet](http://shakespeare.mit.edu/romeo_juliet/full.html)
- [Frequently Used HDFS Shell Commands](https://stepupanalytics.com/frequently-used-hdfs-shell-commands/)

## Repository

- <https://github.com/denisecase/basic-hadoop-on-windows>
