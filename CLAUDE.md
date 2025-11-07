# ln-capacitor-wifi - Project Documentation

## Project Overview

**ln-capacitor-wifi** is a Capacitor 7 plugin that provides programmatic WiFi connectivity management for iOS and Android mobile applications. It's specifically designed for IoT device connections where apps need to connect to specific WiFi networks programmatically.

- **Version**: 0.2.20
- **Author**: Marcelo Luz
- **License**: MIT
- **Repository**: https://github.com/Lindsor/capacitor-wifi.git
- **Capacitor**: 7.x (recently updated from 5.x)

## Technology Stack

### Core Technologies
- **Capacitor**: 7.x - Cross-platform native runtime
- **TypeScript**: 5.x - Type-safe plugin interface
- **Swift**: 5.1 - iOS native implementation
- **Java**: 21 - Android native implementation

### Build Tools
- **TypeScript Compiler**: Compiles TypeScript to ES2017
- **Rollup**: 5.x - Bundles to IIFE and CommonJS formats
- **Gradle**: 8.11.1 - Android builds
- **CocoaPods**: iOS dependency management

### Platform Requirements
- **Node.js**: 20+ (required by Capacitor 7)
- **iOS**: Deployment target 14.0+, Xcode 16.0+
- **Android**: minSdk 26, targetSdk 35, compileSdk 35, Java 21

## Project Structure

```
/
├── src/                          # TypeScript source
│   ├── index.ts                  # Plugin registration & exports
│   ├── definitions.ts            # TypeScript interfaces & types
│   └── web.ts                    # Web stub (unimplemented)
│
├── android/                      # Android native implementation
│   ├── build.gradle              # Android build config (SDK 35, Gradle 8.7.2)
│   ├── gradle/wrapper/           # Gradle 8.11.1
│   └── src/main/
│       ├── AndroidManifest.xml   # Required permissions
│       └── java/com/lindsor/capacitor/wifi/
│           ├── WifiPlugin.java   # Capacitor plugin entry point
│           ├── Wifi.java         # Core WiFi implementation (389 lines)
│           ├── WifiEntry.java    # WiFi network data model
│           ├── WifiError.java    # Error wrapper
│           ├── WifiErrorCode.java # Error codes enum
│           └── *Callback.java    # Async operation callbacks
│
├── ios/                          # iOS native implementation
│   ├── Podfile                   # iOS 14.0+ requirement
│   └── Plugin/
│       ├── WifiPlugin.swift      # Main plugin implementation
│       ├── WifiPlugin.m          # Objective-C bridge
│       └── Wifi.swift            # Helper utilities
│
├── dist/                         # Build output (generated)
│   ├── esm/                      # ES modules + type definitions
│   ├── plugin.js                 # IIFE browser bundle
│   └── plugin.cjs.js             # CommonJS bundle
│
├── package.json                  # NPM package config
├── tsconfig.json                 # TypeScript config (ES2017, skipLibCheck)
├── rollup.config.js              # Rollup bundler config
└── LnCapacitorWifi.podspec       # CocoaPods spec (iOS 14.0+)
```

## Architecture & Design Patterns

### Plugin Architecture
The plugin follows Capacitor's standard plugin pattern with clear separation:

1. **TypeScript Interface Layer** (src/)
   - Defines the API contract
   - Registers plugin with Capacitor
   - Provides type definitions

2. **Platform-Specific Native Implementations**
   - iOS: Swift implementation in ios/Plugin/
   - Android: Java implementation in android/src/main/java/

3. **Web Stub** (src/web.ts)
   - Throws `METHOD_UNIMPLEMENTED` errors
   - This is intentionally mobile-only

### Design Patterns Used

- **Bridge Pattern**: Objective-C bridges Swift to Capacitor runtime
- **Callback Pattern**: Android uses abstract callback classes for async operations
- **Repository Pattern**: Platform-specific implementations behind common interface
- **Error Code Enum**: Consistent error handling across platforms

### Code Organization

**Android Package**: `com.lindsor.capacitor.wifi`
- `WifiPlugin.java` - Capacitor plugin wrapper with @PluginMethod annotations
- `Wifi.java` - Core WiFi logic with Android API handling
- `WifiEntry.java` - Data model for WiFi network info
- `WifiError.java` / `WifiErrorCode.java` - Error handling
- `*Callback.java` - Async callback interfaces

**iOS Implementation**:
- `WifiPlugin.swift` - Main plugin extending CAPPlugin
- `WifiPlugin.m` - Objective-C bridge using CAP_PLUGIN macros
- Implements CLLocationManagerDelegate for permission handling

## API Capabilities

### Main Methods

1. **scanWifi()** - Scans for available WiFi networks (Android only)
2. **getCurrentWifi()** - Gets currently connected WiFi info
3. **connectToWifiBySsidAndPassword()** - Connects to specific SSID
4. **connectToWifiBySsidPrefixAndPassword()** - Connects to SSID matching prefix
5. **checkPermissions()** - Checks location/network permission status
6. **requestPermissions()** - Requests required permissions
7. **disconnectAndForget()** - Disconnects from current WiFi

### Key Types

```typescript
interface WifiEntry {
  bssid?: string;
  ssid?: string;
  level?: number;
  capabilities?: WifiCapability[];
  isCurrentWifi?: boolean;
}

enum WifiCapability {
  WPA2, RSN, WEP, WPA, WPS, ESS, IBSS
}

interface PermissionStatus {
  location: string;
  network: string;
}
```

## Platform-Specific Implementation Details

### Android Implementation (Wifi.java:389 lines)

**API Level Differences**:
- **Android Q+ (API 29+)**: Uses `WifiNetworkSpecifier` + `NetworkRequest`
- **Pre-Android Q**: Uses deprecated `WifiConfiguration` API

**Key Features**:
- `BroadcastReceiver` pattern for WiFi scan results
- Callback-based async operations
- Permission checks for location and network access
- Hidden SSID detection support
- WiFi capability parsing from scan results

**Required Permissions** (AndroidManifest.xml):
- `ACCESS_WIFI_STATE`
- `CHANGE_WIFI_STATE`
- `ACCESS_NETWORK_STATE`
- `ACCESS_COARSE_LOCATION`
- `ACCESS_FINE_LOCATION`
- `INTERNET`

### iOS Implementation (WifiPlugin.swift)

**Frameworks Used**:
- `SystemConfiguration.CaptiveNetwork` - WiFi information
- `NetworkExtension.NEHotspotConfiguration` - Programmatic connections
- `CoreLocation.CLLocationManager` - Location permissions

**Limitations**:
- iOS doesn't allow full network scanning (security restriction)
- Can only retrieve current WiFi network info
- Requires location permissions even just to read SSID
- Uses `joinOnce` flag for temporary connections

**iOS 14.0+ Requirement**: Uses NEHotspotConfiguration API

## Development Workflow

### Build Commands

```bash
# Build the plugin
npm run build
# Runs: clean → docgen → tsc → rollup

# Development watch mode
npm run watch

# Code quality
npm run lint      # ESLint + Prettier + SwiftLint
npm run fmt       # Auto-fix formatting

# Generate API documentation
npm run docgen    # Updates README.md and dist/docs.json

# Verify all platforms
npm run verify
# Runs: verify:ios && verify:android && verify:web
```

### Platform Verification

```bash
# iOS verification (requires Xcode 16.0+)
npm run verify:ios
# cd ios && pod install && xcodebuild

# Android verification (requires Android SDK)
npm run verify:android
# cd android && ./gradlew clean build test

# Web verification (just builds)
npm run verify:web
# npm run build
```

### Build Output Formats

- **ESM**: `dist/esm/` - ES modules with TypeScript declarations
- **CommonJS**: `dist/plugin.cjs.js` - CommonJS bundle
- **IIFE**: `dist/plugin.js` - Browser-compatible bundle
- **Docs**: `dist/docs.json` - API documentation in JSON format

## Recent Updates (Capacitor 7 Migration)

### What Changed

1. **Capacitor Dependencies**: Updated from 5.x to 7.x
   - @capacitor/core: ^7.0.0
   - @capacitor/android: ^7.0.0
   - @capacitor/ios: ^7.0.0
   - @capacitor/cli: ^7.0.0

2. **iOS Updates**:
   - Deployment target: 13.0 → 14.0
   - Updated Podfile platform requirement
   - Updated podspec deployment target

3. **Android Updates**:
   - Gradle plugin: 8.0.0 → 8.7.2
   - Gradle wrapper: 8.0.2 → 8.11.1
   - compileSdk: 33 → 35
   - targetSdk: 33 → 35
   - Java compatibility: 17 → 21

4. **Build Tools**:
   - TypeScript: ~4.1.5 → 5.x
   - Rollup: 2.32.0 → 5.x
   - Added `skipLibCheck: true` to tsconfig.json

### Breaking Changes (Capacitor 7)

- Node.js 20+ required (was 18+)
- iOS 14.0+ required (was 13.0+)
- Android SDK 35 required (was 33)
- Java 21 required for Android builds
- Xcode 16.0+ required for iOS builds

## Code Style & Conventions

### Naming Conventions
- **TypeScript**: camelCase for methods, PascalCase for types/interfaces
- **Java**: PascalCase for classes, camelCase for methods
- **Swift**: PascalCase for classes, camelCase for methods
- **Constants**: UPPER_SNAKE_CASE

### Code Quality Tools
- **ESLint**: @ionic/eslint-config
- **Prettier**: @ionic/prettier-config
- **SwiftLint**: @ionic/swiftlint-config

### Formatting
- **Java**: 4-space indentation
- **TypeScript**: Per Prettier config
- **Swift**: Per SwiftLint config

## Cross-Platform Differences

### Feature Parity

| Feature | Android | iOS |
|---------|---------|-----|
| Scan WiFi Networks | ✅ Full | ❌ Not allowed |
| Get Current WiFi | ✅ | ✅ |
| Connect to SSID | ✅ | ✅ |
| Connect by Prefix | ✅ | ✅ |
| Disconnect | ✅ | ✅ |
| Location Permissions | ✅ Required | ✅ Required |

### Permission Models

**Android**:
- Requires location permissions for WiFi scanning
- Requires network state permissions
- Different APIs for Android Q+ vs legacy

**iOS**:
- Requires location permissions just to read SSID
- Cannot enumerate available networks
- More restrictive due to iOS security model

## Common Development Tasks

### Adding a New Method

1. Add method signature to `WifiPlugin` interface in `src/definitions.ts`
2. Implement in `src/web.ts` (throw unimplemented error)
3. Implement in `android/src/main/java/.../WifiPlugin.java` with `@PluginMethod`
4. Implement in `ios/Plugin/WifiPlugin.swift` with `@objc` decorator
5. Add Objective-C bridge in `ios/Plugin/WifiPlugin.m` using `CAP_PLUGIN_METHOD`
6. Run `npm run docgen` to update documentation
7. Build and test on both platforms

### Testing Strategy

- **Android**: Unit tests in `android/src/test/`, instrumentation tests in `android/src/androidTest/`
- **iOS**: Unit tests in `ios/PluginTests/`
- **Web**: No tests (intentionally unimplemented)
- Test in consuming app for integration testing

### Publishing Workflow

1. Update version in `package.json`
2. Run `npm run build` to generate dist/
3. Run `npm run lint` to verify code quality
4. Run `npm run verify` to check platform builds
5. Commit changes
6. Tag release: `git tag v0.2.20`
7. Push: `git push && git push --tags`
8. Publish: `npm publish`

## Known Issues & Limitations

### Android
- Legacy WiFi configuration API deprecated in Android Q
- Location permission required even for basic WiFi operations
- Different behavior between API levels 28 and 29+

### iOS
- Cannot scan for available networks (iOS security restriction)
- Location permission required just to read WiFi SSID
- `joinOnce` connections are temporary

### General
- Web platform intentionally not supported
- Requires physical device testing (simulators have limited WiFi)
- Some features require specific device capabilities

## Troubleshooting

### Build Failures

**TypeScript errors**:
- Check Node.js version (must be 20+)
- Run `npm install` to update dependencies
- Verify `skipLibCheck: true` in tsconfig.json

**Android build fails**:
- Ensure ANDROID_HOME is set
- Check Java version (must be 21)
- Update Gradle wrapper: `cd android && ./gradlew wrapper --gradle-version 8.11.1`

**iOS build fails**:
- Run `cd ios && pod install`
- Check Xcode version (must be 16.0+)
- Clean build folder in Xcode

### Runtime Issues

**Permissions not working**:
- Check AndroidManifest.xml includes all required permissions
- Verify Info.plist includes location usage descriptions
- Call `checkPermissions()` before `requestPermissions()`

**WiFi connection fails**:
- Verify SSID and password are correct
- Check platform-specific API level requirements
- Android Q+ requires different approach than legacy

## Git Workflow

**Current Branch**: develop
**Main Branch**: develop

**Recent Commits**:
- 97d84f2 - Fix iOS Crash Report
- 440c7f2 - Fix NullPointerException
- e266e98 - Fix Android Connection
- 661d8fa - Fix parse error
- 4bdf2aa - Add iOS Error handling Prefix connect

## Additional Resources

- **Capacitor Documentation**: https://capacitorjs.com/docs
- **Capacitor 7 Migration Guide**: https://capacitorjs.com/docs/updating/7-0
- **Android WiFi APIs**: https://developer.android.com/reference/android/net/wifi/WifiManager
- **iOS Network Extension**: https://developer.apple.com/documentation/networkextension

## Notes for Contributors

- Follow existing code patterns and conventions
- Test on both iOS and Android devices
- Update documentation when adding features
- Run linter before committing
- Write descriptive commit messages
- Focus on "why" in commit messages, not "what"
- Android and iOS implementations may differ significantly due to platform APIs
- Always check both API level compatibility (Android) and iOS version requirements

## Project Maintenance

- Keep Capacitor dependencies in sync across all packages
- Update iOS deployment target and Android SDK versions together
- Run verification builds regularly
- Monitor for deprecated API warnings
- Update documentation when APIs change
- Follow semantic versioning for releases
