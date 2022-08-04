---
title: Mobile development with Nix
date: 2022-08-01
---

## Adb setup

We need to setup `adb` for all further interations with an android device. This can be done by using two packages, one for udev rules and one for android platform tools.

Our `shell.nix` will look like this:

```nix
{ pkgs ? import <nixpkgs> { } }:

pkgs.mkShell {
  buildInputs =
    [ pkgs.android-udev-rules pkgs.androidenv.androidPkgs_9_0.platform-tools ];
}
```

For NixOS you can simply do

```nix
{
  services.udev.packages = [ pkgs.android-udev-rules ];
}
```

And `/etc/udev/rules.d/51-android.rules` should be created.
