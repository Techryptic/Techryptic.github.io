---
layout:     post
title:      "Annoying Apple Fans: The Flipper Zero Bluetooth Prank Revealed"
date:       2023-09-01
author:     "Tech"
header-img: "img/in-post/post-js-version/annoying-apple-fans/apple-ble-annoyed.png"
tags:
    - Bluetooth
---
> **9/6/2023 Update:** I'd like to draw your attention to the fact that I've been addressing this matter since November 2022. Feel free to examine my Twitter or YouTube history for further information.

> **9/6/2023 Update:** I've already submitted this concern to the Apple Research team, and it's crucial to emphasize its significance. I propose two potential solutions for Apple, with the added benefit of reducing the threat:
First, they should bolster the proximity RSSI check, ensuring that receiving devices establish a stronger connection with Apple devices.
Secondly, from a technical perspective, it's worth noting that the BLE 4.0 specification has a limited payload size of only 31 bytes, which isn't sufficient for proper checksums. In contrast, the BLE 5.0 specification increases this limit to 255 bytes, allowing for more robust packet checks. Keep in mind that these values are subject to implementation specifics and may vary between different Bluetooth chipsets and devices. 

> **9/5/2023 Update:** Featured by TechCrunch, you can read it here: [Hacking device Flipper Zero can spam nearby iPhones with Bluetooth pop-ups](https://techcrunch.com/2023/09/05/flipper-zero-hacking-iphone-flood-popups/)

> **9/3/2023 Update:** Youtube Video Demo

{% include youtube.html id="OWXt8oTJ1lo" %}

---

#  Annoying Apple Fans: The Flipper Zero Bluetooth Prank Revealed
The Bluetooth Low Energy (BLE) protocol, a cornerstone of modern wireless communication, has been instrumental in enabling seamless interactions between devices. Its advertising packets, in particular, are broadcast signals that devices use to announce their presence and capabilities. Apple's ecosystem, with its myriad of interconnected devices, heavily relies on these packets for functionalities ranging from AirDrop file transfers to Apple Watch connectivity.

In November 2022, I released a [Youtube Video](http://www.youtube.com/watch?v=m_-nMw5bzjI) discussing AirTag spoofing. We're going to extend that to other services.

# Bluetooth Low Energy (BLE) and ADV Packets:

Bluetooth Low Energy (BLE), as part of the Bluetooth 4.0 specification, was introduced to cater to applications that require minimal power consumption. It's especially suitable for devices that need periodic or occasional transfer of data.

One of the primary mechanisms by which BLE devices communicate or make their presence known is through advertising packets, commonly referred to as ADV packets.

## ADV Packets:

> **Purpose:** ADV packets are broadcasted by devices to announce their presence. These can be picked up by any device that's listening, without pairing.

**Types of Advertising:**

- **Connectable Directed Advertising:** Targeted advertising for a specific device.
- **Non-connectable Undirected Advertising:** For devices broadcasting information without connecting, like a beacon.
- **Scannable Undirected Advertising:** Allows a scan request from a receiving device but doesn't establish a connection.

**Frequency:** Devices can adjust their ADV packet broadcast frequency, balancing power consumption against discoverability.

**Data Payload:** Contains information like the device's name or services it offers.

### Data Payload Notifications

Let's break this down using an example from capturing AirTags data, although the same principles apply to any type of notification you might want to send in a spam-like manner.

In the Wireshark capture I conducted back in January, I observed that the notifications followed a standardized format, with only a few bytes distinguishing them for each type of notification. To conduct these captures, I have my own Faraday Box, which provides a controlled environment for capturing various types of notifications.

<img src="/img/in-post/post-js-version/annoying-apple-fans/wireshark1.PNG" width="" style="border: 1px solid black;">

In the image provided, we should focus on the Data column, highlighted in blue.

Now, let's dive deeper into the analysis:

<img src="/img/in-post/post-js-version/annoying-apple-fans/wireshark2.PNG" width="" style="border: 1px solid black;">


From the image, several key points become evident. Our goal is to replicate this data to be sent by the Flipper or any other device. The data adheres to a standardized format, as previously mentioned.


Here's a breakdown of the data:
```c
0x1E, // Length
0xFF, // Type
0x4C, 0x00, // Apple, Inc.

// Now comes the Data of the AirTag packet
0x0719050055100000014eca9189f2e68aa8c7dc1e4ac7d331ef52d3
```

From this point, you can replay it on any device, triggering a notification.
However, for consistency, we can perform some data cleanup.
Like all Apple BLE Notifications, AirTag data follows a structured format, and by starting from the end and zeroing out two bytes at a time, we can identify which ones are unnecessary.

I've written a script to handle this, but in essence, we realize that approximately 90% of the data in the packet is redundant.

```c
0x0719050055100000014eca9189f2e68aa8c7dc1e4ac7d331ef52d3 // AirTag with Capture Data
0x071905005500000000000000000000000000000000000000000000 // AirTag with only the essential hex data required for the notification to pop.
```

For other types of notifications, the process follows a similar pattern. An event is triggered on the device, and the corresponding BLE data can be captured. This data can be easily replicated by copying it and, if desired, trimming away any unnecessary portions. **It's worth noting that this standardized format, serving as a signature of the BLE packet, remains consistent across all devices, ensuring accessibility for everyone within the Apple Ecosystem.**


# Implications for Apple

Apple's ecosystem relies on BLE for many integrations and features. Here's the role of ADV packets for Apple:

- **AirDrop:** Uses BLE to discover nearby devices with ADV packets identifying potential share targets.

- **Handoff:** Uses BLE to detect nearby devices, with ADV packets determining device presence.

- **Apple Watch:** Uses BLE for a low-energy connection with the iPhone, with ADV packets aiding in discovery and connection.

- **HomeKit:** Many HomeKit devices use BLE, advertising their status or availability.

- **Security and Privacy:** Apple ensures user privacy by implementing measures like rotating the Bluetooth address to prevent tracking, even though ADV packets can be picked up by any listening device.

- **iBeacons:** Uses BLE's non-connectable undirected advertising to broadcast a unique identifier for location-based services.

In conclusion, ADV packets in BLE are crucial for many of Apple's features. Apple harnesses BLE advertising for efficient device communication while prioritizing user privacy and security.

# Flipper Zero Bluetooth Prank Revealed
Enter the Flipper Zero, a multi-tool device for hackers and tinkerers. One of its capabilities is to interact with BLE protocols, and more specifically, to mimic or spoof these advertising packets.

When a device like Flipper Zero mimics the advertising packets of legitimate devices or services, it can create a plethora of phantom devices in the vicinity of an iOS user. Imagine searching for a device to connect to and being presented with a list of dozens, if not hundreds, of fake device names. Or attempting an AirDrop and being flooded with fictitious recipients. It's not just a minor inconvenience; it can disrupt the seamless experience that Apple users are accustomed to.

But why would someone do this? The reasons can vary:

- **Pranks:** Some might find it amusing to watch Apple aficionados grapple with a sudden influx of mysterious devices appearing on their screens.
- **Testing and Research:** Cybersecurity professionals might use such tactics to study vulnerabilities or test the robustness of BLE implementations.
- **Malicious Intent:** While less common, there's potential for malicious actors to exploit this for nefarious purposes, such as a type of phishing attack by mimicking trusted notifications. (Blog Post being written)

For iOS users, this mimicry can be more than just an annoyance. It can lead to confusion, disrupt workflows, and in rare cases, pose security concerns. It underscores the importance of being aware of the devices around us and the potential vulnerabilities inherent in wireless communications.


But why shed light on this? It's essential to note that the Flipper Zero's range is limited. To truly impact an iOS device, you'd need to be in close proximity. This isn't about widespread disruption, but rather an exploration of the playful potential and boundaries of wireless communication.


# Here's how it's done.
We'll be updating the Flipper Zero by altering a specific file responsible for its Bluetooth functionality, then compiling and applying the firmware update.

Make sure you have enough space and clone the source code:

> git clone --recursive https://github.com/flipperdevices/flipperzero-firmware.git

I've streamlined the process for you, allowing you to effortlessly select the type of notification you'd like to display on nearby iOS devices.

{% include flipper-BLE.html %}

### Updating the 'gap.c' file and then compiling the updated firmware.

After your selection of notification type, copy the provided code output and then update the contents within the 'gap.c' file located at the specified location.

<img src="/img/in-post/post-js-version/annoying-apple-fans/gap.c_location.PNG" width="" style="border: 1px solid black;">

After updating the file, proceed to compile the firmware. Ensure you're in the root directory before executing the following command:

> ./fbt COMPACT=1 DEBUG=0 VERBOSE=1 updater_package

As an illustration:

<img src="/img/in-post/post-js-version/annoying-apple-fans/compile.PNG" width="" style="border: 1px solid black;">

Once the command executes, near the end, you'll identify the directory where the compilation occurred: 

<img src="/img/in-post/post-js-version/annoying-apple-fans/compile_directory.PNG" width="" style="border: 1px solid black;">

Within that folder, your new compiled firmware is the .tgz file that we will open in the qFlipper app.

<img src="/img/in-post/post-js-version/annoying-apple-fans/firmware_tgz.PNG" width="" style="border: 1px solid black;">

In that directory, the newly compiled firmware is represented by the .tgz file, which we'll load into the qFlipper app.

<img src="/img/in-post/post-js-version/annoying-apple-fans/update_flipper.PNG" width="" style="border: 1px solid black;">


## Have fun!
Here are some examples of what the notifications look like.
<video src='/img/in-post/post-js-version/annoying-apple-fans/applevideo.mp4' style="max-width: 100%; width: 100%; height: auto;" autoplay muted loop></video>
