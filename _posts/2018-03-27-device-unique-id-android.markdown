---
title: "Using MediaDrm for device ID tracking"
layout: post
date: 2018-03-27 14:00
image: /assets/images/kotlin-android-extensions.png
headerImage: false
tag:
- android
category: blog
author: miquel
description: How to use MediaDrm to obtain a device unique ID
---

If you read the [Best Practices for Unique Identifiers](https://developer.android.com/training/articles/user-data-ids.html)
you will see that Google discourages the usage of device (hardware) identifiers
in order to identify users.

With the recent news regarding Facebook data privacy practices, it's important
to raise awareness on how can app developers track your devices, match users
across different apps and how is the operating system protecting us.

Until now, requesting the `ANDROID_ID` has been the go-to solution for many.

```kotlin
val androidId = Settings.Secure.getString(contentResolver,
                                          Settings.Secure.ANDROID_ID)
```

If you use this snippet, Android Studio will warn you to instead use less
confidential IDs like the Advertising ID or a custom instant ID.

And in fact, [since Android 8.0](https://developer.android.com/reference/android/provider/Settings.Secure.html#ANDROID_ID)
, the `ANDROID_ID` is generated for each app, making not possible for app
developers to use it to track across different apps.

I welcome this change, although it comes very late.

The `ANDROID_ID` originally is stored in the device operating system and will
change with a factory reset. As well, this ID is different for each
user in the platform.

Also rooted users can change their `ANDROID_ID`. There's apps that allow you
to do that.

Other solutions involve requiring user permissions. For example `PHONE`
permission is required to read the user IMEI.

### Introducing: MediaDrm

MediaDrm is an Android framework API that allows users to securely provide
encryption keys to MediaCodec for playing premium content in a secure way.

MediaDrm works with different DRM providers. Widevine from Google, but also
PlayReady from Microsoft depending on the device.

When a device uses DRM for the first time, a device provisioning occurs, which
means that the device will obtain a unique certificate and it will be stored
in the DRM service of the device.

This provisioning profile has a unique ID, and you can obtain it with a simple
call.

```kotlin
val id = MediaDrm(WIDEVINE_UUID)
            .getPropertyByteArray(MediaDrm.PROPERTY_DEVICE_UNIQUE_ID)
```

You will need the UUID that identifies the different DRM providers. Here's a
list of the common ones (extracted from ExoPlayer):

```kotlin
val COMMON_PSSH_UUID = UUID(0x1077EFECC0B24D02L, -0x531cc3e1ad1d04b5L)
val CLEARKEY_UUID = UUID(-0x1d8e62a7567a4c37L, 0x781AB030AF78D30EL)
val WIDEVINE_UUID = UUID(-0x121074568629b532L, -0x5c37d8232ae2de13L)
val PLAYREADY_UUID = UUID(-0x65fb0f8667bfbd7aL, -0x546d19a41f77a06bL)
```

Widevine is the most common DRM found on Android devices, but you can use the
rest as an alternative. MediaDrm constructor will throw an exception if the DRM
is not available.

The ID you will obtain is a 16 byte array, which you can conveniently convert
to an int or to a Base64 String for your own use.

### The Problem

This ID is not only the same on all apps, but also it is the same for all
users of the device. So a guest account, for example, will also obtain the same
ID, as opposed to the `ANDROID_ID`.

As well, no permissions are required to access this ID.

There's not much you can do as the user to avoid this. Only a factory reset will
restart this provisioning profile. While Google introduced ways to improve
privacy around the `ANDROID_ID` on Android 8.0, making it unique per app,
the design of DRM systems does not allow much to do against it.

Maybe in the future apps should require permissions to access DRM services.

