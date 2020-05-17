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

## Copy OpenJDK to C:\ for Hadoop

Choco installs OpenJDK to C:\Program Files\OpenJDK\jdk-14.0.1.

- This works fine for most programs, but not Hadoop.
- My solution is to COPY the most recent OpenJDK to C:\
- Windows Environment Variables still use the Chocolatey path. (Windows 1 below.)
- Hadoop will use JAVA_HOME=C:\OpenJDK\jdk-14.0.1 (Windows 2 below.)

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

- Go to <https://github.com/steveloughran/winutils>
- Or <https://github.com/cdarlint/winutils> (for newer versions)
- Download the correct version of winutils.exe to C:\hadoop-3.2.1\bin.
- Download the correct version of hadoop.dll to C:\hadoop-3.2.1\bin.

## Hadoop 3 - Create Node Directories

- Create C:\hadoop-3.2.1\nodes
- Create C:\hadoop-3.2.1\nodes\namenode

## Hadoop 4 - Update jar

- Rename your C:\hadoop-3.2.1\share\hadoop\hdfs\hadoop-hdfs-3.2.1.jar to hadoop-hdfs-3.2.1.jar.bk (5820 KB)
- Save the share-hadoop-hdfs\hadoop-hdfs-3.2.1.jar in this repo (5780 KB) to your C:\hadoop-3.2.1\share\hadoop\hdfs\ folder.

---

## Windows 1 - Configure System Environment Variables

Edit the system environment variables. All paths must reflect the installation location on your machine. These paths reflect the locations on my machine.

- Open with Windows key + "Edit the system environment variables" / Environment variables / System variables (bottom half of the window).
- Set HADOOP_HOME to C:\hadoop-3.2.1\bin
- Verify JAVA_HOME is C:\Program Files\OpenJDK\jdk-14.0.1
- Verify M2_HOME C:\ProgramData\chocolatey\lib\maven\apache-maven-3.6.3

Edit Path - use Edit or New entry to add these locations to your path. Note the \bin at the end. Spelling and capitalization are critical.

- Verify: %JAVA_HOME%\bin
- Verify: %M2_HOME%\bin
- New: %HADOOP_HOME%\bin

Caution: You may get errors if you have multiple Java options in your path. I delete Oracle JDK and use only the most current OpenJDK.

## Windows 2 - Edit Hadoop Files

Open C:\hadoop-3.2.1\etc\hadoop in VS Code to edit these files as shown in <./etc-hadoop>:

- core-site.xml
- hadoop-env.cmd (set JAVA_HOME=C:\OpenJDK\jdk-14.0.1)
- hdfs-site.xml (values must be formatted as shown)
- mapred-site.xml
- yarn-site.xml

## Windows 3 - Format new namenode

We need to format a new namenode. In PowerShell as Adminstrator, run:

```PowerShell
hdfs namenode –format
```

It will ask: "Re-format filesystem in Storage Directory root= C:\hadoop-3.2.1\nodes\namenode; location= null ? (Y or N)"

Type Y and hit ENTER to continue. 

Verify it formats without errors. The process will shutdown the namenode after formatting.

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

## References

- [Romeo and Juliet](http://shakespeare.mit.edu/romeo_juliet/full.html)
- [Frequently Used HDFS Shell Commands](https://stepupanalytics.com/frequently-used-hdfs-shell-commands/)

## Repository

- <https://github.com/denisecase/basic-hadoop-on-windows>
