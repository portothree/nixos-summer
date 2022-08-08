---
title: Mobile development with Nix
date: 2022-08-01
---

## Adb setup

We need to setup `adb` for all further interations with an android device. This can be done by using two packages, one for udev rules and one for android platform tools.

```nix
# flake.nix
{
  description = "StreetComplete";
  inputs.nixpkgs.url = "nixpkgs/nixos-22.05";
  outputs = { self, nixpkgs }:
    let
      system = "x86_64-linux";
      pkgs = import nixpkgs {
        inherit system;
        config.android_sdk.accept_license = true;
      };
    in {
      devShell."${system}" = import ./shell.nix {
        inherit pkgs; 
      };
    };
}
```

```nix
# shell.nix
{ pkgs }:

pkgs.mkShell {
  buildInputs = with pkgs; [ 
    android-udev-rules
    androidenv.androidPkgs_9_0.platform-tools
  ];
}
```

For NixOS you can simply do

```nix
{
  services.udev.packages = [ pkgs.android-udev-rules ];
}
```

And `/etc/udev/rules.d/51-android.rules` should be created.

## Android SDK

Later we are going to use a build tool called Gradle that relies on android SDK to build and test our application. You can use the `nixpkgs.androidenv` helper to compose an Android SDK installation with plugins, but in the spirit of flakes, we are going to use the [android-nixpkgs](https://github.com/tadfisher/android-nixpkgs) flake.

```nix
# flake.nix
{
  description = "StreetComplete";
  inputs.nixpkgs.url = "nixpkgs/nixos-22.05";
  inputs.android-nixpkgs.url = "github:tadfisher/android-nixpkgs";
  inputs.android-nixpkgs.inputs.nixpkgs.follows = "nixpkgs";
  outputs = { self, nixpkgs, android-nixpkgs }:
    let
      system = "x86_64-linux";
      pkgs = import nixpkgs {
        inherit system;
        config.android_sdk.accept_license = true;
      };
      androidSdk = android-nixpkgs.sdk.${system} (sdkPkgs: with sdkPkgs; [
        cmdline-tools-latest
        build-tools-30-0-3
        platform-tools
        platforms-android-31
        emulator
      ]);
    in {
      devShell.${system} = import ./shell.nix {
        inherit pkgs; 
        inherit androidSdk;
      };
    };
}
```

```nix
# shell.nix
{ pkgs, androidSdk }:

pkgs.mkShell {
  buildInputs = with pkgs; [ 
    android-udev-rules 
    androidenv.androidPkgs_9_0.platform-tools
    androidSdk
  ];
}
```

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
