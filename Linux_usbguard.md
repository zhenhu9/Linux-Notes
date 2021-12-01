# usbguard

This App controls who can control what usb devices. The default root and the members of wheel group can manager usb devices.

The most simpliest way to use it:

```
usbguard list-devices	/* list all devices which are allowed and blocked.
usbguard allow-device ID	/* Allow the blocked device ID in the list.
```

------
