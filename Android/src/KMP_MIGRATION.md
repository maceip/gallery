# Kotlin Multiplatform Migration Guide

This document describes the migration of the AI Edge Gallery Android app to Kotlin Multiplatform (KMP) with Compose Multiplatform support for Android and iOS.

## Project Structure

```
Android/src/
├── app/                      # Android-specific application code
├── shared/                   # KMP shared module
│   └── src/
│       ├── commonMain/       # Shared code for all platforms
│       ├── androidMain/      # Android-specific implementations
│       └── iosMain/          # iOS-specific implementations
├── iosApp/                   # iOS application (to be created)
├── build.gradle.kts          # Root build configuration
├── settings.gradle.kts       # Project structure configuration
└── gradle/
    └── libs.versions.toml    # Dependency version catalog
```

## Key Changes

### 1. Dependency Replacements

The migration replaces Google Maven dependencies with open-source GitHub alternatives where possible:

#### Replaced Dependencies

| Original (Google Maven) | Replacement (GitHub/Maven Central) | Notes |
|------------------------|-------------------------------------|-------|
| `com.google.android.gms:play-services-tflite-java` | `org.tensorflow:tensorflow-lite` | Direct TensorFlow Lite from GitHub |
| `com.google.android.gms:play-services-tflite-gpu` | `org.tensorflow:tensorflow-lite-gpu` | TFLite GPU delegate |
| `com.google.dagger:hilt-*` | `io.insert-koin:koin-*` | KMP-compatible DI framework |
| `com.google.code.gson:gson` | `org.jetbrains.kotlinx:kotlinx-serialization-json` | KMP-compatible serialization |

#### Platform-Specific Dependencies (Keep Both)

- **MediaPipe** (`tasks-text`, `tasks-genai`, `tasks-imagegen`) - Android only, needs iOS alternatives
- **CameraX** - Android only, use native camera APIs on iOS
- **Firebase Analytics** - Optional, has KMP alternatives available

### 2. New KMP Dependencies

#### Core KMP Libraries
- **Koin** (`4.0.1`) - Dependency injection for KMP
- **Ktor** (`3.0.3`) - Networking library for KMP
- **kotlinx-coroutines** (`1.9.0`) - Coroutines for all platforms
- **kotlinx-datetime** (`0.6.1`) - Date/time for KMP
- **Compose Multiplatform** (`1.7.1`) - UI framework

#### Platform-Specific Engines
- **Android**: `ktor-client-android`
- **iOS**: `ktor-client-darwin`

### 3. Build Configuration

#### Root `build.gradle.kts`
Added plugins:
- `kotlin-multiplatform`
- `android-library`
- `compose-compiler`

#### `shared/build.gradle.kts`
New shared module configuration with:
- Android target (JVM 11)
- iOS targets (x64, arm64, simulatorArm64)
- Compose Multiplatform dependencies
- Source sets: `commonMain`, `androidMain`, `iosMain`

### 4. Expect/Actual Pattern

The shared module uses Kotlin's expect/actual mechanism for platform-specific implementations:

**Example: Platform Information**

```kotlin
// commonMain/Platform.kt
expect class Platform() {
    val name: String
    val version: String
}

// androidMain/Platform.android.kt
actual class Platform {
    actual val name: String = "Android ${android.os.Build.VERSION.SDK_INT}"
    actual val version: String = android.os.Build.VERSION.RELEASE
}

// iosMain/Platform.ios.kt
actual class Platform {
    actual val name: String = UIDevice.currentDevice.systemName()
    actual val version: String = UIDevice.currentDevice.systemVersion
}
```

## Migration Strategy

### Phase 1: Foundation (Current)
- ✅ Create shared module structure
- ✅ Set up KMP build configuration
- ✅ Add Compose Multiplatform support
- ✅ Replace Google Maven dependencies with GitHub alternatives
- ✅ Create expect/actual for platform APIs
- ✅ Move common data models to shared module

### Phase 2: Code Migration (Next Steps)
1. Move UI components to shared Compose Multiplatform
2. Migrate ViewModels to shared module
3. Create platform-specific implementations for:
   - File system access
   - Camera APIs
   - ML/AI inference (MediaPipe alternatives for iOS)
4. Replace Hilt with Koin DI
5. Migrate navigation to KMP-compatible solution

### Phase 3: iOS Implementation
1. Create iOS app target
2. Implement iOS-specific UI adaptations
3. Set up iOS camera and permissions
4. Integrate iOS ML frameworks (Core ML, TensorFlow Lite iOS)
5. Test on iOS devices

### Phase 4: Testing & Optimization
1. Unit tests for shared code
2. Platform-specific integration tests
3. Performance optimization
4. Code review and refactoring

## Building the Project

### Android
```bash
cd Android/src
./gradlew :app:assembleDebug
```

### iOS (After iOS app setup)
```bash
cd Android/src
./gradlew :shared:linkDebugFrameworkIosArm64
```

Then open the iOS project in Xcode and build.

## Dependencies Reference

### GitHub Sources for Replaced Dependencies

1. **TensorFlow Lite**
   - Repository: https://github.com/tensorflow/tensorflow
   - Maven: `org.tensorflow:tensorflow-lite:2.16.1`

2. **Koin**
   - Repository: https://github.com/InsertKoinIO/koin
   - Maven: `io.insert-koin:koin-core:4.0.1`

3. **Ktor**
   - Repository: https://github.com/ktorio/ktor
   - Maven: `io.ktor:ktor-client-core:3.0.3`

4. **kotlinx.serialization**
   - Repository: https://github.com/Kotlin/kotlinx.serialization
   - Maven: `org.jetbrains.kotlinx:kotlinx-serialization-json:1.7.3`

### Platform-Specific Alternatives Needed

For iOS implementations, consider:
- **Camera**: AVFoundation (native iOS)
- **ML/AI**:
  - TensorFlow Lite Swift (https://github.com/tensorflow/tensorflow)
  - Core ML
  - ONNX Runtime (https://github.com/microsoft/onnxruntime)

## Known Issues and Considerations

1. **MediaPipe on iOS**: MediaPipe has limited iOS support. Consider:
   - Using TensorFlow Lite directly with custom models
   - Exploring Core ML conversions
   - Using ONNX Runtime for cross-platform ML

2. **Firebase**: Current Firebase dependency is Android-only. For KMP support:
   - Use Firebase KMP libraries (experimental)
   - Or use platform-specific analytics

3. **Camera APIs**: CameraX is Android-only. iOS needs AVFoundation implementation.

4. **Testing**: Set up platform-specific test targets for both Android and iOS.

## Resources

- [Kotlin Multiplatform Documentation](https://kotlinlang.org/docs/multiplatform.html)
- [Compose Multiplatform](https://www.jetbrains.com/lp/compose-multiplatform/)
- [Koin Documentation](https://insert-koin.io/)
- [Ktor Documentation](https://ktor.io/)
- [TensorFlow Lite](https://www.tensorflow.org/lite)

## Contributing

When adding new features:
1. Implement in `commonMain` when possible
2. Use expect/actual for platform-specific code
3. Keep platform implementations minimal
4. Document platform differences
5. Test on both Android and iOS

## License

Copyright 2025 Google LLC - Apache License 2.0
