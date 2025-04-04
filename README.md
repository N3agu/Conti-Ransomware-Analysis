# Conti Ransomware Analysis
An in-depth analysis of the leaked Conti Ransomware source code and its Ransomware-as-a-Service (RaaS) business model.

![RaaS](https://www.microsoft.com/en-us/security/blog/wp-content/uploads/2022/05/ransomware-as-a-service-affiliate-model-social-4.png)

# Table of contents
- [Introduction](#introduction)
  - [What is Ransomware-as-a-Service (RaaS)](#what-is-ransomware-as-a-service-raas)
  - [Overview of Conti Ransomware](#overview-of-conti-ransomware)
  - [Most Famous Attacks](#most-famous-attacks)
  - [Evolution](#evolution)
  - [The Leak](#the-leak)
- [Code Analysis](#code-analysis)
  - [Version Timeline](#version-timeline)
  - [File Architecture Overview](#file-architecture-overview)
  - [Analysis of main.cpp](#analysis-of-maincpp)
- [Credits](#credits)
- [Disclaimer](#disclaimer)

# Introduction
### What is Ransomware-as-a-Service (RaaS)?
Ransomware-as-a-Service (RaaS) is a cybercrime business model where ransomware developers lease their malware to affiliates, who then execute attacks on target systems. This model lowers the barrier to entry for cybercriminals lacking technical expertise, as they can utilize sophisticated ransomware tools developed by others. Affiliates typically share a portion of the ransom payments with the developers, creating a profit-sharing scheme that has contributed to the proliferation of ransomware attacks.

### Overview of Conti Ransomware
Conti (often considered as the successor to Ryuk ransomware because it's derived from the same codebase and relies on the same TrickBot infrastructure) is a ransomware strain that first appeared in December 2019, attributed to the Russia-based hacking group "Wizard Spider" (developed and maintained by the so-called TrickBot gang). It operates as a RaaS platform, allowing various threat actors to conduct ransomware attacks using its infrastructure. Once deployed, Conti not only encrypts data on infected devices but also spreads laterally across networks, enhancing its impact.

### Most Famous Attacks
1. Irish Health Service Executive (HSE): In May 2021, the Conti group attacked Ireland’s national healthcare system, causing widespread disruption and forcing the HSE to shut down many of its IT systems. The attackers demanded a ransom of $20 million in exchange for the decryption key.
2. AXA Group: In June 2021, the French insurance company AXA Group suffered a ransomware attack by the Conti group, which resulted in the theft of sensitive customer data. The group demanded a ransom of $300 million in exchange for the data and threatened to release the data if the ransom was not paid.
3. University of Utah: In July 2020, the Conti group targeted the University of Utah and demanded a ransom of $457,059 in exchange for the decryption key. The attack disrupted the university’s computer systems, including email and online coursework, causing significant disruption to students and faculty.
4. Brown-Forman: In June 2020, the Conti group targeted the American alcohol company Brown-Forman, the maker of Jack Daniel’s whiskey. The attackers demanded a ransom of $1.5 million in exchange for the decryption key.
5. City of Tulsa: In May 2021, the Conti group targeted the City of Tulsa, Oklahoma, and demanded a ransom of $7.5 million in exchange for the decryption key. The attack disrupted the city’s computer systems, including email and online services, causing significant disruption to city operations.

<p align="center">
  <img src="https://www.tenable.com/sites/default/files/images/blog/abb32c57-3cce-4228-b18e-ed44ec0cacbb.png">
  <br>Source: https://zcybersecurity.com/conti-ransomware/
</p>

### Evolution
* **Obfuscation improvements**: Since the early ‘test samples’ (late 2019), Conti started implementing a simple XOR mechanism to hide the API names resolved at runtime. From June 2020, a custom encoding function for string obfuscation was also employed.
* **Focus on speed**: Conti developers focus mainly on speed; Conti uses up to 32 concurrent CPU threads for file encryption operations and, starting from the iteration of September 2020, they switched the encryption algorithm from AES to CHACHA in order to further speed-up the encryption process. Efficiency seems to be a functional requirement in the Conti development process. This translates into less time required to lock victim’s data and reduce the chance of the operation being blocked.
* **Optimization of file encryption**: from 2nd September 2020, a new logic for file encryption was added. The logic implements two different modes: full and partial. depending on file extension and file dimension. Moreover, encryption through IoCompletionPorts was replaced by C++ queues and locks in January 2021.

Source: https://assets.sentinelone.com/ransomware-enterprise/conti-ransomware-unpacked

### The Leak
In February 2022, during the onset of the Russian invasion of Ukraine, the Conti ransomware group publicly declared its support for Russia, threatening to retaliate against any cyberattacks on Russian infrastructure. In response, an anonymous individual, believed to support Ukraine, leaked a substantial collection of the group’s internal data. This included approximately 60,000 chat messages, as well as source code and other files used by the group. The leaked messages span from early 2020 through February 27, 2022. Most of the communications were direct messages sent via Jabber, and Conti's operations were coordinated using Rocket.Chat. The leak is fragmented, but it provides deep insight into the group's inner workings.

Source: https://en.wikipedia.org/wiki/Conti_(ransomware)

# Code Analysis
### Version Timeline
A summarized list of all the major Conti milestones was released by SentinelOne:

| Variant       | Date           | Features / Updates |
|---------------|----------------|--------------------|
| A1a, A1b, A1c, A2 | 2019/10/06 | - Limited imports at load-time<br>- API names XOR-encoded with 0x99 |
| B1, B2        | 2019/11/29     | - API names now XOR-encoded with 0x0F<br>- All imports loaded at runtime<br>- 32 threads for file encryption<br>- Parallelized drive encryption<br>- Improved stability during file writing |
| C1, C2        | 2020/06/04     | - Custom string obfuscation replaces simple XOR<br>- Mutex created to prevent multiple infections<br>- Command-line options for encryption mode and network locations<br>- 36 fewer services are stopped |
| D1a, D1b, D1c | 2020/09/02     | - API hashing with Murmur2A (seed 0x5B2D)<br>- Imports cached in a single array<br>- Logging and directory-targeting options added<br>- Ransom note moved from `.rsrs` to `.data`<br>- Shadow copies deleted via `wmic`<br>- Threads based on CPU count and execution mode<br>- New file encryption logic with full/partial modes<br>- Only one thread used to search shares<br>- Port checks before share querying |
| E1–E6         | 2020/10–12     | - Ransom note expanded with contact info & UUID<br>- Logging errors now supported via CLI<br>- Added single-directory encryption mode<br>- Dead code and busy loops to hinder analysis |
| F1, F2        | 2021/01–02     | - Optional mutex creation via CLI<br>- Switched from IoCompletionPorts to C++ queues/locks<br>- Murmur seed updated to 0xB801FCDA |
| G1, G2        | 2021/03–04     | - Uses `PathIsDirectoryW` to distinguish files from folders<br>- Murmur seed changed to 0xFF889912 |

Source: https://assets.sentinelone.com/ransomware-enterprise/conti-ransomware-unpacked

The builder, locker and decryptor variants I will analyze are [available here](https://github.com/gharty03/Conti-Ransomware).

### File Architecture Overview
#### Locker
- `locker/`: Directory contains the core components responsible for the encryption process. Notable files are:
  - `chacha20/`: Directory containing files related to the ChaCha20 encryption algorithm (Contains `chacha.c`, `chacha.h`, `ecrypt-config.h`, `ecrypt-machine.h`, `ecrypt-portable.h` and `ecrypt-sync.h`)
  - `antihook/`: Contains `antihooks.h` (which defines removeHooks(), seems like antihooks.c/cpp is missing)
  - `network_scanner.cpp`: Implements functionality to scan the network for shared folders and accessible network locations to propagate the ransomware
  - `threadpool.cpp`: Implements a thread pool to manage concurrent tasks efficiently during the encryption process
  - `disks.cpp`: Handles disk operations, including enumeration of disk drives and partitions
  - `memory.cpp`: Manage memory allocation and deallocation routines during encryption
  - `search.cpp`: Implements search functions to locate target files for encryption (also contains the list of blacklisted extensions: .exe, .dll, .lnk, .sys, .msi, R3ADM3.txt and CONTI_LOG.txt)
  - `main.cpp`: The entry point of the program
#### Decryptor
Both the decryptor and the locker contain their own implementation of ChaCha20, suggesting that encryption and decryption processes are handled separately.

### Analysis of main.cpp
This file is the entry point and controller of the ransomware execution. It parses command-line arguments, initializes critical components, configures encryption settings, and manages the local and network file search and encryption process.

#### 1. Header Inclusions
```cpp
#include "common.h"
#include "filesystem.h"
#include "network_scanner.h"
#include "threadpool.h"
#include "global_parameters.h"
#include "locker.h"
#include "api.h"
#include "logs.h"
#include "process_killer.h"
```
These headers provide:
- API hooking/disabling
- Filesystem enumeration
-Encryption
- Network share scanning
- Logging
- Multithreading/thread pool management
- Process whitelisting/killing

#### 2. Global Variables
```c
STATIC STRING_LIST g_HostList;
STATIC STRING_LIST g_PathList;
STATIC process_killer::PID_LIST g_WhitelistPids;
STATIC INT g_EncryptMode = ALL_ENCRYPT;
```
These store:
- A list of hostnames to scan for network encryption
- A list of paths to encrypt
- A whitelist of process IDs to avoid during encryption
- The current encryption mode: all, local, net, or backups

#### 3. Command Parsing
Key flags:
- `-h <hostfile>`: List of remote hosts to scan
- `-p <pathfile>`: File paths to encrypt
- `-m <mode>`: all, local, net, backups
- `-log enabled`: Enables event logging

Commented out in this version: `-prockiller` and `-pids`

The ransomware reads config from command-line args or files.

#### 4. Config Parsing Functions
```cpp
ParseFile(LPCWSTR FilePath, PSTRING_LIST List)
```
This function reads newline-delimited strings (paths/hostnames) from a file and stores them in a TAILQ list. Used for:
- Target folders (-p)
- Remote hosts (-h)

<details open><summary>Custom std::stoi() implementation called my_stoi() for safely converting strings to integers.</summary>

```cpp
STATIC int my_stoi(char* str) {
	unsigned int strLen = 0;
	unsigned int i = 0;
	while (str[i] != '\0') {
		strLen += 1;
		i++;
	}

	int num = 0;
	int ten;
	BOOL signFlag = TRUE; //true: +, false: -
	for (i = 0; i < strLen; i++) {
		if (str[i] < '0' || str[i] > '9') {
			if (i == 0 && str[i] == '-') {
				signFlag = FALSE;
				continue;
			}
			if (i == 0 && str[i] == '+') {
				signFlag = TRUE;
				continue;
			}

			return 0;
		}

		ten = 1;
		for (unsigned int j = 0; j < strLen - 1 - i; j++) {
			ten *= 10;
		}

		num += ten * (str[i] - '0');
	}

	if (signFlag) {
		return num;
	}
	else {
		return -num;
	}
}
```
</details>

#### 5. API Unhooking
```cpp
api::InitializeApiModule();
api::DisableHooks();
```
#### 6. Thread Pools
```cpp
DWORD dwLocalThreads = g_EncryptMode == LOCAL_ENCRYPT ? SysInfo.dwNumberOfProcessors * 2 : SysInfo.dwNumberOfProcessors;
DWORD dwNetworkThreads = g_EncryptMode == NETWORK_ENCRYPT ? SysInfo.dwNumberOfProcessors * 2 : SysInfo.dwNumberOfProcessors;
```
The ransomware creates separate threadpools for local file encryption and network file encryption, which leads to faster encryption.

#### 7. Mutex
```cpp
HANDLE hMutex = pCreateMutexA(NULL, TRUE, OBFA("kjsidugidf99439"));
if ((DWORD)pWaitForSingleObject(hMutex, 0) != WAIT_OBJECT_0) {
	return EXIT_FAILURE;
}
```
Creats a mutex with a hardcoded string "kjsidugidf99439" (which gets obfuscated at compile-time) to prevent multiple instances of the ransomware running at the same time. If the mutex exists, the program exits.

#### 8. Shadow Copies
Deletes Windows Volume Shadow Copies to prevent restoration of encrypted files.

#### 9. Process Whitelist
```cpp
process_killer::GetWhiteListProcess(&g_WhitelistPids);
locker::SetWhiteListProcess(&g_WhitelistPids);
```
The ransomware identifies certain processes and saves their PID.

#### 10. Scanning
Conti starts scanning for files on the system based on g_EncryptMode.
- File System (enumarates all drvies and starts looking for new files to encrypt)
```cpp
if (filesystem::EnumirateDrives(&DriveList)) {
	hLocalSearch = pCreateThread(NULL, 0, filesystem::StartLocalSearch, &DriveList, 0, NULL);
}
```
- Network (Enumerates remote shares on provided hostnames)
```
PSTRING String = NULL;
TAILQ_FOREACH(String, &g_HostList, Entries) {
	network_scanner::EnumShares(String->wszString, &ShareList);
}

network_scanner::PSHARE_INFO ShareInfo = NULL;
TAILQ_FOREACH(ShareInfo, &ShareList, Entries) {
	filesystem::SearchFiles(ShareInfo->wszSharePath, threadpool::NETWORK_THREADPOOL);
}

network_scanner::StartScan();
```
- Path-Based
```cpp
PSTRING Path = NULL;
TAILQ_FOREACH(Path, &g_PathList, Entries) {
	filesystem::SearchFiles(Path->wszString, threadpool::LOCAL_THREADPOOL);
}
```
and
```cpp
PSTRING Path = NULL;
TAILQ_FOREACH(Path, &g_PathList, Entries) {
	filesystem::SearchFiles(Path->wszString, threadpool::NETWORK_THREADPOOL);
}
```
Encrypts specific paths listed in g_PathList.

#### 11. Encryption
Uses ChaCha20 for file encryption.

Encryption respects the selected mode:
- local: local drives only
- net: network shares only
- all: both local and network
- backups: targets shadow copies and backup files

# Credits
- [vxUnderground](https://vx-underground.org/) for [samples, chat logs and much more](https://vx-underground.org/Samples)
- [SentinelOne](https://www.sentinelone.com/) for [CONTI UNPACKED: UNDERSTANDING RANSOMWARE DEVELOPMENT AS A RESPONSE TO DETECTION – A DETAILED TECHNICAL ANALYSIS](https://assets.sentinelone.com/ransomware-enterprise/conti-ransomware-unpacked)
- [gharty03](https://github.com/gharty03) for [Conti-Ransomware](https://github.com/gharty03/Conti-Ransomware)
- [ForbiddenProgrammer](https://github.com/ForbiddenProgrammer) for [conti-pentester-guide-leak](https://github.com/ForbiddenProgrammer/conti-pentester-guide-leak)

# Disclaimer
This project is intended solely for educational and research purposes. It provides an analysis of the leaked Conti Ransomware source code and its associated Ransomware-as-a-Service (RaaS) model.

Do not use any part of this code or information to target systems without explicit authorization. Unauthorized use of this software or its derivatives may be illegal and is strictly prohibited.

The author does not accept any responsibility or liability for misuse, damages, or legal consequences arising from the use of this material.
