---
layout:     post
title:      "Tile's Anti-Theft Mode: Separating Hype from Reality"
date:       2023-03-11
author:     "Tech"
header-img: "img/in-post/post-js-version/Tile/bg_x.png"
tags:
    - Bluetooth
---

#  Tile's Anti-Theft Mode: Separating Hype from Reality

Tile is a technology company specializing in Bluetooth Low Energy (BLE or Bluetooth LE) enabled trackers. With the help of these trackers, users can locate their belongings quickly via a smartphone app. [Tile](https://tile.com) offers a wide range of trackers to suit different needs, such as Tile Pro, Tile Mate, Tile Slim, and Tile Sticker. These trackers come in various sizes and shapes and can be attached to objects such as keys, wallets, and even laptops.

<img src="/img/in-post/post-js-version/Tile/Tile-Trackers.jpeg" width="400" style="border: 1px solid black;">

# Tile 'Scan and Secure'
The Scan and Secure feature enables you to quickly scan for and detect nearby Tiles and Tile-enabled devices that may be traveling with you. You can use this feature on iOS or Android, even if you do not have an active Tile account, as long as you have the latest version of the Tile app on your mobile device.

> https://tileteam.zendesk.com/hc/en-us/articles/4563823537431-Tile-Scan-and-Secure-Overview

When the 'Scan and Secure' feature is initiated, it starts a 10-minute timer and activates Bluetooth and location services on the user's device. The objective of this feature is to encourage the user to move around for 10 minutes, during which the app detects any Tile devices that may be nearby. This mechanism provides a proactive approach to ensure that users are aware of any Tile devices that may be following them, thus providing an added layer of security to the user's privacy. Furthermore, by encouraging users to move around, the app increases the likelihood of detecting any Tile devices attempting to track them without their knowledge or consent.

Regrettably, this concept has several limitations compared to a block of Swiss cheese. First, in practice, most tracking devices are not typically positioned directly on a user's forehead. Instead, these devices are typically attached to an item the user carries, such as a bag or a set of keys. This means that the 'Scan and Secure' feature may not detect all Tile devices that are close to the user since the app relies on the user's movement and proximity to the devices in question. Therefore, while the feature may provide a certain level of security and peace of mind to users, it may not be foolproof in detecting all Tile devices used to track the user's movements.

# What is Anti-Theft Mode for Tile Trackers?

As written by the company, Tile's Anti-Theft Mode can improve the chances of retrieving stolen property by hiding Tile trackers from scans made by others. This feature can be used with the Scan and Secure function of the Tile app to detect Tile-enabled devices nearby, even without an active Tile account.

Tile has an explanatory blog post on it:

> https://www.tile.com/blog/how-does-tile-anti-theft-mode-work


## Following that, I did it!
I enabled the Anti-Theft Mode on one of my four Tiles for testing purposes, and the process was seamless. However, as I discuss in my blog, I found the feature to be **pointless**.

<img src="/img/in-post/post-js-version/Tile/Tile-Enabled.jpeg" width="250" style="border: 1px solid black;">


# Separating Hype from Reality

## 1. It is visible to everyone
It has been observed that Tile's application for both iOS and Android does not display the device with Anti-Theft Mode enabled. This particular app is an exception, as all other apps show the device with Anti-Theft Mode enabled. However, it is essential to note that despite the absence of the device on Tile's app, it can still be tracked and located through other means available on the device.

Upon conducting a detailed analysis, it has been found that there is no noticeable variation in the packet values between a Tile device with Anti-Theft mode enabled and a standard Tile device. 

* Notably, all Tile devices have the UUID16 set to the value of 0xfeed, which corresponds to the manufacturer **Tile, Inc.**
* Tile devices advertise their Bluetooth Low Energy packet with the initial bytes set to **02:00**.


| Feature                                      | Value                                           |
|----------------------------------------------|-------------------------------------------------|
| frame.protocols                              | bluetooth:btle_rf:btle:btcommon                 |
| btle_rf.channel                              | 0,12,39                                         |
| btle.advertising_header                      | 0x1b60                                          |
| btle.advertising_header.pdu_type             | 0x00                                            |
| btle.advertising_header.rfu.1                | 0                                               |
| btle.advertising_header.ch_sel               | 1                                               |
| btle.advertising_header.randomized_tx        | 1                                               |
| btle.advertising_header.rfu.4                | 0                                               |
| btle.advertising_header.length               | 27                                              |
| btle.length                                  | 27                                              |
| btle.advertising_address                     | XX:XX:XX:XX:XX:XX                               |
| btcommon.eir_ad.advertising_data             | 0                                               |
| btcommon.eir_ad.entry                        | 3,13,2                                          |
| btcommon.eir_ad.entry.length                 | 0x03,0x01,0x16                                  |
| btcommon.eir_ad.entry.type                   | 0x00                                            |
| btcommon.eir_ad.entry.flags.reserved         | 0x00                                            |
| btcommon.eir_ad.entry.flags.le_bredr_support_host | 0x01                                       |
| btcommon.eir_ad.entry.flags.le_bredr_support_controller | 0x01                                 |
| btcommon.eir_ad.entry.flags.bredr_not_supported | 0x00                                         |
| btcommon.eir_ad.entry.flags.le_general_discoverable_mode | 0x01                                |
| btcommon.eir_ad.entry.flags.le_limited_discoverable_mode | 0x00                                |
| btcommon.eir_ad.entry.uuid_16                | 0xfeed                                          |
| btcommon.eir_ad.entry.service_data           | 02:00:.*                                        |
| Mac Company                                  | Tile, Inc.                                      |


## 2. No MAC Address Randomization:
This function is an entertaining example of a 'Fake step forward, three steps back' feature. The code generates a hash of the MAC address associated with the device on which Anti-Theft mode is enabled. The Scan and Secure application then utilize this hash to determine whether to include the device in the list of devices to be scanned.

The Tile Tracker with Anti-Theft mode has a static Mac-Address, which may make it even more trackable. This static address is assigned to the device and does not change, making it possible for someone to track the device's location over time by monitoring its MAC address.

It is worth noting that Apple AirTags, a superior product by default, has its MAC-Address Randomization feature turned on. This feature is an added layer of privacy and security that ensures that the AirTag's unique MAC address is not easily traceable by potential eavesdroppers or malicious actors. With MAC-Address Randomization turned on, the AirTag's MAC address is randomized and changes frequently, making it difficult for anyone to track the user's movements through the AirTag. 

## 3. Anti-Theft Mode is "Hush Money" but with your Identity.
In the world of personal privacy, a new question has emerged: Is it worth giving up personal data to avoid the public display of personal information? Tile, a company specializing in Bluetooth tracking devices, has introduced the Anti-Theft mode. This mode, when enabled, suppresses the public display of device information in the event of theft or loss.

<img src="/img/in-post/post-js-version/Tile/Tile-theftmode.PNG" width="" style="border: 1px solid black;">

Tile's Anti-Theft mode, however, is not a monetary transaction. Instead, the company requests personal information from users in exchange for the suppression of device information. This information includes the user's identity, license, or passport, a 3D scan of their entire face, and location.

This is much personal information to share, particularly for those concerned about privacy and data protection. However, Tile argues that this information is necessary to use the Anti-Theft mode and prevent fraud properly.

# Conclusion

The Anti-Theft Mode offered by Tile seems to be a mere publicity stunt, which is unsurprising given the competition posed by Apple's Airtags. However, do not let the hype blind you from reality, for only by seeing things as they are can we make informed decisions. 

* **Hype** refers to the exaggerated or excessive promotion of a product, idea, or phenomenon, often done to generate excitement, interest, or buzz among the public.
    * https://www.macrumors.com/2023/02/16/tile-anti-theft-mode/
    * https://www.theverge.com/2023/2/17/23603989/tile-location-tracker-anti-theft-mode
    * https://securityboulevard.com/2023/02/tile-trackers-accountability-mode/

* Conversely, **reality** refers to the actual state of things, free from embellishment or exaggeration. It is the truth or actual experience of something without preconceived notions or expectations.
    * https://Bambi.ai/ - Allows you to see ALL tracking devices around you instantly. Android and IOS apps are releasing very soon!
