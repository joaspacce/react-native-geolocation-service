# react-native-geolocation-service
React native geolocation service for iOS and android.

# Why ?
This library is created in an attempt to fix the location timeout issue on android with the react-native's current implementation of Geolocation API. This library tries to solve the issue by using Google Play Service's new `FusedLocationProviderClient` API, which Google strongly recommends over android's default framework location API. It automatically decides which provider to use based on your request configuration and also prompts you to change the location mode if it doesn't satisfy your current request configuration.

> NOTE: Location request can still timeout since many android devices have GPS issue in the hardware level or in the system software level. Check the [FAQ](#faq) for more details.

# Installation
```bash
yarn add react-native-geolocation-service
```
or 
```bash
npm install react-native-geolocation-service
```

# Setup

## iOS
No additional setup is required, since it uses the React Native's default Geolocation API. Just follow the [React Native documentation](https://facebook.github.io/react-native/docs/geolocation.html#ios) to modify the `.plist` file.

## Android
1. In `android/app/build.gradle`

    ```gradle
    ...
    dependencies {
        ...
        implementation project(':react-native-geolocation-service')
    }
    ```

    If you've defined [project-wide properties](https://developer.android.com/studio/build/gradle-tips#configure-project-wide-properties) (recommended) in your root build.gradle, this library will detect the presence of the following properties:

    ```gradle
    buildscript {...}
    allprojects {...}

    /**
     + Project-wide Gradle configuration properties
     */
    ext {
        compileSdkVersion   = 26
        targetSdkVersion    = 26
        buildToolsVersion   = "27.0.3"
        googlePlayServicesVersion = "15.0.1"
    }
    ```

    If you do not have *project-wide properties* defined and have a different play-services version than the one included in this library, use the following instead. But play service version should be `11+` or the library won't work.

    ```gradle
    ...
    dependencies {
        ...
        implementation(project(':react-native-geolocation-service')) {
            exclude group: 'com.google.android.gms', module: 'play-services-location'
        }
        implementation 'com.google.android.gms:play-services-location:<insert your play service version here>'
    }
    ```

2. In `android/setting.gradle`

    ```gradle
    ...
    include ':react-native-geolocation-service'
    project(':react-native-geolocation-service').projectDir = new File(rootProject.projectDir, '../node_modules/react-native-geolocation-service/android')
    ```

3. In `MainApplication.java`

    ```java
    ...
    import com.agontuk.RNFusedLocation.RNFusedLocationPackage;

    public class MainApplication extends Application implements ReactApplication {
        ...
        @Override
        protected List<ReactPackage> getPackages() {
            return Arrays.<ReactPackage>asList(
                ...
                new RNFusedLocationPackage()
            );
        }
    }
    ```

# Usage
Since this library was meant to be a drop-in replacement for the RN's Geolocation API, the usage is pretty straight forward, with some extra error cases to handle.

> One thing to note, this library assumes that location permission is already granted by the user, so you have to use `PermissionsAndroid` to request for permission before making the location request.

```js
...
import Geolocation from 'react-native-geolocation-service';
...

componentDidMount() {
    // Instead of navigator.geolocation, just use Geolocation.
    if (hasLocationPermission) {
        Geolocation.getCurrentPosition(
            (position) => {
                console.log(position);
            },
            (error) => {
                // See error code charts below.    
                console.log(error.code, error.message);
                
            },
            { enableHighAccuracy: true, timeout: 15000, maximumAge: 10000 }
        );
    }
}
```

# API
#### `getCurrentPosition(successCallback, ?errorCallback, ?options)`
 - **successCallback**: Invoked with latest location info.
 - **errorCallback**: Invoked whenever an error is encountered.
 - **options**:

    | Name | Type | Default | Description |
    | -- | -- | -- | -- |
    | timeout | `ms` | -- | Request timeout |
    | maximumAge | `ms` | `INFINITY` | How long previous location will be cached |
    | enableHighAccuracy | `bool` | `false` | Use high accuracy mode
    | distanceFilter | `m` | `0` | Minimum displacement in meters
    | showLocationDialog | `bool` | `true` | whether to ask to enable location in Android

#### `watchPosition(successCallback, ?errorCallback, ?options)`
 - **successCallback**: Invoked with latest location info.
 - **errorCallback**: Invoked whenever an error is encountered.
 - **options**:

    | Name | Type | Default | Description |
    | -- | -- | -- | -- |
    | enableHighAccuracy | `bool` | `false` | Use high accuracy mode
    | distanceFilter | `m` | `100` | Minimum displacement between location updates in meters
    | interval | `ms` | `10000` |  Interval for active location updates
    | fastestInterval | `ms` | `5000` | Fastest rate at which your application will receive location updates, which might be faster than `interval` in some situations (for example, if other applications are triggering location updates)
    | showLocationDialog | `bool` | `true` | whether to ask to enable location in Android

#### `clearWatch(watchId)`
 - watchId (id returned by `watchPosition`)

Checkout [React Native documentation](https://facebook.github.io/react-native/docs/geolocation.html#reference) to see the list of available methods.

# Missing API for Android.
#### `getCurrentLocationSettings(successCallback, ?errorCallback)`
 - **successCallback**: Invoked with latest location settings info.
 - **errorCallback**: Invoked whenever an error is encountered.
 
 - settings  (current location settings on the device returned in `successCallback`)
 ```js          
      {
        "locationMode":"LOCATION_MODE_HIGH_ACCURACY",
        "locationEnabled": "true",
        "gpsProvider": {
            "name": "gps",
            "enabled": "true",
            "accuracy": "ACCURACY_FINE",
            "requiresCell": "false",
            "requiresNetwork": "false",
            "requiresSatellite": "false",
        },
        "networkProvider": {
            "name": "network",
            "enabled": "true",
            "accuracy": "ACCURACY_COARSE",
            "requiresCell": "false",   
            "requiresNetwork": "false",
            "requiresSatellite": "false"
        }
      }                
 ```

 - settings.locationMode (current location mode setted on the device, could be any of these values: "LOCATION_MODE_OFF", "LOCATION_MODE_SENSORS_ONLY", "LOCATION_MODE_BATTERY_SAVING", "LOCATION_MODE_HIGH_ACCURACY" or "LOCATION_MODE_UNDEFINED". Taken from [here](https://developer.android.com/reference/android/provider/Settings.Secure#LOCATION_MODE))

 - settings.[gpsProvider|networkProvider].accuracy (accuracy of respective location provider, could be any of these values: "ACCURACY_FINE" or "ACCURACY_COARSE". Taken from [here](https://developer.android.com/reference/android/location/LocationProvider.html#getAccuracy()))

# Error Codes
| Name | Code | Description |
| --- | --- | --- |
| PERMISSION_DENIED | 1 | Location permission is not granted |
| POSITION_UNAVAILABLE | 2 | Unable to determine position (not used yet) |
| TIMEOUT | 3 | Location request timed out |
| PLAY_SERVICE_NOT_AVAILABLE | 4 | Google play service is not installed or has an older version |
| SETTINGS_NOT_SATISFIED | 5 | Location service is not enabled or location mode is not appropriate for the current request |
| INTERNAL_ERROR | -1 | Library crashed for some reason or the `getCurrentActivity()` returned null |

# TODO
- [x] Implement `watchPosition` & `clearWatch` methods for android
- [x] Implement `stopObserving` method for android

# FAQ
1. **Location timeout still happening ?**

    Try the following steps: (Taken from [here](https://support.strava.com/hc/en-us/articles/216918967-Troubleshooting-GPS-Issues))
    - Turn phone off/on
    - Turn GPS off/on
    - Disable any battery saver settings, including Power Saving Mode, Battery Management or any third party apps
    - Perform an "AGPS reset": Install the App GPS Status & Toolbox, then in that app, go to Menu > Tools > Manage A-GPS State > Reset

    Adjusting battery saver settings on different devices:

    - HTC: Access your phone settings > battery > power saving mode > battery optimization > select your app > don't optimize > save
    - Huawei: Turn Energy Settings to Normal and add your app to "Protected Apps"
    - LG If you're running Android 6 or higher: Settings > battery & power saving > battery usage > ignore optimizations > turn ON for your app
    - Motorola If you're running Android 6 or higher: Battery > select the menu in the upper right-hand corner > battery optimization > not optimized > all apps > select your app > don't optimize
    - OnePlus (using OxygenOS Settings): Battery > battery optimization > switch to 'all apps' > select your app > don't optimize
    - Samsung: Access battery settings > app power saving > details > your app > disabled
    - Sony If you're running Android 6 or higher: Battery > from the menu in the upper right-hand corner > battery optimization > apps > your app
    - Xiomi (MIUI OS) If you're running Android 6 or higher: Access your phone settings > additional settings > battery and performance > manage battery usage > apps > your app
