---
title: "Old macOS"
date: 2024-01-21T16:54:39+01:00
draft: false
---

I just spent more time than I'd liked reinstalling macOS on a core 2 due era 13 " MacBook. Apple kindly provides images of old versions that can in theory be used to create install media, but on this machine the .pkg file refused to work with the installer. On each try I'd get the same error message and a new MacBook would give a different error when trying to extract an Intel OS .pkg file.

The thing that finally worked was to download Lion (from Apple at https://support.apple.com/en-us/102662), get the InstallESD.dmg-file from the package by manually extracting the pkg and then flashing that to a usb stick using Balena etcher.
