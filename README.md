# <img src="https://cdn-icons-png.flaticon.com/128/5088/5088992.png" width="3%" height="3%"> Volatility2 profiles

![](https://img.shields.io/badge/Profiles-8519-seagreen?style=flat-square)

![](https://img.shields.io/badge/Ubuntu%20kernels/amd64-2.6.10%20--%3E%206.5.0-dodgerblue?labelColor=lightsteelblue&style=for-the-badge&logo=ubuntu)  
![](https://img.shields.io/badge/Ubuntu%20kernels/i386-2.6.15%20--%3E%205.4.0-darkcyan?labelColor=lightsteelblue&style=for-the-badge&logo=ubuntu)  
![](https://img.shields.io/badge/Debian%20kernels/amd64-2.6.23%20--%3E%206.4.0-dodgerblue?labelColor=lightsteelblue&style=for-the-badge&logo=debian)  
![](https://img.shields.io/badge/KaliLinux%20kernels/amd64-3.18.0%20--%3E%206.5.0-darkcyan?labelColor=lightsteelblue&style=for-the-badge&logo=kalilinux)  
![](https://img.shields.io/badge/AlmaLinux%20kernels/amd64-4.18.0%20%7C%205.14.0-dodgerblue?labelColor=lightsteelblue&style=for-the-badge&logo=almalinux)  
![](https://img.shields.io/badge/RockyLinux%20kernels/amd64-4.18.0%20%7C%205.14.0-darkcyan?labelColor=lightsteelblue&style=for-the-badge&logo=rockylinux)  


## About the project 

This repository contains Volatility2 profiles, generated against a panel of Linux kernels. It includes most version and subversion that ever existed for a kernel release.

## Related work 

A similar project for Volatility3 symbols is available here : https://github.com/Abyss-W4tcher/volatility3-symbols

Generate your own Volatility3 symbols with : https://github.com/Abyss-W4tcher/volatility-scripts/tree/master/symbols_builder

## Format

| Distribution | Path | Profile | Example |
| ------------ | ---- | ------- | ------- |
| Ubuntu       | Ubuntu/<*architecture*>/<*base-kernel-version*>/<*ABI*>/<*kernel-flavour*>/ | Ubuntu_<*kernel-version*>\_<*package-revision*>\_<*architecture*>.zip | Ubuntu/amd64/3.0.0/19/generic/Ubuntu_3.0.0-19-generic_3.0.0-19.33_amd64.zip |
| Debian       | Debian/<*architecture*>/<*base-kernel-version*>/<*ABI*>/<*kernel-flavour*>/ | Debian_<*kernel-version*>\_<*package-revision*>\_<*architecture*>.zip | Debian/amd64/3.1.0/1/Debian_3.1.0-1-amd64_3.1.1-1_amd64.zip |
| KaliLinux       | KaliLinux/<*architecture*>/<*base-kernel-version*>/<*kernel-flavour*>/ | KaliLinux_<*kernel-version*>\_<*package-revision*>\_<*architecture*>.zip | KaliLinux/amd64/5.2.0/KaliLinux_5.2.0-kali2-amd64_5.2.9-2kali1_amd64.zip |
| AlmaLinux       | AlmaLinux/<*architecture*>/<*base-kernel-version*>/<*kernel-flavour*>/ | AlmaLinux_<*kernel-version*>_<*architecture*>.zip | AlmaLinux/x86_64/4.18.0/AlmaLinux_4.18.0-477.13.1.el8_8_x86_64.zip |
| RockyLinux       | RockyLinux/<*architecture*>/<*base-kernel-version*>/<*kernel-flavour*>/ | RockyLinux_<*kernel-version*>_<*architecture*>.zip | RockyLinux/x86_64/4.18.0/RockyLinux_4.18.0-477.10.1.el8_8_x86_64.zip |

## Usage

Place every profile you plan to use inside your `[volatility2_installation]/volatility/plugins/overlays/linux/` directory. Then, it will appear in the list of arguments usable for the `--profile` parameter.

Be aware that including too much content inside the profiles directory will considerably slow down Volatility2.

If you cannot find the exact match for your memory dump kernel version in this repository, search the closest one available and give it a try.

## FAQ

- *Why can't I locate a profile for a particular subversion of a listed distribution ?*

Due to missing dependencies, some kernels specific versions may not be available here.  

- *Why do recent Linux kernel profiles don't work with Volatility2 ?*

Volatility2 isn't maintained anymore, so changes in recent Linux kernels aren't updated in the master branch. However, patching your Volatility2 installation with the following PR may fix errors with DWARFv5 and undefined symbols :

https://github.com/volatilityfoundation/volatility/pull/854  
https://github.com/volatilityfoundation/volatility/pull/852

- *A profile does not work. Can you help me with this ?*

If you are indeed using the correct profile from this repository, and it doesn't work with your memory dump, check online if someone already ran into the same problem. The opposite case, feel free to open an issue, by describing clearly what you are experiencing.

## Volatility patches 

Due to the usage of a recent "dwarfdump" version against old Linux kernels, some profiles output debug symbols in a format not implemented by Volatility2. For example :

`ValueError: invalid literal for int() with base 10: 'len 0x0002: 0x2300: DW_OP_plus_uconst 0'`

To patch this issue, save the following patch as `dw_format_patch.diff` in your Volatility2 base directory, and run `git apply dw_format_patch.diff` :

```diff
diff --git a/volatility/dwarf.py b/volatility/dwarf.py
index 01164f86..19178c2d 100644
--- a/volatility/dwarf.py
+++ b/volatility/dwarf.py
@@ -269,6 +269,10 @@ class DWARFParser(object):
 
                 if idx != -1:
                     d = d[:idx]
+                if not d.strip().isdigit():
+                    # DW_AT_data_member_location with following format : "len 0x0002: 0x2300: DW_OP_plus_uconst 0"
+                    d = d.split(': ')[-1]
+                    d = d.split(' ')[1]
 
                 off = int(d)
 
```
