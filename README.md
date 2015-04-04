# Table of contents

* [Android _PDF417.mobi_ integration instructions](#intro)
* [Quick Start](#quickStart)
  * [Quick start with demo app](#quickDemo)
  * [Quick integration of _PDF417.mobi_ into your app](#quickIntegration)
  * [Eclipse integration instructions](#eclipseIntegration)
  * [How to integrate _PDF417.mobi_ into your project using Maven](#mavenIntegration)
  * [Performing your first scan](#quickScan)
* [Advanced _PDF417.mobi_ integration instructions](#advancedIntegration)
  * [Checking if _PDF417.mobi_ is supported](#supportCheck)
  * [Customization of `Pdf417ScanActivity` activity](#scanActivityCustomization)
  * [Embedding `RecognizerView` into custom scan activity](#recognizerView)
  * [`RecognizerView` reference](#recognizerViewReference)
  * [Using direct API for recognition of android Bitmaps](#directAPI)
* [Recognition settings and results](#recognitionSettingsAndResults)
  * [Generic settings](#genericSettings)
  * [Scanning PDF417 barcodes](#pdf417Recognizer)
  * [Scanning US Driver's licence barcodes](#usdl)
  * [Scanning one dimensional barcodes with _PDF417.mobi_'s implementation](#custom1DBarDecoder)
  * [Scanning barcodes with ZXing implementation](#zxing)
* [Translation and localization](#translation)
* [Processor architecture considerations](#archConsider)
  * [Reducing the final size of your app](#reduceSize)
  * [Combining _PDF417.mobi_ with other native libraries](#combineNativeLibraries)
* [Troubleshooting](#troubleshoot)
* [Additional info](#info)

# <a name="intro"></a> Android _PDF417.mobi_ integration instructions

The package contains Android Archive (AAR) that contains everything you need to use _PDF417.mobi_ library. Besides AAR, package also contains a demo project that contains two modules:

 - pdf417MobiDemo module demonstrates quick and simple integration of _PDF417.mobi_ library
 - pdf417MobiDemoCustomUI demonstrates advanced integration within custom scan activity
 
_PDF417.mobi_ is supported on Android SDK version 8 (Android 2.2) or later.
 
The library contains one activity: `Pdf417ScanActivity`. It is responsible for camera control and barcode recognition. If you create your own scanning UI, you will need to embed `RecognizerView` into your activity and pass activity's lifecycle events to it and it will control the camera and recognition process.

# <a name="quickStart"></a> Quick Start

## <a name="quickDemo"></a> Quick start with demo app

1. Open Android Studio.
2. In Quick Start dialog choose _Import Non-Android Studio project_.
3. In File dialog select _Pdf417MobiDemo_ folder.
4. Wait for project to load. If Android studio asks you to reload project on startup, select `Yes`.

## <a name="quickIntegration"></a> Quick integration of _PDF417.mobi_ into your app

1. In Android Studio menu, click _File_, select _New_ and then select _Module_.
2. In new window, select _Import .JAR or .AAR Package_, and click _Next_.
3. In _File name_ field, enter the path to _LibRecognizer.aar_ and click _Finish_.
4. If you plan to use ProGuard, add following lines to your `proguard-rules.pro`:
	
	```
	-keep class com.microblink.** { *; }
	-keepclassmembers class com.microblink.** {
		*;
	}
	-dontwarn android.hardware.**
	-dontwarn android.support.v4.**
	```
	
## <a name="eclipseIntegration"></a> Eclipse integration instructions

We do not provide Eclipse integration demo apps. We encourage you to use Android Studio. However, if you still want to use Eclipse, you will need to convert AAR archive to Eclipse library project format. You can do this by doing the following:

1. In Eclipse, create a new _Android library project_ in your workspace.
2. Clear the `src` and `res` folders.
3. Unzip the `LibRecognizer.aar` file. You can rename it to zip and then unzip it or use any tool.
4. Copy the `classes.jar` to `libs` folder of your Eclipse library project. If `libs` folder does not exist, create it.
5. Copy `android-support-v4.jar` to `libs` folder of your Eclipse library project. You can find `android-support-v4.jar` in `/path/to/your/android/SDK/extras/android/support/v4/android-support-v4.jar`.
6. Copy the contents of `jni` folder to `libs` folder of your Eclipse library project.
7. Replace the `res` folder on library project with the `res` folder of the `LibRecognizer.aar` file.

You’ve already created the project that contains almost everything you need. Now let’s see how to configure your project to reference this library project.

1. In the project you want to use the library (henceforth, "target project") add the library project as a dependency
2. Open the `AndroidManifest.xml` file inside `LibRecognizer.aar` file and make sure to copy all permissions, features and activities to the `AndroidManifest.xml` file of the target project.
3. Clean and Rebuild your target project

## <a name="mavenIntegration"></a> How to integrate _PDF417.mobi_ into your project using Maven

Maven repository for _PDF417.mobi_ SDK is: [http://maven.microblink.com](http://maven.microblink.com).

### Using gradle
In your build.gradle you first need to add _PDF417.mobi_ maven repository to repositories list:

```
repositories {
	maven { url 'http://maven.microblink.com' }
}
```

After that, you just need to add _PDF417.mobi_ as a dependency to your application:

```
dependencies {
    compile 'com.microblink:pdf417.mobi:4.0.1'
}
```

### Using android-maven-plugin

Open your pom.xml file and add these directives as appropriate:

```xml
<repositories>
   	<repository>
       	<id>pdf417repo</id>
       	<url>http://maven.microblink.com</url>
   	</repository>
</repositories>

<dependencies>
	<dependency>
		  <groupId>com.microblink</groupId>
		  <artifactId>pdf417.mobi</artifactId>
		  <version>4.0.1</version>
  	</dependency>
<dependencies>
```

Maven dependency requires android-maven-plugin version 4.0.0 (AAR support is required).

## <a name="quickScan"></a> Performing your first scan
1. You can start recognition process by starting `Pdf417ScanActivity` activity with Intent initialized in the following way:
	
	```java
	// Intent for Pdf417ScanActivity Activity
	Intent intent = new Intent(this, Pdf417ScanActivity.class);

	// setup array of recognition settings (described in chapter "Recognition 
	// settings and results")
	RecognizerSettings[] settArray = setupSettingsArray();
	intent.putExtra(Pdf417ScanActivity.EXTRAS_RECOGNIZER_SETTINGS_ARRAY, settArray);

	// Starting Activity
	startActivityForResult(intent, MY_REQUEST_CODE);
	```
2. After `Pdf417ScanActivity` activity finishes the scan, it will return to the calling activity and will call method `onActivityResult`. You can obtain the scanning results in that method.

	```java
	@Override
	protected void onActivityResult(int requestCode, int resultCode, Intent data) {
		super.onActivityResult(requestCode, resultCode, data);
		
		if (requestCode == MY_REQUEST_CODE) {
			if (resultCode == Pdf417ScanActivity.RESULT_OK && data != null) {
				// perform processing of the data here
				
				// for example, obtain parcelable recognition result
				Bundle extras = data.getExtras();
				Parcelable[] resultArray = data.getParcelableArrayExtra(Pdf417ScanActivity.EXTRAS_RECOGNITION_RESULT_LIST);
				
				// Each element in resultArray inherits BaseRecognitionResult class and
				// represents the scan result of one of activated recognizers that have
				// been set up. More information about this can be found in 
				// "Recognition settings and results" chapter
						
				// Or, you can pass the intent to another activity
				data.setComponent(new ComponentName(this, ResultActivity.class));
				startActivity(data);
			}
		}
	}
	```
	
	For more information about defining recognition settings and obtaining scan results see [Recognition settings and results](#recognitionSettingsAndResults).

# <a name="advancedIntegration"></a> Advanced _PDF417.mobi_ integration instructions
This section will cover more advanced details in _PDF417.mobi_ integration. First part will discuss the methods for checking whether _PDF417.mobi_ is supported on current device. Second part will cover the possible customization of builtin `Pdf417ScanActivity` activity, third part will describe how to embed `RecognizerView` into your activity and fourth part will describe how to use direct API to recognize directly android bitmaps without the need of camera.

## <a name="supportCheck"></a> Checking if _PDF417.mobi_ is supported

### _PDF417.mobi_ requirements
Even before starting the scan activity, you should check if _PDF417.mobi_ is supported on current device. In order to be supported, device needs to have camera. 

Android 2.3 is the minimum android version on which _PDF417.mobi_ is supported, but if required we may support even Android 2.2 devices, however additional testing on those devices will be required.

_PDF417.mobi_ is native application, written in C++ and available for multiple platforms. Because of this, _PDF417.mobi_ cannot work on devices that have obscure hardware architectures. We have compiled _PDF417.mobi_ native code for `armeabi`, `armeabi-v7a` and `x86` [ABIs](https://en.wikipedia.org/wiki/Application_binary_interface) because those architectures cover almost entire Android device market (actually more than 98% of devices use `armeabi-v7a` ABI). See [Processor architecture considerations](#archConsider) for more information about native libraries in _PDF417.mobi_ and instructions how to disable certain architectures in order to reduce the size of final app.

### Checking for _PDF417.mobi_ support in your app
To check whether the _PDF417.mobi_ is supported on the device, you can do it in the following way:
	
```java
// check if PDF417.mobi is supported on the device
RecognizerCompatibilityStatus supportStatus = RecognizerCompatibility.getRecognizerCompatibilityStatus(this);
if(status == RecognizerCompatibilityStatus.RECOGNIZER_SUPPORTED) {
	Toast.makeText(this, "PDF417.mobi is supported!", Toast.LENGTH_LONG).show();
} else {
	Toast.makeText(this, "PDF417.mobi is not supported! Reason: " + supportStatus.name(), Toast.LENGTH_LONG).show();
}
```

## <a name="scanActivityCustomization"></a> Customization of `Pdf417ScanActivity` activity

### `Pdf417ScanActivity` intent extras

This section will discuss possible parameters that can be sent over `Intent` for `Pdf417ScanActivity` activity that can customize default behaviour. There are several intent extras that can be sent to `Pdf417ScanActivity` actitivy:

* **`Pdf417ScanActivity.EXTRAS_CAMERA_TYPE`** - with this extra you can define which camera on device will be used. To set the extra to intent, use the following code snippet:
	
	```java
	intent.putExtra(Pdf417ScanActivity.EXTRAS_CAMERA_TYPE, (Parcelable)CameraType.CAMERA_FRONTFACE);
	```
	
* **`Pdf417ScanActivity.EXTRAS_RECOGNIZER_SETTINGS_ARRAY`** - with this extra you must set the array of `RecognizerSettings` objects. Each `RecognizerSettings` object will define settings for specific recognizer object. Each recognizer object then creates its version of `BaseRecognitionResult` object in array returned via `Pdf417ScanActivity.EXTRAS_RECOGNITION_RESULT_LIST` extra. For more information about recognition settings and result, see [Recognition settings and results](#recognitionSettingsAndResults).  After defining recognition settings array, you need to put them into intent extra with following code snippet:
	
	```java
	intent.putExtra(Pdf417ScanActivity.EXTRAS_RECOGNIZER_SETTINGS_ARRAY, settings);
	```
	
* **`Pdf417ScanActivity.EXTRAS_RECOGNITION_RESULT_LIST`** - you can use this extra in `onActivityResult` method of calling activity to obtain array with recognition results. For more information about recognition settings and result, see [Recognition settings and results](#recognitionSettingsAndResults). You can use the following snippet to obtain array of scan results:

	```java
	Parcelable[] resultArray = data.getParcelableArrayExtra(Pdf417ScanActivity.EXTRAS_RECOGNITION_RESULT_LIST);
	```
	
* **`Pdf417ScanActivity.EXTRAS_GENERIC_SETTINGS`** - with this extra you can define additional settings that affect all recognizers or whole recognition process. More information about generic settings can be found in chapter [Generic settings](#genericSettings). To set the extra to intent, use the following code snippet:
	
	```java
	GenericRecognizerSettings genSett = new GenericRecognizerSettings();
	// define additional settings; e.g set timeout to 10 seconds
	genSett.setNumMsBeforeTimeout(10000);
	intent.putExtra(Pdf417ScanActivity.EXTRAS_GENERIC_SETTINGS, genSett);
	```
	
* **`Pdf417ScanActivity.EXTRAS_OPTIMIZE_CAMERA_FOR_NEAR_SCANNING`** - with this extra you can give a hint to _PDF417.mobi_ to optimize camera parameters for near object scanning. When camera parameters are optimized for near object scanning, macro focus mode will be preferred over autofocus mode. Thus, camera will have easier time focusing on to near objects, but might have harder time focusing on far objects. If you expect that most of your scans will be performed by holding the device very near the object, turn on that parameter. By default, this parameter is set to false.
	
* **`Pdf417ScanActivity.EXTRAS_BEEP_RESOURCE`** - with this extra you can set the resource ID of the sound to be played when scan completes. You can use the following snippet to set this extra:

	```java
	intent.putExtra(Pdf417ScanActivity.EXTRAS_BEEP_RESOURCE, R.raw.beep);
    ```
	

* **`Pdf417ScanActivity.EXTRAS_LICENSE_KEY`** - with this extra you can set the license key for _PDF417.mobi_. You can use _PDF417.mobi_ free of charge and without license key for development and non-commercial projects. Once you obtain a license key from [pdf417.mobi website](http://pdf417.mobi/), you can set it with following snippet:

	```java
	// set the license key
	intent.putExtra(Pdf417ScanActivity.EXTRAS_LICENSE_KEY, "Enter_License_Key_Here");
	```
	
	License key is bound to package name of your application. For example, if you have license key that is bound to `mobi.pdf417.demo` app package, you cannot use the same key in other applications. However, if you purchase Premium license, you will get license key that can be used in multiple applications. This license key will then not be bound to package name of the app. Instead, it will be bound to the licensee string that needs to be provided to the library together with the license key. To provide license owner string, use the `EXTRAS_LICENSEE` intent extra like this:

	```java
	// set the license key
	intent.putExtra(Pdf417ScanActivity.EXTRAS_LICENSE_KEY, "Enter_License_Key_Here");
	intent.putExtra(Pdf417ScanActivity.EXTRAS_LICENSEE, "Enter_Licensee_Here");
	```

* **`Pdf417ScanActivity.EXTRAS_SHOW_DIALOG_AFTER_SCAN`** - with this extra you can prevent showing of dialog after each barcode scan. By default, each time scanner finds and decodes a barcode, a dialog with barcode's contents will be shown. To prevent this, use the following snippet:
	
	```java
	// set the license key
	intent.putExtra(Pdf417ScanActivity.EXTRAS_SHOW_DIALOG_AFTER_SCAN, false);
	```

### Customizing `Pdf417ScanActivity` appearance

Besides possibility to put various intent extras for customizing `Pdf417ScanActivity` behaviour, you can also change strings it displays. The procedure for changing strings in `Pdf417ScanActivity` activity are explained in [Translation and localization](#stringChanging) section.

#### Camera splash screen

While loading camera, `Pdf417ScanActivity` displays a splash screen. The layout of splash screen is defined in `res/layout/camera_splash.xml`. If you are not satisfied with default splash screen design, you can overwrite that file as you wish.

#### Modifying other resources.

Generally, you can also change other resources that `Pdf417ScanActivity` uses, but you are encouraged to create your own custom scan activity instead (see [Embedding `RecognizerView` into custom scan activity](#recognizerView)). Just do not modify the contents of `raw` folder, as it contains files necessary for native part of the library - without those files _PDF417.mobi_ will not work.

#### Changing viewfinder appearance

To change the colour of viewfinder in `Pdf417ScanActivity`, change or override the colours defined in `res/values/colors.xml` (colours `default_frame` and `recognized_frame`).

## <a name="recognizerView"></a> Embedding `RecognizerView` into custom scan activity
This section will discuss how to embed `RecognizerView` into your scan activity and perform scan.

1. First make sure that `RecognizerView` is a member field in your activity. This is required because you will need to pass all activity's lifecycle events to `RecognizerView`.
2. It is recommended to keep your scan activity in one orientation, such as `portrait` or `landscape`. Setting `sensor` as scan activity's orientation will trigger full restart of activity whenever device orientation changes. This will provide very poor user experience because both camera and _PDF417.mobi_ native library will have to be restarted every time. There are measures for this behaviour and will be discussed [later](#scanOrientation).
3. In your activity's `onCreate` method, create a new `RecognizerView`, define its [settings and listeners](#recognizerViewReference) and then call its `create` method. After that, add your views that should be layouted on top of camera view.
4. Override your activity's `onStart`, `onResume`, `onPause`, `onStop` and `onDestroy` methods and call `RecognizerView's` lifecycle methods `start`, `resume`, `pause`, `stop` and `destroy`. This will ensure correct camera and native resource management. If you plan to manage `RecognizerView's` lifecycle independently of host activity's lifecycle, make sure the order of calls to lifecycle methods is the same as is with activities (i.e. you should not call `resume` method if `create` and `start` were not called first).

Here is the minimum example of integration of `RecognizerView` as the only view in your activity:

```java
public class MyScanActivity extends Activity implements ScanResultListener, CameraEventsListener {
	private RecognizerView mRecognizerView;
		
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		// create RecognizerView
		mRecognizerView = new RecognizerView(this);
		   
		// setup array of recognition settings (described in chapter "Recognition 
		// settings and results")
		RecognizerSettings[] settArray = setupSettingsArray();
		if(!RecognizerCompatibility.cameraHasAutofocus(CameraType.CAMERA_BACKFACE, this)) {
			setarr = RecognizerSettingsUtils.filterOutRecognizersThatRequireAutofocus(setarr);
		}
		mRecognizerView.setRecognitionSettings(settings);
		
        // set license key
        mRecognizerView.setLicenseKey("your license key here");
           
		// scan result listener will be notified when scan result gets available
		mRecognizerView.setScanResultListener(this);
		// camera events listener will be notified about camera lifecycle and errors
		mRecognizerView.setCameraEventsListener(this);
		
		// set camera aspect mode
		// ASPECT_FIT will fit the camera preview inside the view
		// ASPECT_FILL will zoom and crop the camera preview, but will use the
		// entire view surface
		mRecognizerView.setAspectMode(CameraAspectMode.ASPECT_FILL);
		   
		mRecognizerView.create();
		setContentView(mRecognizerView);
	}
	
	@Override
	protected void onStart() {
	   super.onStart();
	   // you need to pass all activity's lifecycle methods to RecognizerView
	   mRecognizerView.start();
	}
	
	@Override
	protected void onResume() {
	   super.onResume();
	   // you need to pass all activity's lifecycle methods to RecognizerView
	   mRecognizerView.resume();
	}

	@Override
	protected void onPause() {
	   super.onPause();
	   // you need to pass all activity's lifecycle methods to RecognizerView
	   mRecognizerView.pause();
	}

	@Override
	protected void onStop() {
	   super.onStop();
	   // you need to pass all activity's lifecycle methods to RecognizerView
	   mRecognizerView.stop();
	}
	
	@Override
	protected void onDestroy() {
	   super.onDestroy();
	   // you need to pass all activity's lifecycle methods to RecognizerView
	   mRecognizerView.destroy();
	}

	@Override
	public void onConfigurationChanged(Configuration newConfig) {
	   super.onConfigurationChanged(newConfig);
	   // you need to pass all activity's lifecycle methods to RecognizerView
	   mRecognizerView.changeConfiguration(newConfig);
	}
	
    @Override
    public void onScanningDone(BaseRecognitionResult[] dataArray, RecognitionType recognitionType) {
    	// this method is from ScanResultListener and will be called when scanning completes
    	// multiple scan results may be returned, depending on generic settings that define
    	// whether all found objects should be returned or only the first one (see subchapter
    	// "Generic settings" in chapter "Recognition settings and results")
    	
    	// When this method gets called, scanning gets paused. To resume scanning after this
    	// method has been called, call resumeScanning method.
    	mRecognizerView.resumeScanning();
    }
    
    @Override
    public void onCameraPreviewStarted() {
        // this method is from CameraEventsListener and will be called when camera preview starts
    }

    @Override
    public void onStartupError(Throwable exc) {
        /** 
         * This method is from CameraEventsListener and will be called when opening of
         * camera resulted in exception. 
         * Known exceptions that can occur are following:
         *      * com.microblink.hardware.camera.CameraResolutionTooSmallException is thrown when largest possible camera preview
         *        resolution is not enough for making a successful scan
         *      * java.lang.UnsatisfiedLinkError is thrown when native library was not successfully loaded thus making scans impossible
         *      * java.lang.Throwable is thrown in all other cases (for example when camera is not ready because it is used by other
         *        apps or some unknown error has occurred)
         */
    }

    @Override
    public void onNotSupported(NotSupportedReason reason) {
        // This method is from CameraEventsListener and will be called when scanning is not supported 
        // on device. Reason for not being supported is given in 'reason' parameter.
    }
    
    @Override
    public void onAutofocusFailed() {
	    /**
	     * This method is from CameraEventsListener will be called when camera focusing has failed. 
	     * Camera manager usually tries different focusing strategies and this method is called when all 
	     * those strategies fail to indicate that either object on which camera is being focused is too 
	     * close or ambient light conditions are poor.
	     */
    }
    
    @Override
    public void onAutofocusStarted(Rect[] areas) {
	    /**
	     * This method is from CameraEventsListener and will be called when camera focusing has started.
	     * You can utilize this method to draw focusing animation on UI.
	     * Areas parameter is array of rectangles where focus is being measured. 
	     * It can be null on devices that do not support fine-grained camera control.
	     */
    }

    @Override
    public void onAutofocusStopped(Rect[] areas) {
	    /**
	     * This method is from CameraEventsListener and will be called when camera focusing has stopped.
	     * You can utilize this method to remove focusing animation on UI.
	     * Areas parameter is array of rectangles where focus is being measured. 
	     * It can be null on devices that do not support fine-grained camera control.
	     */
    }
}
```

### <a name="scanOrientation"></a> Scan activity's orientation

If activity's `screenOrientation` property in `AndroidManifest.xml` is set to `sensor`, `fullSensor` or similar, activity will be restarted every time device changes orientation from portrait to landscape and vice versa. While restarting activity, its `onPause`, `onStop` and `onDestroy` methods will be called and then new activity will be created anew. This is a potential problem for scan activity because in its lifecycle it controls both camera and native library - restarting the activity will trigger both restart of the camera and native library. This is a problem because changing orientation from landscape to portrait and vice versa will be very slow, thus degrading a user experience. **We do not recommend such setting.**

For that matter, we recommend setting your scan activity to either `portrait` or `landscape` mode and handle device orientation changes manually. To help you with this, `RecognizerView` supports adding child views to it that will be rotated regardless of activity's `screenOrientation`. You add a view you wish to be rotated (such as view that contains buttons, status messages, etc.) to `RecognizerView` with `addChildView` method. The second parameter of the method is a boolean that defines whether the view you are adding will be rotated with device. To define allowed orientations, implement [OrientationAllowedListener](javadoc/com/microblink/view/OrientationAllowedListener.html) interface and add it to `RecognizerView` with method `setOrientationAllowedListener`. **This is the recommended way of rotating camera overlay.**

However, if you really want to set `screenOrientation` property to `sensor` or similar and want Android to handle orientation changes of your scan activity, then we recommend to set `configChanges` property of your activity to `orientation|screenSize`. This will tell Android not to restart your activity when device orientation changes. Instead, activity's `onConfigurationChanged` method will be called so that activity can be notified of the configuration change. In your implementation of this method, you should call `changeConfiguration` method of `RecognizerView` so it can adapt its camera surface and child views to new configuration. Note that on Android versions older than 4.0 changing of configuration will require restart of camera, which can be slow.

__Important__

If you use `sensor` or similar screen orientation for your scan activity there is a catch. No matter if your activity is set to be restarted on configuration change or only notified via `onConfigurationChanged` method, if your activity's orientation is changed from `portrait` to `reversePortrait` or from `landscape` to `reverseLandscape` or vice versa, your activity will not be notified of this change in any way - it will not be neither restarted nor `onConfigurationChanged` will be called - the views in your activity will just be rotated by 180 degrees. This is a problem because it will make your camera preview upside down. In order to fix this, you first need to [find a way how to get notified of this change](https://stackoverflow.com/questions/9909037/how-to-detect-screen-rotation-through-180-degrees-from-landscape-to-landscape-or) and then you should call `changeConfiguration` method of `RecognizerView` so it will correct camera preview orientation.

## <a name="recognizerViewReference"></a> `RecognizerView` reference
The complete reference of `RecognizerView` is available in [Javadoc](javadoc/com/microblink/view/recognition/RecognizerView.html). The usage example is provided in `pdf417MobiDemoCustomUI` demo app provided with SDK. This section just gives a quick overview of `RecognizerView's` most important methods.

##### `create()`
This method should be called in activity's `onCreate` method. It will initialize `RecognizerView's` internal fields and will initialize camera control thread. This method must be called after all other settings are already defined, such as listeners and recognition settings. After calling this method, you can add child views to `RecognizerView` with method `addChildView(View, boolean)`.

##### `start()`
This method should be called in activity's `onStart` method. It will initialize background processing thread and start native library initialization on that thread.

##### `resume()`
This method should be called in activity's `onResume` method. It will trigger background initialization of camera.

##### `pause()`
This method should be called in activity's `onPause` method. It will stop the camera, but will keep native library loaded.

##### `stop()`
This method should be called in activity's `onStop` method. It will deinitialize native library, terminate background processing thread and free all resources that are no longer necessary.

##### `destroy()`
This method should be called in activity's `onDestroy` method. It will free all resources allocated in `create()` and will terminate camera control thread.

##### `changeConfiguration(Configuration)`
This method should be called in activity's `onConfigurationChanged` method. It will adapt camera surface to new configuration without the restart of the activity. See [Scan activity's orientation](#scanOrientation) for more information.

##### `setCameraType(CameraType)`
With this method you can define which camera on device will be used. Default camera used is back facing camera.

##### `setAspectMode(CameraAspectMode)`
Define the aspect mode of camera. If set to `ASPECT_FIT` (default), then camera preview will be fit inside available view space. If set to `ASPECT_FILL`, camera preview will be zoomed and cropped to use the entire view space.

##### `setRecognitionSettings(RecognizerSettings[])`
With this method you can set the array of `RecognizerSettings` objects. Those objects will contain information about what will be scanned and how will scan be performed. For more information about recognition settings and results see [Recognition settings and results](#recognitionSettingsAndResults). This method must be called before `create()`.

##### `setGenericRecognizerSettings(GenericRecognizerSettings)`
With this method you can set the generic settings that will be affect all enabled recognizers or the whole recognition process. For more information about generic settings, see [Generic settings](#genericSettings). This method must be called before `create()`.

##### `reconfigureRecognizers(RecognizerSettings[], GenericRecognizerSettings)`
With this method you can reconfigure the recognition process while recognizer is active. Unlike `setRecognitionSettings` and `setGenericRecognizerSettings`, this method can be called while recognizer is active (i.e. after `resume` was called), but paused (either `pauseScanning` was called or `onScanningDone` callback is being handled). For more information about recognition settings see [Recognition settings and results](#recognitionSettingsAndResults).

##### `setOrientationAllowedListener(OrientationAllowedListener)`
With this method you can set a [OrientationAllowedListener](javadoc/com/microblink/view/OrientationAllowedListener.html) which will be asked if current orientation is allowed. If orientation is allowed, it will be used to rotate rotatable views to it and it will be passed to native library so that recognizers can be aware of the new orientation.

##### `setRecognizerViewEventListener(RecognizerViewEventListener)`
With this method you can set a [RecognizerViewEventListener](javadoc/com/microblink/view/recognition/RecognizerViewEventListener.html) which will be notified when certain recognition events occur, such as when object has been detected.

##### `setScanResultListener(ScanResultListener)`
With this method you can set a [ScanResultListener](javadoc/com/microblink/view/recognition/ScanResultListener.html) which will be notified when recognition completes. After recognition completes, `RecognizerView` will pause its scanning loop and to continue the scanning you will have to call `resumeScanning` method. In this method you can obtain data from scanning results. For more information see [Recognition settings and results](#recognitionSettingsAndResults).

##### `setCameraEventsListener(CameraEventsListener)`
With this method you can set a [CameraEventsListener](javadoc/com/microblink/view/CameraEventsListener.html) which will be notified when various camera events occur, such as when camera preview has started, autofocus has failed or there has been an error while starting the camera.

##### `pauseScanning()`
This method pauses the scanning loop, but keeps both camera and native library initialized. This method is called internally when scan completes before `onScanningDone` is called.

##### `resumeScanning()`
With this method you can resume the paused scanning loop.

##### `addChildView(View, boolean)`
With this method you can add your own view on top of `RecognizerView`. `RecognizerView` will ensure that your view will be layouted exactly above camera preview surface (which can be letterboxed if aspect ratio of camera preview size does not match the aspect ratio of `RecognizerView` and camera aspect mode is set to `ASPECT_FIT`). Boolean parameter defines whether your view should be rotated with device orientation changes. The rotation is independent of host activity's orientation changes and allowed orientations will be determined from [OrientationAllowedListener](javadoc/com/microblink/view/OrientationAllowedListener.html). See also [Scan activity's orientation](#scanOrientation) for more information why you should rotate your views independently of activity.

##### `isCameraFocused()` 
This method returns `true` if camera thinks it has focused on object. Note that camera has to be loaded for this method to work.

##### `focusCamera()` 
This method requests camera to perform autofocus. If camera does not support autofocus feature, method does nothing. Note that camera has to be loaded for this method to work.

##### `isCameraTorchSupported()` 
This method returns `true` if camera supports torch flash mode. Note that camera has to be loaded for this method to work.

##### `setTorchState(boolean, SuccessCallback)` 
If torch flash mode is supported on camera, this method can be used to enable/disable torch flash mode. After operation is performed, [SuccessCallback](javadoc/com/microblink/hardware/SuccessCallback.html) will be called with boolean indicating whether operation has succeeded or not. Note that camera has to be loaded for this method to work and that callback might be called on background non-UI thread.

##### `setScanningRegion(Rectangle, boolean)`
You can use this method to define the scanning region and define whether this scanning region will be rotated with device if [OrientationAllowedListener](javadoc/com/microblink/view/OrientationAllowedListener.html) determines that orientation is allowed. This is useful if you have your own camera overlay on top of `RecognizerView` that is set as rotatable view - you can thus synchronize the rotation of the view with the rotation of the scanning region native code will scan.

Scanning region is defined as [Rectangle](javadoc/com/microblink/geometry/Rectangle.html). First parameter of rectangle is x-coordinate represented as percentage of view width, second parameter is y-coordinate represented as percentage of view height, third parameter is region width represented as percentage of view width and fourth parameter is region height represented as percentage of view height.

View width and height are defined in current context, i.e. they depend on screen orientation. If you allow your ROI view to be rotated, then in portrait view width will be smaller than height, whilst in landscape orientation width will be larger than height. This complies with view designer preview. If you choose not to rotate your ROI view, then your ROI view will be laid out either in portrait or landscape, depending on setting for your scan activity in `AndroidManifest.xml`

Note that scanning region only reflects to native code - it does not have any impact on user interface. You are required to create a matching user interface that will visualize the same scanning region you set here.

##### `setLicenseKey(String licenseKey)`
This method sets the license key that will unlock all features of the native library. You can obtain your license key from [pdf417.mobi website](http://pdf417.mobi/).

##### `setLicenseKey(String licenseKey, String licenseOwner)`
Use this method to set a license key that is bound to a license owner, not the application package name. You will use this method when you obtain a license key that allows you to use _PDF417.mobi_ SDK in multiple applications. You can obtain your license key from [pdf417.mobi website](http://pdf417.mobi/).

## <a name="directAPI"></a> Using direct API for recognition of android Bitmaps

This section will describe how to use direct API to recognize android Bitmaps without the need for camera. You can use direct API anywhere from your application, not just from activities.

1. First, you need to obtain reference to [Recognizer singleton](javadoc/com/microblink/directApi/Recognizer.html).
2. Second, you need to initialize the recognizer.
3. After initialization, you can use singleton to process images. You cannot process multiple images in parallel.
4. Do not forget to terminate the recognizer after usage (it is a shared resource).

Here is the minimum example of usage of direct API for recognizing android Bitmap:

```java
public class DirectAPIActivity extends Activity implements ScanResultListener {
	private Recognizer mRecognizer;
		
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		// initialize your activity here
	}
	
	@Override
	protected void onStart() {
	   super.onStart();
	   mRecognizer = Recognizer.getSingletonInstance();
		
	   // set license key
	   boolean success = mRecognizer.setLicenseKey(this, "your license key");
	   if (!success) {
	   		return;
	   }

		// setupSettingsArray method is described in chapter "Recognition 
		// settings and results")
		mRecognizer.initialize(this, null, setupSettingsArray());
	}
	
	@Override
	protected void onResume() {
	   super.onResume();
		// start recognition
		Bitmap bitmap = BitmapFactory.decodeFile("/path/to/some/file.jpg");
		mRecognizer.recognize(bitmap, this);
	}

	@Override
	protected void onStop() {
	   super.onStop();
	   mRecognizer.terminate();
	}

    @Override
    public void onScanningDone(BaseRecognitionResult[] dataArray, RecognitionType recognitionType) {
    	// this method is from ScanResultListener and will be called when scanning completes
    	// multiple scan results may be returned, depending on generic settings that define
    	// whether all found objects should be returned or only the first one (see subchapter
    	// "Generic settings" in chapter "Recognition settings and results")
    	
    	finish(); // in this example, just finish the activity
    }
    
}
```

# <a name="recognitionSettingsAndResults"></a> Recognition settings and results

This chapter will discuss various recognition settings used to configure different recognizers and scan results generated by them.

## <a name="genericSettings"></a> Generic settings

Generic settings affect all enabled recognizers and the whole recognition process. The complete reference can be found in [javadoc](javadoc/com/microblink/recognizers/settings/GenericRecognizerSettings.html). Here is the list of methods that are most relevant:

##### `setAllowMultipleScanResultsOnSingleImage(boolean)`
Sets whether or not outputting of multiple scan results from same image is allowed. If that is `true`, it is possible to return multiple recognition results from same image. By default, this option is `false`, i.e. the array of `BaseRecognitionResults` will contain at most 1 element. The upside of setting that option to `false` is the speed - if you enable lots of recognizers, as soon as the first recognizer succeeds in scanning, recognition chain will be terminated and other recognizers will not get a chance to analyze the image. The downside is that you are then unable to obtain multiple results from single image.

##### `setNumMsBeforeTimeout(int)`
Sets the number of miliseconds _PDF417.mobi_ will attempt to perform the scan it exits with timeout error. On timeout returned array of `BaseRecognitionResults` might be null, empty or may contain only elements that are not valid (`isValid` returns `false`) or are empty (`isEmpty` returns `true`).

## <a name="pdf417Recognizer"></a> Scanning PDF417 barcodes

This section discusses the settings for setting up PDF417 recognizer and explains how to obtain results from PDF417 recognizer.

### Setting up PDF417 recognizer

To activate PDF417 recognizer, you need to create a [Pdf417RecognizerSettings](javadoc/com/microblink/recognizers/barcode/pdf417/Pdf417RecognizerSettings.html) and add it to `RecognizerSettings` array. You can do this using following code snippet:

```java
private RecognizerSettings[] setupSettingsArray() {
	Pdf417RecognizerSettings sett = new Pdf417RecognizerSettings();
	// disable scanning of white barcodes on black background
	sett.setInverseScanning(false);
	// allow scanning of barcodes that have invalid checksum
	sett.setUncertainScanning(true);
	// disable automatic scale detection
	sett.setAutoScaleDetection(false);
	// disable scanning of barcodes that do not have quiet zone
	// as defined by the standard
	sett.setNullQuietZoneAllowed(false);

	// now add sett to recognizer settings array that is used to configure
	// recognition
	return new RecognizerSettings[] { sett };
}
```

As can be seen from example, you can tweak PDF417 recognition parameters with methods of `Pdf417RecognizerSettings`.

##### `setUncertainScanning(boolean)`
By setting this to `true`, you will enable scanning of non-standard elements, but there is no guarantee that all data will be read. This option is used when multiple rows are missing (e.g. not whole barcode is printed). Default is `false`.

##### `setNullQuietZoneAllowed(boolean)`
By setting this to `true`, you will allow scanning barcodes which don't have quiet zone surrounding it (e.g. text concatenated with barcode). This option can significantly increase recognition time. Default is `false`.

##### `setInverseScanning(boolean)`
By setting this to `true`, you will enable scanning of barcodes with inverse intensity values (i.e. white barcodes on dark background). This option can significantly increase recognition time. Default is `false`.

##### `setAutoScaleDetection(boolean)`
If set to `true`, prior reading barcode, image scale will be corrected. This enables correct reading of barcodes on high resolution images but slows down the recognition process. Default is `false`.

### Obtaining results from PDF417 recognizer
PDF417 recognizer produces [Pdf417ScanResult](javadoc/com/microblink/recognizers/barcode/pdf417/Pdf417ScanResult.html). You can use `instanceof` operator to check if element in results array is instance of `Pdf417ScanResult` class. See the following snippet for an example:

```java
@Override
public void onScanningDone(BaseRecognitionResult[] dataArray, RecognitionType recognitionType) {
	for(BaseRecognitionResult baseResult : dataArray) {
		if(baseResult instanceof Pdf417ScanResult) {
			Pdf417ScanResult result = (Pdf417ScanResult) baseResult;
			
	        // getStringData getter will return the string version of barcode contents
			String barcodeData = result.getStringData();
			// isUncertain getter will tell you if scanned barcode is uncertain
			boolean uncertainData = result.isUncertain();
			// getRawData getter will return the raw data information object of barcode contents
			BarcodeDetailedData rawData = result.getRawData();
			// BarcodeDetailedData contains information about barcode's binary layout, if you
			// are only interested in raw bytes, you can obtain them with getAllData getter
			byte[] rawDataBuffer = rawData.getAllData();
		}
	}
}
```

As you can see from the example, obtaining data is rather simple. You just need to call several methods of the `Pdf417ScanResult` object:

##### `String getStringData()`
This method will return the string representation of barcode contents. Note that PDF417 barcode can contain binary data so sometimes it makes little sense to obtain only string representation of barcode data.

##### `boolean isUncertain()`
This method will return the boolean indicating if scanned barcode is uncertain. This can return `true` only if scanning of uncertain barcodes is allowed, as explained earlier.

##### `BarcodeDetailedData getRawData()`
This method will return the object that contains information about barcode's binary layout. You can see information about that object in [javadoc](javadoc/com/microblink/results/barcode/BarcodeDetailedData.html). However, if you only need to access byte array containing, you can call method `getAllData` of `BarcodeDetailedData` object.

## <a name="usdl"></a> Scanning US Driver's licence barcodes

This section discusses the settings for setting up USDL recognizer and explains how to obtain results from it.

### Setting up USDL recognizer
To activate USDL recognizer, you need to create [USDLRecognizerSettings](javadoc/com/microblink/recognizers/barcode/usdl/USDLRecognizerSettings.html) and add it to `RecognizerSettings` array. You can do this using following code snippet:

```java
private RecognizerSettings[] setupSettingsArray() {
	USDLRecognizerSettings sett = new USDLRecognizerSettings();
	// disallow scanning of barcodes that have invalid checksum
	sett.setUncertainScanning(false);
	// disable automatic scale detection
	sett.setAutoScaleDetection(false);
	// disable scanning of barcodes that do not have quiet zone
	// as defined by the standard
	sett.setNullQuietZoneAllowed(false);
       
	// now add sett to recognizer settings array that is used to configure
	// recognition
	return new RecognizerSettings[] { sett };
}
```

As can be seen from example, you can tweak USDL recognition parameters with methods of `USDLRecognizerSettings`.

##### `setUncertainScanning(boolean)`
By setting this to `true`, you will enable scanning of non-standard elements, but there is no guarantee that all data will be read. This option is used when multiple rows are missing (e.g. not whole barcode is printed). Default is `false`.

##### `setNullQuietZoneAllowed(boolean)`
By setting this to `true`, you will allow scanning barcodes which don't have quiet zone surrounding it (e.g. text concatenated with barcode). This option can significantly increase recognition time. Default is `false`.

##### `setAutoScaleDetection(boolean)`
If set to `true`, prior reading barcode, image scale will be corrected. This enables correct reading of barcodes on high resolution images but slows down the recognition process. Default is `false`.

### Obtaining results from USDL recognizer

USDL recognizer produces [USDLScanResult](javadoc/com/microblink/recognizers/barcode/usdl/USDLScanResult.html). You can use `instanceof` operator to check if element in results array is instance of `USDLScanResult`. See the following snippet for an example:

```java
@Override
public void onScanningDone(BaseRecognitionResult[] dataArray, RecognitionType recognitionType) {
	for(BaseRecognitionResult baseResult : dataArray) {
		if(baseResult instanceof USDLScanResult) {
			USDLScanResult result = (USDLScanResult) baseResult;
			
	        // getStringData getter will return the string version of barcode contents (not parsed)
			String barcodeData = result.getStringData();
			// isUncertain getter will tell you if scanned barcode is uncertain
			boolean uncertainData = result.isUncertain();
			// getRawData getter will return the raw data information object of barcode contents
			BarcodeDetailedData rawData = result.getRawData();
			// BarcodeDetailedData contains information about barcode's binary layout, if you
			// are only interested in raw bytes, you can obtain them with getAllData getter
			byte[] rawDataBuffer = rawData.getAllData();
			
			// if you need specific parsed driver's licence element, you can
			// use getField method
			// for example, to obtain AAMVA version, you should use:
			String aamvaVersion = result.getField(USDLScanResult.kAamvaVersionNumber);
		}
	}
}
```

##### `String getStringData()`
This method will return the string representation of barcode contents (not parsed). Note that PDF417 barcode can contain binary data so sometimes it makes little sense to obtain only string representation of barcode data.

##### `boolean isUncertain()`
This method will return the boolean indicating if scanned barcode is uncertain. This can return `true` only if scanning of uncertain barcodes is allowed, as explained earlier.

##### `BarcodeDetailedData getRawData()`
This method will return the object that contains information about barcode's binary layout. You can see information about that object in [javadoc](javadoc/com/microblink/results/barcode/BarcodeDetailedData.html). However, if you only need to access byte array containing, you can call method `getAllData` of `BarcodeDetailedData` object.

##### `getField(String)`
This method will return a parsed US Driver's licence element. The method requires a key that defines which element should be returned and returns either a string representation of that element or `null` if that element does not exist in barcode. To see a list of available keys, refer to [Keys for obtaining US Driver's license data](DriversLicenseKeys.md)

## <a name="custom1DBarDecoder"></a> Scanning one dimensional barcodes with _PDF417.mobi_'s implementation

This section discusses the settings for setting up 1D barcode recognizer that uses _PDF417.mobi_'s implementation of scanning algorithms and explains how to obtain results from that recognizer. Henceforth, the 1D barcode recognizer that uses _PDF417.mobi_'s implementation of scanning algorithms will be refered as "Bardecoder recognizer".

### Setting up Bardecoder recognizer

To activate Bardecoder recognizer, you need to create a [BarDecoderRecognizerSettings](javadoc/com/microblink/recognizers/barcode/bardecoder/BarDecoderRecognizerSettings.html) and add it to `RecognizerSettings` array. You can do this using following code snippet:

```java
private RecognizerSettings[] setupSettingsArray() {
	BarDecoderRecognizerSettings sett = new BarDecoderRecognizerSettings();
	// activate scanning of Code39 barcodes
	sett.setScanCode39(true);
	// activate scanning of Code128 barcodes
	sett.setScanCode128(true);
	// disable scanning of white barcodes on black background
	sett.setInverseScanning(false);
	// disable slower algorithm for low resolution barcodes
	sett.setTryHarder(false);

	// now add sett to recognizer settings array that is used to configure
	// recognition
	return new RecognizerSettings[] { sett };
}
```

As can be seen from example, you can tweak Bardecoder recognition parameters with methods of `BarDecoderRecognizerSettings`.

##### `setScanCode128(boolean)`
Method activates or deactivates the scanning of Code128 1D barcodes. Default (initial) value is `false`.

##### `setScanCode39(boolean)`
Method activates or deactivates the scanning of Code39 1D barcodes. Default (initial) value is `false`.

##### `setInverseScanning(boolean)`
By setting this to `true`, you will enable scanning of barcodes with inverse intensity values (i.e. white barcodes on dark background). This option can significantly increase recognition time. Default is `false`.

##### `setTryHarder(boolean)`
By setting this to `true`, you will enabled scanning of lower resolution barcodes at cost of additional processing time. This option can significantly increase recognition time. Default is `false`.

### Obtaining results from Bardecoder recognizer

Bardecoder recognizer produces [BarDecoderScanResult](javadoc/com/microblink/recognizers/barcode/bardecoder/BarDecoderScanResult.html). You can use `instanceof` operator to check if element in results array is instance of `BarDecoderScanResult` class. See the following snippet for example:

```java
@Override
public void onScanningDone(BaseRecognitionResult[] dataArray, RecognitionType recognitionType) {
	for(BaseRecognitionResult baseResult : dataArray) {
		if(baseResult instanceof BarDecoderScanResult) {
			BarDecoderScanResult result = (BarDecoderScanResult) baseResult;
			
			// getBarcodeType getter will return a BarcodeType enum that will define
			// the type of the barcode scanned
			BarcodeType barType = result.getBarcodeType();
	        // getStringData getter will return the string version of barcode contents
			String barcodeData = result.getStringData();
			// getRawData getter will return the raw data information object of barcode contents
			BarcodeDetailedData rawData = result.getRawData();
			// BarcodeDetailedData contains information about barcode's binary layout, if you
			// are only interested in raw bytes, you can obtain them with getAllData getter
			byte[] rawDataBuffer = rawData.getAllData();
		}
	}
}
```

As you can see from the example, obtaining data is rather simple. You just need to call several methods of the `BarDecoderScanResult` object:

##### `String getStringData()`
This method will return the string representation of barcode contents. 

##### `BarcodeDetailedData getRawData()`
This method will return the object that contains information about barcode's binary layout. You can see information about that object in [javadoc](javadoc/com/microblink/results/barcode/BarcodeDetailedData.html). However, if you only need to access byte array containing, you can call method `getAllData` of `BarcodeDetailedData` object.

##### `String getExtendedStringData()`
This method will return the string representation of extended barcode contents. This is available only if barcode that supports extended encoding mode was scanned (e.g. code39).

##### `BarcodeDetailedData getExtendedRawData()`
This method will return the object that contains information about barcode's binary layout when decoded in extended mode. You can see information about that object in [javadoc](javadoc/com/microblink/results/barcode/BarcodeDetailedData.html). However, if you only need to access byte array containing, you can call method `getAllData` of `BarcodeDetailedData` object. This is available only if barcode that supports extended encoding mode was scanned (e.g. code39).

##### `getBarcodeType()`
This method will return a [BarcodeType](javadoc/com/microblink/recognizers/barcode/BarcodeType.html) enum that defines the type of barcode scanned.

## <a name="zxing"></a> Scanning barcodes with ZXing implementation

This section discusses the settings for setting up barcode recognizer that use ZXing's implementation of scanning algorithms and explains how to obtain results from it. _PDF417.mobi_ uses ZXing's [c++ port](https://github.com/zxing/zxing/tree/00f634024ceeee591f54e6984ea7dd666fab22ae/cpp) to support barcodes for which we still do not have our own scanning algorithms. Also, since ZXing's c++ port is not maintained anymore, we also provide updates and bugfixes to it inside our codebase.

### Setting up ZXing recognizer

To activate ZXing recognizer, you need to create [ZXingRecognizerSettings](javadoc/com/microblink/recognizers/barcode/zxing/ZXingRecognizerSettings.html) and add it to `RecognizerSettings` array. You can do this using the following code snippet:

```java
private RecognizerSettings[] setupSettingsArray() {
	ZXingRecognizerSettings sett=  new ZXingRecognizerSettings();
	// disable scanning of white barcodes on black background
	sett.setInverseScanning(false);
	// activate scanning of QR codes
	sett.setScanQRCode(true);

	// now add sett to recognizer settings array that is used to configure
	// recognition
	return new RecognizerSettings[] { sett };
}
```

As can be seen from example, you can tweak ZXing recognition parameters with methods of `ZXingRecognizerSettings`. Note that some barcodes, such as Code 39 are available for scanning with [_PDF417.mobi_'s implementation](#custom1DBarDecoder). You can choose to use only one implementation or both (just put both settings objects into `RecognizerSettings` array). Using both implementations increases the chance of correct barcode recognition, but requires more processing time. Of course, we recommend using the _PDF417.mobi_'s implementation for supported barcodes.

##### `setScanAztecCode(boolean)`
Method activates or deactivates the scanning of Aztec 2D barcodes. Default (initial) value is `false`.

##### `setScanCode128(boolean)`
Method activates or deactivates the scanning of Code128 1D barcodes. Default (initial) value is `false`.

##### `setScanCode39(boolean)`
Method activates or deactivates the scanning of Code39 1D barcodes. Default (initial) value is `false`.

##### `setScanDataMatrixCode(boolean)`
Method activates or deactivates the scanning of Data Matrix 2D barcodes. Default (initial) value is `false`.

##### `setScanEAN13Code(boolean)`
Method activates or deactivates the scanning of EAN 13 1D barcodes. Default (initial) value is `false`.

##### `setScanEAN8Code(boolean)`
Method activates or deactivates the scanning of EAN 8 1D barcodes. Default (initial) value is `false`.

##### `shouldScanITFCode(boolean)`
Method activates or deactivates the scanning of ITF 1D barcodes. Default (initial) value is `false`.

##### `setScanQRCode(boolean)`
Method activates or deactivates the scanning of QR 2D barcodes. Default (initial) value is `false`.

##### `setScanUPCACode(boolean)`
Method activates or deactivates the scanning of UPC A 1D barcodes. Default (initial) value is `false`.

##### `setScanUPCECode(boolean)`
Method activates or deactivates the scanning of UPC E 1D barcodes. Default (initial) value is `false`.

##### `setInverseScanning(boolean)`
By setting this to `true`, you will enable scanning of barcodes with inverse intensity values (i.e. white barcodes on dark background). This option can significantly increase recognition time. Default is `false`.

### Obtaining results from ZXing recognizer

Bardecoder recognizer produces [ZXingScanResult](javadoc/com/microblink/recognizers/barcode/zxing/ZXingScanResult.html). You can use `instanceof` operator to check if element in results array is instance of `ZXingScanResult` class. See the following snippet for example:

```java
@Override
public void onScanningDone(BaseRecognitionResult[] dataArray, RecognitionType recognitionType) {
	for(BaseRecognitionResult baseResult : dataArray) {
		if(baseResult instanceof ZXingScanResult) {
			ZXingScanResult result = (ZXingScanResult) baseResult;
			
			// getBarcodeType getter will return a BarcodeType enum that will define
			// the type of the barcode scanned
			BarcodeType barType = result.getBarcodeType();
	        // getStringData getter will return the string version of barcode contents
			String barcodeData = result.getStringData();
		}
	}
}
```

As you can see from the example, obtaining data is rather simple. You just need to call several methods of the `ZXingScanResult` object:

##### `String getStringData()`
This method will return the string representation of barcode contents. 

##### `getBarcodeType()`
This method will return a [BarcodeType](javadoc/com/microblink/recognizers/barcode/BarcodeType.html) enum that defines the type of barcode scanned.

# <a name="translation"></a> Translation and localization

`PDF417.mobi` can be localized to any language. If you are using `RecognizerView` in your custom scan activity, you should handle localization as in any other Android app - `RecognizerView` does not use strings nor drawables, it only uses raw resources from `res/raw` folder. Those resources must not be touched as they are required for recognition to work correctly.

However, if you use our builtin `Pdf417ScanActivity` activity, it will use resources packed with library project to display strings and images on top of camera view. We have already prepared string in several languages which you can use out of the box. You can also [modify those strings](#stringChanging), or you can [add your own language](#addLanguage).

To use a language, you have to enable it from the code:

* To enable usage of predefined language you should call method `LanguageUtils.setLanguage(language, context)`. For example, you can set language like this:

	```java
	// define PDF417.mobi language
	LanguageUtils.setLanguage(Language.Croatian, this);
	```
		
* To enable usage of language that is not available in predefined language enum (for example, if you added your own language), you should call method `LanguageUtils.setLanguageAndCountry(language, country, context)`. For example, you can set language like this:
	
	```java
	// define PDF417.mobi language
	LanguageUtils.setLanguageAndCountry("hr", "", this);
	```

### <a name="addLanguage"></a> Adding new language

_PDF417.mobi_ can easily be translated to other languages. The `res` folder in `LibRecognizer.aar` archive has folder `values` which contains `strings.xml` - this file contains english strings. In order to make e.g. croatian translation, create a folder `values-hr` in your project and put the copy od `strings.xml` inside it (you might need to extract `LibRecognizer.aar` archive to get access to those files). Then, open that file and change the english version strings into croatian version. 

### <a name="stringChanging"></a> Changing strings in the existing language
	
To modify an existing string, the best approach would be to:

1. choose a language which you want to modify. For example Croatia ('hr').
2. find strings.xml in `LibRecognizer.aar` archive folder `res/values-hr`
3. choose a string key which you want to change. For example, ```<string name="PhotoPayHelp">Help</string>```
4. in your project create a file `strings.xml` in the folder `res/values-hr`, if it doesn't already exist
5. create an entry in the file with the value for the string which you want. For example ```<string name="PhotoPayHelp">Pomoć</string>```
6. repeat for all the string you wish to change

# <a name="archConsider"></a> Processor architecture considerations

_PDF417.mobi_ is distributed with both ARMv6, ARMv7 and x86 native library binaries.

ARMv7 architecture gives the ability to take advantage of hardware accelerated floating point operations and SIMD processing with [NEON](http://www.arm.com/products/processors/technologies/neon.php). This gives _PDF417.mobi_ a huge performance boost on devices that have ARMv7 processors. Most new devices (all since 2012.) have ARMv7 processor so it makes little sense not to take advantage of performance boosts that those processors can give. 

x86 architecture gives the ability to obtain native speed on x86 android devices, like [Prestigio 5430](http://www.gsmarena.com/prestigio_multiphone_5430_duo-5721.php). Without that, _PDF417.mobi_ will not work on such devices, or it will be run on top of ARM emulator that is shipped with device - this will give a huge performance penalty.

However, there are some issues to be considered:

- ARMv7 processors understand ARMv6 instruction set, but ARMv6 processors do not understand ARMv7 instructions.
- if ARMv7 processor executes ARMv6 code, it does not take advantage of hardware floating point acceleration and does not use SIMD operations
- ARMv7 build of native library cannot be run on devices that do not have ARMv7 compatible processor (list of those old devices can be found [here](http://www.getawesomeinstantly.com/list-of-armv5-armv6-and-armv5-devices/))
- neither ARMv6 nor ARMv7 processors understand x86 instruction set
- x86 processors do not understand neither ARMv6 nor ARMv7 instruction sets
- however, some x86 android devices ship with the builtin [ARM emulator](http://commonsware.com/blog/2013/11/21/libhoudini-what-it-means-for-developers.html) - such devices are able to run ARM binaries (both ARMv6 and ARMv7) but with performance penalty. There is also a risk that builtin ARM emulator will not understand some specific ARM instruction and will crash.

`LibRecognizer.aar` archive contains both ARMv6, ARMv7 and x86 builds of native library. By default, when you integrate _PDF417.mobi_ into your app, your app will contain native builds for all processor architecture. Thus, _PDF417.mobi_ will work on ARMv6 and x86 devices and will use ARMv7 features on ARMv7 devices. However, the size of your application will be rather large.

## <a name="reduceSize"></a> Reducing the final size of your app

If your final app is too large because of _PDF417.mobi_, you can decide to create multiple flavors of your app - one flavor for ARMv6, one for ARMv7 and one for x86 devices. With gradle and Android studio this is very easy - just add the following code to `build.gradle` file of your app:

```
productFlavors {
   x86 {
       ndk {
           abiFilter "x86"
       }
   }
   armv7 {
       ndk {
           abiFilter "armeabi-v7a"
       }
   }
   arm {
       ndk {
           abiFilter "armeabi"
       }
   }
}
```

With that build instructions, gradle will build three different APK files for your app. Each APK will contain only native library for one processor architecture. You can find more information about multiple APK support in Google Play Store on [this link](https://developer.android.com/google/play/publishing/multiple-apks.html).

However, if you are using Eclipse, things get complicated. Eclipse does not support build flavors and you will either need to remove support for some processors or create three different library projects from `LibRecognizer.aar` - each one for specific processor architecture. In the next section, we will discuss how to remove processor architecture support from Eclipse library project.

### Removing processor architecture support in Eclipse

This section assumes that you have set up and prepared your Eclipse project from `LibRecognizer.aar` as described in chapter [Eclipse integration instructions](#eclipseIntegration).

Native libraryies in eclipse library project are located in subfolder `libs`:

- `libs/armeabi` contains native libraries for ARMv6 processor architecture
- `libs/armeabi-v7a` contains native libraries for ARMv7 processor arhitecture
- `libs/x86` contains native libraries for x86 processor architecture

To remove a support for processor architecture, you should simply delete appropriate folder inside Eclipse library project:

- to remove ARMv6 support, delete folder `libs/armeabi`
- to remove ARMv7 support, delete folder `libs/armeabi-v7a`
- to remove x86 support, delete folder `libs/x86`

### Consequences of removing processor architecture

However, removing a processor architecture has some consequences:

- by removing ARMv6 support _PDF417.mobi_ will not work on devices that have ARMv6 processors. 
- by removing ARMv7 support, _PDF417.mobi_ will work on both devices that have ARMv6 and ARMv7 processor. However, on ARMv7 processors, hardware floating point and SIMD acceleration will not be used, thus making _PDF417.mobi_ much slower. Our internal tests have shown that running ARMv7 version of _PDF417.mobi_ on ARMv7 device is more than 50% faster than running ARMv6 version on same device.
- by removing x86 support, _PDF417.mobi_ will not work on devices that have x86 processor, except in situations when devices have ARM emulator - in that case, _PDF417.mobi_ will work, but will be slow

Our recommendation is to include both ARMv6, ARMv7 and x86 versions into your app - it will work on all devices and will provide best user experience. However, if you really need to reduce the size of your app, we recommend releasing three versions of your app - one version with only ARMv6 version for old devices, one version with only ARMv7 version for new devices and one version with only x86 version for those rare x86 devices.

## <a name="combineNativeLibraries"></a> Combining _PDF417.mobi_ with other native libraries

If you are combining _PDF417.mobi_ library with some other libraries that contain native code into your application, make sure you match the architectures of all native libraries. For example, if third party library has got only ARMv6 and x86 versions, you must use exactly ARMv6 and x86 versions of _PDF417.mobi_ with that library, but not ARMv7. Using ARMv7 will most definitely crash your app in initialization step on some devices because it will try to load all its native dependencies in same preferred architecture - for example Nexus 4 preferres ARMv7 native libraries so it will see that there is a _PDF417.mobi_ ARMv7 native library and will load it. After that, it will try to load ARMv7 version of your third party library which does not exist - therefore app will crash with `UnsatisfiedLinkException`.

# <a name="troubleshoot"></a> Troubleshooting

In case of problems with using the SDK, you should do as follows:

* enable logging to get the ability to see what is library doing. To enable logging, put this line in your application:

	```java
	com.microblink.util.Log.setLogLevel(com.microblink.util.Log.LogLevel.LOG_VERBOSE);
	```

After this line, library will display as much information about its work as possible. Make sure to remove this line in your production code as lots of log outputs may slow down the performance of library.

If you cannot solve problems by yourself, do not hesitate to contact us at <support@microblink.com>. Make sure you include the logs when contacting us to minimize the time to find and correct a bug. Also, if having misrecognitions, please send us high resolution images that are not scanned correctly.

# <a name="info"></a> Additional info
For any other questions, feel free to contact us at <support@microblink.com>.

