# CHALLENGE 1 - ExifMetadata #

## OBJECTIVE : ##
>HELP!  That wily Jack Frost modified one of our naughty/nice records, and right before Christmas!  Can you help us figure out which one?  We’ve installed `exiftool` for your convenience!

#  

## PROCEDURE : ##

This is a super quick one just ran the following command to filter out any exif entries containing the word “Jack” and to display the preceding 40 lines to get the document’s file name.
```
:~$ exiftool . | grep -B40 Jack
```
