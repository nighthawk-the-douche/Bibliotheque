---
icon: window
---

# Android Applications

## <mark style="color:$primary;">ABOUT</mark>

## <mark style="color:blue;">Project Structure</mark>

#### app/

| File                                            | Description                                                                                   |
| ----------------------------------------------- | --------------------------------------------------------------------------------------------- |
| <mark style="color:$success;">`manifest`</mark> | Package name, components (activities, services, etc.), permissions, network config, API level |
| <mark style="color:$success;">`java`</mark>     | Application Java source code                                                                  |
| <mark style="color:$success;">`res`</mark>      | UI strings, images, layout XMLs, static assets                                                |

#### Gradle Scripts

| File                                                      | Description                                               |
| --------------------------------------------------------- | --------------------------------------------------------- |
| <mark style="color:$success;">`build.gradle`</mark>       | Build config — dependencies, build types, ProGuard toggle |
| <mark style="color:$success;">`proguard-rules.pro`</mark> | Custom ProGuard rules                                     |

### <mark style="color:blue;">activity\_main.xml</mark>

| Attribute                                                            | Description                                                                                                            |
| -------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| <mark style="color:$success;">`tools:context=".MainActivity"`</mark> | Binds layout to Activity in editor preview only. No runtime effect.                                                    |
| <mark style="color:$success;">`android:id`</mark>                    | Unique identifier for referencing in Java. `@+id` creates new resource; `@id` references existing.                     |
| <mark style="color:$success;">`android:text`</mark>                  | Text content of TextView/Button. Hardcoded or referenced from `res/values/strings.xml` (recommended for localization). |

HTML/JS files → `app/assets` (`app/src/main/assets` in Project view).

## <mark style="color:$primary;">APPLICATION COMPONENTS</mark>

Building blocks that define different parts of an Android app — UI, background logic, data access, and event handling. All components must be declared in `AndroidManifest.xml` to be recognized by the system. They can be used individually or in combination.

Components communicate with each other — both within the same app and across different apps — via **IPC** (Interprocess Communication). This is what allows, for example, one app to trigger an action in another, or separate processes within the same app to exchange data.

Main components: Activities, Services, Broadcast Receivers, Content Providers.

### <mark style="color:$primary;">Activities</mark>

Single screen with a UI. Can appear as full-screen, floating, embedded, or multi-window.

#### Lifecycle Callbacks

| Callback                                           | What happens                                                                                                                                                                                                    |
| -------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| <mark style="color:$success;">`onCreate()`</mark>  | Activity created. UI setup, data binding, listener config. **Key pentest entry point** — hard-coded credentials/keys often initialized here; Intent parameters passed between activities are also handled here. |
| <mark style="color:$success;">`onStart()`</mark>   | Activity becomes visible to user. Resource initialization.                                                                                                                                                      |
| <mark style="color:$success;">`onResume()`</mark>  | Activity interacting with user. Animations, media, input handling. Always followed by `onPause()`.                                                                                                              |
| <mark style="color:$success;">`onPause()`</mark>   | Another app or dialog is on top. Activity still visible, non-needed resources released. Followed by `onResume()` or `onStop()`.                                                                                 |
| <mark style="color:$success;">`onStop()`</mark>    | Activity no longer visible. Resources released. Followed by `onRestart()` or `onDestroy()`.                                                                                                                     |
| <mark style="color:$success;">`onDestroy()`</mark> | Activity destroyed — system needs memory or user closed it.                                                                                                                                                     |
| <mark style="color:$success;">`onRestart()`</mark> | Called when activity restarts after `onStop()`. Followed by `onStart()`.                                                                                                                                        |

Android maintains an **Activity stack** per task. New Activity → pushed on top → becomes active. Back navigation → popped from stack.

#### Declaring Activities

Must be declared in `AndroidManifest.xml` as child of `<application>`:

```xml
<activity android:name=".MainActivity">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```

`android.intent.action.MAIN` marks the entry point of the app — first Activity launched on start. Entry point identification is a priority during pentesting (maps attack surface and app flow).

#### Intents

Messaging objects used to request an action from another component (same or other app).

## <mark style="color:$primary;">SERVICES</mark>

An Android component that performs long-running operations in the background without a user interface. The key point is that a Service continues running even after the user has left the app — it is not tied to the Activity lifecycle. Used for tasks like downloading files, playing music, or maintaining a connection to a remote server.

There are three types:

| Type           | Description                                                                                                                                                                                                                               |
| -------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Foreground** | Requires user attention. Must show a persistent notification so the user is aware it's running. Continues even when app is minimized. Started with `startService()`. Examples: media players, navigation apps.                            |
| **Background** | Performs work that doesn't need user awareness. Android 8.0+ (API 26): restricted from running unless the app is in the foreground — introduced to conserve battery and system resources.                                                 |
| **Bound**      | Other components (even from different processes) bind to it via `bindService()`. Provides a client-server interface for IPC — client can invoke methods and receive results. Connection lasts as long as at least one component is bound. |

```java
public class ExampleService extends Service {
    int startMode;       // behavior if service is killed
    IBinder binder;      // interface for bound clients
    boolean allowRebind; // whether onRebind should be used
}
```

### <mark style="color:blue;">Broadcast Receivers</mark>

Serve two roles simultaneously: an Application Component and an IPC mechanism.

As an **IPC mechanism** — enable communication between different applications by sending and receiving Intents. These Intents can come from the Android system, other apps, or the app itself.

As an **Application Component** — designed to respond to system-wide or custom events. Act as a messaging system across the entire Android ecosystem. For example, the system broadcasts an event when the device starts charging, and any app with a registered receiver for that event will be notified.

Broadcast Receivers do not have a UI and are not long-running — `onReceive()` is called and expected to finish quickly.

```java
public class MyBroadcastReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        String action = intent.getAction();
        if (action != null) {
            switch (action) {
                case Intent.ACTION_POWER_CONNECTED:    break;
                case Intent.ACTION_POWER_DISCONNECTED: break;
                default: break;
            }
        }
    }
}
```

| Method                                                                      | Description                                            |
| --------------------------------------------------------------------------- | ------------------------------------------------------ |
| <mark style="color:$success;">`sendBroadcast(Intent)`</mark>                | Sends to all receivers simultaneously, undefined order |
| <mark style="color:$success;">`sendOrderedBroadcast(Intent, String)`</mark> | Sends to one receiver at a time, in priority order     |

### <mark style="color:blue;">Content Providers</mark>

Serve two roles simultaneously: an Application Component and an IPC mechanism.

As an **IPC mechanism** — enable communication between apps by allowing them to access, modify, or delete data through a consistent interface via the `ContentResolver` class.

As an **Application Component** — responsible for managing and exposing structured data, either within the same app or to external apps. Act as an intermediate layer between the app and its underlying data storage.

Data can be stored in: local SQLite databases, internal/external device storage, or a remote server. All interaction goes through a standardized **CRUD** (Create, Read, Update, Delete) API, regardless of the storage backend — this is what makes Content Providers a clean abstraction for data sharing across process boundaries.

## <mark style="color:$primary;">IPC — Application Level</mark>

High-level Java abstractions built on top of the Binder driver. They define the logic and format of interaction — the actual data transfer is delegated to Binder underneath.

### <mark style="color:purple;">Intents (Point-to-Point)</mark>

Directed message passing. A single, isolated data packet sent to activate a specific component (Activity or Service). Two types:

* **Explicit Intent** — targets a specific component by class name. Used within the same app.
* **Implicit Intent** — declares an action (e.g., `VIEW`, `SEND`) and lets the OS resolve which component handles it.

### <mark style="color:purple;">Deep Links (External Trigger)</mark>

URL-based routing mechanism (`scheme://...`) that causes the OS to generate an implicit Intent from an external source (browser, another app) and route it into the app. The app receives it like any other Intent.

Attack vectors: parameter manipulation, authentication bypass, XSS if the target is a WebView.

### <mark style="color:purple;">Broadcast Receivers (Publish / Subscribe)</mark>

Global or local event broadcasting. A process emits a message into the system (e.g., "screen unlocked", "battery low"), and all previously registered receivers intercept it. No direct targeting — any component registered for that action receives it.

* **Global broadcast** — system-wide, any app can receive
* **Local broadcast** (`LocalBroadcastManager`) — scoped to the same app, not visible to other apps

### <mark style="color:purple;">Content Providers (Data Export / CRUD)</mark>

Expose access to a local SQLite database or files to other apps through a standardized URI interface (`content://authority/path`). The requesting app interacts via `ContentResolver` — it never touches the storage directly.

All operations map to CRUD: `query()`, `insert()`, `update()`, `delete()`. Access can be permission-gated per URI path.

### <mark style="color:purple;">Bound Services / AIDL (Remote Procedure Call)</mark>

Persistent, bidirectional connection between processes. Process A binds to Process B's Service and can directly call its methods as if they were local — the AIDL (Android Interface Definition Language) interface defines the contract, Binder handles the transport.

Unlike Intents (fire-and-forget), a bound connection stays alive as long as at least one client is bound. The Service is destroyed when all clients unbind.
