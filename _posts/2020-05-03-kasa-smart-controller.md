---
title: "Raspberry pi smart controller for tp-link Kasa devices"
layout: post
date: 2020-05-03
headerImage: false
tag:
- tp-link
- raspberrypi
- eink
blog: true
author: gdf
description: "Raspberry pi smart controller for tp-link Kasa devices"
---

## What to expcet after this post?
{% youtube lXhoo_O_KDQ 600 400 %}
![image](/assets/images/posts/waveshare.jpg)

## Hardware that you needs
- Tp-link Kasa plug (A list of supported device can be found in [here](https://github.com/plasticrake/tplink-smarthome-api#supported-devices))
- [Raspberry pi 4 2GB edition](https://www.raspberrypi.org/products/raspberry-pi-4-model-b/)
- [Waveshare 2.7inch E-Ink display HAT for Raspberry Pi](https://www.waveshare.com/2.7inch-e-paper-hat.htm)
- [Micro SD card for Raspberry pi](https://www.amazon.com/s?k=16+gb+microsd+class+10+card&crid=SPHY8QRZJD8E&sprefix=16+gb+microsd%2Caps%2C236&ref=nb_sb_ss_i_4_13)

## Details steps to reproduce this project

### Step 1: Setup Micro SD card for Raspberry pi.
Please reference the [offical guide](https://projects.raspberrypi.org/en/projects/raspberry-pi-setting-up) from raspberry pi to install Raspbian operating system on the Micro SD card. 

(Optional) You can also setup remote ssh connection to raspberrypi with the [guideline](https://itsfoss.com/ssh-into-raspberry/)

### Step 2: Setup Waveshare 2.7inch E-Ink display HAT
Inside this step, we will install necessary library and driver for Waveshare 2.7inch E-Ink display.
- Mount the Waveshare 2.7inch E-Ink display HAT to the raspberry pi. 
- Open terminal: Enalbe SPI interface 
  - `sudo raspi-config`
  - Choose `Interfacing Options -> SPI -> Yes`  to enable SPI interface
  - `sudo reboot`
- Open terminal: Install necessary library
  - Install BCM2835 libraries
    ```
    wget http://www.airspayce.com/mikem/bcm2835/bcm2835-1.60.tar.gz
    tar zxvf bcm2835-1.60.tar.gz 
    cd bcm2835-1.60/
    sudo ./configure
    sudo make
    sudo make check
    sudo make install
    #For more details, please refer to http://www.airspayce.com/mikem/bcm2835/
    ```
  - Install wiringPi libraries
    ```
    sudo apt-get install wiringpi

    #For Pi 4, you need to update it：
    cd /tmp
    wget https://project-downloads.drogon.net/wiringpi-latest.deb
    sudo dpkg -i wiringpi-latest.deb
    gpio -v
    #You will get 2.52 information if you install it correctly
    ```
  - Install Python libraries
    ```
    #python2
    sudo apt-get update
    sudo apt-get install python-pip
    sudo apt-get install python-pil
    sudo apt-get install python-numpy
    sudo pip install RPi.GPIO
    sudo pip install spidev
    #python3
    sudo apt-get update
    sudo apt-get install python3-pip
    sudo apt-get install python3-pil
    sudo apt-get install python3-numpy
    sudo pip3 install RPi.GPIO
    sudo pip3 install spidev
    ```
More details can be found in reference: https://www.waveshare.com/wiki/2.7inch_e-Paper_HAT

### Step 3: Kasa controller program
Inside this step we will run a kasa controller program (NodeJS) to control the Kasa devices inside your home. 
- Open terminal: Install/Upgrade node and npm. 
  - `curl -sL https://deb.nodesource.com/setup_12.x | sudo bash -`
  - `sudo apt install nodejs`
  - `node --version` to check the version of the node
- Open terminal: Setup `kasaControl.js`
  - `mkdir -p ~/smartcontroller/kasacontroller && cd $_`
  - Create the `kasaControl.js` and `package.json` file under the `kasacontroller` folder. File can be found in the following [gist](gudongfeng/16fb67b6c6a26fdba597166cfd67bca6)
  - `npm install && npm start`
  - Verify `cat /tmp/devicelist.json` has content and it shows a list of kasa devices in your home. 
  - `Control + C` to terminate the program
{% gist gudongfeng/16fb67b6c6a26fdba597166cfd67bca6 %}

### Step 4: Program for Waveshare 2.7inch E-Ink display HAT
Inside this step, we will setup the program for Waveshare 2.7inch E-Ink display HAT to dispaly a list of kasa device in the screen and the button control for the device. 
- Open terminal: Install dependencies
  - `pip install watchdog`
- Open terminal: Run Waveshare E-Ink display control program
  - `mkdir -p ~/smartcontroller/waveshareControl && cd $_`
  - `git clone https://github.com/waveshare/e-Paper .`
  - `cd ~/smartcontroller/waveshareControl/RaspberryPi&JetsonNano/python`
  - Create the `deviceInfo.py` file under the `~/smartcontroller/waveshareControl/RaspberryPi&JetsonNano/python` folder. File can be found in [gist](https://gist.github.com/gudongfeng/f08c4c9907f95b99f0f5a80791609d1b)
  - Make sure the `/tmp/devicelist.json` file is not empty when it has been created in Step 3. 
  - `python deviceInfo.py`
  - The Waveshare display should now show a list of kasa devices in your home. 
{% gist gudongfeng/f08c4c9907f95b99f0f5a80791609d1b %}

### Step 5: Run the program during start time in raspberry pi
- Open terminal: Edit the start up script
  - `sudo vi /etc/rc.local`
  - Append the following text before the `exit 0` line
  ```
  exec 1>/tmp/rc.local.log 2>&1  # send stdout and stderr from rc.local to a log file
  set -x

  sudo -u pi node /home/pi/smartcontroller/kasacontroller/kasaControl.js &
  sleep 5
  sudo -u pi python /home/pi/smartcontroller/waveshareControl/RaspberryPi&JetsonNano/python/deviceInfo.py &
  ```
  - `sudo reboot`

After Setup 5, you should be able to use it to control your kasa devices. 
