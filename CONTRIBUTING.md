Contriubtions are very welcome! Please raise a PR/issue.

For new devices, please include the Mark number in the directory name so they sort in order. You can find the mark number of devices [here](https://wiki.mobileread.com/wiki/Kobo_Firmware_Releases).

The .config settings you need to build the current set of modules are:
```
# UHID
CONFIG_UHID=m

# RAID
CONFIG_MD=y
CONFIG_BLK_DEV_MD=m
CONFIG_MD_LINEAR=m
```
