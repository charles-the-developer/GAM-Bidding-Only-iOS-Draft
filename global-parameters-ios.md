# Global Parameters iOS


This page documents various global parameters you can set on the Prebid SDK. It describes the various properties and methods of the Prebid SDK, First-Party Data used for Targeting Ads, and consent options under various privacy regulations and User Identity API.  


## Prebid SDK Properties and Methods

The `Prebid` class is a singleton that enables you to apply global settings.


### Properties

`prebidServerAccountId`: [String] containing the Prebid Server account ID.

`prebidServerHost`: [String] This property is crucial as it contains the configuration for your Prebid Server host, with which the Prebid SDK will communicate. Depending on your specific needs, you can choose from the system-defined Prebid Server hosts or define your own custom Prebid Server host.

`shareGeoLocation`: [Optiona]l [Bool]; if this flag is True AND the app collects the user’s geographical location data, Prebid Mobile will send the user’s geographical location data to the Prebid Server. Suppose this flag is false OR the app does not collect the user’s geographical location data. In that case, Prebid Mobile will not populate any user's geographical location information in the call to Prebid Server. The default setting is false.

`logLevel`: This property controls the level of logging output to the console.

`timeoutMillis`: [Int]

The Prebid timeout (available in SDK v1.2+), when set, returns control to the ad server SDK to fetch an ad once the specified timeout has elapsed. Because the Prebid SDK gets bids from the Prebid Server in one payload, setting the Prebid timeout too low can reduce demand, resulting in a potential negative revenue impact.

`creativeFactoryTimeout`: [int] This parameter controls how long a banner creative has to load before it is considered a failure.

`creativeFactoryTimeoutPreRenderContent`: This parameter controls how much time video and interstitial creatives have to load before it is considered a failure.

`storedAuctionResponse`: Set as type string, stored auction responses signal Prebid Server to respond with a static response matching the storedAuctionResponse found in the Prebid Server Database, useful for debugging and integration testing. No bid requests will be sent to bidders when a matching storedAuctionResponse is found. For more information on how stored auction responses work, refer to the [written description on github issue 133.](https://github.com/prebid/prebid-mobile-android/issues/133)

`pbsDebug:` adds the debug flag (“test”:1) on the outbound http call to the Prebid Server. The test:1 flag signals to the Prebid Server to emit the full resolved request (resolving any Stored Request IDs) and the full Bid Request and Bid Response to and from each bidder.


### Methods


#### Stored Responses

`addStoredBidResponse`: This method takes two parameters



* `bidder`: Bidder name as defined by Prebid Server bid adapter of type string.
* `responseId`: Configuration ID used in the Prebid Server Database to store static bid responses.

Stored Bid Responses are similar to Stored Auction Responses in that they signal to Prebid Server to respond with a static pre-defined response, except Stored Bid Responses is done at the bidder level, with bid requests sent out for any bidders not specified in the bidder parameter. For more information on how stored auction responses work, refer to [github issue 133](https://github.com/prebid/prebid-mobile-android/issues/133).


```
func addStoredBidResponse(bidder: String, responseId: String)
```


`clearStoredBidResponses`: This method clears any stored bid responses. It doesn’t take any parameters.

Custom Headers

`addCustomHeader`: This method enables you to customize the HTTP call to the prebid server. It takes two parameters



* `name`: a name for the custom header
* `value`: a value for the custom header

`clearCustomHeaders`: Allows you to clear any custom headers you have previously set.


### SDK Properties and Methods Example usage


## Consent Management

_This section describes how publishers can provide info on user consent to the SDK and how SDK behaves under different kinds of restrictions._


### App Tracking Transparency

You should follow Apple's Guidelines on implementing [App Tracking Transparency](https://developer.apple.com/documentation/apptrackingtransparency). The Prebid SDK automatically sends ATT signals, so no prebid-specific work is required.


### GDPR

There are two ways to provide information on user consent to the Prebid SDK according to The Transparency & Consent Framework (TCF)



* Explicitly via SDK API: publishers can provide TCF data via SDK’s Targeting API
* Implicitly set through the CMP: SDK reads the TCF data stored in the `UserDefaults`

**Important**: The SDK explicitly prioritizes the values set through the API over those stored by the CMP and treats the latter as a fallback.  If the publisher provides TCF data both ways, the values set through the API will be sent to the PBS, and values stored by the CMP will be ignored. 


#### Setting GDPR Values with the API

SDK provides three properties to set the TCF values explicitly, though this method is not preferred. Ideally, the Consent Management Platform will set these values – see the next section.

If you need to set the values directly, here's how to indicate that the user is subject to GDPR:

Swift: 


```
Targeting.shared.subjectToGDPR = false
```


To provide the consent string:

Swift:


```
Targeting.shared.gdprConsentString = "BOMyQRvOMyQRvABABBAAABAAAAAAEA"
```


To set the purpose consent: 

Swift:


```
Targeting.shared.purposeConsents = "100000000000000000000000"
```



#### Getting Consent Values from the CMP

SDK reads the values for the following keys from the `UserDefaults` object 



* **IABTCF_gdprApplies** - indicates whether the user is subject to GDPR
* **IABTCF_TCString** - full encoded TC string
* **IABTCF_PurposeConsents** - indicates the consent status for the purpose. 

For more detailed information, read the [In-App Details section](https://github.com/InteractiveAdvertisingBureau/GDPR-Transparency-and-Consent-Framework/blob/master/TCFv2/IAB%20Tech%20Lab%20-%20CMP%20API%20v2.md#in-app-details) of the TCF.

Here is an example of a `UserDefaults` file with TCF signals: 

**Note**: Publishers shouldn’t explicitly assign values for these keys (unless they have a custom-developed CMP).  It is a privilege of the Consent Management Provider (CMP) SDKs. If the publisher wants to provide this data to the Prebid SDK, they should use the explicit APIs described above.

Here are several essential implementation details on how SDK processes CMP values:



* Prebid SDK reads CMP values from the `UserDefaults `during the initialization 
* Prebid SDK doesn’t verify or validate CMP values in any way
* Prebid SDK reads CMP values for each bid request, so the latest value is always used.


### CCPA

Prebid SDK reads and sends CCPA signals according to the [US Privacy User Signal Mechanism](https://github.com/InteractiveAdvertisingBureau/USPrivacy/blob/master/CCPA/USP%20API.md) and [OpenRTB extension](https://github.com/InteractiveAdvertisingBureau/USPrivacy/blob/7f4f1b2931cca03bd4d91373bbf440071823f257/CCPA/OpenRTB%20Extension%20for%20USPrivacy.md). 

SDK reads the value for the `IABUSPrivacy_String` key from the `UserDefaults` and sends it in the `regs.ext.us_privacy` object of the OpenRTB request.


### COPPA

Prebid SDK follows the OpenRTB 2.6 spec and provides an API to indicate whether the current user falls under COPPA regulation. Publishers can set the respective flag using the targeting API: 

Swift:


```
Targeting.shared.subjectToCOPPA = true
```


SDK passes this flag in the `regs.coppa` object of the bid requests. 

SDK relies on the section “**7.5 COPPA Regulation Flag**” of the OpenRTB spec and does not refrain from reading user and device data since suppressing these fields should be an exchange obligation. 


### GPP

Note: GPP is supported starting from version Prebid SDK 2.0.6.

Prebid SDK reads and sends GPP signals according to the [CMP API Specification](https://github.com/InteractiveAdvertisingBureau/Global-Privacy-Platform/blob/main/Core/CMP%20API%20Specification.md) and OpenRTB 2.6 spec (so far [GPP in proposal](https://github.com/prebid/prebid-server/issues/2442)). 

SDK reads the value for the `IABGPP_HDR_GppString` key in the `UserDefaults` and sends it in the `regs.gpp` object of the OpenRTB request.

## First Party User Data

Prebid provides following functions to manage First Party User Data:

```swift
func addUserData(key: String, value: String)

func updateUserData(key: String, value: Set<String>)

func removeUserData(forKey: String)

func clearUserData()
```

Example:

```swift
Targeting.shared.addUserData(key: "globalUserDataKey1", value: "globalUserDataValue1")
```

### First Party Inventory (Context) Data

Prebid provides following functions to manage First Party Inventory Data:

```swift
func addContextData(key: String, value: String)

func updateContextData(key: String, value: Set<String>)

func removeContextData(forKey: String)

func clearContextData()
```

Example:

```swift
Targeting.shared.addContextData(key: "globalContextDataKey1", value: "globalContextDataValue1")
```

### Access Control

The First Party Data Access Control List provides a methods to restrict access to first party data to a supplied list of bidders.

```swift
func addBidderToAccessControlList(_ bidderName: String)

func removeBidderFromAccessControlList(_ bidderName: String)

func clearAccessControlList()
```

Example:

```swift
Targeting.shared.addBidderToAccessControlList(Prebid.bidderNameRubiconProject)
```

## User Identity API

Prebid SDK supports two interfaces to pass / maintain User IDs and ID vendor details:

* Real-time in Prebid SDK's API field externalUserIdArray
* Store User Id(s) in local storage

Any identity vendor's details in local storage will be sent over to Prebid Server as is, unadulterated. If data is sent in the API and entered into local storage, the API detail will prevail.

### Prebid SDK API Access

Prebid SDK supports passing an array of UserID(s) at auction time in the field externalUserIdArray, which is globally scoped. Setting the externalUserIdArray object once per user session is sufficient, as these values would be used in all consecutive ad auctions in the same session.

```swift
public var externalUserIdArray = [ExternalUserId]()
```

**Exmaples**

```swift
// User Id from External Third Party Sources
var externalUserIdArray = [ExternalUserId]()

externalUserIdArray.append(ExternalUserId(source: "adserver.org", identifier: "111111111111", ext: ["rtiPartner" : "TDID"]))
externalUserIdArray.append(ExternalUserId(source: "netid.de", identifier: "999888777")) 
externalUserIdArray.append(ExternalUserId(source: "criteo.com", identifier: "_fl7bV96WjZsbiUyQnJlQ3g4ckh5a1N")) 
externalUserIdArray.append(ExternalUserId(source: "liveramp.com", identifier: "AjfowMv4ZHZQJFM8TpiUnYEyA81Vdgg"))
externalUserIdArray.append(ExternalUserId(source: "sharedid.org", identifier: "111111111111", atype: 1, ext: ["third" : "01ERJWE5FS4RAZKG6SKQ3ZYSKV"]))

Prebid.shared.externalUserIdArray = externalUserIdArray
```

### Local Storage

Prebid SDK provides a local storage interface to set, retrieve, or update an array of user IDs with associated identity vendor details. Prebid SDK will retrieve and pass User IDs and ID vendor details to PBS if values are present in local storage. The main difference between the Prebid API interface and the local storage interface is data storage persistence. Local Storage data will persist across user sessions, whereas the Prebid API interface (externalUserIdArray) persists only for the user session. If a vendor's details are passed into local storage and the Prebid API simultaneously, the Prebid  API data (externalUserIdArray) will prevail.

Prebid SDK Provides five functions to handle User ID details:

```swift
public func storeExternalUserId(_ externalUserId: ExternalUserId)

public func fetchStoredExternalUserIds() -> [ExternalUserId]?

public func fetchStoredExternalUserId(_ source : String) -> ExternalUserId?

public func removeStoredExternalUserId(_ source : String)

public func removeStoredExternalUserIds()
```

**Examples**

```swift
//Set External User ID
Targeting.shared.storeExternalUserId(ExternalUserId(source: "sharedid.org", identifier: "111111111111", atype: 1, ext: ["third" : "01ERJWE5FS4RAZKG6SKQ3ZYSKV"]))

//Get External User ID
let externalUserIdSharedId = Targeting.shared.fetchStoredExternalUserId("sharedid.org")

//Get All External User IDs
let externalUserIdsArray = Targeting.shared.fetchStoredExternalUserIds()

//Remove External UserID
Targeting.shared.removeStoredExternalUserId("sharedid.org")

//Remove All External UserID
Targeting.shared.removeStoredExternalUserIds()
```