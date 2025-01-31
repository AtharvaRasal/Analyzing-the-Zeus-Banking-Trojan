# Analyzing-the-Zeus-Banking-Trojan

Description 

This project guides you through setting up a secure virtual environment for analyzing malicious software, specifically the notorious Zeus Banking Trojan. By establishing a Windows VM with analysis tools and a REMnux distribution, you'll gain practical experience in static analysis, dynamic analysis, network monitoring, and rule-based detection to understand the malware's behavior and impact. Perfect for cybersecurity professionals, researchers, and enthusiasts interested in hands-on malware analysis.


<h2>Table of Contents</h2>

 - Analyzing the Zeus Banking Trojan
  - Background Information: Zeus Banking Trojan
  - Tools for Analyzing Zeus Banking Trojan
  - Downloading Zeus Banking Trojan
  - Analysis
  - Basic Static Analysis
  - Advanced Static Analysis
  - Dynamic Analysis
  - Writing a Yara Rule
- Conclusion

<h2>Analyzing the Zeus Banking Trojan</h2>

For this part, we will be analyzing the Zeus banking trojan. First, some background information - basically what happened, and we will be overviewing the analysis tools. Then we go and dissect our malware, provision our lab, and conduct our analysis, which includes downloading the Trojan, labeling the malware, and going through basic static and dynamic analysis with reporting and indicators of compromise.

<h3>Background Information: Zeus Banking Trojan</h3>

The Zeus Banking Trojan, also known as Zbot, is a notorious piece of malware designed to steal banking information. First discovered in 2007, Zeus is known for its stealth and persistence, allowing it to remain undetected while capturing sensitive data such as banking credentials and credit card information through techniques like keylogging and form grabbing.

Zeus often operates as part of a botnet, controlled by a central command server, and is capable of performing man-in-the-browser (MitB) attacks, intercepting and manipulating online transactions in real-time.

<b>Infection Vectors</b>

- Phishing Emails: Malicious attachments or links in emails.
- Drive-by Downloads: Automatic downloads from compromised websites.
- Social Engineering: Convincing users to download and execute the malware.

<b>Impact</b>

Zeus has caused significant financial losses globally, affecting both individuals and large organizations. Despite numerous law enforcement efforts to dismantle Zeus botnets, variants of this malware continue to emerge, highlighting its resilience.

<b>Defense Strategies</b>

- Endpoint Protection: Use robust antivirus and anti-malware solutions.
- Network Security: Monitor network traffic for malicious activity.
- User Education: Raise awareness about phishing and safe browsing practices.
- Two-Factor Authentication (2FA): Enhance security for online banking.
- Regular Updates: Keep systems and software up-to-date with security patches.
By understanding the Zeus Trojan's characteristics and infection methods, we can better prepare to defend against and analyze this persistent threat.

Tools for Analyzing Zeus Banking Trojan
1. VirusTotal
   
VirusTotal is an online service that analyzes files and URLs for viruses and other types of malicious content. It aggregates results from multiple antivirus engines to provide a comprehensive analysis.

Example Usage:

import requests

file_path = 'path_to_file'
api_key = 'your_api_key'

with open(file_path, 'rb') as file:
    response = requests.post(
        'https://www.virustotal.com/api/v3/files',
        headers={'x-apikey': api_key},
        files={'file': file}
    )

print(response.json())

2. PeStudio
   
PeStudio is a tool for static analysis of Windows executables. It provides information about the file's hashes, entropy, imports, and potential indicators of malicious behavior.

3. FLOSS
FLOSS (FireEye Labs Obfuscated String Solver) extracts obfuscated strings from malware, which can reveal hidden commands, URLs, and other useful data.

floss path_to_malware_sample

4. Capa
   
Capa is an open-source tool that analyzes the capabilities of malicious programs by identifying behaviors and functionalities within the code.

capa path_to_malware_sample

5. Cutter
   
Cutter is an advanced, open-source GUI for the Rizin reverse engineering framework, used for disassembling and analyzing malware.

6. INetSim
   
INetSim is a tool that simulates various internet services to analyze malware's network behavior in a controlled environment.

7. Wireshark
    
Wireshark is a network protocol analyzer that captures and inspects packets in real-time, useful for analyzing the network traffic generated by malware.

8. Procmon
    
Procmon (Process Monitor) is a Windows tool that provides real-time file system, registry, and process/thread activity monitoring.

9. YARA
    
YARA is a tool used to identify and classify malware by creating rules that look for specific characteristics.

Example YARA Rule:

rule ZeusBankingTrojan
{
    meta:
        description = "Detects Zeus Banking Trojan samples"
    strings:
        $string1 = "Zeus"
        $string2 = { 6A 40 68 00 30 00 00 6A 14 8D 91 }
    condition:
        $string1 or $string2
}

Example Usage:

yara rule_file.yara path_to_malware_sample

<h3>Downloading Zeus Banking Trojan</h3>

Before moving at this point, make sure you have both of your VMs snapshots are taken. Safety is key when dealing with malware.

Now, previously we set our 2 VMs to talk only with each other and not external access. To download the Trojan, we will momentarily connect FlareVM to the internet. To do this, you can set the network adapter settings on VMware to NAT and the connection to the internet will start. Do not forget to remove the DNS server IP (which was our REMnux machine IP) to connect to the internet.

Connect to Internet img

Let's proceed to download the trojan. https://github.com/ytisf/theZoo/tree/master/malware/Binaries/ZeusBankingVersion_26Nov2013

Be very careful with this repository when downloading and detonating these malwares.

Here we have 4 files in this repository. We will click on the .zip file and download it. Use Microsoft Edge browser if the Chrome browser automatically blocks the file.

Download Trojan img

Right after we download the malware, we will change the network adapter setting back to Host-only private network. At this point, we have no internet access; only our 2 VMs are connected to each other.

We put our malware on our desktop, and we take another snapshot of our FlareVM before we detonate the trojan.

Now when we look into the zip file that we downloaded, we see an odd file:

Odd File img

We pull this to our desktop, which then asks us for a password, which is a security measure of the theZoo repository. The password is "infected".

Enter Password img

Now remember, before detonating this malware, we are disconnected from the internet.

While doing various analyses, we will fill out this report that we created, so we are going to iteratively go through this process and add details.

Analysis Report img

<h3>Analysis</h3>

We can start with a very default way to check a suspicious file with VirusTotal. We connect to the internet and go to VirusTotal, where it's very easy to choose the file from our computer and upload it.

Upload to VirusTotal img

As expected, 67/74 security vendors and 5 sandboxes flagged this file as malicious. We can add this page screen to our report to start.

VirusTotal Results img

There is lots of information on VirusTotal, like all the hash values of the file with the History and various other names that have been used to infect with this malware. At this point, we can disconnect our machine from the internet.

Now there are many ways to get information about the file that we are suspicious of. One easy-to-use tool is PeStudio, which is installed with the FlareVM. Just search the name and drag and drop the suspicious file into the program:

PeStudio img

You will have various information about the file, and even the VirusTotal link if you have an internet connection. We can add the hash values to our report, with the filename, and continue.

<h3>Basic Static Analysis</h3>

So with basic static analysis, what we are trying to do is examine the program or code and identify any malicious artifacts without running the actual program. We will be employing a few different tools to gather information. As we are on PeStudio, let's try to gather more information about the malware.

We will look for some interesting strings that may pop out, such as a URL, a domain name, an IP address, or maybe some different types of imports such as DLLs.

As we see on the PeStudio screen, there is an export URL called corect.com, which is misspelled and embedded into this file. Let's put this URL and also the file name into our report.

Next, we go to the sections from the left side menu and compare the raw-size and virtual-size of the file.

Raw vs Virtual Size img

Comparing the raw size and virtual size of malware helps detect packing or obfuscation, which can indicate that the malware expands in memory, revealing additional hidden code or data not immediately visible in the raw file. This discrepancy can signal malicious intent and the presence of hidden functionalities. Here we see there is not a clear discrepancy. Let's add this to our report as well.

Next, we will go to the strings part. This is a very important part. The Strings tab in PeStudio helps identify human-readable strings within the malware, which can reveal important information such as URLs, IP addresses, file paths, registry keys, commands, and other indicators of compromise (IOCs). Analyzing these strings can provide insights into the malware's behavior, potential targets, and communication channels, aiding in understanding and mitigating the threat.

Strings img

<h3>Investigating Strings in PeStudio as a Malware Analyst</h3>

 - Open the Malware Sample: Load the executable in PeStudio and navigate to the Strings tab.
 - Identify Suspicious Indicators: Look for URLs, IP addresses, file paths, registry keys, and commands.
 - Highlight IOCs: Note down any indicators of compromise, like C2 server addresses or persistence mechanisms.
 - Correlate with Malware Behavior: Match strings to known malware behaviors, e.g., "cmd.exe" indicating command execution.
 - Search Known Patterns: Use search engines and threat intelligence databases to find matches with known malware.
 - Analyze Context: Understand the context and functionality related to each string.
   
By investigating these strings, we can uncover valuable insights into the malware's operations and potential impact.

Next, the Libraries tab shows DLLs that the malware imports, revealing its capabilities. Key libraries include:

 - SHLWAPI.dll: Handles string manipulation and file operations.
 - KERNEL32.dll: Manages system-level tasks like memory management and process creation.
 - USER32.dll: Manages user interface elements such as windows and menus.
   
The tab indicates if libraries are duplicated, any flags, and the addresses of imported functions. "Implicit" type means these libraries load automatically at startup. The import count shows how many functions are imported from each DLL. By analyzing these libraries and functions, we can identify the malware's critical operations, cross-reference with known threats, and document suspicious activities.

![Libraries](https://github.com/uruc/Malware-Analysis-Lab/blob/main/

For the next tool, we will use FLOSS. This is a command-line utility. Let's use the more advanced terminal emulator Cmder. As we mentioned earlier, FLOSS (FireEye Labs Obfuscated String Solver) extracts obfuscated strings from malware, which can reveal hidden commands, URLs, and other useful data. Since this binary is not likely packed, it won't have to use any of those tools or tactics.

FLOSS Output

And as we see, FLOSS pulls out all the strings into this text file that we wanted. These strings are the same as we see in PeStudio.

FLOSS Strings

Another useful thing about FLOSS is you can extract the strings based on their length:

floss -n 6 filename

Will extract the strings with 6 or more characters.

Next tool that we will use is Capa. We open a PowerShell and, as we mentioned earlier, just writing capa filename and this is the result:

Capa Output

So based on the result of this malware, detailed capabilities are:

 - Reference Anti-VM Strings Targeting VMWare: Indicates that the malware contains strings specifically used to detect the presence of VMWare, which is a common technique to evade analysis in virtualized environments.
 - Resolve Function by Parsing PE Exports: Demonstrates that the malware can dynamically resolve functions by analyzing the Portable Executable's export table, a method often used to obfuscate its intentions and bypass certain security mechanisms.
 - 
You can also use the verbose modes of Capa with -v and -vv. The -vv option in the Capa command increases the verbosity level of the output, providing more detailed information about the analysis process. This includes additional context about the rules being matched, the features being extracted, and more granular details of the malware's capabilities. This is useful for deeper inspection and understanding of how Capa arrives at its conclusions.

<b>Analysis of Extracted Strings from PeStudio Report</b>

The extracted strings from the malware sample provide insights into its potential behavior and capabilities. Here's a brief analysis of these strings:

<b>SHLWAPI.dll</b> Functions:

 - PathRelativePathToW, PathParseIconLocationW, PathCombineW, PathAddExtensionW: Functions related to file path manipulation, suggesting the malware may alter or construct file paths.
 - ChrCmpIA, ChrCmpIW: String comparison functions, indicating it may be comparing file names or other strings.
 - PathIsPrefixA, PathIsRootW, PathIsUNCServerA, PathIsSameRootA, PathIsRelativeA: Functions to determine the nature of file paths, suggesting checks on file locations.
 - PathRenameExtensionA, PathRemoveArgsA, PathQuoteSpacesA, PathMakeSystemFolderA: Functions for modifying file properties and paths.
 - PathMatchSpecW, StrCmpNIA: Functions for pattern matching and string comparison.
   
<b>KERNEL32.dll</b> Functions:

 - LocalUnlock, FreeLibrary, LocalAlloc, LocalFree: Memory management functions, indicating dynamic memory allocation and deallocation.
 - GetEnvironmentVariableW, GetEnvironmentVariableA: Functions to read environment variables, possibly to gather system information.
 - GetSystemDefaultUILanguage, GetSystemDefaultLCID, GetUserDefaultUILanguage: Functions related to system locale and language settings, potentially for environment-specific behavior.
 - HeapFree, GlobalAddAtomA: Memory and string management functions.
 - GetLogicalDrives, GetDriveTypeA, GetCompressedFileSizeA: Functions to gather information about the system's drives.
 - VirtualQueryEx, IsBadReadPtr: Functions for memory inspection and validation.
 - WriteFile, CreateFileMappingA: Functions for file writing and memory-mapped files, indicating possible file manipulation or data storage.
 - WinExec: Function to execute a program, suggesting the malware can launch other processes.
 - DeleteCriticalSection: Indicates the use of synchronization mechanisms, possibly to manage concurrency in multithreading.
   
<b>USER32.dll</b> Functions:

 - VkKeyScanA, GetAsyncKeyState: Functions for key input, possibly indicating keylogging capabilities.
 - GetClipboardOwner, GetClipboardData, EnumClipboardFormats: Functions to access clipboard data, suggesting the malware might be stealing clipboard content.
 - CallWindowProcW, SetLastErrorEx: Functions to manage window messages and error handling.
 - AllowSetForegroundWindow, UpdateWindow, FlashWindowEx: Functions related to window focus and updates, indicating potential manipulation of window states.
 - ShowCaret, HideCaret: Functions for caret manipulation, which could be related to keylogging or input handling.
 - GetCapture, IsWindowEnabled, IsWindowVisible: Functions to interact with window properties, potentially for spying on window states.
   
The presence of these functions suggests that the malware has capabilities for:

 - File and path manipulation.
 - Memory and process management.
 - Gathering system and environment information.
 - Interacting with and manipulating windows and user input, potentially for spying or keylogging.
 - Executing other processes and managing files.
   
This indicates that the malware is quite versatile and capable of a range of activities that are typically associated with malicious behavior, such as spying, stealing information, and manipulating system resources. We add these information to our report.

<h3>Advanced Static Analysis</h3>

For our advanced static analysis, we will download and use Cutter. Cutter is an advanced, open-source GUI for the Rizin reverse engineering framework, used for disassembling and analyzing malware. You can enable the internet of the FlareVM to install Cutter. What would be better was actually to install everything we need beforehand and then cut the internet off the VM before detonating the malware. https://cutter.re/

Cutter Website img

Download and extract the folder, and start the cutter.exe. And click continue on the start page and select the suspicious file you want to investigate and click open.

Open File in Cutter img

On the next page, you can customize any settings depending on the needs. For now, we will continue with the default settings.

Cutter Settings img

The Overview tab in Cutter displays general information about the binary:

 - File Info: Shows the path, format, size, and other details.
 - Hashes: Provides MD5, SHA1, and SHA256 hashes to uniquely identify the file.
 - Libraries: Lists dynamically linked libraries (DLLs) used by the binary, including SHLWAPI.dll, KERNEL32.dll, and USER32.dll, indicating standard Windows API usage.
 - Analysis Info: Provides details on functions, cross-references, calls, and other elements within the binary.
   
Cutter Overview img

The Graph View in Cutter visualizes the control flow of the executable:

 - Functions and Calls: Displays the flow between different functions and calls within the code.
 - GetTickCount: The presence of the GetTickCount function suggests timing checks, potentially to evade analysis by detecting sandbox environments.
 - AllowSetForegroundWindow: Indicates manipulation of window focus, which could be used for stealing user input or interacting with the user interface in a deceptive manner.
   
![Cutter Graph View](https://github.com/uruc/Malware-Analysis-Lab/blob/

The Decompiler View translates assembly code into a more readable C-like pseudo code:

 - Function Analysis: Shows detailed pseudo code of functions, making it easier to understand the logic and identify malicious behavior.
 - System Calls and Memory Operations: Frequent use of system calls and memory operations points towards potential manipulation of system resources and memory, typical of malware behavior.
   
Cutter Decompiler View img

<h3>Dynamic Analysis</h3>

Alright, to do our dynamic analysis, first, we need to have our REMnux ready. As we went through the configuration earlier, let's go through the configuration again:

On REMnux: Edit /etc/inetsim/inetsim.conf: dns_default_ip          <REMnux IP> service_bind_address    0.0.0.0

Start INetSim: sudo inetsim Start FakeDNS: sudo inetsim --service fakdns or just fakedns

Start INetSim and FakeDNS

On FlareVM: Enter the REMnux IP as the preferred DNS server. Open Chrome on FlareVM and navigate to google.com. You should see the INetSim default HTML page:

INetSim Default Page img

One tool that we will use now is Procmon (Process Monitor). It is a powerful monitoring tool for Windows that provides real-time visibility into file system, registry, and process/thread activities, helping us understand the behavior and impact of the malware on the system. Search for it on the FlareVM and start it.

And now it is time for detonation! We double-click and run this malware:

Run Malware img

And now it is time for detonation! We double-click and run the malware, which presents a User Account Control (UAC) prompt for "Adobe Flash Player." Interestingly, the malware prevents us from clicking "No" on this prompt. Additionally, if we wait on this page, the malware restarts the application and displays the UAC prompt again. This behavior suggests that the malware uses techniques to manipulate UAC prompts and force user interaction, potentially to gain elevated privileges.

We click "Yes" to run the malware, and it immediately starts some form of Adobe Flash installer and then deletes itself from the desktop. Using Procmon, we utilize the Process Tree feature to trace the behavior of the invoice_2318362983713_823931342io.pdf.exe process.

Procmon Process Tree Procmon Process Tree Procmon Process Tree img

The process tree shows the following:

 - InstallFlashPlayer.exe: This is likely a decoy or part of the malware's payload.
 - WerFault.exe: The Windows Error Reporting tool, often exploited by malware to mask its true activity.
 - cmd.exe: The command prompt, suggesting the malware may execute further commands or scripts.
 - conhost.exe: The Console Window Host, often associated with cmd.exe operations.
   
Procmon Filtering img

Using Procmon's filtering feature, we focus on the invoice_2318362983713_823931342io.pdf.exe process name and operations containing RegSetValue. This shows the binary writing values to the registry, particularly under \Run\Google\Update, indicating an attempt to maintain persistence by setting the malware to run at startup.

Registry Persistence Registry Persistence

RegSetValue: This operation indicates that the malware writes specific values to the Windows registry. In this case, it creates a registry entry under HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run\Google Update, ensuring the malware runs automatically upon system startup.

Registry Entry img

For our next step, we will gather network-based indicators of the malware's behavior. Network-based indicators can reveal if the malware communicates with command-and-control (C2) servers or performs any network activity that could be flagged as malicious.

Wireshark Capture img

To collect network-based indicators, we use Wireshark to capture and analyze the network traffic generated by the malware. Since the malware deletes itself after execution, we need to unzip the malware again from the zip file before detonating it.

We run the malware while Wireshark is capturing the traffic, looking for any signs of communication with a C2 server. The captured traffic shows various DNS and HTTP requests, but we did not find any definitive indicators of C2 communication.

<b>File Analysis</b>

Next, we investigate the files created by the invoice_2318362983713_823931342io.pdf.exe malware. We observe that the malware creates several files in the C:\Users\uruc\AppData\Local\Temp directory. Among these files, we find a suspicious DLL file named msimg32.dll.

Suspicious DLL

Suspicious DLL

To further analyze msimg32.dll, we connect the machine to the internet and upload the file to VirusTotal for scanning. VirusTotal aggregates results from multiple antivirus engines to determine if the file is malicious. The scan results indicate that 59 out of 70 security vendors flagged msimg32.dll as malicious, identifying it as various types of malware, including Trojans and backdoors.

<h3>Writing a Yara Rule</h3>

YARA is a powerful tool used to identify and classify malware by creating rules that look for specific patterns and characteristics within files. Below is an example of a YARA rule written to detect the Zeus Banking Trojan.

rule Zeus {
    meta:
        Author="random"
        description="A rule against the Zeus Banking Trojan"
    strings:
        $file_name="invoice_2318362983713_823931342io.pdf.exe"
        // Suspected name of functions and DLL functionalities.
        $function_name_KERNEL32_CreateFileA="CellrotoCrudUntohighCols" ascii
        // PE Magic Byte
        $PE_magic_byte="MZ"
        // Hex String Function name
        $hex_string = {43 61 6D 65 56 61 6C 65 57 61 75 6C 65 72}
    condition:
        $PE_magic_byte at 0 and $file_name
        and $function_name_KERNEL32_CreateFileA
        or $hex_string
}

<h3>Explanation of the YARA Rule</h3>

 1.Rule Name:

 - rule Zeus {: This defines the name of the rule, which is "Zeus" in this case. This helps in identifying the rule and its purpose.
   
 2.Meta Section:

 - The meta section provides metadata about the rule, such as the author and a description.
 - Author = "random": Specifies the author of the rule.
 - description = "A rule against the Zeus Banking Trojan": Describes the purpose of the rule.
   
 3.Strings Section:

 - The strings section defines the patterns to look for within the file.
 - $file_name = "invoice_2318362983713_823931342io.pdf.exe": Looks for the specific file name associated with the malware.
 - $function_name_KERNEL32_CreateFileA = "CellrotoCrudUntohighCols" ascii: This string represents a suspected function name or DLL functionality that the malware might use, defined as an ASCII string.
 - $PE_magic_byte = "MZ": This string checks for the "MZ" header, which is the magic byte for PE (Portable Executable) files, indicating the file is an executable.
 - $hex_string = {43 61 6D 65 56 61 6C 65 72 57 61 75 6C 65 72}: This hexadecimal string represents a specific sequence of bytes within the file, related to a function name.
   
Hex String img

 4.Condition Section:

 - The condition section defines the logic that determines when the rule matches.
 - $PE_magic_byte at 0 and $file_name and $function_name_KERNEL32_CreateFileA or $hex_string: This condition requires the presence of the PE magic byte at the beginning of the file and the specified file name. Additionally, it checks for the function name string and the hexadecimal string, making the rule more specific to the Zeus Banking Trojan.
   
By creating and using YARA rules like this one, analysts can efficiently identify and classify malware based on known patterns and characteristics, aiding in the detection and response to cyber threats.

After writing our YARA rule, we run it against the malware sample to check for matches. The command and output are shown below:

YARA Rule Output img

This output shows that the YARA rule successfully matched the specified patterns within the malware sample, confirming that our rule is correctly identifying characteristics of the Zeus Banking Trojan.

<h3>Conclusion</h3>

In this malware analysis lab, we thoroughly examined a suspicious executable named invoice_2318362983713_823931342io.pdf.exe to understand its behavior and impact. We began by performing static analysis using Cutter to inspect the file's structure, identifying key libraries and functions that suggested malicious activities. We then used dynamic analysis with Procmon to monitor the malware's real-time behavior, revealing its process tree and registry modifications aimed at maintaining persistence.

Network analysis with Wireshark was conducted to capture and scrutinize network traffic, where we searched for signs of communication with command-and-control servers. Although no definitive C2 communication was detected, other suspicious network activities were observed. Further file analysis led us to identify a suspicious DLL (msimg32.dll), which, when uploaded to VirusTotal, was flagged as malicious by multiple antivirus engines.

Finally, we created a YARA rule tailored to detect specific patterns associated with the Zeus Banking Trojan. Running this rule against the malware sample successfully identified its malicious characteristics, confirming the effectiveness of our detection methods.

Overall, this lab showcased a comprehensive approach to malware analysis, integrating static and dynamic analysis, network monitoring, and rule-based detection to uncover the malware's behavior and persistence mechanisms. This thorough examination equips us with the knowledge and tools needed to effectively detect and respond to similar threats in the future.
