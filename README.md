# Basic Hadoop on Windows

> How to install Apache Hadoop (HDFS, YARN, MapReduce) on Windows 10.

---

## Recommended Prerequisites

- Be able to open PowerShell as administrator and run commands
- Be able to stop a PowerShell process with CTRL+C
- Be able to close PowerShell with ALT+SPACE C
- Chocolatey, the Windows package manager

```PowerShell
choco install hadoop -y
refreshenv
choco list -local
```
Close the window with ALT+SPACE C and reopen. 

## JDK 8 With No Spaces

Choco installs JDK 8 with spaces. We need a path with no spaces for Hadoop. 

- Hadoop cannot have spaces in the path.
- Yarn cannot have a version greater than 8.
- Keep Windows Environment Variables at the original Chocolatey path. (Windows 1 below.)
- Use a special "no spaces in the path" JDK 8 version for Hadoop (set in hadoop-env.cmd below.)

An easy way to get this version:

- Go to https://adoptopenjdk.net/upstream.html
- Download OpenJDK 8 (LTS) for Windows x64 JDK 103 MB as a zip file to C:.
- If not permitted to download to C:, download to your "Downloads" folder and move to C:\ when done.
- Right-click / Extract all / Extract.
- Move the openjdk-8u252-b09 folder from C:\OpenJDK8U-jdk_x64_windows_8u252b09 to C:.

Now we have JDK 8 with no spaces available for Hadoop.

Verify you have C:\openjdk-8u252-b09 with the necessary files. 

## Edit Windows System Environment Variables

HADOOP_HOME = C:\Hadoop

JAVA_HOME = location of your most current OpenJDK (this can have spaces)

PATH = only one JDK in the path and %HADOOP_HOME%\hadoop-3.3.0\bin.

## Add winutil files

- Go to [https://github.com/cdarlint/winutils](https://github.com/cdarlint/winutils).
- Get winutils.exe and download the lastest to your C:\Hadoop\hadoop-3.3.0\bin folder.

## Windows 2 - Edit Hadoop Files

Open C:\Hadoop\hadoop-3.3.0\etc\hadoop in VS Code to edit these files as shown in [./etc-hadoop](./etc-hadoop).

- core-site.xml
- hadoop-env.cmd (set JAVA_HOME=C:\openjdk-8u252-b09 - no spaces, must be version 8)
- hdfs-site.xml (path values must be formatted as shown in the example)
- mapred-site.xml
- workers
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

Start your namenode and datanode services as shown above.

Open PowerShell as an Administrator and run: 

```PowerShell
c:\hadoop-3.2.1\sbin\start-yarn.cmd
```

This will start the YARN Resource Manager and the YARN Node Manager services.

Open a browser to <http://localhost:8088/cluster/nodes> to see your managed cluster. 

## Troubleshooting

Issues?  Key things to check for include the following.

Windows Environment Variables

1. Verify you only have one java bin directory in your path.
2. Verify the paths provided match the locations on your machine (use File Explorer to check). 

## References

- [Romeo and Juliet](http://shakespeare.mit.edu/romeo_juliet/full.html)
- [Frequently Used HDFS Shell Commands](https://stepupanalytics.com/frequently-used-hdfs-shell-commands/)

## Repository

- <https://github.com/denisecase/basic-hadoop-on-windows>
