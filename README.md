# Light Puzzle
A puzzle for your Philips Hue Smart Bulbs and iOS device.

## Developing a Hue Light Puzzle iOS App

My goal is to develop an iOS app that connects with Philips Hue Lights and uses the lights to show a puzzle. The user would use the app to solve the puzzle and the app would change the lights to match the current state of the puzzle.

I already have a developer account with Apple and Xcode which is needed to develop iOS apps. I've never worked with Philips Hue lights, so first I found their developer site and signed up for a developer account.

I have broken the problem into 2 parts: connecting to the Philips Hue bulbs and connecting an iOS app to a Philips Hue bridge.

### Connecting to Philips Hue Bulbs
To learn how to develop for the Hue Light bulb, I went to the [Hue developer site](https://developers.meethue.com/develop/get-started-2/). It lists 3 easy steps for getting started:
1. Test your bridge network by making sure you can control your lights with your iOS device
1. Find your bridge's IP address
1. Load the test app in your browser at:
https://\<bridge ip address\>/debug/clip.html

I immediately got blocked on step 2 of the 3 easy steps to get started which was to obtain the IP address of the bridge. I was frustrated that I couldn't find it so I gave up working on the project to focus on my classes. Going back to step 2, there are multiple ways of obtaining the IP address for the bridge, so I used a different method this time (using the official Philips Hue app) with success.

The next step was visiting the API debug tool in a web browser at: `https://<bridge ip address>/debug/clip.html`. The debugger lets you create HTTPS calls. This allows me to develop the code for the lights separately from the iOS app.

#### Use GET to query available resources
If you try to use the GET command, you will get an error message like:
```
[
	{
		"error": {
			"type": 1,
			"address": "/",
			"description": "unauthorized user"
		}
	}
]
```
If you create a user and use it with a GET command, you can get info about the lights and their states.

#### Create a new user with POST
The bridge will randomly generate a username for you. To get this username you have to press the link button on the bridge and send a POST command to /api with a body of the user you want to whitelist such as `{"devicetype":"hue_light_puzzle_app#iphone Joe"}`. This is a security step to limit control of your lights to someone who has physical access to the bridge. The app will have a helper message to instruct the user to authenticate themselves by pressing the button on their bridge. If you send the request without pressing the link button, you will get an error message like:
```
[
	{
		"error": {
			"type": 101,
			"address": "",
			"description": "link button not pressed"
		}
	}
]
```

#### Use PUT to change a setting

##### Test 1: turning a light on and off
Perform another GET command to get info about a light with the url: `https://<bridge ip address>/api/<username>/lights/1`

The JSON response shows all the resources that light has. My lights are on right now, so the "on" attribute is set to "true". To test turning off the light, I will use a PUT command with the url: `https://<bridge ip address>/api/<username>/lights/1/state` and a JSON body of `{"on":false}`.

I get the JSON response:
```
[
	{
		"success": {
			"/lights/1/state/on": false
		}
	}
]
```
Also the light in my room has turned off. The first test has been a success, on to test 2.

##### Test 2: Change the colors
I modified the last command to turn the light back on and also set the hue and saturation to change the color.
* Brightness values are from 0 to 254.
* Hue values are from 0 to 65535.
* Saturation values are from 0 to 254.
* XY values are from 0 to 1
* Color Point values are from 2000K/500 mirek (warm) to 6500K/153 mirek (cold)

The json body looks like this: `{"on":true, "sat":254, "bri":254,"hue":30000}`

I get the response:
```
[
	{
		"success": {
			"/lights/1/state/on": true
		}
	},
	{
		"success": {
			"/lights/1/state/hue": 30000
		}
	},
	{
		"success": {
			"/lights/1/state/sat": 254
		}
	},
	{
		"success": {
			"/lights/1/state/bri": 254
		}
	}
]
```
Also the light turns back on and has changed to the new color. The second test has also been a success. Now that I've learned the basics of working with Philips Hue Lights, the next step is to learn how to send these commands as part of an app.

### Connecting an iOS app to a Philips Hue Bridge
A Hue developer account is needed to send commands to the bridge through an app and to access the [core concepts page](https://developers.meethue.com/develop/get-started-2/core-concepts/) on the developer site to learn how to connect to the bridge through an app.


## Resources available
* /lights - contains all the light resources
* /groups - contains all the groups
* /config - contains all the configuration items
* /schedules - contains all the schedules
* /scenes - contains all the scenes
* /sensors - contains all the sensors
* /rules - contains all the rules

All the properties that can be controlled are in the /state resource of a light.

### Potential puzzle problems
#### Transition Time
The documentation says that it could take a few minutes for the bridge to update its representation. There is also a transition time to the new state of 400 milliseconds. If you want the light to respond quickly to a state change, set the transition time in the light state to zero milliseconds.
#### Command Maximums
Also you can’t send commands to the lights too fast. A max of around 10 commands per second to the /lights resource. For /groups commands the maximum is 1 per second. It is generally more efficient to use the lights API (unless there are >= 12 lights).
All the lights have to be in range, but the lights act as a signal repeater.
#### QuickStart iOS Hue App
I may not be able to use the QuickStart iOS app as a base for the app. I downloaded the Hue Apple SDK by Philips : https://github.com/PhilipsHue/PhilipsHueSDK-iOS-OSX
I tried running the QuickStart iOS app, but it did not compile. The last update was 4 years ago, so I'm not surprised, but this means getting started won't be quick.

### Notes
Send ‘ON’ attribute to a light only once so as not to slow the responsiveness of the bridge.
Sending multiple conflicting parameters at once: xy beats ct beats hue, sat.
The hue API is a RESTful interface, but local (only accessible on your local WiFi network).
provides an SDK for iOS.
The SDK provide utility methods to convert RGB to one of the accepted settings.
The SDK also helps with finding the address of the bridge on the local network.
The SDK is in Objective-C.

### Next Steps
- [ ] Learn about the Hue SDK for iOS:https://developers.meethue.com/develop/tools-and-sdks/hue-sdk-guide-for-ios-and-os-x/
- [ ] research incorporating the Hue SDK into an existing iOS project
- [ ] create a blank iOS project
- [ ] add an on/off button
- [ ] create a network connection between iOS app and Hue bridge
- [ ] hook up the button to the Hue bridge to turn the lights on/off
