# Android System
System (usually Linux UID 1000) is a special user that bypasses many permission checks in android. This user is highly priviledged and is what runs apps like Settings. This user can access many protected areas of your device. Below is stuff you can do if you get access to System (either by an exploit in Android (like [CVE 31317](https://github.com/oddbyte/CVE-2024-31317)), or by a exploit in a specific app that has runs as the System shared UID (like the [Samsung TTS exploit](https://www.google.com/search?q=samsung+tts+shell)).

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
