---
layout: post
date:   2019-07-07 12:59:00 -0500
categories: research
title: SpiderMonkey Research - 0x01 - Setup & Debug
---

Anyone else find Setup of any sort to be somewhat always complicated? Whether its setting up dev environment, going through documented installation steps etc. For me most things just would not work like they have been documented to. Maybe it's just me. Well, this was one of those. Check out the "Into the weeds the section" at the end of the post for more details on mistakes, frustrations, challenges and general lolz and facepalm moments and troubleshooting tips.<br />
<br />
<b><u>Obligatory Environment Details</u></b><br />
<br />
1. Windows 10 Enterprise Evaluation v1809<br />
<br />
<b><u>Setup &amp; Debug</u></b><br />
In each step, where applicable, the superscript numbers reference the official Mozilla Documentation. This would help with troubleshooting when required.<br />
<br />
1. <u>Getting SpiderMonkey Source</u>.<br />
<ul>
<li>Get the latest <a href="https://ftp.mozilla.org/pub/mozilla.org/mozilla/libraries/win32/MozillaBuildSetup-Latest.exe" target="_blank">Mozilla-build</a> <sup><a href="https://developer.mozilla.org/en-US/docs/Mozilla/Developer_guide/Build_Instructions/Windows_Prerequisites#Required_tools">1</a></sup>. Use default installation settings.</li>
<li>Run the <code>start-shell.bat</code> file in <code>C:\mozilla-build</code></li>
<li>From within a new Shell navigate to a location where you would want the source to reside</li>
<li>Fetch the source using mecurial<br /><code>hg clone https://hg.mozilla.org/mozilla-central</code><br />This is going to take some time</li>
</ul>
<ul></ul>
2.<u> Compiling SpiderMonkey</u> <sup><a href="https://developer.mozilla.org/en-US/docs/Mozilla/Projects/SpiderMonkey/Build_Documentation#Developer_(debug)_build">2</a></sup><br />
<br />
&nbsp; &nbsp; SpiderMonkey requires some pre-requisites as mention in Mozilla build docs.<br />
<ol>
<li>Follow the instructions from <a href="https://developer.mozilla.org/en-US/docs/Mozilla/Developer_guide/Build_Instructions/Windows_Prerequisites#Getting_ready" target="_blank">Getting-Ready till Required Tools. </a>During the Visual Studio Setup also select clang for Windows in the individual components.&nbsp;</li>
<li>Download and Install <a href="https://www.microsoft.com/EN-US/DOWNLOAD/DETAILS.ASPX?ID=44266" target="_blank">Microsoft Visual C++ Compiler for Python</a>&nbsp;</li>
<li>Install .NET Framework 3.5 from "Turn Windows Features Off or On"</li>
<li>launch the start-shell.bat script and navigate to your repo location</li>
<li>Execute the following command to properly configure your environment and catch any missing deps</li>
<br /><code><code>mozilla-central$ </code>./mach bootstrap</code>
<li>In the js/src folder create a build directory. Mine is called BUILD_DBG.OBJ (as recommended by Mozilla)<br /><code>mozilla-central/js/src$ mkdir BUILD_DBG.OBJ</code> </li>
<li>From within the new directory run the following commands </li>
</ol>
```
mozilla-central/js/src/BUILD_DBG.OBJ$ autoconf-2.13
mozilla-central/js/src/BUILD_DBG.OBJ$ ../configure --enable-debug --disable-optimize --enable-nspr-build
mozilla-central/js/src/BUILD_DBG.OBJ$ mozmake -j4 -s
```
*Note the official docs use autoconf2.13 but checking `C:\mozilla-build\msys\local\bin` we see the correct command*

3. <u>Debugging</u><br />
<u><br /></u>
&nbsp;&nbsp;&nbsp; We want to be able to instrument the JS engine from within the debugger and break into it to examine memory. We would be using WinDbg.<br />
<ol>
<li>Install the WinDbg Preview App from the Windows Store. You can find the app by searching WinDBG in Cortana search box. If you prefer, you can install WinDbg by following these instructions from <a href="https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/debugger-download-tools" target="_blank">Download Debugging Tools for Windows</a> </li>
<li>Configure WinDbg as a Postmortem debugger by following these instructions from <a href="https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/enabling-postmortem-debugging" target="_blank">Microsoft</a></li>
<li>Configure Symbol path in the debugger<br /> <pre><code>.sympath srv*c:\symbols*https://msdl.microsoft.com/download/symbols</code></pre>
<pre><code>.sympath+ srv*C:\symbols*https://symbols.mozilla.org/</code></pre>
</li>
<li>Next, we would be using a tool by <a href="https://twitter.com/0vercl0k/status/1084555927783563264?s=20" target="_blank">@0vercl0k</a>. Follow the installation guide on <a href="https://github.com/0vercl0k/windbg-scripts" target="_blank">GitHub</a><ol>
<li>Note that to use the command interface on the debugger, you'll have to 
have attached to a process. You can start and attach to the JS shell at 
<pre><code>/path/to/debug/build/dist/bin/js.exe</code></pre>
</li>
</ol>
</li>
<li>Install python3.7 from <a href="https://www.python.org/downloads/release/python-373/" target="_blank">Python</a></li>
<li>Two notable functions only available in Debug build of JS shell are <code>dumpObject()</code> and <code>objectAddress()</code></li>
<li>Now we should be set to start our Journey</li>
</ol>
<br />
<b><u>Resources</u></b><br />
<br />
<ol>
<li>https://developer.mozilla.org/en-US/docs/Mozilla/Developer_guide/Build_Instructions/Windows_Prerequisites</li>
<li>https://developer.mozilla.org/en-US/docs/Mozilla/Projects/SpiderMonkey/Build_Documentation</li>
<li>https://doar-e.github.io/blog/2018/11/19/introduction-to-spidermonkey-exploitation/#setting-it-up</li>
</ol>
<br />
<br />
<b><u>Into the Weeds</u></b><br />
I originally tried to use the source from GitHub using the following command. <br />
<code>git clone --depth 1 https://github.com/mozilla/gecko-dev.git</code><br />
The build time using this method was supposed to be significantly faster. However, I ran into problems running step 7 using that source.The major reason was <code>./mach bootstrap</code> which is supposed to help with dependencies only works with the Mecurial source. Technically, I could go through the code, see what it does and validate the environment myself but that would be way too much work (I think).<br />
<br />
Now, there's definitely a way to use Mecurial and still get cloning to complete in a significantly shorter time but I did not explore this option. Feel free to share your quick clone/build tips in the comment below.<br />
<br />
Oh, I almost forgot. Step 2 and 3 where written after multiple fails in step 5. So you may have more fails but that means step 5 is really doing it's job. Just make the fix to your environment as it suggest. Also it does take sometime.<br />
<br />
I also ran into problems using the following options for configure. <code>--host=x86_64-pc-mingw32 --target=x86_64-pc-mingw32</code>. Definitely, If you have some advice regarding this please share in the comments! But I don't think we need this.<br />
<br />
I ran config without the <code>--enable-nspr-build</code> option and mozmake failed with <code>'prinit.h' not found</code>. This was frustrating because I thought I had done everything right. A quick check on my configure and I returned here to update Step 7!<br />
<br />
In the debugging stage. Setting the symbols path, at this point of writing. I'm not sure if that step is required. I seemed to be able to find symbols without specifying the symbols path. This is probably a result of the debug build.<br />
<br />
Special Thanks to 0vercl0k, his blog on Introduction to SpiderMonkey Exploitation<sup><a href="https://doar-e.github.io/blog/2018/11/19/introduction-to-spidermonkey-exploitation">3</a></sup> is a useful reference as I compare and contrast with LiveOverflow's work on Webkit. It has really helped fast-track my progress