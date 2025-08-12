# Android System
System (usually Linux UID 1000) is a special user that bypasses many permission checks in android. This user is highly privileged and is what runs apps like Settings. This user can access many protected areas of your device. Below is stuff you can do if you get access to System (either by an exploit in Android (like [CVE 31317](https://github.com/oddbyte/CVE-2024-31317)), or by a exploit in a specific app that has runs as the System shared UID (like the [Samsung TTS exploit](https://www.google.com/search?q=samsung+tts+shell)).

If you have access to System and run `id` you should get back something like this:
```
$ id
uid=1000(system) gid=1000(system) groups=1000(system),1065(reserved_disk),3009(readproc) context=u:r:system_app:s0
```

### Aliases to make your life easier:
By default, everything is black/white only which can get pretty annoying quickly. Enable color and extra information in the `ls` command using:
```
alias ls="ls -la --color"
```

## I got access to system, what can I do?
First, you need to understand something: All of the following data is based on ASOP. Things **WILL** be different / not exist. Because this is all "hidden", it really isnt standard and has no reason to be. Therefore all vendors that pull from ASOP will most likely modify properties, not include specific paths, etc.

### Carrier features:
On certian Samsung devices, you can access `/optics/configs/carriers/*`. This allows you to edit carrier features. Use [this decoder](https://github.com/fei-ke/OmcTextDecoder) to decode the .xml files, and read https://xdaforums.com/t/custom-csc-features-in-samsung-using-system-shell-guide-no-root.4631423/ for instructions on how to use this. Read the comments for features. For example, adding
```
    <CscFeature_Setting_SupportRealTimeNetworkSpeed>TRUE</CscFeature_Setting_SupportRealTimeNetworkSpeed>
    <CscFeature_Common_SupportZProjectFunctionInGlobal>TRUE</CscFeature_Common_SupportZProjectFunctionInGlobal>
```
to the features will show you the device network speed in the notification area.
Read more here: https://xdaforums.com/t/csc-features-10-jan13-enable-secret-csc-features.2033894/

### Enable Multi-user on unsupported devices
You can enable Multi-User support on any android device (even ones like Samsung which do not show this feature to users by default) using the following commands:
```
$ setprop persist.sys.max_users 15000 # Sets the max users to 15000 (extremely overkill)
$ setprop persist.sys.show_multiuserui 1 # Tells Android to show the multiuserui (you can access this in the pull-down notification area, usually near the settings button)
```

### Access to `/data/system/*`:
This is a extremely powerful place in your system. It includes **A LOT** of system information and protected databases.

### Direct r/w to `system`, `secure`, and `global` settings.
You can find the db files at `/data/system/users/0`, like `/data/system/users/0/settings_global.xml`.

### Force profile owner
**This is untested**

You can write to `/data/system/users/0`, which means that you can make a file called `/data/system/users/0/profile_owner.xml` with the following content:
```
<?xml version=’1.0’ encoding=’utf-8’ standalone=’yes’ ?>
```
Which (from what I can tell) will make Device Admins profile owners (?????) (untested)

## What system cannot access
System is prevented from r/w/x on `/data/local/tmp`. This means that any binaries you want to run (like the `exp.so` used to throw the shell) must be included as a "library" in a installed app.

## Below is technical information on what it can access:
## Properties:
`bluetooth.*`

`persist.bluetooth.*`

`debug.*`

`net.*`

`sys.*`

`persist.sys.*`

`log.tag.*`

`persist.log.tag`

You can find what selinux context goes to what properties [here](https://android.googlesource.com/platform/system/sepolicy/+/main/private/property_contexts) and what `system_app` can access [here](https://android.googlesource.com/platform/system/sepolicy/+/main/private/system_app.te)

`system_app` can access any system properties that are protected by the following selinux context:
```
Bluetooth properties

adaptive_haptics_prop
arm64_memtag_prop
bluetooth_a2dp_offload_prop
bluetooth_audio_hal_prop
bluetooth_prop
bluetooth_lea_mode_prop
exported_bluetooth_prop

System properties

debug_prop
system_prop
exported_system_prop
exported3_system_prop
gesture_prop
locale_prop
log_tag_prop
logd_prop
net_radio_prop
timezone_prop
drm_forcel3_prop
dynamic_system_prop

USB properties

usb_control_prop
usb_prop

Control properties

ctl_default_prop
ctl_bugreport_prop
```

## Filesystem:

### Read/Write Access

1. `system_app_data_file` (full create/read/write permissions)

`/data/data/[system apps]` (system_app_data_file)

2. `misc_user_data_file` (full create/read/write permissions)

`/data/misc/user` (misc_user_data_file)

3. `anr_data_file` (create/read/write permissions)

`/data/anr` (anr_data_file)

4. `connectivityblob_data_file` (read/write permissions)

`/data/connectivity/blobs` (connectivityblob_data_file)

5. `cgroup` files (write permissions)

`/sys/fs/cgroup/*` (cgroup files)

6. `cgroup_v2` files and directories (write permissions)

`/sys/fs/cgroup/v2/*` (cgroup_v2 files)

### Read-Only Access

1. `apex_data_file` (search permission)

`/apex/*` (apex_data_file - search only (no read))

2. `staging_data_file` (read permissions)

`/data/staging/*` (staging_data_file)

3. `wallpaper_file` (read permissions)

`/data/system/users/*/wallpaper` (wallpaper_file)

4. `icon_file` (read permissions)

`/data/system/users/*/icons` (icon_file)

5. `asec_apk_file` (read permissions)

`/data/app-asec/*/pkg.apk` (asec_apk_file)

6. `vendor_boot_ota_file` (read directory permissions)

`/vendor_boot_ota` (vendor_boot_ota_file)

7. `system_dlkm_file` (limited access - search, getattr, open, read)

`/system_dlkm/etc/NOTICE.xml.gz` (system_dlkm_file)

8. `proc_version` (read permissions)

`/proc/version` (proc_version)

### Special Access

keystore and wifi_key keystores (various operations)

Can connect to system_server UDP sockets
