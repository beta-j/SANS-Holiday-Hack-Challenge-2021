# OBJECTIVE 11 - Customer Complaint Analysis #

## OBJECTIVE : ##
>A human has accessed the Jack Frost Tower network with a non-compliant host. [Which three trolls complained about the human](https://downloads.holidayhackchallenge.com/2021/jackfrosttower-network.zip)? Enter the troll names in alphabetical order separated by spaces. Talk to Tinsel Upatree in the kitchen for hints. 
#  

## HINTS: ##
<details>
  <summary>Hints provided for Objective 11</summary>
  
>-	Different from BPF capture filters, Wireshark's [display filters](https://wiki.wireshark.org/DisplayFilters) can find text with the `contains` keyword - and evil bits with `ip.flags.rb`.
>-	[RFC3514](https://datatracker.ietf.org/doc/html/rfc3514) defines the usage of the "Evil Bit" in IPv4 headers.

</details>

#  

## PROCEDURE : ##

It’s clear that this is going to be a WireShark filtering challenge – so first thing to do is to fire up WireShark and open the provided pcap file.

Looking at the pcap file I can see that complaints are registered through a web form called `/feedback/guest_complaint.php` in plaintext through an HTTP PUT method.  We can filter for these events by using the following simple display filter:
```
http contains "complaint.php" and http contains "POST"
```

The hints make reference to **RFC3514** which sets the reserved bit in an IPv4 header to mark the packet as ‘evil’.  The objective’s question also lets us know that the human in question has accessed the network with a non-compliant host.  So in this case I’m expecting traffic coming from the human to have the reserved bit unset.  I used the following Wireshark display filter to see the complaint made by the human:
```
http contains "complaint.php" and http contains "POST" and  ip.flags.rb==0
```

This filter results in a single HTTP POST from a lady in Room 1024

![image](https://github.com/beta-j/SANS-Holiday-Hack-Challenge-2021/assets/60655500/616aa097-47b9-4a8c-b375-3b4e8801a3f6)

So now I can simply filter for complaints about room 1024 that have the evil bit set to `1`:
```
http contains "complaint.php" and http contains "POST" and http contains "1024" and  ip.flags.rb==1 
```

This leaves me with three complaints made by three trolls named **Yaqh**, **Flud** and **Hagg**.

![image](https://github.com/beta-j/SANS-Holiday-Hack-Challenge-2021/assets/60655500/11ff3206-9c41-46cb-9fbf-062fd7f1a412)


