Lighthouse iOS SDK
===============

The Lighthouse iOS SDK is designed to be simple to develop with, allowing you to easily integrate Lighthouse iBeacon software into your apps. For more info about Lighthouse visit the [Lighthouse Website](http://lighthousebeacon.io)

## Install Guide

Installing the client should be a breeze. If it's not, please let us know at [team@lighthousebeacon.io](mailto:team@lighthousebeacon.io)

### Universal Library

Our recommended way of installing the Lighthouse SDK is to use the universal library we've created. With this, you'll be able to instrument your app regardless of whether or not it uses ARC (Automatic Reference Counting).

### Download

Download the contents of this repository.
Within the folder called "Lighthouse" should be the following contents:

- libLighthouse.a
- Lighthouse.h

### Add Files to Xcode
Just drag the Lighthouse folder into your Xcode project.

### Add Required Frameworks
Go into your app's target’s Build Phases screen and add the following (if they don't already exist) to the "Link Binary With Libraries" section.

- CoreLocation.framework
- CoreBluetooth.framework

### Build Settings
Under your application targets "Build Settings" configuration find the "Other Linker Flags" property and set it to "-ObjC".

### Import LighthouseManager.h
You'll need to import the LighthouseManager.h header into the files that contain code relating to lighthouse. You can either add them to individual files or include it in your ApplicationName-Prefix.pch file.

	#import "LighthouseManager.h"

### Add Background Modes
In your application plist file (often called "ApplicationName-Info.plist") add a row for
"Required background modes" of type Array. It then needs:

- "App registers for location updates"
- "App communicates using CoreBluetooth"

These are needed to receive the background beacon notifications.

### Compile
Try and compile. It should work!

If it doesn't work, let us know at [team@lighthousebeacon.io](mailto:team@lighthousebeacon.io) and one of us will help you right away.

### Get Application ID & Keys
If you haven't done so already, login to Ligthouse to get your Application ID & Keys.


## Instrumentation
Now it's time to actually use the client!

### Register Client
Register the LighthouseManager with your Application ID and access keys. The recommended place to do this is in one of your application delegates. Here's some example code:

	- (void)applicationDidBecomeActive:(UIApplication *)application {
		// Enable logging (optional)
		[LighthouseManager enableLogging];

		// Configure the manager using Lighthouse keys
		[[LighthouseManager sharedInstance] configure:@{
			@"appId": @"your_app_id",
			@"appKey": @"your_app_key",
			@"appToken": @"your_app_token",
			@"appVersion": @"your_app_version", // optional
			@"uuids": @[@"beacon_uuids_to_monitor"]
		}];

		// Start monitoring
		[[LighthouseManager sharedInstance] launch];

		// Request permission (you can do this whenever you like - it will ask user for location permission - possibly you want to wait until they reach a certain section of your app)
		[[LighthouseManager sharedInstance] requestPermission];
	}


From now on, in your code, you can just reference the shared client by calling [LighthouseManager sharedInstance].

### Application Suspend & Resume
To best conserve battery and correctly trigger Lighthouse events you'll need to add the following lines to your application delegate.

	- (void)applicationWillResignActive:(UIApplication *)application {
		[[LighthouseManager sharedInstance] suspend];
	}

	- (void)applicationDidBecomeActive:(UIApplication *)application {
		[[LighthouseManager sharedInstance] activate];
	}

	- (void)applicationWillTerminate:(UIApplication *)application {
		[[LighthouseManager sharedInstance] terminate];
	}

### Debugging
The Lighthouse iOS SDK code does a lot of logging, but it's turned off by default. If you'd like to see the log lines generated by your usage of the client, you can enable logging easily:

	[LighthouseManager enableLogging];

Just put this at any point before you use LighthouseManager. A good place is in your application delegate.

To disable logging (by default it is disabled), simply call:

	[LighthouseManager disableLogging];

### Events
The Lighthouse iOS SDK doesn't keep all the beacon events for itself, after all sharing is caring. Using the SDK you can subscribe to events such as when a user enters, exits or ranges (usually every second when within a beacon). These events contain a NSDictionary full of useful information such as the beacon ids, distance, accuracy, user's compass direction, etc.

You can then listen to these NSNotifications using the following example commands:

	[[LighthouseManager sharedInstance] subscribe:@"LighthouseDidEnterBeacon" observer:self selector:@selector(didEnterBeacon:)];
	[[LighthouseManager sharedInstance] subscribe:@"LighthouseDidExitBeacon" observer:self selector:@selector(didExitBeacon:)];
	[[LighthouseManager sharedInstance] subscribe:@"LighthouseDidRangeBeacon" observer:self selector:@selector(didRangeBeacon:)];
	[[LighthouseManager sharedInstance] subscribe:@"LighthouseDidReceiveNotification" observer:self selector:@selector(didReceiveNotification:)];
	[[LighthouseManager sharedInstance] subscribe:@"LighthouseDidReceiveCampaign" observer:self selector:@selector(didReceiveCampaign:)];

	#pragma mark - Notification Handers

	- (void)didEnterBeacon:(NSDictionary *)data {
		NSLog(@"didEnterBeacon %@", data);
	}

	- (void)didExitBeacon:(NSDictionary *)data {
		NSLog(@"didExitBeacon %@", data);
	}

	- (void)didRangeBeacon:(NSDictionary *)data {
		NSLog(@"didRangeBeacon %@", data);
	}

	- (void)didReceiveNotification:(NSDictionary *)data {
		NSLog(@"didReceiveNotification %@", data);
		// You could listen to this event and show a loading indicator while you make a request for detailed campaign data
	}

	- (void)didReceiveCampaign:(NSDictionary *)data {
		NSLog(@"didReceiveCampaign %@", data);
		// The reponse for a request for detailed campaign data
	}

In every observer's dealloc you MUST run the following to prevent leaking memory and crashes. Basically you are saying to Lighthouse that you want to unsubscribe this object from all the events it previously subscribed to.

	- (void)dealloc {
		[[LighthouseManager sharedInstance] removeAll:self];
	}

You can also unsubscribe to any event at any time by using the UUID returned from the subscribe method. See below:

	NSString *subscribeKeyForEnterEvent = [[LighthouseManager sharedInstance] subscribe:@"LighthouseDidEnterBeacon" observer:self selector:@selector(didEnterBeacon:)];

	[[LighthouseManager sharedInstance] unsubscribe:subscribeKeyForEnterEvent];

NOTE: The old method of using NSNotification observing prior to 1.1.5 is no longer available. We hope this new method is more useful and give you more control with events.

You'll also see the "LighthouseDidReceiveCampaign" event has been added in 1.1.5. This event is triggered whenever a request to get more detail about a campaign is made. More details in the next "Detailed Campaign Data" section.

### Event Payloads

This section outlines the NSDictionary structure of the data parameter passed from the events you subscribe to.
The following three events have the same payload structure: LighthouseDidEnterBeacon, LighthouseDidExitBeacon, LighthouseDidRangeBeacon

	@{
		@"properties": NSDictionary, // These are your custom set properties
		@"device": NSString, // Derived from [[[UIDevice currentDevice] identifierForVendor] UUIDString]
		@"key": NSString, // String of uuid-major-minor all joined together
		@"uuid": NSString, // String of uuid
		@"major": NSString, // String of major
		@"minor": NSString, // String of minot
		@"direction": CLLocationDirection (double), // Derived from CLLocationDirection trueHeading,
		@"moving": NSNumber, // Boolean in NSNumber format as to whether the device is stationary or if the user is moving (ie, walking). In testing, distance becomes more accurate if the device is moving.
		@"distance": NSNumber, // Average meters from beacon from the last 5 readings (is used as approximate guide only, beacon accuracy isn't very reliable and based on environment). Sometimes this value can appear as a negative number which should be discarded as it means an accurate distance couldn't be determined. Derived from CLBeacon accuracy. This value is very unreliable, we wouldn't recommend depending on it.
		@"mode": NSString, // Articulates whether you are in production or development mode. Either @"production" or @"development",
		@"state": NSString, // Based on [[UIApplication sharedApplication] applicationState]. Value is either @"Active", @"Inactive", @"Background",
		@"timestamp": NSNumber, // Under the hood it is @([[NSDate date] timeIntervalSince1970])
	}

The LighthouseDidReceiveNotification event has this structure:

	@{
		@"properties": NSDictionary, // These are your custom set properties
		@"device": NSString, // Derived from [[[UIDevice currentDevice] identifierForVendor] UUIDString]
		@"state": NSString, // Based on [[UIApplication sharedApplication] applicationState]. Value is either @"Active", @"Inactive", @"Background",
		@"mode": NSString, // Articulates whether you are in production or development mode. Either @"production" or @"development",
		@"notification": NSDictionary, // userInfo dictionary that came with push notification
		@"timezoneOffset": NSNumber, // Under the hood it is @([[NSTimeZone systemTimeZone] secondsFromGMT]),
		@"timestamp": NSNumber, // Under the hood it is @([[NSDate date] timeIntervalSince1970])
	}

The LighthouseDidReceiveCampaign event has the following structure. More fields will be added to the campaign dictionary in the future as the campaigns become more advanced.

	@{
		@"notification": NSDictionary, // Whole dictionary of the one described above in LighthouseDidReceiveNotification
		@"campaign": @{
			@"_id": NSString, // campaign Id from the Lighthouse Backend
			@"meta": Either NSDictionary or NSString based on whether vaild json was entered in the campaign creation stage. It it can parse it as JSON it will be NSDictionary, but if it can't it just spits out the NSString.
		}
	}

### Detailed Campaign Data
In 1.1.5 we added the ability to retrieve detailed campaign data that is too large to fit in the 256 byte limit of a push notification. This is useful if you are using the "Meta" field in the Advanced Fields section of campaign creation. In future you will also use this method for getting images, videos, rules etc from the API. To get detailed campaign data you need to give Lighthouse context of the notification to get the corresponding campaign data for.

	NSDictionary *mostRecentNotification = [[LighthouseManager sharedInstance] notifications][0];
	[[LighthouseManager sharedInstance] campaign:mostRecentNotification];

This will make an API call to the Lighthouse server. On completion it will use the "LighthouseDidReceiveCampaign" event so you'll need to subscribe to it.

	[[LighthouseManager sharedInstance] subscribe:@"LighthouseDidReceiveCampaign" observer:self selector:@selector(didReceiveCampaign:)];

	....

	- (void)didReceiveCampaign:(NSDictionary *)data {
		NSLog(@"didReceiveCampaign %@", data);
	}

An example of the data returned is below. You'll see it includes the campaign data as well as the original notification so you can determine which notification was used as context in retrieving the campaign.

	{
		campaign: {
			_id: '537c314c43083e7358000022',
			meta: {
				foo: 'bar'
			}
		},
		notification: {
			device: '576C3487-45D2-487D-95AF-4241C1F57671',
			mode: 'development',
			notification: {
				aps: {
					alert: 'Welcome to the Railway Hotel. Enjoy $5 Pints for the next hour',
					badge: 1,
					sound: 'default'
				},
				d: {
					a: '537c35bc43083e7358000034',
					c: '537c314c43083e7358000022'
				},
				s: 'lh'
			},
			properties: {},
			state: 'Active',
			timestamp: '1400649148.683397',
			timezoneOffset: 36000
		}
	}

NOTE: If valid JSON data is added in the 'meta' field of admin campaign console then we will automatically parse it for transmission down to the iOS app so you get an NSDictionary of values. However if its not valid and our JSON parser can't parse it then a string will be sent. You should check the meta type as to whether it is an NSString or NSDictionary before assuming it's a NSDictionary otherwise your application could run into a bug/crash.

### Transmission
There are times when you will want to leverage the Lighthouse SDK to easily access iBeacon events but you may not want it to transmit data to Lighthouse API. This could be the case during development or possibly if a user does not want their movements tracked. You can use the following methods to control whether the SDK transmits data to the server. By default it is enabled.

	[LighthouseManager enableTransmission];

Just put this at any point before you use LighthouseManager. A good place is in your application delegate.

To disable transmission to the Ligthhouse server, simply call:

	[LighthouseManager disableTransmission];

### Production Mode
By default the Lighthouse SDK operates in Development mode. Every analytic sent to Lighthouse API will record which mode the device was in when it was sent. This means that we can send push notifications using the correct certificates for the device. Also in future we'll investigate how you can filter and test your analytics more by segregating development data from your production data.

	[LighthouseManager enableProduction];

Just put this at any point before you use LighthouseManager. A good place is in your application delegate.

To put it back into development mode, you can disable production at any time (by default production is disabled):

	[LighthouseManager disableProduction];

### Requesting Permission
You've all had that experience when you launch an app for the first time and get bombarded with permission requests for notifications, location, movement, microphones - the list goes on. In some situations you may wish to avoid this and only request location permission at a relevant time in the user experience. We added the ```requestPermission``` method for this reason. Lighthouse won't work until permission has been requested (and accepted by the user).

	[[LighthouseManager sharedInstance] requestPermission];

You can also check whether permission has been requested previously using:

	[[LighthouseManager sharedInstance] hasRequestedPermission];

### Requesting Push Notification Permission
Likewise with location permission above you can request push notification permission from a user with the following methods. If your app has already requested permission this will operate silently in the background, it is still important to run them though if you want to receive push notifications for campaigns from the Lighthouse API.

Put this after Lighthouse has been configured and launched, you can request immediately at startup or later after you have explained to the user that you are about to ask permission.

	[[LighthouseManager sharedInstance] requestPushNotifications];

You will also need to add the following methods in your AppDelegate so Lighthouse can respond to registeration and notification events.

	- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
		...

		// If we have a notification in the payload get it out of launch options
		NSDictionary *notification = [launchOptions objectForKey:UIApplicationLaunchOptionsRemoteNotificationKey];
		if (notification) {
			[[LighthouseManager sharedInstance] didReceiveRemoteNotification:notification];
		}

		...
	}

	- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo {
		[[LighthouseManager sharedInstance] didReceiveRemoteNotification:userInfo];
	}

	- (void)application:(UIApplication *)app didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {
		[[LighthouseManager sharedInstance] didRegisterForRemoteNotificationsWithDeviceToken:deviceToken];
	}

	- (void)application:(UIApplication *)app didFailToRegisterForRemoteNotificationsWithError:(NSError *)err {
		[[LighthouseManager sharedInstance] didRegisterForRemoteNotificationsWithDeviceToken:nil];
	}

You can also view an array of all notifications received (you could for instance pull the last object off that array and show to users)

	[[LighthouseManager sharedInstance] notifications]

It can also be helpful to have access to the devices push notification token. You can use this in the Admin Integration page to test your push notifications certificates. Just copy/paste the value into the Test Push Notification section and hit send.

	[[LighthouseManager sharedInstance] pushNotificationToken]

### Generating Push Notification Certificates for Admin Portal

Ther are a few steps involved in getting push notification certificates from Apple Developer Portal into a usable format for the application. This [push notification tutorial](http://www.raywenderlich.com/32960/apple-push-notification-services-in-ios-6-tutorial-part-1) is an excellent resource to guide through the process of generating Certificate Signing Requests, App IDs and SSL Certs. After you've got your certificate from that process, use the following commands (which are based off the ones mentioned in that tutorial) to get the certificate and your private key into the correct format for the Lighthouse portal.

Using the aps_development.cer certificate downloaded from Apple we convert it to PEM format.

	openssl x509 -in aps_development.cer -inform der -out development-cert.pem

Now that we have development-private-key.p12 we convert it to PEM format.

	openssl pkcs12 -nocerts -out development-key-enc.pem -in development-private-key.p12

The key is encrypted so we have to remove this for lighthouse to be able to send the notification.

	openssl rsa -in development-key-enc.pem -out development-key.pem

Test it is working by connecting to Apple's sandbox environment using the newly created cert and key.

	openssl s_client -connect gateway.sandbox.push.apple.com:2195 -cert development-cert.pem -key development-key.pem

If all is working, open the cert (development-cert.pem) and key (development-key.pem) files and copy their contents into the Integration page on [admin.lighthousebeacon.com.au](http://admin.lighthousebeacon.com.au).

You can now use the Test Push Notification section to send a test notification to your device. You can also use the same process for generating your production keys (replacing "development" in filenames with "production")

## FAQs

What is the default behaviour when a beacon is detected when the app is in the foreground? Can this be customised?

If a campaign is triggered then a push notification will be sent to the device, but because the app is open it won't make a noise or display an alert, the AppDelegate "didReceiveRemoteNotification" code will still trigger though so you can handle this situation. If no campaign is triggered for the beacon and the app is in the foreground then it will still fire the events such as "LighthouseDidEnterBeacon", "LighthouseDidExitBeacon", "LighthouseDidRangeBeacon" if you are subscribed to them.

## Changelog

##### 1.1.5

+ Removed use of NSNotifications for communicating events, instead a subscribe/unsubscribe system has been implement
+ Added a "campaign" method to retrieve detailed information about a campaign. This is needed to access advanced fields such as "meta" which are too large to send in a 256 byte push notification.

##### 1.1.4

+ Updated push notification code for receiving notifications and reading them within the application

##### 1.1.3

+ Added ability to switch between development and production (particularly helpful for push notifications)

##### 1.1.2

+ Added timezoneOffset to payload sent to API

##### 1.1.1

+ Updated internal domain name that data is POSTed to.

##### 1.1.0

+ Added push notification permission support.
+ Added enable/disable transmission to Lighthouse API support.
+ README updated to explain above additions. Also added "Build Settings" in installation instructions.

##### 1.0.0

+ Initial SDK Release

### Questions & Support

If you have any questions, bugs, or suggestions, please email them to [team@lighthousebeacon.io](mailto:team@lighthousebeacon.io). We'd love to hear your feedback and ideas!
