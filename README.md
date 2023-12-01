# Disable Swap Memory on macOS Sonoma Version 14.1.1

**Save your SSD life!**

*Warning: With insufficient memory, the kernel may potentially terminate processes. This guide is recommended for users with ample memory space based on their usage.*

## Disable SIP (System Integrity Protection)

### Start up your computer in macOS Recovery (Apple Silicon)
(Intel-based Mac: https://support.apple.com/en-ca/guide/mac-help/mchl338cf9a8/14.0/mac/14.0)

1. On your Mac, choose Apple menu > Shut Down.

2. Wait for your Mac to shut down completely.

3. A Mac is completely shut down when the screen is black and any lights (including in the Touch Bar and keyboard) are off.

4. Press and hold the power button on your Mac until the system volume and the Options button appear.

5. Click the Options button, then click Continue.

6. If asked, select a volume to recover, then click Next.

7. Select an administrator account, then click Next.

8. Enter the password for the administrator account, then click Continue.


**Note:** Disabling System Integrity Protection (SIP) is a powerful operation that should be done with caution. It allows for advanced system modifications, but it also makes your system more vulnerable to security threats. Only proceed if you understand the implications and have a specific need for disabling SIP.

### Disable SIP in Terminal under macOS Recovery

1. menu bar -> tool -> Open Terminal
```bash
% csrutil disable    
```
2. restart computer, check SIP
```bash
% csrutil status  
Output: System Integrity Protection status: disabled.
```
## Disable Swap Memory

### Compressor mode in Vertual Memory
1. Check current mode:
```bash
% sysctl -a vm.compressor_mode
```

  * *1 -> Compress memory: Disabled, Swap memory: Disabled*

  * *2 -> Compress memory: Enabled, Swap memory: Disabled*

  * *3 -> Compress memory: Disabled, Swap memory: Enabled*

  * *4 -> Compress memory: Enabled, Swap memory: Enabled*

2. Setup boot configuration to disable Swap memory:
```bash
% sudo nvram boot-args="vm_compressor=2"
```

3. Disable dynamic_pager:

```bash
% sudo launchctl unload -w /System/Library/LaunchDaemons/com.apple.dynamic_pager.plist

% sudo rm /private/var/vm/swapfile*
```
4. Restart and check current mode:
```bash
% sysctl -a vm.compressor_mode
Output: 2
```

## Enable SIP
1. Open Terminal under macOS Recovery

2. Enable SIP without NVRAM Protections	& Boot-arg Restrictions:
```bash
% csrutil enable --without nvram
```

3. Check status:
```bash
% csrutil status
System Integrity Protection status: unknown (Custom Configuration).

Configuration:
	Apple Internal: disabled
	Kext Signing: enabled
	Filesystem Protections: enabled
	Debugging Restrictions: enabled
	DTrace Restrictions: enabled
	NVRAM Protections: disabled
	BaseSystem Verification: enabled
	Boot-arg Restrictions: disabled
	Kernel Integrity Protections: enabled
	Authenticated Root Requirement: enabled

This is an unsupported configuration, likely to break in the future and leave your machine in an unknown state.
```
---

4. Restart computer and check Compreassor mode
```bash
% sysctl vm.compressor_mode    
vm.compressor_mode: 2
```

***Swap Memory Disabled***

**note: To enable Swap memory back**


In terminal under macOS Recovery:
```bash
% csrutil enable
```

**Important:** Please be aware that modifying system settings can have significant consequences. This guide is provided as-is, and the author takes no responsibility for any issues or damage resulting from its use.

