## Introduction

iBeacon is a new technology that extends Location Services in iOS. It enables a low cost, low powered indoor proximity monitoring for devices. A beacon is simply a device that transmits a signal that allows other devices to detect its proximity. An iBeacon users Bluetooth Low Energy advertisement packets to advertise its availability and the devices can then listen and decode the advertisement packet to find more details about the beacon. 

## How can I get some beacons?
There are a lot of possible options for buying low cost beacons for creating wonderful indoor experiences. There's [Estimote](http://estimote.com/) which is probably the most famous one in the market right now. But there are plenty of others too including [Kontakt](http://kontakt.io), [Radius Networks](http://www.radiusnetworks.com/) or you could even [roll out your own](http://www.theregister.co.uk/2013/11/29/feature_diy_apple_ibeacons/) without much trouble. If you ever want to decide which one to buy, there is a really good comparision [here](http://beekn.net/guide-to-ibeacons/). 

## Integrating with your apps
Lets get to the playing part now. Although the iBeacon specification is released by Apple, the use is not limited to iOS devices. iOS 7 does come up with inbuilt support for detecting iBeacons with the CoreLocation framework, but its not limited to this. You can always scan for bluetooth devices (and therefore the iBeacons) with a compatible Android device, or even a simple bluetooth LE adapter like [ConnectBlue OBS 421](http://support.connectblue.com/display/PRODBTSPA/Bluetooth+Low+Energy+Serial+Port+Adapter+-+Getting+Started).

Lets talk a bit about how to detect beacons with iOS. The CoreLocation framework makes it really easy to [monitor beacon regions](https://developer.apple.com/library/ios/documentation/userexperience/conceptual/LocationAwarenessPG/RegionMonitoring/RegionMonitoring.html#//apple_ref/doc/uid/TP40009497-CH9-SW7) and get notified when the device enters/exits a region. Once the user is in the beacon region, the device can even [range for iBeacons](https://developer.apple.com/library/ios/documentation/userexperience/conceptual/LocationAwarenessPG/RegionMonitoring/RegionMonitoring.html#//apple_ref/doc/uid/TP40009497-CH9-SW15) in that region to determine the relative proximity of one or more beacons in the region and to be notified when that distance changes. All you need to do is create regions and start monitoring and ranging. Don't beleive me, take a look. You would need the UUID of the beacons that you are going to monitor and optionally the major and minor identifiers if you want to monitor specific beacons. 

```objective-c
// Create a region to be monitored. You can omit the major and/or minor to create regions with more than one beacon.
CLBeaconRegion *region = [[CLBeaconRegion alloc] initWithProximityUUID:beaconUUID major:2 minor:1 identifier:@"My-Beacon-Region"];

// Register the beacon region with the location manager.
[self.locManager startMonitoringForRegion:beaconRegion];

⋮

// Delegate method from the CLLocationManagerDelegate protocol.
- (void)locationManager:(CLLocationManager *)manager didRangeBeacons:(NSArray *)beacons inRegion:(CLBeaconRegion *)region {
   if ([beacons count] > 0) {
      CLBeacon *nearestExhibit = [beacons firstObject];
 
      // Present the exhibit-specific UI only when
      // the user is relatively close to the exhibit.
      if (CLProximityNear == nearestExhibit.proximity) {
         [self presentExhibitInfoWithMajorValue:nearestExhibit.major.integerValue];
      } else {
         [self dismissExhibitInfo];
   }
}

- (void)locationManager:(CLLocationManager *)manager didEnterRegion:(CLRegion *)region {
	// Do something. e.g. Show a region specific notification or start ranging for this 
	// region of the app is in to receive detailed proximity info.
}

- (void)locationManager:(CLLocationManager *)manager didExitRegion:(CLRegion *)region {
	// Do something. e.g. Stop ranging for this region to save battery.
}
```

The default UUID for Estimote beacons is `B9407F30-F5F8-466E-AFF9-25556B57FE6D` and for Kontakt, its `f7826da6-4fa2-4e98-8024-bc5b71e0893e`.

## Conclusion
Enough code, lets see how it works. First thing to note: Your app *might not* receive the notifications for boundary crossings immediately, specially if it is in the background. There is a detailed study about this behaviour [here](http://developer.radiusnetworks.com/2013/11/13/ibeacon-monitoring-in-the-background-and-foreground.html), but here's what I have noticed. If the app is in the foreground and ranging, boundary crossings delivered within a second. If not, it usually takes some time (a few minutes). If you are trying to navigate users with beacons, try to keep ranging always on instead of turning it on only when the user enters a region. If the app is in background, its even more unreliable. It might take several minutes to get the boundary crossing notification. You can improve it a bit by setting  `notifyEntryStateOnDisplay` to `YES` in which case your app is immediately notified of a boundary crossing when the device display is switched on (e.g. when user unlocks the phone).