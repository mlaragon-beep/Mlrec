# FrameLock Recorder — Native Android Gameplay Recorder for Mobile Legends: Bang Bang

Optimized for **Xiaomi Pad 6 (Snapdragon 870, Adreno 650, Android 14 / HyperOS)** with broad compatibility across Android 10–15 (API 29–34).

## Architecture Highlights
- **Zero-Copy Hardware Video Pipeline**: `MediaProjection` -> `VirtualDisplay` -> `MediaCodec.createInputSurface()` (`video/avc` or `video/hevc`). Zero CPU pixel readback.
- **Dual-Track Fault-Isolated Audio Engine**:
  - Track 1: `AudioPlaybackCaptureConfiguration` (`USAGE_GAME` & `USAGE_MEDIA`) prioritizing Mobile Legends (`com.mobile.legends`).
  - Track 2: Independent `AudioRecord` (`MIC`) encoded to a separate AAC track inside the MP4 or saved as a synchronized `.m4a` stem file.
  - **Automatic Fallback & Silent-Frame Detector**: Actively checks PCM RMS energy on internal audio. If Android or Mobile Legends blocks internal capture (`ALLOW_CAPTURE_BY_NONE`), FrameLock surfaces a non-blocking diagnostic banner and keeps recording video + microphone seamlessly.
- **Zero WebView Dependency**: 100% pure native Jetpack Compose + Android `WindowManager` overlay. Avoids Chromium `RenderThread` / `SurfaceTexture` deadlocks when Mobile Legends opens in-game event WebViews.
- **Xiaomi HyperOS / MIUI Safeguards**: Built-in detection for HyperOS background pop-up permissions (`OP_BACKGROUND_START_ACTIVITY`), floating window permission, battery optimization bypass, and Game Turbo whitelist guidance.
- **Adaptive Performance Governor**: Monitors `PowerManager.getCurrentThermalStatus()` and encoder queue pressure, stepping video bitrate dynamically via `MediaCodec.setParameters()` during intense 5v5 team fights so gameplay FPS stays locked.

---

## Building the Installable APK

### Option 1: Android Studio (Recommended)
1. Open the `android-project` folder in **Android Studio Jellyfish (2023.3.1)** or newer.
2. Allow Gradle Sync to download Android SDK Platform 34 and dependencies.
3. Connect your **Xiaomi Pad 6** via USB with **USB Debugging** and **USB Debugging (Security Settings)** enabled in Developer Options.
4. Click **Run > Run 'app'** or **Build > Build Bundle(s) / APK(s) > Build APK(s)**.
5. Output APK path:
   `app/build/outputs/apk/debug/app-debug.apk`

### Option 2: Command Line (Gradle)
Ensure JDK 17 and Android SDK (API 34) are installed (`ANDROID_HOME` set):
```bash
# If gradle wrapper jar isn't present locally, use installed gradle 8.7+:
gradle assembleDebug
# Or with wrapper:
./gradlew assembleDebug

# Install directly onto connected Xiaomi Pad 6:
adb install -r app/build/outputs/apk/debug/app-debug.apk
```

### Option 3: Zero-Local-Setup Cloud Build (GitHub Actions)
1. Push the extracted `android-project` folder to a GitHub repository.
2. Open the **Actions** tab and run **Build FrameLock Android APK**.
3. Download the compiled `FrameLock-MLBB-Recorder-debug.apk` artifact and sideload onto your Xiaomi Pad 6.

---

## Xiaomi Pad 6 (Android 14 / HyperOS) Setup Checklist
1. **Allow Display Over Other Apps**: Settings -> Apps -> Special app access -> Display over other apps -> **FrameLock** -> Allow.
2. **HyperOS Battery Saver**: Settings -> Apps -> Manage apps -> FrameLock -> Battery saver -> **No restrictions**.
3. **HyperOS Background Pop-ups**: App info -> Other permissions -> **Display pop-up windows while running in the background** -> Allow.
4. **Game Turbo Whitelist**: Security App -> Game Turbo -> Settings -> **Memory exceptions** -> Add **FrameLock** so HyperOS does not kill the recording service mid-match.

## Technical Note on Mobile Legends Internal Audio Capture
Android 10+ `AudioPlaybackCapture` only captures audio from apps that permit capture (`allowAudioPlaybackCapture="true"` or default targeting API >= 29). If Moonton sets `allowAudioPlaybackCapture="false"` in a specific game client build or routes voice chat to `MODE_IN_COMMUNICATION`, Android zeroes out the internal PCM stream at the AudioFlinger mixer level. FrameLock detects this within 1.5 seconds, notifies you in the UI, and automatically preserves your Microphone track without interrupting video capture.
