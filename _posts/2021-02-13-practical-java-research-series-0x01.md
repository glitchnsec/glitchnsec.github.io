---
layout: post
date:   2021-02-13 02:26:00 -0500
categories: research java
---

Research Methodologies and Exploitation Considerations for Java-based targets
---

<h3><b>Abstract</b></h3>
This is the first blog of this series and we hope to cover some general knowledge required for Java research. Topics covered include: De-compiling Java Classes, Version Diffing, Rebuilding Source Tree, IDEs, Remote Debugging and Java versioning. This post assumes knowledge of basic Java and programming terminologies and basic knowledge of the *NIX OSes. If you encounter an unexplained term, feel free to do some Google-ing and then return!

I'll try to be accurate in my description or explanations, but errors are bound to exist. I'm not a Java developer so if you come across something outdated, wrong or incomplete, please feel free to reach out!
<br>

<h3>De-compiling Java Classes</h3>

The Java compiler `javac` compiles Java source code into bytecode which the JVM (Java Virtual Machine) will execute at runtime. Often times, i.e in the absence of heavy obfuscation and anti-decompilation, the original source may be retrieved from compiled java archives and class files.

Many tools exist to achieve this, some standalone and some built into IDEs. Of relevance to this blog is the [CFR Java decompiler](https://www.benf.org/other/cfr/)

<h3>Rebuilding Source Tree</h3>
The goal here is not to reconstruct the source tree perfectly but to get it just right enough to be used in a remote debugging environment. Find all the JAR archives you are interested in and decompile them into an output directory with CFR.

Example:
```
find <location> -name '*.jar' -exec java -jar <CFR jar path> --outputdir <output dir> {} \;
```

For open source projects, simply download the sources for the relevant versions

<h3>Version Diffing</h3>

Diffing is a common technique in the Software Engineering world. However, it also plays a crucial role in Security Research. Diffing different product binaries or sources to find changes between versions or to trace the origin or fix of a vulnerability is an essential research step.

Many tools exist to achieve this, for the purpose of this blog we will be using the GNU diff `man 1 diff` and the diffing capabilities of VSCode and GitHub

Performing a recursive diff on two directories can be achieved with following command

```sh
diff -r <source tree root (before)> <source tree root (after)> diff_output.diff
```

For an open source repository hosted on GitHub, find the relevant tag/branch versions and navigate to the appropriate URL
Template:

```url
https://github.com/user/<repo-name>/compare/<base version>...<compare version>
```
Example: Apache MQ diff between 5.15.14 and 5.15.13

```url
https://github.com/apache/activemq/compare/activemq-5.15.13...activemq-5.15.14
```

<h3>IDEs</h3>

Whilst it may be very tempting to engage in the battle of the IDEs, permit me to rule that the IDE of choice in these series is the [IntelliJ IDEA](https://www.jetbrains.com/idea/) by JetBrains. The community version works just fine for our needs.

<h3>Remote Debugging</h3>
Most times, the research setup involves, a running instance of the target application, a client application (optionally) and the source. Most applications have startup scripts that contain the arguments to the JVM. To setup up remote debugging, we simply find the appropriate files, either through grepping and enable the debug lines (often commented), or we introduce the debug lines in the appropriate startup files and re-launch the application. Finally, we connected the IDE to the debug server and add in the sources.. More details on this later.

<h3>Java versioning</h3>
It is important to recognize the Java environment requirement the target application is intended for. Most applications support at least Java 8. Once the appropriate Java environment has been identified, relevant binaries can be obtained from [OpenJDK](https://openjdk.java.net/) downloading only the JDKs is sufficient. Configure the installations such that tools and IDEs can easily identify Java on the system.

Example: Installing Open JDK 8 on a Ubuntu system

```bash
sudo apt-get install openjdk-8-jdk
```
<h3>Conclusion</h3>

Java targets may appear as application platforms, libraries, web applications and client applications. In the next few blogs on this series, we discuss research specifics to these target groups. The series concludes with an exploration of critical vulnerability classes plaguing Java applications, a shallow dive into exploitation techniques focused on RCE and additional resources.