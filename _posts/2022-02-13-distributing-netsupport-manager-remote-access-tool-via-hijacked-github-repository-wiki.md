---
layout: post
date:   2022-02-13 02:26:00 -0500
categories: threat-investigations
---

Distributing NetSupport Manager Remote Access Tool via Hijacked GitHub Repository Wikis
---

<h3><b>Abstract</b></h3>
The following details an interesting threat activity that seemed to have spanned Oct 2021 - Jan 2022. Certain now defunct GitHub accounts were found to update wiki pages having weak write permissions with links to trojanized binaries. Unsuspecting victims were compromised with the known remote access software NetSupport Manager Application. A remote access tool/trojan is a software designed to allow a remote personnel fully control/manipulate a client computer.
<br>

<h3><b>911, What's your emergency?</b></h3>

It's afternoon of Jan 23 2022 and I receive a message from a friend.

![911](/assets/images/help!.png)

Fig.1

I ask a few interrogative questions just to make sure he knows what he saw. Most importantly, I care about Windows Defender has to say about the situation.

![FAQ](/assets/images/faq.png)

Fig.2

I kid you not, I'm not allowing myself to believe it's a remote access trojan. I'm hoping it's like, wireless mice, delayed keyboard strokes etc. Great, Windows Defender is enabled. Call me old fashioned but Defender has come a long way, we'll give it that. Anyways, I ask a few more questions but eventually I scheduled a visit to the crime scene. It is important to note, the scene has been tainted because the victim did destroy some evidence by deleting files in the Downloads folder out of panic.

<h3><b>Defender, what's your report</b></h3>

**Step one**: Ask the guard.

According to the on-prem security guard, Windows Defender, the last potential threat activity was `Nov. 11, 2021`.

![wd_report](/assets/images/wd_report.png)

Fig.3

Defender reports the threat may not have been fully remediated. Defender also notes that threat did exhibit unwanted behavior on `Jan 23, 2022`. This is the date I was contacted.


<h3><b>Do you even persist?</b></h3>

Assuming we have a real threat actor, they have likely prepared for "turning off" the compromised device. Of course, who goes through all the trouble of compromising a victim and doesn't consider maintaining a foothold? In attacker Technique Tactics and Procedure, this is a tactic known as [Persistence](https://attack.mitre.org/tactics/TA0003/).

**Step two**, identify the foothold. There are a couple techniques but one of the first I checked for was auto logon scripts. Specifically, [Startup Folder](https://attack.mitre.org/techniques/T1547/001).

In the user startup folder, `%USERPROFILE%\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\`, the first Indicator of Compromise is discovered.

![Autoruns.ini.lnk image](/assets/images/autoruns_ini_lnk.png)

Fig.14

![Autoruns info](/assets/images/autoruns_dets.png)

Fig.15

A few telltale signs:

- The filename leverages the default windows behavior to hide extensions to fool a user into thinking its a benign file. `Autoruns.ini.lnk` shows as `Autoruns.ini`. A `.lnk` file is a shortcut file to another application in Windows.

- The target is `client32.exe`.
- The target path is not `Program Files` or `Program Files (x86)`
- The file write date is `Nov. 11. 2021`
- The file location was hidden (Not shown. Screenshot not taken prior to remediation)

What is `client32.exe`? Yeah I didn't know too, but let's Google it!

According to `https://file.info` 

![fileinfo](/assets/images/fileinfo.png)

Fig.4

it is a component of NetSupport Manager Application. This application appears to be a product of a legitimate business in the UK. `https://www.netsupportmanager.com/platforms/windows/` 

![net sup remote info](/assets/images/netsupmanreminfo.png)

Fig.5

According to the victim's account, he is unaware of this application's presence.

Alright, so he did in fact get compromised. But How??

<h3><b>Establishing Timelines</b></h3>

Cursory examination confirms the suspicion. Time to roll out the tools and put the puzzle pieces together.

In digital forensics, we care about the five W's and one H. What, Where, Who, When, Why and How.

The device is a 250 GB SSD and 8 GB RAM. Whilst I had the storage to image the RAM, I did not have the storage to image the disk. Of course the scene has been manipulated so pure memory forensics might offer little to no help. A disk image may offer visibility into deleted files. However, time and storage were a limitation.

Given the scenario, I opted to use [Mandiant's Redline](https://www.fireeye.com/services/freeware/redline.html) for the preliminary host investigation. Copy Redline to flash disk and install on the victim machine offline. Then select type of artifacts to be gathered and execute.

Redline has a TimeLine feature and we have a date `Nov 11. 2021` so let's see what artifacts were created around 2pm (according to Windows Defender's report.)

![redline timeline 1](/assets/images/redline_p1.png)

Fig.6

It appears from the prefetch files, SmartScreen prompted the victim for verification. Then a scan by Windows Defender was triggered. Right after the scan concludes, NetSupport Manager file gets written to disk. This behavior suggests two possibilities. Either an RCE in Windows Defender or a self extracting ZIP archive. The later was found to be the case. However, the sample was not reverse engineering so it's inner workings are not known.

<h3><b>The 'What'</b></h3>

Two artifacts are known to us. `Gow-0.8.0.zip` and `client32.exe`. According to Redline's timeline summary, the `Gow-0.8.0.zip` file was obtained from GitHub.

![gow-download](/assets/images/download_dets.png)

Fig.7

The download URL `https://github-releases.githubusercontent.com/426168326/57107ec2-d330-4ed8-829a-35b82902f7de...` gives us the repository ID from which this file was downloaded. Using the undocumented but useful API `/repositories/<ID>`, we can query a repository by ID. 

![github_repo](/assets/images/github_repo.png)

Fig.8

Great (sarcastically), the repository is gone. Interesting because there's a legitimate `Gow` [repository on Github](https://github.com/bmatzelle/gow/releases/tag/v0.8.0) and the latest version? `0.8.0`. At this point, the connection is unclear.

Since the sample on disk is deleted and the repository is gone, we can only learn about the sample if we knew it's hash.

Unfortunately, the download metadata does not contain the file hash, but we see a hash for the `NSM.LIC` file for NetSupport Manager Application. Whilst there are other files like `.dll` and `.ini` files, the binary files may not lead us to much and the `.ini` file is a configuration file likely unique to the victim. The license file gives us the details on **Who** this application was licensed to!

At the time, `NSM.LIC` file was hidden and I missed it during my initial evidence collection. Luckily searching the hash led me to this `https://any.run` [report](https://any.run/report/3dfd99f196890ba6195f1ed90da2f3263b5a2f001d313741df764e7f14606b40/f64884ff-e947-4929-9d9d-101a2de7340d). This was the first sign of another ITW activity. Any run allows the viewing of files uploaded to the community domain.

![NSM license](/assets/images/nsm_license.png)

Fig.9

Alright, we have pseudo-WHO. Save from contacting the Vendor, threat actor `Freddy` could be whoever. The `https://any.run` report was generated `Jan 20. 2022`. The License was generated `Nov. 14 2017`. Threat actor `Freddy` has been in business for over 3 years.

The sample from this `https://any.run` [report](https://any.run/report/3dfd99f196890ba6195f1ed90da2f3263b5a2f001d313741df764e7f14606b40/f64884ff-e947-4929-9d9d-101a2de7340d) appears to exhibit similar behavior to our sample. Are they the same sample? We still need our hash to tell!

Enter Windows Defender. Redline TimeLine suggests Windows Defender wrote scan file for this sample (See Fig.6 - Highlight 3). The file written by Windows Defender was `C:\Users\All Users\Microsoft\Windows Defender\Scans\History\Service\DetectionHistory\17\4AEF7B1B-238E-4397-A59E-67DDDDF5DE97`

The content of the file is binary, the format was not investigated, but it has some obvious wide character strings.

![](/assets/images/wd_detail.png)

Fig.10

Thanks to Windows Defender, we have all three hash types to search the internet with. Our sample hash differs from the sample in the discovered report. However the threat actor is either the same or a licensed copy of Net Support Manager was stolen and used for these attacks.

Searching our sample SHA1 hash `5eaf9684a6f80fcd59aa00228cb5379ec3f026438b3ccdd8d26b25a6f4b53b97` yields the following:

- A [GitHub issue on GammaRay repository](https://github.com/KDAB/GammaRay/issues/658) on GitHub.
  
  ![](/assets/images/hash_found!.png)
  
  Fig.11

- VirusTotal [report](https://www.virustotal.com/gui/file/5eaf9684a6f80fcd59aa00228cb5379ec3f026438b3ccdd8d26b25a6f4b53b97/details). Perfect! Our sample is out there and recognized by a few endpoint protection software as malicious. This is good news. It was first uploaded to VirusTotal `Nov 07, 2021`. So far, threat actor has had 3 months (Oct 2021 - Jan 2022) to work undetected.

We have found our sample on another repository. The GitHub user `dantti` mentions the suggestion '"Restrict editing to users in teams with push access only" on Wiki pages...'. The GitHub user `mleev` had updated the GammaRay Wiki home to include a shortened URL link to another repository containing the malicious sample.

This could mean the `Gow` repository suffered the same attack and sure enough [it did](https://github.com/bmatzelle/gow/issues?q=is%3Aissue+is%3Aclosed).

![](/assets/images/gow_hacked.png)

Fig.12

The GitHub issues were opened `Dec 23, 2021`. The threat actor began making the link updates `Oct 21, 2021` under different GitHub usernames.

![](/assets/images/wiki_updates.png)

Fig.13

![](/assets/images/gow_hacked2.png)

Fig.16

![](/assets/images/urlscan.png)

Fig.17

<h3><b>The 'How'</b></h3>

<h4><b>What really happened on `Nov 11. 2021` ?</b></h4>

The victim accounts that on this day, it was a day off; Remembrance day. He was working on a programming project. Something leveraging AWS Lambda functions for automation but does not recall visiting the repository. Browser history and Redline Timeline suggests the file was manually downloaded. It is likely the victim's memory is not accurate. Understandably, it's been some time and over the holidays.

At this point, the victim recalls he was in search of some linux tools for Windows environment after briefly complaining to a friend on social media.

The victim had visited this repository on `Nov 11. 2021` during which the Wiki download links were trojanized.

<h3><b>The 'When'</b></h3>

The scope of the attack or how many users may have been compromised is still unknown. The earliest Wiki corruption known is still Oct 2021 but the NetSupport Manager license dates back to Nov 2017.

<h3><b>The 'Why'</b></h3>

The motive of the attacker is unclear but the attacks are likely targeting developers using or working on large C++ Windows projects.

It is unknown what other repositories may have been compromised. GitHub support was contacted with the findings but is yet to respond.

A close examination of the properties of the `Gow` and `GammaRay` such as repository user size, relevance etc may yield other potentially compromised repositories.

It is important to note that GitHub Wiki pages are [locked down](https://docs.github.com/en/communities/documenting-your-project-with-wikis/changing-access-permissions-for-wikis) by default.

<h3><b>The 'Who'</b></h3>

Although still unclear, as the licensed software may have been stolen, the licensee is named `Freddy`. Investigating the compromised wikis yields the following now defunct GitHub User names:

`kirniko`, `arseij`, `anokn`, `arnoyr`, `mleev`, `immki`, `sa3388`, `da4567`, `rinlt`, `nikefi`

Network traffic suggests the control server resides in Russia.

![](/assets/images/network_p1.png)

Fig.16

<h2><b>Remediation - Victim</b></h2>

To conduct remediation, the sample was detonated on `https://any.run` and the [generated report](https://any.run/report/5eaf9684a6f80fcd59aa00228cb5379ec3f026438b3ccdd8d26b25a6f4b53b97/e2ad6b3d-99e7-4bcc-af0e-cc0c8763aa4f) was used to craft a PowerShell script to remove artifacts and restore normal operating conditions.

The sample appears to install Net Support Manager client service with a configuration specifying the remote control server.

Reverse engineering of the dropper malware was not performed. An exercise left for the curious reader!

The remediation script is shown and linked below:

```ps1
## Use at your own risk
## This is a dirty tool
## This tool is not written with high standards
## @glitchnsec 2022

## Remove IoCs
function Remove-IOCs {

  # Terminate the running Net Support Manager
  # stop and delete the associated service
  $service = $(gwmi win32_service -Filter "DisplayName like '%Net%Support%Manager%'")
  if ($service) {
    $service.StopService()
    $service.Delete()
  }

  # Filesystem IOCs are located in "$env:USERPROFILE\AppData\Roaming\WinSup"
  If ($(Test-Path -Path "$env:USERPROFILE\AppData\Roaming\WinSup")) {
    Remove-Item -Path "$env:USERPROFILE\AppData\Roaming\WinSup" -Recurse -Force

  }

  # The autoruns file that ensures persistence
  If ($(Test-Path -Path "$env:USERPROFILE\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\autoruns.ini.lnk")){
    Remove-Item -Path "$env:USERPROFILE\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\autoruns.ini.lnk" -Force
  }

  # Find the file saved by client.exe
  # C:\Users\admin\AppData\Local\Microsoft\Windows\Temporary Internet Files\Content.IE5\PO2HN1X2\loca[1].htm
  # likely not relevant
  $f = $(Get-ChildItem -Recurse -Filter "loca[1].htm" -Path "$env:USERPROFILE\AppData\Microsoft\Windows\Temporary Internet Files\" -ErrorAction SilentlyContinue)

  if ($f) {

    Remove-Item -Path "$f.FullName" -Force
  
  }

  # Registry IoCs
  if ($(Test-Path -Path "HKCU:\SOFTWARE\WinRAR SFX")) {
    # Remove this, the program wrote it
    Remove-Item -Path "HKCU:\SOFTWARE\WinRaR SFX" -Recurse -Force
  }

  
  # Not sure about the other Registry writes. Most look benign or default.
  # They are registry write events so might be sus but we'll see
  # https://any.run/report/5eaf9684a6f80fcd59aa00228cb5379ec3f026438b3ccdd8d26b25a6f4b53b97/e2ad6b3d-99e7-4bcc-af0e-cc0c8763aa4f#registry

}

## Block communication on Firewall
# incase we missed anything
function Update-Firewall {
  New-NetFirewallRule -DisplayName "Block Brassaffid.com C&C" -Direction Outbound -LocalPort Any -Protocol TCP -Action Block -RemoteAddress 185.87.49.233/23
  New-NetFirewallRule -DisplayName "Block Brassaffid.com C&C" -Direction Outbound -LocalPort Any -Protocol TCP -Action Block -RemoteAddress 62.172.138.35/23
  New-NetFirewallRule -DisplayName "Block Brassaffid.com C&C" -Direction Outbound -LocalPort Any -Protocol TCP -Action Block -RemoteAddress 23.108.57.144/23

}

Remove-IOCs
Update-Firewall

```

<h2><b>Remediation - Public GitHub Repository Wiki</b></h2>

Open source GitHub repository owners should verify their Wiki pages have the appropriate read/write permissions set. Review the [GitHub documentation](https://docs.github.com/en/communities/documenting-your-project-with-wikis/changing-access-permissions-for-wikis). Make sure the "**Restrict editing to collaborators only**" is checked 

If your Wiki Home contains links to Windows Executable binaries, and your Wiki home page was not previously hardened, verify that your Wiki history does not contain edits by the following users: `kirniko`, `arseij`, `anokn`, `arnoyr`, `mleev`, `immki`, `sa3388`, `da4567`, `rinlt`, `nikefi`. This list may be non-exhaustive. Additionally, verify that shortened links have  NOT been used to link binaries. Especially links hosted by the URL shortener service: `https:\\linkify.me`. This was the shortner service observed to be used by the attacker.

If your Wiki Home was found to be compromised, announce to and advice your users accordingly.

<h2><b>Threat IoCs</b></h2>

See `https://any.run` [report](https://any.run/report/5eaf9684a6f80fcd59aa00228cb5379ec3f026438b3ccdd8d26b25a6f4b53b97/e2ad6b3d-99e7-4bcc-af0e-cc0c8763aa4f#files) for full details

|FileName|Hash - sha1|Comments |
|---|---|---|
|N/A|B26BC424724A69565AF64F70AACE5890671254E2| FileName not provided. artifact is likely unique to victim |
|`%USERPROFILE%\AppData\Roaming\WinSup\`|N/A| Folder is hidden and read-only|
|`%USERPROFILE%\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\autoruns.ini.lnk`|N/A|Hash not provided, artifact is unique to victim|

<h2><b>ShoutOuts</b></h2>

- Shout out to users who reported the breach to the repository owners for `Gow` and `GammaRay`.
- Shout out to the repository maintainers for `Gow` and `GammaRay` for responding and locking down their wikis
- Shout out to the user(s) that uploaded the sample to Virus Total and VX Intel.
- Shout out to the endpoint detection products that probably protected users from further compromise.