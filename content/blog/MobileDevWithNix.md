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

## Gradle

Our example application (StreetComplete) uses gradlew, a gradle wrapper which facilitates the usage of gradle.

`shell.nix`

```nix
{ pkgs ? import <nixpkgs> {config.android_sdk.accept_license = true;} }:

let
  androidSdk = pkgs.androidenv.androidPkgs_9_0.androidsdk;
in
pkgs.mkShell {
  buildInputs = with pkgs; [
    androidSdk
    glibc
  ];
  # override the aapt2 that gradle uses with the nix-shipped version
  GRADLE_OPTS = "-Dorg.gradle.project.android.aapt2FromMavenOverride=${androidSdk}/libexec/android-sdk/build-tools/28.0.3/aapt2";
}
```
