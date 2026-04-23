# burp-android-setup
Step-by-step notes on pulling APKs via ADB, handling split APKs, patching certificate pinning with apk-mitm, and routing traffic through Burp Suite for mobile security testing.

---

## Proxy Android Apps Through Burp Suite (Windows)

### Note

If other apps stop working, reset the wifi proxy settings

* Social logins (Google, Facebook, etc.) may **not work** in patched apps

  * These flows often rely on secure webviews, integrity checks, or external app handoffs that break after modification
  * Use **email/password login** wherever possible for testing

### Disclaimer

For testing apps you are authorized to test only. Some apps may break due to anti-tampering or certificate pinning protections.

---

## Requirements

* ADB (Platform Tools)
* Java (for APKEditor)
* Node.js (for apk-mitm)
* Android device with USB debugging enabled

---

## 1. Setup ADB

1. Download Platform Tools from Google
2. Extract the folder
3. Open terminal inside the folder

Enable **USB Debugging** on your phone and connect it.

Check connection:

```
adb devices
```

---

## 2. Find Target App Package

```
adb shell pm list packages | findstr appname
```

Example output:

```
package:com.example.app
```

---

## 3. Get APK Paths

```
adb shell pm path com.example.app
```

This many times return **multiple paths** (split APKs), e.g.:

```
package:/data/app/.../base.apk
package:/data/app/.../split_config.arm64_v8a.apk
package:/data/app/.../split_config.en.apk
```

---

## 4. Pull APKs

Create a folder:

```
mkdir apks
```

Pull all APKs:

```
adb pull <path-from-above> apks/
```
**Pull all paths returned in the previous step**

Copy the paths from previous output.

---

## 5. Merge Split APKs

Modern apps use split APKs. These must be merged before patching.

Download APKEditor (jar file), then:

```
java -jar APKEditor-1.4.8.jar m -i apks/
```

Output:

```
apks_merged.apk
```

---

## 6. Install apk-mitm

```
npm install -g apk-mitm
```

---

## 7. Patch APK (Bypass Certificate Pinning)

```
apk-mitm apks_merged.apk
```
apk-mitm modifies the app to trust user-installed certificates and bypass certificate pinning

Output:

```
apks_merged-patched.apk
```

---

## 8. Install Patched APK

Uninstall original app first:

```
adb uninstall com.example.app
```

Then install patched version:

```
adb install apks_merged-patched.apk
```

If install fails, make sure the original app is fully removed.

---

## 9. Configure Proxy

On your phone:

* Connect to same WiFi as your PC
* Set manual proxy:

  * IP = your PC IP
  * Port = 8080 (default for Burp)

Install Burp certificate on device (required for HTTPS interception).
Make sure the certificate is installed as a user CA and trusted by the device
In Burp proxy settings, edit the proxy listeners to listen from all interfaces. 

---

## Notes

* Some apps will **not work** after patching due to:

  * Root detection
  * Integrity checks
  * Advanced pinning
* Play Protect may warn about modified apps
* Apps from Play Store may auto-update and overwrite patched versions
* Not all apps can be pulled easily 
---
