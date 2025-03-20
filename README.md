# Sharing Cloud

A swift library that extends the [swift-sharing](https://github.com/pointfreeco/swift-sharing) library with support for iCloud key-value synchronization through NSUbiquitousKeyValueStore.

## Overview

SharingCloud provides a simple and elegant way to sync shared data across multiple Apple devices using iCloud's key-value store. The library implements the `SharedKey` protocol from the swift-sharing library, enabling you to use the `@Shared` property wrapper with iCloud as a persistence strategy.

## Features

- **Seamless iCloud Integration**: Automatically sync data between iOS, macOS, watchOS, and tvOS devices
- **Simple Property Wrapper Syntax**: Use the familiar `@Shared` syntax with the `.iCloudKV` storage strategy
- **Comprehensive Type Support**:
  - Primitives: Bool, Int, Double, String
  - Complex Types: URL, Data, Date
  - Optional variants of all supported types
  - RawRepresentable types using Int or String as raw values
- **Automatic Change Detection**: Observes iCloud changes and updates UI automatically
- **Entitlement Checking**: Checks for proper iCloud entitlements and triggers the warning by `reportIssue`
- **Key Length Validation**: Checks that the key is under the 64-byte limit and triggers the warning by `reportIssue`

## Installation

### Swift Package Manager

Add the following to your `Package.swift` file:

```swift
dependencies: [
    .package(url: "https://github.com/KeithBird/sharing-cloud", from: "1.0.0")
]
```

Then add the product to any target that needs it:

```swift
.product(name: "SharingCloud", package: "sharing-cloud")
```

## Usage

### Basic Example

```swift
import SharingCloud

// In an ObservableObject or @Observable class
@Observable
class SettingsModel {
    @ObservationIgnored
    @Shared(.iCloudKV("userPreferences")) 
    var isDarkModeEnabled: Bool = false
    
    @ObservationIgnored
    @Shared(.iCloudKV("notificationsEnabled")) 
    var notificationsEnabled: Bool = true
    
    @ObservationIgnored
    @Shared(.iCloudKV("fontSize")) 
    var fontSize: Int = 14
}

// In a SwiftUI view
struct SettingsView: View {
    @Shared(.iCloudKV("userPreferences")) private var isDarkModeEnabled: Bool = false
    @Shared(.iCloudKV("notificationsEnabled")) private var notificationsEnabled: Bool = true
    @Shared(.iCloudKV("fontSize")) private var fontSize: Int = 14
    
    var body: some View {
        Form {
            Toggle("Dark Mode", isOn: $isDarkModeEnabled)
            Toggle("Notifications", isOn: $notificationsEnabled)
            Stepper("Font Size: \(fontSize)", value: $fontSize, in: 10...30)
        }
    }
}
```

### Using with RawRepresentable Types

```swift
enum Theme: String, Sendable {
    case light
    case dark
    case system
}

struct ThemeSettingsView: View {
    @Shared(.iCloudKV("appTheme")) private var theme: Theme = .system
    
    var body: some View {
        Picker("Theme", selection: $theme) {
            Text("Light").tag(Theme.light)
            Text("Dark").tag(Theme.dark)
            Text("System").tag(Theme.system)
        }
    }
}
```

### Storing Complex Types

Storing a URL example:

```swift
@Shared(.iCloudKV("lastVisitedURL")) var lastURL: URL?
```

### Other Shared Use Casess

More examples can be found in the [swift-sharing documentation](https://swiftpackageindex.com/pointfreeco/swift-sharing/main/documentation/sharing/).

## Setting Up iCloud in Your App

1. In Xcode, select your target and navigate to "Signing & Capabilities"
2. Add the iCloud capability
3. Check "Key-value storage"
4. Ensure your app has the proper entitlements:
   - `com.apple.developer.ubiquity-kvstore-identifier`
5. Replace `$(CFBundleIdenfier)` with your key-value store ID (i.e. `$(TeamIdentifierPrefix)<your key-value_store ID>`).

For more information, please see: [Synchronizing App Preferences with iCloud](https://developer.apple.com/documentation/foundation/icloud/synchronizing_app_preferences_with_icloud)

## Limitations

- NSUbiquitousKeyValueStore has a maximum total storage size of 1MB per user
- Each key-value pair is limited to 1MB in size
- Maximum of 1024 keys allowed
- Keys can't be larger than 64 bytes using UTF-8 encoding
- Synchronization is not immediate; iCloud syncs on its own schedule

## Best Practices

- Use short, descriptive key names to stay under the 64-byte limit
- Store only small pieces of data like preferences and settings
- Do not store sensitive user information in iCloud key-value store
- Check for iCloud availability before assuming data will be synced

## License

This library is available under the MIT license.
