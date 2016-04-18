# How to use Unity 3D within an iOS app

This is going to appear to be complicated based on the length of this article
it's really not. I try to fully show some examples here, and provide some images
for those who may not know where certain things are in xcode.


This would not be possible without [www.the-nerd.be],  Frederik Jacques.
All of the settings in the xcconfig file, the `UnityProjectRefresh.sh`
script and the project import are directly derieved from his work. The video
he made in the provided link is worth watching.

This covers Unity 5+. At the time of this writing this has been
successfully used with Unity `5.3.4f1` under `Xcode 7.3`.

This works with storyboards.

You only get **ONE** unity view. You **CANNOT** run multiple Unity
Views in your application at once. You will also need a way to
communicate to <-> from your unity content to your iOS app.
I would recommend an event bus in both your Unity code and
your iOS code. AKA one central place on both sides to emit events
to and listen to events on each side.

In other words you will need 2 busses, 1 on the Unity side that you can
call into to emit events from on the iOS side, and one on the iOS side that
Unity can call into to emit events on.

You can read more about communication between the 2 worlds from
the following links:

**More about embedding**

http://forum.unity3d.com/threads/unity-appcontroller-subclassing.191971/

Specifically there is a bit on commuicating here with some sample code.
Note, this is not for UNITY 5, but it shows the samples in OverlayUI related
making functions available to the Objective-C side of things to be called from
your Unity Code.

http://forum.unity3d.com/threads/unity-appcontroller-subclassing.191971/#post-1341666


**Communicating from Unity -> ObjC**

http://blogs.unity3d.com/2015/07/02/il2cpp-internals-pinvoke-wrappers/

http://forum.unity3d.com/threads/unity-5-2-2f1-embed-in-ios-with-extern-dllimport-__internal-methods-fails-to-compile.364809/


**Communicating from Unity <-> ObjC**

http://alexanderwong.me/post/29861010648/call-objective-c-from-unity-call-unity-from


## Lets get started.

### From Unity

First you need to have a project in unity, and you need to build it for iOS.

Under Unity 5 the project's scripting backend is already set to `il2cpp` so you
pretty much just have to :

- `File -> Build Settings`
- Select your scene(s)
- Press the build button
- Remember the folder you built the project too.


### From Xcode

There is a bit more to do here, but ideally the `Unity.xcconfig` and
the `UnityProjectRefresh.sh` script make this easier.

Setting expectations, the project import process here takes some time,
it's not instant, Unity generates a lot of files and Xcode has to import them
all. So expect to stare a beachball for a few minuts while it does it's thing.

Ok! Fire up Xcode and create a new `Objective-C` project or open an existing
`Objective-C` project.

Here is what we will be doing, this will seem like a lot, but it's pretty straight
forward. You will fly through these steps minus the unity project import/cleanup
which is not diffiucilt, it's just time consuming given the number of files.

- Add the Unity.xcconfig file provided in this repo
- Adjust 1 project dependent setting
- Add a new `run script` build phase
- Import your unity project
- Clean up your unity project
- Create and update a PrefixHeader.pch file
- Modify the `main.m` to handle unity initialization
- Wrap the UnityAppController into your application delegate
- Adjust the `GetAppController` function in `UnityAppController.h`
- Make few adjustments to support Vuforia
- Go bananas, you did it! Add the unity view wherever you want!

#### Add the Unity.xcconfig file provided in this repo

Drag and drop the `Unity.xcconfig` file into your Xcode project.
Set the project to use those settings.

<img src="https://dl.dropboxusercontent.com/u/20065272/forums/github/ios-unity5/set_xcconfig.png">

If you are using Cocoapods or have other xconfig already in place, you will have to manually add configs from `Unity.xcconfig` to Build Settings

#### Adjust 1 project dependent setting
So that does a lot for you in terms of configuration, now we need to adjust 1 setting in it.
Since we don't know where you decided to export your unity project too, you need to configure that.

The default location is `${SOURCE_ROOT}/Unity`

Open up your project's build settings and scroll all the way to bottom, you will see:

```
UNITY_IOS_EXPORT_PATH
```

Adjust that path to point to your ios unity export path


<img src="https://dl.dropboxusercontent.com/u/20065272/forums/github/ios-unity5/unity_ios_export_path.png">

You can also adjust your

```
UNITY_RUNTIME_VERSION
```

If you are not using  `5.3.4f1`.


#### Add a new `run script` build phase

Now we need to ensure we copy our fresh unity project on each build, so we add a
new run script build phase.

Select Build Phases from your project settings to add a new build phase.

Copy the contents of the UnityProjectRefresh.sh script into this phase.

<img src="https://dl.dropboxusercontent.com/u/20065272/forums/github/ios-unity5/run_script_phase.png">


#### Import your unity project

This is outlined in this [www.the-nerd.be] video at around 5:35 - 7:30 as well, but it's now time to import our Unity project.

Create a new group and call it `Unity`, the name doesn't matter it's just helpful to name things so you know what they are).
<img src="https://dl.dropboxusercontent.com/u/20065272/forums/github/ios-unity5/new_group.png">

You will need to open the folder you built your Unity iOS project into. It will be the same folder you
specified for the `UNITY_IOS_EXPORT_PATH` above.

Do 1 folder at a time, this will take a minute or more to do, there are lots of files.

We are going to drag in the following folders (You don't need to copy them):

- `/your/unity/ios/export/path/Classes`
- `/your/unity/ios/export/path/Libraries`


#### Clean up your unity project

This is all in the [www.the-nerd.be] video as well 7:35 -
There is two location we will clean up for convenience. For both of these we
*ONLY WANT TO REMOVE REFERENCES DO NOT MOVE TO TRASH*

We don't need the `Unity/Classes/Native/*.h`  and we don't need `Unity/Libraries/libl2cpp/`.

The Unity.xcconfig we applied knows where they are for compiling purposes.

- Remove `Unity/Libraries/libl2cpp/` 7:35 - 7:50 in [www.the-nerd.be] video.
- Remove `Unity/Classes/Native/*.h` 7:55- 8:44 in [www.the-nerd.be] video.


#### Create and update a PrefixHeader.pch file

This is all in the [www.the-nerd.be] video as well, begining at 14:55.
Create a new `PrefixHeader.pch` file and place it in project's root directory.
`Unity.xcconfig` will update Build Settings and link this file for you.

Next find the Unity Prefix.pch file, located in `Classes` folder. Copy the content and paste it into our `PrefixHeader.pch`, between `#define PrefixHeader_pch` and `#endif`.

Now add `#import "UnityAppController.h"` under `#import <UIKit/UIKit.h>`

#### Add the `UnityUtils` folder from this repo

Copy these into our project.

- `UnityUtils.h/mm` is our new custom init function.

The new custom unity init function is pulled directly out of the main.mm file in your unity project.

#### Modify the `main.m` to handle unity initialization

In your xcode project under `Unity/Classses` locate the `main.mm` file. Within that file locate

```cpp
int main(int argc, char* argv[])
```
Once you find that you can go ahead and see that `UnityUtils.mm`, which we imported
above, is effectively this function. If Unity change this initialization you will need
to update your `UnityUtils.mm` file to match their initialization.

Anyway, now we need to update our `main.m`.

Rename our `main.m` to `main.mm`, import `UnityUtils.h` and add

```cpp
custom_unity_init(argc, argv);
```

above

```cpp
return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
```

Now go to Build Phases and search for `main.mm`. You should see two `main.mm` under `Compile sources`
Remove the one located in Unity folder.

#### Wrap the UnityAppController into your application delegate

We are taking away control from the unity generated application delegate, we
need to act as a proxy for it in our `AppDelegate`.

First add the following variable to your `AppDelegate.h`

```objc
@property (nonatomic) UnityAppController *unityController;
```
Now we need to initialize and proxy through the calls to the `UnityAppController`.
All said and done you will be left with the following:

```objc
#import "AppDelegate.h"

@interface AppDelegate ()

@end

@implementation AppDelegate

-(BOOL)application:(UIApplication *)application willFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
    [self.unityController application:application willFinishLaunchingWithOptions:launchOptions];
    return YES;
}

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    // Override point for customization after application launch.
    [self.unityController application:application didFinishLaunchingWithOptions:launchOptions];
    return YES;
}

- (void)applicationWillResignActive:(UIApplication *)application {
    // Sent when the application is about to move from active to inactive state. This can occur for certain types of temporary interruptions (such as an incoming phone call or SMS message) or when the user quits the application and it begins the transition to the background state.
    // Use this method to pause ongoing tasks, disable timers, and throttle down OpenGL ES frame rates. Games should use this method to pause the game.
    [self.unityController applicationWillResignActive:application];
}

- (void)applicationDidEnterBackground:(UIApplication *)application {
    // Use this method to release shared resources, save user data, invalidate timers, and store enough application state information to restore your application to its current state in case it is terminated later.
    // If your application supports background execution, this method is called instead of applicationWillTerminate: when the user quits.
    [self.unityController applicationDidEnterBackground:application];
}

- (void)applicationWillEnterForeground:(UIApplication *)application {
    // Called as part of the transition from the background to the inactive state; here you can undo many of the changes made on entering the background.
    [self.unityController applicationWillEnterForeground:application];
}

- (void)applicationDidBecomeActive:(UIApplication *)application {
    // Restart any tasks that were paused (or not yet started) while the application was inactive. If the application was previously in the background, optionally refresh the user interface.
    [self.unityController applicationDidBecomeActive:application];
}

- (void)applicationWillTerminate:(UIApplication *)application {
    // Called when the application is about to terminate. Save data if appropriate. See also applicationDidEnterBackground:.
    [self.unityController applicationWillTerminate:application];
}

-(void)applicationDidReceiveMemoryWarning:(UIApplication *)application
{
    [self.unityController applicationDidReceiveMemoryWarning:application];
}

#pragma mark - Lazy Load

-(UnityAppController *)unityController
{
    if (!_unityController) {
        _unityController = [[UnityAppController alloc] init];
    }
    return _unityController;
}

@end
```

#### Adjust the `GetAppController` function in `UnityAppController.h`

Locate the file `UnityAppController.h` in the xcode group `Unity/Classes/`

Find the following function:

```objc
inline UnityAppController*GetAppController()
{
    return (UnityAppController*)[UIApplication sharedApplication].delegate;
}
```

Replace that with this:

```objc
NS_INLINE UnityAppController* GetAppController()
{
    NSObject<UIApplicationDelegate>* delegate = [UIApplication sharedApplication].delegate;
    UnityAppController* unityController = (UnityAppController *)[delegate valueForKey:@"unityController"];
    return unityController;
}
```


#### Make few adjustments to support Vuforia

Locate the file `UnityAppController.h` in the xcode group `Unity/Classes/`.

We need to update import statements for all frameworks. Replace `#import "framework"` with `#import <framework>`

E.g.
```objc
#import <QuartzCore/CADisplayLink.h>
#import <UIKit/UIKit.h>
#import <QuartzCore/CADisplayLink.h>

#include "PluginBase/RenderPluginDelegate.h"

@class UnityView;
@class DisplayConnection;
@class UnityViewControllerBase;
```

Now open the `UnityAppController.mm` and import `VuforiaNativeRendererController.h`


Next, locate the file `VuforiaNativeRendererController.mm` in the xcode group `Unity/Libraries/Plugins/iOS`.

Comment out or remove the last line
```cpp
IMPL_APP_CONTROLLER_SUBCLASS(VuforiaNativeRendererController)
```

Next, copy the `shouldAttachRenderDelegate` function
```cpp
- (void)shouldAttachRenderDelegate
{
	self.renderDelegate = [[VuforiaRenderDelegate alloc] init];

// Unity native rendering callback plugin mechanism is only supported
// from version 4.5 onwards
#if UNITY_VERSION>434
	UnityRegisterRenderingPlugin(&VuforiaSetGraphicsDevice, &VuforiaRenderEvent);
#endif
}
```
In `UnityAppController.h` you should find function with the same name, replace or update it with copied version.

Next, copy the top part from `VuforiaNativeRendererController.mm`
```cpp
// Unity native rendering callback plugin mechanism is only supported
// from version 4.5 onwards
#if UNITY_VERSION>434

// Exported methods for native rendering callback
extern "C" void VuforiaSetGraphicsDevice(void* device, int deviceType, int eventType);
extern "C" void VuforiaRenderEvent(int marker);

#endif
```
And paste it into `UnityAppController.h`

Even though we added a run script for Unity `Data` folder to be copied, Vuforia requires `QCAR` folder to be added to our project.

Go to your Unity build folder and locate `QCAR` folder under `Data/Raw`
Drag this folder into our project, uncheck `Copy items if needed` and select `Create folder references `

This is it. Vuforia should now work as intended.

#### Go bananas, you did it! Add the unity view wherever you want!

I happen to do this in a stock, single view application, so xcode generated a `ViewController`
files for me attached to a storyboard. Here is how I hooked up my little demo:

```swift
#import "ViewController.h"

@interface ViewController ()

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
}

- (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
    // Dispose of any resources that can be recreated.
}

- (IBAction)openUnity:(id)sender
{
    UIView *unityView = UnityGetGLView();
    [self.view addSubview:unityView];

    unityView.translatesAutoresizingMaskIntoConstraints = NO;

    NSDictionary *views = @{@"view": unityView};
    NSArray *verticalConstraints = [NSLayoutConstraint constraintsWithVisualFormat:@"|[view]-0-|" options:0 metrics:nil views:views];
    [self.view addConstraints:verticalConstraints];

    NSArray *horizontalConstraints = [NSLayoutConstraint constraintsWithVisualFormat:@"V:|-0-[view]-0-|" options:0 metrics:nil views:views];
    [self.view addConstraints:horizontalConstraints];
}
```

[www.the-nerd.be]: http://www.the-nerd.be/2015/08/20/a-better-way-to-integrate-unity3d-within-a-native-ios-application/  "The Nerd"
