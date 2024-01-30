# OBJECTIVE 3 - Thaw Frost Tower's Entrance #

## OBJECTIVE : ##
>Turn up the heat to defrost the entrance to Frost Tower.  Click on the Items tab in your badge to find a link to the Wifi Dongle’s CLI interface. Talk to Greasy Gopherguts outside the tower for tips.#  

## HINTS: ##
<details>
  <summary>Hints provided for Objective 3</summary>
  
>-	The [iwlist](https://linux.die.net/man/8/iwlist) and [iwconfig](https://linux.die.net/man/8/iwconfig) utilities are key for managing Wi-Fi from the Linux command line.
>-	[cURL](https://linux.die.net/man/1/curl) makes HTTP requests from a terminal - in Mac, Linux, and modern Windows!
>-	When sending a [POST request with data](https://www.educative.io/edpresso/how-to-perform-a-post-request-using-curl), add `--data-binary` to your curl command followed by the data you want to send.

</details>

#  

## PROCEDURE : ##

There’s an open window conveniently located close to Greasy Gopherguts – I’m thinking I can probably do a Wi-Fi scan from somewhere near this window...
```console
elf@4979e808b4f5:~$ iwconfig
wlan0     IEEE 802.11  ESSID:off/any  
          Mode:Managed  Access Point: Not-Associated   Tx-Power=22 dBm   
          Retry:off   RTS thr:off   Fragment thr=7 B   
          Power Management:on
```

Running `iwconfig` confirms that I have a wifi interface named `wlan0`, so I can now scan for Wi-Fi networks on this interface:
```console
elf@4979e808b4f5:~$ iwlist wlan0 scan    
wlan0     No scan results
```

No luck.... oh wait there’s another open window and I think I can spot a round access point stuck to the wall too!  OK, so let’s give this another go:

![image](https://github.com/beta-j/SANS-Holiday-Hack-Challenge-2021/assets/60655500/f0096453-14bb-47de-833a-d510af7d5fb1)

```console
elf@b72ff8033ef5:~$ iwlist wlan0 scan
wlan0     Scan completed :
          Cell 01 - Address: 02:4A:46:68:69:21
                    Frequency:5.2 GHz (Channel 40)
                    Quality=48/70  Signal level=-62 dBm  
                    Encryption key:off
                    Bit Rates:400 Mb/s
                    ESSID:"FROST-Nidus-Setup"
```
That did the trick – we now have an ESSID: **`Frost-Nidus-Setup`** which we can try to connect to using `iwconfig`.

Connecting to the (thankfully unsecured) Wi-Fi network we get a helpful MOTD:
```console
elf@b72ff8033ef5:~$ iwconfig wlan0 essid FROST-Nidus-Setup
** New network connection to Nidus Thermostat detected! Visit http://nidus-setup:8080/ to complete setup
(The setup is compatible with the 'curl' utility)
```

So I followed it’s advice:
```console
elf@b72ff8033ef5:~$ curl http://nidus-setup:8080/
◈──────────────────────────────────────────────────────────────────────────────◈

Nidus Thermostat Setup

◈──────────────────────────────────────────────────────────────────────────────◈

WARNING Your Nidus Thermostat is not currently configured! Access to this
device is restricted until you register your thermostat » /register. Once you
have completed registration, the device will be fully activated.

In the meantime, Due to North Pole Health and Safety regulations
42 N.P.H.S 2600(h)(0) - frostbite protection, you may adjust the temperature.

API

The API for your Nidus Thermostat is located at http://nidus-setup:8080/apidoc
```

Trying to register the thermostat seems useless at this point as it requires us to know the thermostat’s serial no, but we know that we should still be able to change the temperature without registering.

If we `curl` to `http://nidus-setup:8080/apidoc` we can see that we are indeed allowed to change the cooler settings without registering and we are conveniently told that we can do this by using a `POST` request with a JSON payload.

Following the command structure suggested by the API itself, we can raise the temperature to a balmy 20 degrees:
```console
elf@290c75e4b151:~$ curl -XPOST -H 'Content-Type: application/json'   --data-binary '{"temperature": 20}'   http://nidus-setup:8080/api/cooler
{
  "temperature": 19.77,
  "humidity": 66.63,
  "wind": 26.79,
  "windchill": 19.43,
  "WARNING": "ICE MELT DETECTED!"
}
```

This successfully thawed and unlocked the Frost Tower’s entrance 😊

 
