# Disable Swap Memory - Yi(Leo) Luan

**Last Updated:** December 1, 2023  
**Location:** Canada

**Save your SSD life!**

Tested with macOS Sonoma Version 14.1.2

Most of the guides online do not work with macOS Sonoma 14.1.1 or later.

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

1. Menu bar: Utilities -> Terminal
```bash
% csrutil disable    
```
2. restart your computer, check SIP
```bash
% csrutil status  
Output: System Integrity Protection status: disabled.
```
## Disable Swap Memory

### Compressor mode in Virtual Memory
1. Check current mode:
```bash
% sysctl vm.compressor_mode
vm.compressor_mode: 4
```

  * *1 -> Compress memory: Disabled； Swap memory: Disabled*

  * *2 -> Compress memory: Enabled； Swap memory: Disabled*

  * *3 -> Compress memory: Disabled； Swap memory: Enabled*

  * *4 -> Compress memory: Enabled； Swap memory: Enabled*

2. Setup boot configuration to disable Swap memory:
```bash
% sudo nvram boot-args="vm_compressor=2"
```
Use```% nvram -p|grep compress``` to check

3. Unload dynamic_pager daemons to prevent it from starting at boot.
```bash
% sudo launchctl unload -w /System/Library/LaunchDaemons/com.apple.dynamic_pager.plist

% sudo rm /private/var/vm/swapfile*
```
4. Restart and check compressor mode:
```bash
% sysctl vm.compressor_mode
vm.compressor_mode: 2
```

## Enable SIP (Recommanded)
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

4. Restart your computer and check Compressor mode
```bash
% sysctl vm.compressor_mode    
vm.compressor_mode: 2
```

***Swap Memory Disabled***

**note: To re-enable Swap memory**

***fully re-enabling SIP will automatically Re-enable Swap Memory for MacOS 14.0+ itself***

In the terminal under macOS Recovery:
```bash
% csrutil enable
```
---

# Re-enable Swap Memory without fully re-enable SIP:

## Step 1: Reset Boot Configuration for Swap Memory

Open the Terminal application on your Mac.
Reset the boot configuration to its default state, which includes enabling swap memory. Execute the following command:

```bash
sudo nvram -d boot-args
```

This command removes the custom boot arguments that were set to disable swap memory.

## Step 2: Reload dynamic_pager Daemon

The dynamic_pager daemon controls the swap file in macOS. To re-enable it, you need to reload the daemon. In Terminal, execute:

```bash
sudo launchctl load -w /System/Library/LaunchDaemons/com.apple.dynamic_pager.plist
```

This command re-enables the daemon, allowing macOS to manage swap files automatically.

## Step 3: Restart Your Mac

After executing these commands, restart your Mac to apply the changes.

## Step 4: Verify Swap Memory Re-activation

After your Mac restarts, open Terminal again and check the compressor mode to ensure that swap memory is re-enabled:

```bash
sysctl vm.compressor_mode
```

The output should indicate that both memory compression and swap memory are enabled (typically vm.compressor_mode: 4).


# Reference
SIP: https://developer.apple.com/documentation/security/disabling_and_enabling_system_integrity_protection

csrutil enable --without nvram: https://developer.apple.com/forums/thread/17452

macOS Recovery: https://support.apple.com/en-ca/guide/mac-help/mchl46d531d6/14.0/mac/14.0

outdated guide: https://windsketch.cc/macbook-disable-swap

vm.compressor_mode: https://apple.stackexchange.com/questions/118839/vm-compressor-mode-vm-compressor-mode-values-for-enabled-compressed-memory-in

launchctl: https://support.apple.com/en-ca/guide/terminal/apdc6c1077b-5d5d-4d35-9c19-60f2397b2369/mac#:~:text=You%20don't%20interact%20with,should%20be%20started%20by%20launchd%20

Manual page of dynmix_pager(MacOSX): https://www.unix.com/man-page/osx/8/dynamic_pager/

**Important:** Please be aware that modifying system settings can have significant consequences. This guide is provided as-is, and the author takes no responsibility for any issues or damage resulting from its use.
