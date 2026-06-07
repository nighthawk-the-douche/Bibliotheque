---
icon: android
---

# Android OS Theory

## <mark style="color:$primary;">ANDROID OS</mark>

Android is a mobile OS based on a modified Linux kernel, developed by the Open Handset Alliance and commercially sponsored by Google. Most devices ship with Google Mobile Services (GMS) — proprietary suite including Play Store and Chrome. Used in phones, tablets, smart TVs, and wearables.

Apps are distributed through Google Play Store, Amazon Appstore, Samsung Galaxy Store, Huawei AppGallery, and open-source platforms (F-Droid, APKPure, APKMirror).

### <mark style="color:purple;">Hardware</mark>

Majority of devices run **ARM (AArch64)**. x86/x86-64 supported in later versions (Intel processors). Since Android is Linux-based, a shell on the device gives full access to Linux commands.

### <mark style="color:purple;">Android Software Stack</mark>

Android's architecture is a six-layer stack where each layer depends on the one below it and exposes functionality to the one above.

<figure><img src="../../.gitbook/assets/Android Schemes(1).png" alt=""><figcaption></figcaption></figure>

#### <mark style="color:red;">Linux Kernel</mark>

The foundation of the entire platform. Responsible for managing device hardware — display, camera, Bluetooth, WiFi, audio, USB, and more. Also handles lower-level OS functions like threading, memory management, and process scheduling, which the Android Runtime relies on. Android does not use a stock Linux kernel — it includes custom patches and drivers (most importantly, the Binder IPC driver) that don't exist in upstream Linux.

#### <mark style="color:red;">HAL (Hardware Abstraction Layer)</mark>

Sits between the Linux kernel and the Android framework. Different hardware components (cameras, sensors, Bluetooth chips) have vendor-specific implementations — HAL provides a standardized interface so the Android framework can talk to any hardware without caring about the specifics underneath. Implemented as a collection of shared libraries that are dynamically loaded at runtime. This is what allows device manufacturers (Samsung, Xiaomi, etc.) to ship custom hardware while remaining compatible with the broader Android platform.

#### <mark style="color:red;">Android Runtime (ART)</mark>

The managed runtime environment that executes Android applications. Replaced the older Dalvik VM in Android 5.0 (Lollipop). The key architectural difference between the two is compilation strategy:

* **Dalvik** used **JIT** (Just-in-Time) compilation — code was interpreted and compiled piece by piece during execution
* **ART** uses **AOT** (Ahead-of-Time) compilation — the entire app is compiled into native machine code at install time

The result is faster app launch times and better runtime performance, at the cost of slightly longer installation and more storage usage. ART also handles garbage collection and provides the sandbox each app runs in.

#### <mark style="color:red;">Native C/C++ Libraries</mark>

A set of core libraries written in C and C++ that are part of the Android OS itself. Components like ART and HAL are built on top of these. They handle things like graphics rendering (OpenGL ES), media processing, database access (SQLite), and SSL/TLS (OpenSSL/BoringSSL). Developers access these from Java/Kotlin code via **JNI** (Java Native Interface), or directly from native code using the **NDK** (Native Development Kit).

#### <mark style="color:red;">Java API Framework</mark>

The layer developers interact with most directly. Provides the building blocks for Android applications — Views, Activities, Services, Content Providers, Broadcast Receivers, system Managers (LocationManager, NotificationManager, etc.). Everything in this layer is a Java/Kotlin API that internally communicates with lower layers via Binder IPC and JNI calls.

#### <mark style="color:red;">System Apps</mark>

Pre-installed applications that ship with the device — Dialer, SMS, Browser, Settings, etc. They sit at the top of the stack and have the same API access as third-party apps, but may hold additional system-level permissions.

## <mark style="color:$primary;">SECURITY MODEL</mark>

Android's security model is built on Linux's multi-user architecture. The core idea: every app is treated as a separate user, and Linux enforces isolation between users at the kernel level.

* Each installed app is assigned a unique **UID** (user ID). The OS uses this UID for all access control decisions — the app itself never sees it.
* Filesystem permissions are UID-based. An app can only read and write its own files by default. Other apps' data directories are inaccessible unless explicitly shared.
* Each app runs in its own process. Each process gets its own ART instance, which means separate memory space — one app cannot read another app's memory.
* Processes are started on demand and killed when idle or when the system needs resources. No app has persistent background presence by default.
* Android enforces **principle of least privilege** — apps only get the permissions they declare in the manifest, and sensitive permissions require explicit user approval at runtime (Android 6.0+).

#### <mark style="color:red;">Verified Boot</mark>

A security feature that ensures the OS hasn't been tampered with between power-off and boot. Works by building a chain of trust: each boot stage cryptographically verifies the integrity of the next before handing control to it. The chain starts from a hardware root of trust (the bootloader) and goes through the kernel, system partition, and beyond.

If any stage fails verification — meaning the signature doesn't match — the device either refuses to boot entirely or displays a warning to the user that the software has been modified. This makes it difficult to persistently modify system partitions without the user knowing.

Verified Boot also includes **Rollback Protection**: the device refuses to boot older versions of Android, preventing attackers from downgrading to a version with a known exploitable vulnerability.

### <mark style="color:purple;">Rooting</mark>

Android separates flash storage into two main partitions:

* <mark style="color:$success;">`/system/`</mark> — the OS partition, mounted **read-only** at boot
* <mark style="color:$success;">`/data/`</mark> — user data and app installations, writable

By default, no user or app has root access. Rooting means gaining root privileges on the device — typically by exploiting a vulnerability in the kernel or bootloader to remount `/system/` as writable and install a root management app (like Magisk). On some devices (Google Pixel, OnePlus), rooting is also achievable by unlocking the bootloader through the OEM Unlock option in developer settings, which disables Verified Boot.

A rooted device gives full control — useful for security research, app analysis, and bypassing restrictions — but disables several built-in protections, making the device significantly more exposed to malware.

## <mark style="color:$primary;">IMPORTANT DIRECTORIES</mark>

| Directory                                                              | Description                                 |
| ---------------------------------------------------------------------- | ------------------------------------------- |
| <mark style="color:$success;">`/data/data`</mark>                      | User-installed apps data                    |
| <mark style="color:$success;">`/data/user/0`</mark>                    | App-private data                            |
| <mark style="color:$success;">`/data/app`</mark>                       | APKs of user-installed apps                 |
| <mark style="color:$success;">`/system/app`</mark>                     | Pre-installed system apps                   |
| <mark style="color:$success;">`/system/bin`</mark>                     | System binaries                             |
| <mark style="color:$success;">`/data/local/tmp`</mark>                 | World-writable directory                    |
| <mark style="color:$success;">`/data/system`</mark>                    | System configuration files                  |
| <mark style="color:$success;">`/etc/apns-conf.xml`</mark>              | Default APN configurations                  |
| <mark style="color:$success;">`/data/misc/wifi`</mark>                 | WiFi configuration files                    |
| <mark style="color:$success;">`/data/misc/user/0/cacerts-added`</mark> | User certificate store                      |
| <mark style="color:$success;">`/etc/security/cacerts/`</mark>          | System certificate store (root only)        |
| <mark style="color:$success;">`/sdcard`</mark>                         | Symlink to DCIM, Downloads, Music, Pictures |

## <mark style="color:$primary;">APK STRUCTURE</mark>

APK is an archive (`.apk`) containing all components needed to install and run an app.

<figure><img src="../../.gitbook/assets/Android Schemes.png" alt=""><figcaption></figcaption></figure>

Compilation flow: Java/Kotlin source → Java bytecode → **DEX** (Dalvik Executable) → executed by ART (Android 5.0+) or Dalvik VM (older).

| Component                                                  | Description                                                                                                                                                                                        |
| ---------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| <mark style="color:$success;">`META-INF/`</mark>           | Generated at signing. Contains `CERT.RSA` (public key + signature), `CERT.SF` (hashes of MANIFEST.MF lines), `MANIFEST.MF` (SHA256 hashes of all APK files). Any modification invalidates the APK. |
| <mark style="color:$success;">`assets/`</mark>             | Raw files bundled with the app (images, videos, DBs). Accessed via `AssetManager`. Xamarin/Cordova/React-Native apps store code and DLLs here.                                                     |
| <mark style="color:$success;">`lib/`</mark>                | Native libraries (`.so` files) per architecture. Present in apps using NDK (C/C++ components).                                                                                                     |
| <mark style="color:$success;">`res/`</mark>                | Predefined resources (XML layouts, colors, fonts, configs). Cannot be modified at runtime.                                                                                                         |
| <mark style="color:$success;">`AndroidManifest.xml`</mark> | App metadata: package name, SDK version, permissions, `NetworkSecurityConfig`, activities, providers, services.                                                                                    |
| <mark style="color:$success;">`classes.dex`</mark>         | All compiled Java/Kotlin classes in DEX format.                                                                                                                                                    |
| <mark style="color:$success;">`resources.arsc`</mark>      | Precompiled resources. Maps resource IDs (e.g., `R.string.app_name`) to actual values.                                                                                                             |

### <mark style="color:purple;">Native Code</mark>

Native code is C/C++ compiled directly for a specific processor architecture, bypassing the Java/ART layer entirely. This gives a performance advantage for computationally heavy tasks — graphics, cryptography, signal processing — where JVM overhead would be a bottleneck.

Native components are packaged as `.so` (shared object) files inside the APK's `lib/` directory, with a separate binary for each supported architecture (arm64-v8a, armeabi-v7a, x86, x86\_64). At runtime, the app loads the appropriate `.so` for the device's architecture.

**JNI (Java Native Interface)** is the bridge that makes this possible. It defines a standard contract for how managed Java/Kotlin code calls into unmanaged native C/C++ code and vice versa — including how types are converted, how method signatures are declared on both sides, and how memory is managed across the boundary.

## <mark style="color:$primary;">IPC — OS LEVEL</mark>

Physical transport mechanisms that move data across isolated process boundaries. Everything at the Application level eventually delegates to one of these.

### <mark style="color:purple;">Binder (</mark><mark style="color:purple;">`/dev/binder`</mark><mark style="color:purple;">)</mark>

Custom Android driver built into the Linux kernel — not part of upstream Linux. Handles \~99% of all IPC on Android. The core mechanism: instead of copying data between processes (which is expensive), Binder uses shared memory mapping. The sending process serializes its data into a **Parcel** — a binary byte stream — and writes it into a shared memory region. The kernel driver then maps that same region into the receiving process's address space. One copy, kernel-mediated, secure.

The kernel driver also handles identity verification — it stamps each transaction with the sender's UID, which the receiving process can query. This is how Android enforces permission checks across IPC: a Service can call `getCallingUid()` and decide whether to honor the request based on who's asking.

All high-level Java IPC abstractions (Intents, Bound Services, Content Providers) are wrappers around Binder. When an Activity starts a Service or an app queries a Content Provider, Binder is what actually moves the data underneath.

### <mark style="color:purple;">Standard Linux IPC</mark>

Inherited from Linux, available at the native layer. Used directly by C/C++ libraries, bypassing the Java framework entirely:

* **Unix domain sockets** — full-duplex stream or datagram communication between processes via filesystem socket files. More flexible than pipes — support bidirectional communication and can pass file descriptors between processes.
* **Pipes** — unidirectional byte stream, typically used between parent and child processes. Simple but limited — one direction only, no addressing.
* **Shared memory** — a memory region mapped into multiple processes simultaneously. Fastest IPC option since there's no kernel copy at all — processes read and write directly. Requires external synchronization (mutexes, semaphores) to avoid race conditions.

Relevant for malware analysis: malicious `.so` libraries commonly use Unix sockets or shared memory to communicate between processes without touching the Android framework, making the traffic invisible to tools that only monitor Java-level IPC.
