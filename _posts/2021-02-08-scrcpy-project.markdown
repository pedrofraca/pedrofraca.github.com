---
layout: post
title:  "scrcpyui - macOS UI app to use scrcpy"
date:   2021-02-05 00:38:24 +0100
categories: mac android swift
---

After our first post about kotlin native, we're back and this time we'll talk about implementing a macOS app that will be the UI connection with the command line tool [scrcpy][scrcpy], this tool is an easy way to mirror your android device in your computer. Long time ago I thought that would be great to have a mac status bar app that can allow users to easily execute the tool, this status bap app will automatically recognize any device connected (using [adb][adb]) and just launch the tool when the user clicks on the desired device. 

Since it turned out to be a really simple an funny application I decided to open source [it][repo-url], since every project has to have a nice logo, I created this one after a few iterations with Gimp. Believe it or not, but my inspiration was the anarchy symbol.

{:refdef: style="text-align: center;"}
![scrcpy logo](/assets/scrcpy-ui-logo.png){:height="400px" width="400px"}
{: refdef}

First iteration
---------------
In this iteration I just wanted to be sure that all the pieces can work together and that we can distribute the app without any problem. Summing up a list with all the things our app has to have to be fully functional:

* Get devices connected from adb
* Parse response from adb and transform it in an array of devices
* Draw the list of devices in the status
* Open scrcpy once user clicks on a device

To do that we need to execute another app and parse their output, we're going to do so using [Process][process-link] from swift. Here how the method looked like in the first version: 

{% highlight swift %}

func shell(launchPath: String, arguments: [String], readOutput : Bool) -> String {
    let task = Process()
    task.launchPath = launchPath
    task.arguments = arguments

    let pipe = Pipe()
    task.standardOutput = pipe

    task.launch()

    if(!readOutput) {
        return ""
    }

    let data = pipe.fileHandleForReading.readDataToEndOfFile()
    guard let output = String(data: data, encoding: String.Encoding.utf8) else { return "" }
    
    return output
}

{% endhighlight %}

This method is executing a command, redirecting the standard output to a pipe that we read until it's finished. The `readOutput` parameter is going to avoid connecting to the pipe if we're not really interested in the output, which is the case of launching the scrcpy command.

After some hacking I got the first draft of the app, all the code was in the AppDelegate as you can see in this [commit][commit]).

A bit of feedback
---------------
Once the first version was implemented I wanted to get quick feedback on things I can improve, decided to share the app with one of my teammates and the first thing he pointed out was that the app was not refreshing the list of devices when opening it, so you have to do that manually everytime to connect a new device. First thing that came to my mind was the macOS wifi app, when you click on it, wifi networks are refreshed. You also get an status update on the first item of the menu:

{:refdef: style="text-align: center;"}
![wifi](/assets/wifi.png)
{: refdef}

Wanted to do it properly, so I decided to split up my logic into different components, and write some unit testing to verify that the logic is properly implemented. Finally the easier pattern to implement was model-view-presenter. Basically the presenter will take the actions user does with the status bar, execute something and draw back again the desired status in the UI.  

* *DeviceRepository*: Class that will basically receive in the constructor an instance of CommandLineProtocol to retrieve the devices from the console and parse it into strings. Will call to adb, as explained in the intro.

* *CommandLineProtocol*: Protocol that represents a command line interaction with only one method.

{% highlight swift %}

import Foundation

protocol CommandLine {
    func execute(arguments: [String], readOutput : Bool) -> String
}

{% endhighlight %}


* *BashCommandLine*: Implementation of the CommandLineProtocol that launches a command as a process and reads the output.

* *StatusBarProtocol*: UI Protocol than represents the status bar and some simple actions.

{% highlight swift %}

import Foundation

protocol StatusBarProtocol {
    func setInformation(info : String) -> Void
    func addItemToMenu(item : String, shortcut : String) -> Void
    func addOptions() -> Void
    func clearList() -> Void
}

{% endhighlight %}

* *ScrCpyPresenter*: Class that interacts with all the previous classes/protocols. Receives all of them and connects them with the business logic.

{% highlight swift %}

//Steps to do here
//1. Get always list of devices from adb. Update view in the meantime to tell user we're refreshing
//2. Once list is received refresh the UI and tell user we're not refreshing anymore.
func refresh() -> Void {
    statusBar.clearList()
    statusBar.setInformation(info: "Refreshing Devices")
    let devices = repo.getDevices()
    var deviceNumber = 1
    devices.forEach { device in
        statusBar.addItemToMenu(item: device, shortcut: String(deviceNumber))
        deviceNumber+=1
    }
    statusBar.setInformation(info: "\(devices.count) device(s) detected" )
    statusBar.addOptions()
}

{% endhighlight %}

Our main mission after these changes was to leave our *AppDelegate* with less code and just UI code, and a few part of presenter calls. Here a small arch representation:

{:refdef: style="text-align: center;"}
![scrcpy arch](/assets/scrcpy-ui-arch.png){:height="398px" width="600px"}
{: refdef}

You said testing, right?
---------------

One of the reasons of such refactor was to enable the code to be more testable. Allowing us to inject some mocks of the protocols and to verify that the logic inside of the components looks good. 

The first logic we wanted to verify was how to obtain the devices from adb and how to parse them in way they can be used in the next step, we created the *DevicesRepositoryTest* class that basically will receive some mock implementation of the CommandLine tool that simulates an empty response as well as one device response. 

{% highlight swift %}
class DevicesRepositoryTest: XCTestCase {

    func testDevicesReturnedProperly() {
        let repo = DevicesRepository(withCommandLine: MockCommandLine())
        XCTAssertEqual(["emulator-5554"], repo.getDevices())
    }
    
    func testNoDevicesAreReturnedIfNoDevicesConnected() {
        let repo = DevicesRepository(withCommandLine: MockCommandLineWithoutDevices())
        XCTAssertEqual([], repo.getDevices())
    }
}
{% endhighlight %}

For all of our tests we decided to use *XCTest* without any framework to mock or verify. All mocks are implemented manually and we implemented a really rudimentary system to verify some interaction with some methods.

{% highlight swift %}

class MockStatusBar : StatusBarProtocol {
      
    var functionCalled = [String: Bool]()
    
    func hasBeenCalled(function: String) -> Bool {
        let result = functionCalled[function] ?? false
        if(!result) {
            debugPrint("👉🏾 We can't find \(function) but we've this in the list of fuctions called")
            debugPrint(functionCalled)
        }
        return result
    }
    
    func setInformation(info: String) {
        functionCalled[#function] = true
    }
    
    func addItemToMenu(item: String, shortcut: String) {
        functionCalled[#function] = true
    }
    
    func addExtraOptions() {
        functionCalled[#function] = true
    }
    
    func clearList() {
        functionCalled[#function] = true
    }
}

{% endhighlight %}

As you can see, we're only adding the name of the method to a list to verify later that has been called, is not optimal, since we're here not keeping track of important things like order of calling or paremeters with some matchers. Since I'm not an expert on mocks/stub frameworks in swift and after a first research all of them seemed to be very heavy I decided to go that way. Feel free to contact me if you know any lightweight framework to use.

Testing is important, so is executing tests everytime you do a change. With GitHub actions you can do that easily, we've created two flows:

* Pull requests that are pointing to main. In this case we juest execute tests. You can take a look to the [yml][pull-action] file for the action.
* Code that arrives to main, in this case we're not only running tests but also building and bundling the binary to later distribute it, here the [action][main-action]. 

Distributing the app
---------------

Once the app is completed the next challenge to solve was how to distribute the binary. Here is when [homebrew][brew-link] comes to the rescue. It was not only about distributing the binary but also to makes sure all the dependencies the app needs are installed in the system, and also homebrew is a great tool for that. Since it's an already bundled macOS app we have to distribute it using [cask][homebrew-cask]. In order to do that you need to create your own homebrew repo to later tap on it. 

{% highlight bash %}
 brew tap-new username/your-repo  
{% endhighlight %}

By executing this command you'll generate in your local taps an empty repository that later you can sync with your github account. And then you can start creating your formulaes or casks in it.

In this case we're doing so for the scrcpyui app, as well as defining the dependency with scrcpy.

{% highlight ruby%}

cask "scrcpyui" do
  version "1.1"
  sha256 "b765ef555e8cab230a16297f1cf9e637d2507c44d896a25dd8fe689409988678"

  url "https://github.com/pedrofraca/scrcpyui/releases/download/v1.1/scrcpyui.dmg"
  name "scrcpyui"
  desc "Small project to provide UI access for the scrcpy tool"
  homepage "https://github.com/pedrofraca/scrcpyui/"

  app "scrcpyui.app"

  depends_on formula: "scrcpy"
end

{% endhighlight %}

After this is pretty easy to tap into your custom tap to install and update the app, you just need to do this:

{% highlight bash %}
 brew tap pedrofraca/brew; brew install scrcpyui
{% endhighlight %}

> Please, give it a try and feel free to distribute, contribute or just provide a bit of feedback.

[pull-action]: https://github.com/pedrofraca/scrcpyui/blob/main/.github/workflows/pull.yml
[main-action]: https://github.com/pedrofraca/scrcpyui/blob/main/.github/workflows/main.yml
[homebrew-cask]: https://github.com/Homebrew/homebrew-cask
[brew-link]: https://brew.sh/
[repo-url]: https://github.com/pedrofraca/scrcpyui
[commit]: https://github.com/pedrofraca/scrcpyui/commit/f0ddc38d558feebb3c5f376e479924886c9ddd43#diff-818b8d5e35c5db962f463c040a8a137a579906c162f3371826c398f8c4e76414
[scrcpy]: https://github.com/Genymobile/scrcpy
[adb]: https://developer.android.com/studio/command-line/adb
[process-link]: https://www.hackingwithswift.com/example-code/system/how-to-run-an-external-program-using-process
