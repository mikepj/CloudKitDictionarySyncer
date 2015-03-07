# CloudKitDictionarySyncer

CloudKitDictionarySyncer is a utility that you can use in your app to save an NSDictionary to both a local plist file and 
remotely in the user's CloudKit private database. If CloudKit is not available, because the user is offline, not logged in or whatever 
 , the NSDictionary will be saved to the local plist only.
 
At app startup, when you load the dictionary, CloudKitDictionarySyncer will return a single dictionary if the plist and iCloud version are 
  identical. If not, a Conflict tuple containing both dictionaries will be returned. Handle the conflict in a way appropriate for your app, 
  for example by merging the data in both dictionaries or by simply choosing the last saved dictionary (last method not recommended). After
  the conflict is solved, you should save the dictionary immediately. This will resync the local plist dictionary and the iCloud dictionary.
  
##Installation

Add the file [CloudKitDictionarySyncer.swift](CloudKitDictionarySyncer/CloudKitDictionarySyncer.swift) to your project


##Usage

### Setup CloudKit

 1. Turn on iCloud in the Capabilities tab for your build target. Make sure to enable CloudKit using the checkbox that shows up.
 
 2. In the [iCloud dashboard](https://icloud.developer.apple.com/dashboard/), add a new Record type called Plists. In the Plists type, add two string attributes: dictname and plistxml. The 
 resulting Record Type should look like this: ![Dashboard example](/images/icloudrecordtype.png?raw=true "Dashboard example")
 
 If you have trouble setting up Cloudkit, read this excellent [blog post](http://shrikar.com/blog/2014/10/12/ios8-cloudkit-tutorial-part-1/)

### Your client code

#### Create a syncer object 

Create a syncer object for each dictionary that you want to persist/sync. If you set the debug flag to true you will get some
 debug log messages in your console log.

```swift
let exampledictSyncer = CloudKitDictionarySyncer(dictname: "exampledict", debug: true)
```

#### Loading your dictionary.
This should be done only once per app session. Pass a function that can receive a LoadResult enum, that will contain either a NSMutableDictionary or a Conflict tuple.
  
```swift
self.exampledictSyncer.loadDictionary(onComplete: {
    loadResult in
    switch loadResult {
        case .Dict(let loadeddict):
            // No conflict
            self.dict = loadeddict
        case .Conflict(let plistdict, let clouddict, let latest):
            // Handle conflict, for example by merging.
            self.dict = myMergeFunction(plistdict, clouddict)
            // Save the merged dict immediately        
            self.syncer.saveDictionary(self.dict, onComplete: {
                status in
                println("Resaved merged dict. Save status = \(status)")
            })
    }
})
```  

#### Saving your NSDictionary. 
To be safe, you should do this whenever you have updated your dictionary. For the battery savers and the more adventurous, save when the
app goes into background. Pass a function that takes a String, that will contain an informational status message from the save operation.
  
```swift
self.exampledictSyncer.saveDictionary(self.dict, onComplete: {
    status in
    println("Save status = \(status)")
})
```  

### Handling conflicts
  
  
##Example Project

For more info, have a look at the example project in this repo.

##Feedback and Contribution

All feedback and contribution is very appreciated. Please send pull requests, create issues
or just send an email to [mats.melke@gmail.com](mailto:mats.melke@gmail.com).

##Copyrights

* Copyright (c) 2015- Mats Melke. Please see LICENSE.txt for details.
