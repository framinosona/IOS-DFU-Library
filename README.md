# Device Firmware Update application

DFU application allows to flash new firmware on the DK. The process has three steps:
1. Selecting zip file with an appropriate firmware.
2. Selecting BLE device with open bootloader.
3. Starting firmware upload.

Settings screen allows to change the DFU library parameters used for uploading firmware.

<img src="App/fastlane/screenshots/en-US/iPhone 8 Plus-01Devices.png" width="300"> <img src="App/fastlane/screenshots/en-US/iPhone 8 Plus-02Settings.png" width="300">

# iOS DFU Library

[![Version](http://img.shields.io/cocoapods/v/NordicDFU.svg)](http://cocoapods.org/pods/NordicDFU)
[![Carthage compatible](https://img.shields.io/badge/Carthage-compatible-4BC51D.svg?style=flat)](https://github.com/Carthage/Carthage)

## Installation

**For Cocoapods:**

- Create/Update your **Podfile** with the following contents

```ruby
target 'YourAppTargetName' do
    use_frameworks!
    pod 'NordicDFU'
end
```

- Install dependencies

```ruby
pod install
```

- Open the newly created `.xcworkspace`

- Import the library to any of your classes by using `import NordicDFU` and begin working on your project


**For Carthage:**

- Create a new **Cartfile** in your project's root with the following contents

```
github "NordicSemiconductor/IOS-DFU-Library" ~> x.y // Replace x.y with your required version
```

- Build with carthage

```sh
carthage update --use-xcframeworks --platform iOS // other supported platforms: macOS, tvOS, watchOS
```

- Carthage will build the **NordicDFU.framework** and **ZipFramework.framework** files in **Carthage/Build/**, 
you may now copy all those files to your project and use the library, additionally, carthage also builds **\*.dsym** files 
if you need to resymbolicate crash logs. you may want to keep those files bundled with your builds for future use.

**For Swift Package Manager:**

```swift
// swift-tools-version:5.6
import PackageDescription

let package = Package(
  name: "<Your Product Name>",
  dependencies: [
    .package(
      url: "https://github.com/NordicSemiconductor/IOS-DFU-Library", 
      .upToNextMajor(from: "<Desired Version>")
    )
  ],
  targets: [.target(name: "<Your Target Name>", dependencies: ["NordicDFU"])]
)
```

---

## Device Firmware Update (DFU)

The nRF5x Series chips are flash-based SoCs, and as such they represent the most flexible solution available. 
A key feature of the nRF5x Series and their associated software architecture and S-Series SoftDevices is the 
possibility for Over-The-Air Device Firmware Upgrade (OTA-DFU). See Figure 1. 
OTA-DFU allows firmware upgrades to be issued and downloaded to products in the field via the cloud and so
enables OEMs to fix bugs and introduce new features to products that are already out on the market. 
This brings added security and flexibility to product development when using the nRF5x Series SoCs.

This repository contains a tested library for iOS 8+ platform which may be used to perform Device Firmware Update 
on the nRF5x device using an iPhone or an iPad.

DFU library has been designed to make it very easy to include these devices into your application. 
It is compatible with all Bootloader/DFU versions.

[![Alt text for your video](http://img.youtube.com/vi/LdY2m_bZTgE/0.jpg)](http://youtu.be/LdY2m_bZTgE)

### Service Changed characteristic

In order the DFU to work with iOS, the target device MUST have the **Service Changed** characteristic with 
Indicate property in the **Generic Attribute** service. Without this characteristic iOS will assume that services of
this device will never change and will not invalidate them after switching to DFU bootloader mode.

##### Service Changed characteristic behavior:

- On paired devices a change of the attribute table must be indicated using an indication to the Service Changed characteristic. 
iOS automatically enables the CCC and handles this indication and performs a service discovery. 
This indication is handled correctly in Legacy DFU since SDK 8.0.
- On non-trusted devices (not paired) iOS will clear the service cache every time the device disconnects.

##### Secure DFU from SDK 12:

- The Secure DFU implementation from SDK 12 does not support bonding (experimental buttonless sample does not 
pass bond information when switching to DFU bootloader mode and the bootloader does not send S-C indication). 
As a workaround, the bootloader starts to advertise with MAC address incremented by 1, so from the phone's perspective 
it's a completely new device and a fresh service discovery will be done. When your new firmware is going to change 
the list of services you may consider adding another 1 to the MAC address for the new application to make sure 
the cache will not conflict (unless the device is not bonded and you have Service Changed characteristic, then no 
caching is used as written above). Be aware, that adding 1 to a public address is not possible (unless you register a new one). 
Also, devices may be sold with following MAC addresses and it may happen that 2 devices have the same one. 
Use this feature carefully.

##### Secure DFU from SDK 14:

- Buttonless DFU with Bonds Sharing has been added to the SDK. Bonded relationship is required to use
this service. Address does not change when in DFU mode, instead the bootloader sends Service Changed
indication when entered DFU mode and app mode. For bonded devices it is recommended to use this service.

---

## Documentation

See the [documentation](https://nordicsemiconductor.github.io/IOS-DFU-Library/documentation/nordicdfu) for more information.

---

## Requirements

The library is compatible with nRF51 and nRF52 devices with S-Series Soft Device and the DFU Bootloader flashed on. 

---

## DFU History

### Legacy DFU

* **SDK 4.3.0** - First version of DFU over Bluetooth Smart. DFU supports Application update.
* **SDK 6.1.0** - DFU Bootloader supports Soft Device and Bootloader update. As the updated Bootloader may be dependent on the new Soft Device, those two may be sent and installed together.

- Buttonless update support for non-bonded devices.

* **SDK 7.0.0** - The extended init packet is required. The init packet contains additional validation information: device type and revision, application version, compatible Soft Devices and the firmware CRC.
* **SDK 8.0.0** - The bond information may be preserved after an application update. The new application, when first started, will send the Service Change indication to the phone to refresh the services. New features:

- Buttonless update support for bonded devices 
- sharing the LTK between an app and the bootloader.

### Secure DFU

* **SDK 12.0.0** - New Secure DFU has been released. This library is fully backwards compatible so supports both the new and legacy DFU.
* **SDK 13.0.0** - Buttonless DFU (still experimental) uses different UUIDs. No bond sharing supported. Bootloader will use address +1.
* **SDK 14.0.0** - Buttonless DFU no longer experimental. New buttonless characteristic added for bonded devices (requires bond, cache cleaning relies on Service Changed indication).
* **SDK 15.0.0** - Support for higher MTUs added.

This library is fully backwards compatible and supports both the new and legacy DFU. The experimental buttonless DFU service from SDK 12 is supported since version 1.1.0. Due to the fact, that this experimental service from SDK 12 is not safe, you have to set [enableUnsafeExperimentalButtonlessServiceInSecureDfu](https://github.com/NordicSemiconductor/IOS-DFU-Library/blob/main/Library/Classes/Implementation/DFUServiceInitiator.swift#L325) to true to enable it, this is off by default. Read the method documentation for details. It is recommended to use the Buttonless service from SDK 13 (for non-bonded devices, or 14 for bonded). Both are supported since DFU Library 1.3.0.

Check platform folders for mode details about compatibility for each library.

## Other frameworks

### React Native

An unofficial library for both iOS and Android that is based on this library is available for React Native: [react-native-nordic-dfu](https://github.com/Salt-PepperEngineering/react-native-nordic-dfu)

### Expo

An unofficial Expo module for both iOS and Android that is based on this library is available: [expo-nordic-dfu](https://github.com/getquip/expo-nordic-dfu)

### Flutter

A library for both iOS and Android that is based on this library is available for Flutter: 
[nordic_dfu](https://pub.dev/packages/nordic_dfu)

### Xamarin

Simple binding library for iOS is available on nuget:
[Laerdal.Dfu](https://www.nuget.org/packages/Laerdal.Dfu/)

---

### Resources

- [DFU Introduction](https://infocenter.nordicsemi.com/topic/sdk_nrf5_v17.0.2/lib_bootloader_modules.html?cp=8_1_3_5 "Documentation")
- [How to create init packet](https://infocenter.nordicsemi.com/topic/sdk_nrf5_v17.0.2/lib_bootloader_dfu_validation.html#lib_dfu_image "Init packet")
- [nRF51 Development Kit (DK)](https://www.nordicsemi.com/Software-and-Tools/Development-Kits/nRF51-DK "nRF51 DK") (compatible with Arduino Uno Revision 3)
- [nRF52 Development Kit (DK)](https://www.nordicsemi.com/Software-and-Tools/Development-Kits/nRF52-DK "nRF52 DK") (compatible with Arduino Uno Revision 3)
- [nRF52840 Development Kit (DK)](https://www.nordicsemi.com/Software-and-Tools/Development-Kits/nRF52840-DK "nRF52840 DK") (compatible with Arduino Uno Revision 3)


