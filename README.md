# Spalla SDK

Spalla SDK Android provides a video player integrated with Spalla services. This document will show you how to add and use the SDK in your Android application.

The SDK offers the following features:
- View player
- Full screen player
- Analytics

## Getting started on Spalla SDK Android

The current version of the Spalla SDK for Android is 1.0.0, which requires the Android 23 API.
With the SDK repository downloaded, extract the contents of the .zip in a folder and modify the build.gradle files to fetch the SDK.

In your top-level (project) build.gradle or settings.gradle file, add the following repositories:
```
repositories {
   google()
   mavenCentral()
   maven { url 'path_to_repository' }
   maven { url 'https://jitpack.io' }
}
```

> Note: the "path_to_repository" references the path to the contents extracted from the .zip.


Next, edit the build.gradle on the module-level (app/build.gradle) adding the dependency:

```
dependencies {
   implementation 'com.spalla.sdk:spalla-android-sdk:x.x.x'
}
```

## Initialize Spalla SDK Android

Add the internet permission in the AndroidManifest.xml file.

```
<uses-permission android:name="android.permission.INTERNET"/>
```

Initialize SDK in Application.onCreate method with the context, Spalla token and THEO license.
```
class AppApplication : Application() {
   override fun onCreate() {
       super.onCreate()

       val token = "XXXXX..."
       val license = "YYYYY..."

       SpallaSDK.initialize(this, token, license)
   }
}
```

Add the Google Cast App Id as meta-data in the AndroidManifest.xml file.

```
<meta-data
   android:name="com.spalla.sdk.CAST_ID"
   android:value="XXXXXXXX"/>
```

## Using player view

The player view called “SpallaPlayerView” can be added to an application either through a) a layout (i.e. XML) or b) programmatically through the constructor API.

Using the `layout.xml`:

```
<com.spalla.sdk.android.core.player.view.SpallaPlayerView
   android:id="@+id/spallaPlayerView"
   android:layout_width="match_parent"
   android:layout_height="300dp"/>
```

Using the constructor API: see the documentation.

Sync the `SpallaPlayerView` with the Activity lifecycle changes calling the onResume, onPause and onDestroy methods:

```
override fun onResume() {
   super.onResume()
   spallaPlayerView.onResume()
}

override fun onPause() {
   super.onPause()
   spallaPlayerView.onPause()
}

override fun onDestroy() {
   super.onDestroy()
   spallaPlayerView.onDestroy()
}
```

Call the SpallaPlayerView.load method to load and initialize the player:
```
spallaPlayerView.load(spallaContentId, isLiveContent, autoPlay)
```

## Using picture-in-picture in player view

The picture-in-picture feature is only supported for Android 8.0+ (min API level 26).

Follow the steps below for picture-in-picture to work properly:

In the activity that contains the SpallaPlayerView add the supportsPictureInPicture and configChanges in the `AndroidManifest.xml` file:

```
<activity
   android:name=".presentation.PlayerActivity"
   android:exported="false"
   android:supportsPictureInPicture="true"
   android:configChanges=
       "screenSize|smallestScreenSize|screenLayout|orientation"/>
```

Call the SpallaPlayerView.enterPictureInPictureMode method to enter picture-in-picture mode. In the example below we call it in Activity.onUserLeaveHint, if the user presses the home or recents button while the app is open, the picture-in-picture will be activated.

```
override fun onUserLeaveHint() {
   super.onUserLeaveHint()
   spallaPlayerView.enterPictureInPictureMode(this)
}
```

Override the Activity.onPictureInPictureModeChanged method and call SpallaPlayerView.onPictureInPictureModeChanged. Also handle UI changes when entering and exiting picture-in-picture mode.

```
@RequiresApi(Build.VERSION_CODES.O)
override fun onPictureInPictureModeChanged(
   isInPictureInPictureMode: Boolean,
   newConfig: Configuration
) {
   super.onPictureInPictureModeChanged(
       isInPictureInPictureMode,
       newConfig
   )
   spallaPlayerView.onPictureInPictureModeChanged(
       this,
       isInPictureInPictureMode
   )
   if (isInPictureInPictureMode) {
       // Hide the full-screen UI (toolbar, etc.)
       // while in PiP mode.
   } else {
       // Restore the full-screen UI.
   }
}
```

## Using full screen player activity

To display the player in a separate full screen, use the `SpallaFullScreenPlayerActivity.Builder` class as shown below:

```
SpallaFullScreenPlayerActivity
   .Builder(
       contentId = spallaContentId,
       isLiveContent = isLiveContent,
       autoPlay = autoPlay
   )
   .build()
   .show(this)
```

## Customization

It is possible to customize the player's colors through a .css file in the assets folder.

In this example the style.css file contains:

```
.theo-primary-color, .vjs-selected {
 color: #cc0000 !important;
}

.theo-primary-background {
 color: #000000 !important;
 background-color: #cc0000 !important;
}

.theo-secondary-color {
 color: #ffffff !important;
}

.theo-secondary-background {
 color: #000000 !important;
 background-color: #ffffff !important;
}

.theo-tertiary-color {
 color: #000000 !important;
}

.theo-tertiary-background {
 color: #ffffff !important;
 background-color: #000000 !important;
}
```

The color relationship follows the table below:
| Color | Affected |
| --- | --- |
| Primary color | Big play button, play progress on seek bar |
| Primary background | Menu header background |
| Secondary color | Control bar icons, time display |
| Secondary background | Close button social sharing |
| Tertiary color | Control bar background |
| Tertiary background | Menu content background |

## Add player customization

SpallaPlayerView: add the cssStyle attribute value.

```
<com.spalla.sdk.android.core.player.view.SpallaPlayerView
   android:id="@+id/spallaPlayerView"
   android:layout_width="match_parent"
   android:layout_height="300dp"
   app:cssStyle="style.css"/>
```

SpallaFullScreenPlayerActivity.Builder: add the cssStyle method.

```
SpallaFullScreenPlayerActivity
   .Builder(
       contentId = spallaContentId,
       isLiveContent = isLiveContent,
       autoPlay = autoPlay
   )
   .cssStyle("style.css")
   .build()
   .show(this)
 ```
 
### Back button icon

Add the SpallaFullScreenPlayerActivity.Builder.backButtonDrawable method to change the back button image.

```
SpallaFullScreenPlayerActivity
   .Builder(
       contentId = spallaContentId,
       isLiveContent = isLiveContent,
       autoPlay = autoPlay
   )
   .cssStyle("style.css")
   .backButtonDrawable(R.drawable.ic_arrow_back)
   .build()
   .show(this)
```

## Analytics

A session identifier is required to collect video playback data. Use the token obtained from the login response header ("x-spalla-ssid") in the SpallaSDK.setSessionId method.

```
SpallaSDK.setSessionId(token)
```

## All Classes


### SpallaSDK

Type: public object.

The entry point of Spalla SDK.

##### Method Summary
| Return Type | Method | Description |
| --- | --- | --- |
| void | initialize(context: Context, token: String, license: String) | Initializes the SDK. |
| String | getToken() | Returns the Spalla token. |
| String | getLicence() | Returns the Spalla license. |
| void | setSessionId(sessionId: String?) | Adds a session identifier for analytics. |
| String? | getSessionId() | Returns the session identifier. |

### SpallaPlayerView
Type: public final class.

A media player interface defining high-level functionality, such as the ability to play, pause, seek and query properties of the currently playing media.

##### Constructor
SpallaPlayerView(context: Context, attrs: AttributeSet?, defStyleAttr: Int, cssStyle: String?, isFullScreen: Boolean, isFullScreenOrientationCoupled: Boolean)

| Parameters | Description | Default value |
| --- | --- | --- |
| context | Android context. | - |
| attrs | An optional AttributeSet instance. | null |
| defStyleAttr |  An optional style. | null |
| cssStyle | An optional player style. | null |
| isFullScreen | Whether the player is in fullscreen mode or not. | false |
| isFullScreenOrientationCoupled | Whether the orientation of the device and the fullscreen state are coupled. When this option is set to true, the player will go fullscreen when the device is rotated to landscape and will also exit fullscreen when the device is rotated back to portrait. Note that this has no relation to the orientation in which the player will be in fullscreen. | true |

##### Method Summary
| Modifier and Type | Method | Description |
| --- | --- | --- |
| void | registerPlayerListener(listener: SpallaPlayerListener) | Register a player listener. |
| void | registerFullScreenListener(listener: SpallaPlayerFullScreenListener) | Register a full screen listener. |
| void | registerCastListener(listener: SpallaPlayerCastListener) | Register a cast listener. |
| void | onResume() | Resumes the SpallaPlayerView. |
| void | onPause() | Pauses the SpallaPlayerView. |
| void | onDestroy() | Destroys the SpallaPlayerView. |
| Boolean | isDestroyed() | Returns whether onDestroy has been called. |
| void | load(id: String, isLiveContent: Boolean, autoPlay: Boolean) | Prepares the SpallaPlayerView. |
| void | play() | Starts or resumes playback. |
| void | pause() | Pauses playback. |
| void | stop() | Stops playback. |
| Double | getDuration() | Returns the duration of the media, in seconds. |
| Double | getCurrentTime() | Returns the current playback position of the media, in seconds. |
| Boolean | enterPictureInPictureMode(activity: Activity) | Returns whether picture-in-picture has been entered successfully. |
| void | onPictureInPictureModeChanged(activity: Activity, isInPictureInPictureMode: Boolean) | Notifies the SpallaPlayerView about picture-in-picture changes. |

### SpallaFullScreenPlayerActivity

Type: public final class.

A full screen player activity. Instantiate with `SpallaFullScreenPlayerActivity.Builder`.

##### Method Summary

| Modifier and Type | Method | Description |
| --- | --- | --- |
| void | show(context: Context) | Shows the activity. |

### SpallaFullScreenPlayerActivity.Builder

Type: public final class.

Fluent API for creating SpallaFullScreenPlayerActivity instances.

##### Constructor

SpallaFullScreenPlayerActivity.Builder(contentId: String, isLiveContent: Boolean, autoPlay: Boolean)

| Parameters | Description | Default value |
| --- | --- | --- |
| contentId | The Spalla content id. | - |
| isLiveContent | Whether the content is live. | - |
| autoPlay | Whether the player should immediately start playback. | - |

##### Method Summary

| Modifier and Type | Method | Description |
| --- | --- | --- |
| SpallaFullScreenPlayerActivity.Builder | cssStyle(cssStyle: String) | Overrides the player style. |
| SpallaFullScreenPlayerActivity.Builder | backButtonDrawable(backButtonDrawable: Int) | Overrides the back button icon. |
| SpallaFullScreenPlayerActivity | build() | Creates the SpallaFullScreenPlayerActivity instance. |

##### SpallaPlayerListener

Type: public interface.

The `SpallaPlayrListener` which can be used to listen for events sent from the player.
This listener can be added on the [`SpallaPlayerView`](#spallaplayerview) by using the `registerPlayerListener` method.

##### Method Summary

| Modifier and Type | Method | Description |
| --- | --- | --- |
| abstract void | onEvent(event: SpallaPlayerEvent) | Called when an event is received from the player. |

### SpallaPlayerFullScreenListener

Type: public interface.

The `SpallaPlayerFullScreenListener` which can be used to listen for changes in full screen mode.
This listener can be added on the SpallaPlayerView by using the registerFullScreenListener method.

##### Method Summary

| Modifier and Type | Method | Description |
| --- | --- | --- |
| abstract void | onEnterFullScreen() | Called when entering full screen. |
| abstract void | onExitFullScreen() | Called when exiting fullscreen. |

### SpallaPlayerCastListener

Type: public interface.

The SpallaPlayerCastListener which can be used to listen for Cast changes.
This listener can be added on the SpallaPlayerView by using the registerCastListener method.

##### Method Summary

| Modifier and Type | Method | Description |
| --- | --- | --- |
| abstract void | onCastChanged(casting: Boolean, castName: String?) | Called when the Cast changes. casting parameter: true if casting to a Chromecast. castName parameter: the name of the Chromecast device that the player is casting to, if any. |

### SpallaPlayerEvent

Type: public sealed class.

The player events. This class is used in the SpallaPlayerListener interface through the onEvent method.

##### Method Summary

| Modifier and Type | Method | Description |
| --- | --- | --- |
| String | toString() | Returns the event in String format |

### SpallaPlayerEvent.Play

Type: public object.

Fired when play() has returned or when autoplay has caused playback to begin

### SpallaPlayerEvent.Playing

Type: public object.

Fired when playback is ready to start after having been paused or delayed due to lack of media data.
Even if this event fires, the player might still not be potentially playing, e.g. if the player is paused for user interaction or paused for in-band content.

### SpallaPlayerEvent.Pause

Type: public object.

Fired when the player is paused.

### SpallaPlayerEvent.Waiting
Type: public object.

Fired when playback has stopped because the next frame is not available, but the user agent expects that frame to become available in due course.

### SpallaPlayerEvent.Ended

Type: public object.

Fired when current time has reached the end of the current source.

### SpallaPlayerEvent.DurationUpdate

Type: public final class.

Fired when the duration is updated, is when you set a new source.

##### Constructor

SpallaPlayerEvent.DurationUpdate(duration: Double)

| Parameters | Description |
| --- | --- |
| duration | The player's new duration in seconds. |

### SpallaPlayerEvent.TimeUpdate 

Type: public final class.

Fired when the current playback position changed as part of normal playback or is some special cases, like discontinuity.

##### Constructor

SpallaPlayerEvent.TimeUpdate(currentTime: Double)

| Parameters | Description |
| --- | --- |
| currentTime | The player's current time in seconds. |

### SpallaPlayerEvent.Error

Type: public final class.

Fired when an error occurs.

##### Constructor

SpallaPlayerEvent.Error(message: String, retryAvailable: Boolean)

| Parameters | Description | 
| --- | --- |
| message | The error's message. |
| retryAvailable | Whether possible to retry. |

