# About Android Security Model

Two distinct layers to Android Security model.

**1. Implemented in the OS, and isolates installed app from one another**

* Each app has a specific UID, dynamically assigned.
* An app can only access its UID files and no other (except if shared by another app or OS)
* Each App runs as a separate process under a sperate UID
* Prior to Android 4.3 = the only thing that was isolating apps = if root compromised entire system was compromised
* Starting from Android 4.3 = SELinux
* SELinux denies all process interaction + create policies to allow only expected applications

**2. Security of an App itself (made by the developers)**

* The dev can selectively expose certain app functions to other apps
* Configures App capabilities
* All apps are in the /data/data folder (except if modified in manifest by dev)
* The permissions declared in the manifest will be translated in permissions in the file system.

# About Apps

## Structure

An Android App comprises two main elements:

1. The program's core functionality, written in Java code or Kotlin (official language today)
2. The XML files that specify various configurations, including string values and the app's identity.

## Types

> **Native:** They are those developed applications only and exclusively for mobile operating systems, either Android or IOS. In Android you use the Java or Kotlin programming language, while in IOS you make use of Swift or Objective-C. These programming languages are the official ones for the respective operating systems.

> **Hybrid**: These applications use technologies such as HTML, CSS and JavaScript, all of these linked and processed through frameworks such as Apache Córdova "PhoneGap", Ionic, among others.

## Directories

**App directories in device**

_/data/data/\<package\_name>._ By default, the apps databases, settings, and all other data go here.

* **databases/**: here go the app's databases
* **lib/**: libraries and helpers for the app
* **files/**: other related files
* **shared\_prefs/**: preferences and settings
* **cache/**: well, caches

**APK Anatomy**

When decompiled :

* **AndroidManifest.xml**: contains the application’s package name, access rights, referenced libraries as well as other metadata.
* **classes.dex**: contains the application source code compiled in .dex file format.
* **resources.arsc**: contains the application’s precompiled resources.
* **res/** : contains the application's resources not compiled into resources.arsc.
* **lib/** : contains compiled code that is platform-dependent. Each sub-directory in lib/ contains the specific source code for respective processors.
* **assets/** : contains the application’s assets.
* **META-INF/** : contains the MANIFEST.MF file, which stores metadata about the application. It also contains the certificate and signature of the APK.
