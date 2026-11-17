# Current Device and Development Environment

## iPad state
- Device: iPad 3,1 (J1AP)
- OS: iOS 9.3.5, build 13G36
- Darwin: 15.6.0
- XNU: 3248.61.1
- Jailbreak setup: checkm8-a5
- Normal boot: working
- SSH and root shell: working

The normal iOS installation is accessible over SSH. Research has deliberately avoided ramdisk SSH because the goal is to inspect the running system.

## Network access
- Local address currently used: 10.0.0.142
- SSH username: root
- SSH server offers older host-key algorithms (ssh-rsa, ssh-dss).

Successful Mac command:

~~~sh
ssh -o HostKeyAlgorithms=+ssh-rsa root@10.0.0.142
~~~

The local address may change if the network assigns a different DHCP lease. Do not assume it is permanent.

## Development Mac
- Computer: M1 MacBook Air
- Shell: macOS Terminal
- Working directory: ~/Desktop/iPad3-Research/
- Downloaded artifact: ~/Desktop/iPad3-Research/kernelcache

ChatGPT Work / Computer Use is unavailable in the user's workflow. Provide commands for the user to run manually and label the execution environment.

## iOS version evidence

Command: sw_vers

~~~text
ProductName: iPhone OS
ProductVersion: 9.3.5
BuildVersion: 13G36
~~~

## NVRAM snapshot

Command: nvram -p

~~~text
auto-boot 1
boot-args
backlight-level 1669
~~~

The boot-args value is empty in this snapshot. This is a recorded observation, not permission to change NVRAM.

## CPU sysctl query

Command:

~~~sh
sysctl -a | grep machdep.cpu
~~~

Output:

~~~text
kern.exec: unknown type returned
sysctl: sysctl fmt -1 1024 2: No such file or directory
~~~

This query did not provide the intended CPU details. It has not been resolved. Do not interpret these messages as proof of a hardware fault; they may reflect unavailable or incompatible sysctl formatting/interface behavior on this older OS.

## Filesystem observations

Kernelcache directory:

Command: ls -lah /System/Library/Caches/com.apple.kernelcaches/

~~~text
total 11M
-rw-r--r-- 1 root wheel 11M kernelcache
~~~

dyld shared cache directory:

Command: ls -lah /System/Library/Caches/com.apple.dyld/

~~~text
total 512M
-rw-r--r-- 1 root wheel 512M dyld_shared_cache_armv7
~~~

The dyld cache filename indicates an ARMv7 shared cache. No extraction or analysis of that cache has yet been documented.

## CoreServices observations

The directory /System/Library/CoreServices/ contains standard iOS 9 components, including:
- SpringBoard.app
- SystemVersion.plist
- DeviceOMatic.app
- AssistiveTouch.app
- MobileStorageMounter.app
- VoiceOverTouch.app

No claim is made that these components are compatible with iOS 10.
