# How to send video feedback with Azbouki SDK in iOS

### What is Azbouki

Capturing video feedback with enough metadata can save a lot of time for the entire dev team. There is already many tools available but I keep hearing from my friends and fellow developers that they are using chats and email for getting bug reports and feedback from their clients, users and beta testers. There are two main reasons for that:

* There isn't a go-to tool
* The most popular tools are either too expensive or closed-source and the developers don't have enough control over what they are recording and sending.

We can't (so far) change the former but there's much we can do about the latter. This is why we started working on the Azbouki SDK and dashboard and being open-source was the most important decision for us. 

### Why Sentry?

We have two priorities - to innovate and to help others innovate, so reinventing the wheel is not a part of our plan. Therefore building on what is already done (well) makes much more sense. Collecting user interaction events and showing them in a dashboard, collecting crash reports and performance monitoring is something that Sentry is doing great and is not our battle. This is why we chose to step on Sentry as our event collecting mechanism. So if you are already using it - great, Azbouki integrates easily with it. If you aren't - currently Azbouki SDK for iOS depends on it so it will be added to your dependencies.

### Setup

Setting up Azbouki SDK for iOS is simple. For the time being you can only install it as a cocoapod:

```
pod 'azbouki'
```

After running `pod install` you have to configure the SDK somewhere in the initialization logic of your app (preferably in the AppDelegate's `application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?)`:


```
import azbouki

...

func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
	...
	AzboukiClient.configure(appId: "<your-app-id>", userId: "<user-id-string>")
	...
}
```

### Send video session

In order to capture a video session you need to add `NSPhotoLibraryUsageDescription` to your `Info.plist` file. Otherwise, the app will crash when the user stops the video recording.

Starting a capture session is simple:

```
AzboukiClient.startVideoSession(message: "<some message that you can get according to your app's logic>") { err in
    if let err = err {
        print("Error starting session: \(err.localizedDescription)")
        return
    }
    if (AzboukiClient.isRecording()) {
        self.toggleRecordingButton.setTitle("stop recording", for: .normal)
    } else {
        print("Error starting session")
    }
    
}
```

In the sample application I created a UIButton that toggles recording and opens a UIAlertController:

```
 @IBAction func start(_ sender: UIButton) {
    if AzboukiClient.isRecording() {
        AzboukiClient.stopVideoSession()
        sender.setTitle("start recording", for: .normal)
    } else {
        openMessageAlert()
    }
}
    
func openMessageAlert() {
    let alert = UIAlertController(title: "Start recording", message: "Add a description", preferredStyle: .alert)
  
    alert.addTextField { (textField) in
        textField.text = ""
    }

    alert.addAction(UIAlertAction(title: "OK", style: .default, handler: { [weak alert] (_) in
        let textField = alert?.textFields![0]
        AzboukiClient.startVideoSession(message: textField?.text) { err in
            if let err = err {
                print("Error starting session: \(err.localizedDescription)")
                return
            }
            if (AzboukiClient.isRecording()) {
                self.toggleRecordingButton.setTitle("stop recording", for: .normal)
            } else {
                print("Error starting session")
            }
            
        }
        
    }))

    self.present(alert, animated: true, completion: nil)
}
```

Recording is ended by calling `AzboukiClient.stopVideoSession()`.



### Review video reports

Now if you login to the Azbouki dashboard you will see a new session added to the list:

![](https://raw.githubusercontent.com/tdermendjiev/tdermendjiev.github.io/master/assets/img/session-list.png)

Select a session and the session details screen will open. On the left is the video and right next to it is the list of events:

![](https://raw.githubusercontent.com/tdermendjiev/tdermendjiev.github.io/master/assets/img/play-events.png)

You can also open the app logs by clickin on `Application Log`:

![](https://raw.githubusercontent.com/tdermendjiev/tdermendjiev.github.io/master/assets/img/session-logs.png)

And see the device data on `Session information`:

![](https://raw.githubusercontent.com/tdermendjiev/tdermendjiev.github.io/master/assets/img/session-information.png)

At the bottom of the page there is some additional metadata formatted as a json object:

![](https://raw.githubusercontent.com/tdermendjiev/tdermendjiev.github.io/master/assets/img/additional-metadata.png)

### Conclusion

Azbouki SDK is fully open-source and you can easily extend it and add additional events to the event log. We do our best to keep the docs in a good shape but if any help or customization is needed, drop us a note.
