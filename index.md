## Why a Bitcoin Full Node?
Bitcoin is a digital currency supported by a peer-to-peer network. In order to run efficiently and effectively, it needs peers run by different people....and the more the better.

This tutorial will describe how to create a Bitcoin "full node" (a Bitcoin server that contains the full blockchain and propagates transactions throughout the Bitcoin network via peers). This system *will not* mine for Bitcoins...it *will* play its part to keep the Bitcoin peer-to-peer network healthy and strong. For a detailed explanation for why it is important to have a healthy Bitcoin peer-to-peer network, read this article [about Bitcoin full nodes](https://medium.com/@lopp/bitcoin-nodes-how-many-is-enough-9b8e8f6fd2cf).

Also please note this will be a "headless" server...meaning we will not be using a GUI to configure Bitcoin or check to see how things are running. In fact, once the server is set up, you will only interact with it using command line calls over [SSH](http://en.wikipedia.org/wiki/Secure_Shell). The idea is to have this full node be simple, low-power, and something that "just runs" in your basement, closet, etc.

## Why a Raspberry Pi?

Raspberry Pi is an inexpensive computing hardware platform that generates little heat, draws little power, and can run silently 24 hours a day without having to think about it. 

## Background

I decided to create my own Bitcoin full node on a Raspberry Pi. My Raspberry Pi full node is up and running, performing well, has about 75 peers and is relaying transactions to the Bitcoin network. I have to say, ever since I got it set up it has been low maintenance. 

I am going to assume that if you are reading this to create your own Raspberry Pi bitcoin full node, then you already know a little bit about linux, electronics, or running command line tools like SSH. 

## Parts List  (total cost ~$80)

* [Raspberry Pi 2 - Model B](https://www.adafruit.com/products/2358) (~$40)
* [Raspberry Pi Case](https://www.adafruit.com/products/2285) (~$5)
* [MicroSD card with 64 GB of storage](http://amzn.com/B00R7CSHWW) (~$25)
* [Mirco USB charger that you can dedicate to the Raspberry Pi](https://www.adafruit.com/products/1995) (~$8)

* Ethernet CAT45 cable (assuming you have one around somewhere)
* (Temporarily you will need a keyboard and monitor that supports HDMI to make things easier. You don't need this technically, but it sure makes the initial config process much easier.)

## Steps to Follow

1. **Prepare the MicroSD Card:** The blockchain is growing quickly (30+ GBs at the time of this writing), so I felt 64 GB was a good size for the Raspberry Pi's storage. I suppose you could go for 128 GBs to be even more future-proof, if you are willing to spend the money. The MicroSD card will likely come formatted as exfat, instead of FAT32, but the Raspberry Pi needs FAT32. I recommend using the built-in tools on Windows or Mac OSX to format the MicroSD. If you only have a Linux box to start, you probably already know how to format the microSD card. Since I run a Mac, I just used the built-in Disk Utility and formatted as "MS-DOS (FAT)," which is really FAT32. 

2. **Install the operating system:** Installing software on a Raspberry Pi can be mildly complicated. I suggest using their [NOOBS](https://www.raspberrypi.org/documentation/installation/noobs.md) install manager to make it painless. Just follow the link for [NOOBS](https://www.raspberrypi.org/documentation/installation/noobs.md), download the files and copy them to your FAT32 microSD card and get ready to turning things on.

3. **Initial configuration:** There are ways to avoid using a keyboard, video display, and mouse (KVM) altogether. But in the interest of keeping things simple I recommend putting your Raspberry Pi into its case, **then insert the microSD card** into your Raspberry Pi (trust me, it is easier to put the case on first), hook up the KVM cables, plugin the ethernet cable, and plugin the power. 

    At boot up, select Raspbian as your operating system and let NOOBS get the OS set up. Do not bother with any setting that will launch "startx" (the GUI interface) at boot time, since this full node will only be configured via command line.

4. **Update Raspberry Pi:** 

  Add "en_US.UTF-8 UTF-8" to the locale list and set your time zone and then reboot: 
  ```
  sudo raspi-config
  sudo reboot
  ```
  Keep everything fresh:
  ```
  sudo apt-get update
  sudo apt-get upgrade
  ```
  Install dependencies (broken up here to show more cleanly - you can do one apt-get call if you want):
  ```
  sudo apt-get install build-essential autoconf libssl-dev libboost-dev 
  sudo apt-get install libboost-chrono-dev libboost-filesystem-dev
  sudo apt-get install libboost-program-options-dev libboost-system-dev 
  sudo apt-get install libboost-test-dev libboost-thread-dev libtool
  ```
  Prepare for and download bitcoin source code:
  ```
  mkdir ~/bin
  cd ~/bin
  git clone -b v0.11.2 https://github.com/bitcoin/bitcoin.git
  cd bitcoin/
  ```
  Configure and compile the source code; install to the bin directory (this will take 30+ minutes):
  ```
  ./autogen.sh
  ./configure CPPFLAGS="-I/usr/local/BerkeleyDB.4.8/include -O2" LDFLAGS="-L/usr/local/BerkeleyDB.4.8/lib" --disable-wallet
  make
  sudo make install
  ```
5. **Verify your Bitcoin full node is working:** Assuming all went well (above) you can just type the following in your command line:

    ```
    bitcoind &
    ```
    The & at the end tells the app to run in the background so that your command line interface can be used for more commands. If you type the following you will see the status of your Bitcoin full node:
    ```
    bitcoin-cli getinfo
    ```
    It will likely tell you that it is loading the blockchain or tell you how many blocks have been loaded thus far.  If you made it this far, *congratulations!* Unfortunately, you are far from over.

    For now, just stop the Bitcoin server you just worked so hard to start with the following command:

    ```
    bitcoin-cli stop
    ````

6. **Side-load the blockchain:** In my experience, the Raspberry Pi 2 with its 1 GB of RAM and quad processors was not able to synchronize the blockchain on its own. Once it gets to about block 300,000 it starts to run out of RAM and all of the coaxing in the world does not help. (Note: one reader of this guide explained that you can control the RAM usage by starting the Bitcoin server with the following command 'bitcoind --dbcache=50 &'. If you want to give it a try, start up the Bitcoin service using that switch and see if that will allow you to synchronize the blockchain without running out of RAM.) Again, just my experience and newer versions of Bitcoin may resolve this issue. 

    Solution? Synchronize the blockchain on your primary machine and then simply copy your personal seed of the Bitcoin blockchain to your Raspberry Pi full node. The files you need are in the 'blocks' folder and the 'chainstate' folder. You can use the following command using SCP (basically SSH for copying files) to move the files from your main computer to your Raspberry Pi full node (you can also use WinSCP on a Windows machine or Cyberduck on a Mac - but I prefer the command line):

    ```
    scp -r blocks your_username@raspberrypiIPaddress:/home/pi/.bitcoin
    scp -r chainstate your_username@raspberrypiIPaddress:/home/pi/.bitcoin
    ```

    Now you should be able to start up your bitcoin server again and start relaying Bitcoin transactions in realtime. Just execute the Bitcoin server command:

    ```
    bitcoind &
    ````

7. **Make sure port forwarding is turned on in your router:** One more, quite important thing...you need to enable port forwarding on your router to point to *port 8333* to your internal Bitcoin full node IP address. I'm not sure if you need both TCP and UDP forwarded, but I did both and everything is working great. How do you do this? Each router is different and each cable/fiber/DSL provider has instructions somewhere. Your router, in fact, might automatically do it for you, since gaming machines like the XBOX and Playstation benefit from port forwarding and ISPs don't want to deal with explaining how to set it up, so they auto-detect services you are running that need port forwarding and make it happen. 

    Why do you need port forwarding? Basically it allows other Bitcoin peers to automatically connect to *you* without the need for you to invite them first. Without port forwarding you will have far fewer peers and not allow the Bitcoin network to be healthy. So much so, that you cannot really claim you are running a full node without port forwarding (or wide open IP access) enabled.

## **You're Done!**

    You should be all set! Remember when you want to check in on things just open and SSH connection and run:

    ```
    bitcoin-cli getinfo
    ```

## Like this information and want to tip me in Bitcoin? -thanks!

![Bitcoin Tip Address](https://s3.amazonaws.com/dcarns/Public/bitcoinaddress.png)

[19qNSoJdTfRLNoXJUBndgcd9uusr1BroY](bitcoin://19qNSoJdTfRLNoXJUBndgcd9uusr1BroY)