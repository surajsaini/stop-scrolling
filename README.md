# Scroll Pause

A native Kotlin / Jetpack Compose Android app that gives you a gentle, customizable reminder during extended Instagram use. Android 8.0 (API 26) or newer. No account or network permission.

## Install the ready-made APK — no Android Studio needed

1. Copy `Scroll-Pause-1.0.0.apk` from the delivered `dist` folder to your Android phone (USB, your own cloud drive, or another file-transfer method).
2. Open the APK in the phone's Files app. If Android asks, allow **Install unknown apps** for the app opening the APK, then tap **Install**. You can turn that permission off afterward.
3. Open **Scroll Pause → Permissions**. Grant **Usage access** and **Display over other apps**. On some phones the permission screen lists all apps; select Scroll Pause.
4. Allow notifications to keep the monitoring controls visible and receive the invitation to resume after reboot.
5. Return to **Home → Start monitoring**. Open Instagram normally. The default first reminder is after 7 minutes, with repeats every 5 minutes and a session reset after a 60-second break.

This is a locally signed, standalone APK. It does not require Android Studio, an emulator, Google Play, or a developer account to install. The source project's debug APK is also installable, but uses a different signing certificate: use the delivered release APK for normal installation.

## Quick phone test

1. In **Settings**, set the first reminder to **10 sec**, reset after **15 sec**, repeat every **10 sec**, and save.
2. In **Customize**, choose a local GIF or photo, enter a message, adjust the appearance, and use **Test this design on screen**. Tap × to dismiss; save when satisfied.
3. Start monitoring, open Instagram for 10 seconds, and check that the card slides in from the right. Swipe outside the card to confirm normal Instagram scrolling. Only the card's small rectangle receives its own touch events.
4. Leave Instagram for 5 seconds and return: active time should resume. Leave for at least 15 seconds and return: the next session should start at zero. Locking the phone also pauses the timer and counts toward the reset break.
5. Rotate the phone, switch between Feed and Reels, and confirm that reminders do not duplicate. Repeats wait until the current card is dismissed or times out.
6. Stop monitoring using Home or the notification. Revoke a required permission and confirm monitoring stops cleanly. Restart the phone and use the resume notification (or open the app) to start again.
7. Choose your normal time limits after testing.

## Features

- Home: monitor toggle, session timer, selected threshold, overlay test, current status.
- Customize: local PNG/JPG/WebP/GIF import, animated preview, message with `{time}` substitution, text size/color, card width, vertical position, animation duration, visibility duration, save and overlay test.
- Settings: thresholds from 5 seconds to 24 hours, reset breaks from 0 to 60 minutes, optional repeats from 5 seconds to 24 hours, floating reminder toggle, explicit Instagram / Instagram Lite selection.
- Permissions: usage access, overlay permission, notifications, battery guidance, privacy and platform limitations.
- One shared overlay controller prevents duplicate windows. The timer lives in a foreground service, independent of screen rotation. Rotating dismisses a visible card; the next reminder is positioned for the new screen bounds.
- Animated images are decoded at a small target size and stopped/released when the card is removed. Files are limited to 20 MB and 4096 pixels per side to bound memory use.

## How monitoring works

`UsageStatsManager.queryEvents()` supplies foreground activity transitions. Only exact enabled package identifiers are matched: `com.instagram.android` and optionally `com.instagram.lite`. The app transiently processes Android's foreground events to distinguish entering/leaving a selected app; it does not record a history of other apps.

A user-started foreground service polls approximately once per second while the screen is on. It uses a monotonic timer, subtracts away time, resets after the selected break, and schedules repeats from actual reminder delivery. A paused activity from an older Instagram screen cannot clear a newer resumed activity. A brief overlap of event queries catches delayed events; duplicate events are discarded.

When the screen turns off or locks, counting pauses immediately and the automatic overlay is removed. No wake lock is held. Polling backs off while the screen is off. A long OS suspension or clock change resets the session conservatively instead of inventing active time. A service restart reconstructs current foreground state but starts timing fresh; it does not backfill earlier usage.

The overlay uses `TYPE_APPLICATION_OVERLAY` and a compact window with `FLAG_NOT_FOCUSABLE` and `FLAG_NOT_TOUCH_MODAL`. It does not take keyboard focus. Touches outside the card go to the underlying app. Touches within the card cannot simultaneously interact with Instagram. It closes when you leave the monitored app, tap ×, disable overlays, or the visibility timer expires.

Settings use private `SharedPreferences`; selected media is copied into internal app storage. No screenshots, accessibility service, message inspection, analytics, account, internet permission, or cloud backup are used. The only current session duration is in memory. Imported files abandoned without saving may remain privately stored until the next successful image save or app data clear; saved replacement images remove older copies.

## Android limitations

- This measures **foreground Instagram time**, not actual finger scrolling. Watching video, reading, composing, or simply leaving Instagram visible counts. Android Usage Access does not expose individual swipes.
- Device vendors may suspend or kill background services. Use the app's battery guidance if needed. Notification permission is recommended but not a prerequisite for starting a foreground service.
- Modern foreground-service rules require a visible notification and an appropriate service type. This app declares `specialUse` with an explanation. Google Play distribution would additionally require the relevant Play declarations/review; the delivered APK is for direct installation.
- After reboot or app update, a notification invites the user to resume. It does not silently restart monitoring. If notifications are denied, open the app manually. Force-stop, the Android active-app Stop control, or process death can end monitoring. A system restart of the service uses a fresh timer.
- Usage events may arrive late. Detection is generally within a polling interval, but exact timing is not guaranteed under power restrictions. Split-screen, picture-in-picture, work profiles, cloned apps, and nonstandard Instagram package IDs are not reliably supported.
- Apps and Android can suppress overlays on sensitive screens. Landscape / large accessibility fonts reduce the space available; a long message is limited to eight lines. Position and width are constrained to the current screen when the card is shown.
- Timing or monitored-app changes reset the current session; appearance changes apply to the next reminder. A repeat interval shorter than the card's display time waits until the card closes.

Platform references: [usage events](https://developer.android.com/reference/android/app/usage/UsageStatsManager#queryEvents(long,long)), [foreground service types](https://developer.android.com/develop/background-work/services/fgs/service-types#special-use), [overlay window behavior](https://developer.android.com/reference/android/view/WindowManager.LayoutParams#TYPE_APPLICATION_OVERLAY), [Android 15 background-start changes](https://developer.android.com/about/versions/15/behavior-changes-15#fgs-background-start), [Coil GIF support](https://coil-kt.github.io/coil/gifs/).

## Build from source without Android Studio

Install a JDK (17 or 21) and Android SDK command-line tools. Install `platforms;android-35` and `build-tools;35.0.0`, accept the SDK license, and point `ANDROID_HOME` or `local.properties` at the SDK.

Windows PowerShell:

```powershell
$env:JAVA_HOME = 'C:\path\to\jdk-21'
$env:ANDROID_HOME = 'C:\path\to\Android\Sdk'
.\gradlew.bat :app:assembleDebug :app:testDebugUnitTest :app:lintDebug
```

macOS / Linux:

```sh
chmod +x gradlew
./gradlew :app:assembleDebug :app:testDebugUnitTest :app:lintDebug
```

The debug APK is at `app/build/outputs/apk/debug/app-debug.apk`. It is automatically signed for testing. An Android device with USB debugging can optionally install it using `adb install -r <apk-path>`, but USB debugging is **not** needed when opening the delivered APK on your phone.

Build an unsigned optimized release using `./gradlew :app:assembleRelease`. Sign it with your own Android signing key via `apksigner`. This workspace keeps the generated delivery key in `.signing/`, excluded from the source ZIP and version control. Keep that local folder private and backed up if you want future APKs to update this installation without uninstalling. An APK signed with a different key cannot replace the installed app directly.

The pinned build uses Gradle 8.11.1, Android Gradle Plugin 8.9.2, Kotlin 2.1.20, Compose BOM 2025.04.01, and Coil 2.7.0. Compile/target SDK is 35; minimum SDK is 26. The complete project can also be opened directly in Android Studio.

## Verification and project layout

- `monitor/SessionTimer.kt`: pure state machine with threshold, break, reset, repeat, stale-event and delayed-poll tests.
- `monitor/ForegroundTracker.kt`: package/activity matching with transition-order tests.
- `monitor/MonitorService.kt`: foreground service, permissions, notification, screen state, lifecycle.
- `overlay/`: compact native overlay and shared preview card.
- `data/`: private settings and bounded local media import.
- `MainActivity.kt`: Compose home, customization, settings, and setup screens.

`dist/BUILD-REPORT.txt` records the checks actually run on the delivered APK. Compilation and automated checks do not substitute for testing on your phone, especially for vendor battery restrictions and Instagram's current behavior.
