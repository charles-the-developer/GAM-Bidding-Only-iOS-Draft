# Global Parameters

This page documents various global parameters you can set on the Prebid SDK. It describes the properties and methods of the Prebid SDK that allow you to supply important parameters to the header bidding auction.

## Prebid Global Properties and Methods

The `Prebid` class is a singleton that enables you to apply global settings. It covers:

- attributes that are defined during initialization (e.g. the Prebid Server connection)
- values affecting the behavior of the Prebid SDK (e.g. timeout)
- items influencing the OpenRTB output (e.g. shareGeoLocation)

### Global Properties

(TBD - where are these set? Need an example. )

{: .table .table-bordered .table-striped }
| Parameter | Scope | Type | Purpose | Description | Example |
| --- | --- | --- | --- | --- |
| prebidServerAccountId | either | string | init | (iOS only) Your Prebid Server team will tell you whether this is required or not and if so, the value. | "abc123" |
| prebidServerHost | optional | enum | init | (iOS only) This can take the values "Appnexus", "Rubicon", or "Custom". If "Custom", you need to use the setCustomPrebidServerUrl() method to set a URL. This is where the Prebid SDK will send the auction information. Your Prebid Server team will tell you which value to use. The default is "Custom". | "Custom" |
| customStatusEndpoint | optional | string | init | (iOS only) Use this URL to check the status of Prebid Server. The default status endpoint is the PBS URL appended with '/status'. | "https://prebidserver<wbr>.example<wbr>.com/custom<wbr>/status" |
| shareGeoLocation | optional | boolean | ORTB | (iOS only) If this flag is true AND the app collects the user’s geographical location data, Prebid Mobile will send the user’s lat/long geographical location data to the Prebid Server. The default is false. | `true` |
| locationUpdatesEnabled | optional | boolean | ORTB | If true, the SDK will periodically try to listen for location updates. Default is `false`. | `true` |
| logLevel | optional | enum | SDK control | (iOS only) This property controls the level of logging output to the console. The value can be .error, .info, .debug, .verbose, .warn, .severe, and .info. The default is `.debug`. | `.error` |
| debugLogFileEnabled | optional | boolean | SDK control | If set to true, the output of PrebidMobile's internal logger is written to a text file. Default is `false`. | `true` |
| timeoutMillis | optional | integer | init | (iOS only) (SDK v1.2+) The Prebid SDK timeout. When this number of milliseconds passes, the Prebid SDK returns control to the ad server SDK to fetch an ad without Prebid bids. | 1000 |
| creativeFactoryTimeout | optional | integer | SDK control | (iOS only) Controls how long a banner creative has to load before it is considered a failure. (TBD - is this milliseconds? Need the default value) | 2000 |
| creativeFactoryTimeoutPreRenderContent | optional | integer | SDK control | (iOS only) Controls how much time video and interstitial creatives have to load before it is considered a failure. (TBD - is this milliseconds? Need the default value) | 2000 |
| storedAuctionResponse | optional | string | ORTB | (iOS only) For testing and debugging. Get this value from your Prebid Server team. It signals Prebid Server to respond with a static response from the Prebid Server Database. See [more information on stored auction responses](/prebid-server/endpoints/openrtb2/pbs-endpoint-auction.html#stored-responses). | "abc123-sar-test-320x50" |
| pbsDebug | optional | boolean | ORTB | (iOS only) Adds the debug flag (`test`:1) on the outbound http call to the Prebid Server. The `test` flag signals to the Prebid Server to emit the full resolved request and the full Bid Request and Bid Response to and from each bidder. | true |
| shouldAssignNativeAssetID | optional | boolean | ORTB | (iOS only) Whether to automatically assign an assetID for a Native ad. Default is `false`. | true |
| useCacheForReportingWithRenderingAPI | optional | boolean | ORTB | Indicates whether PBS should cache the bid on the server side. If the value is `true` the Prebid SDK will make the cache request to retrieve the cached asset. Default is `false`. | true |
| useExternalClickthroughBrowser | optional | boolean | SDK control | (iOS only) Controls whether to use PrebidMobile's in-app browser or the Safari App for displaying ad clickthrough content. Default is false. | true |
| useExternalClickthroughBrowser | optional | enum | ORTB | (iOS only) Indicates the type of browser opened upon clicking the creative in an app. This corresponds to the OpenRTB imp.clickbrowser field. Values are "embedded" and "native". Default is "native". | "native".
| includeWinners | optional | boolean | ORTB | (iOS only) If `true`, Prebid sdk will add `includewinners` flag inside the targeting object described in [PBS Documentation](prebid-server/endpoints/openrtb2/pbs-endpoint-auction.html#targeting) . Default is `false`. | `true` |
| includeBidderKeys | optional | boolean | ORTB | (iOS only) If `true`, Prebid sdk will add `includebidderkeys` flag inside the targeting object described in [PBS Documentation](/prebid-server/endpoints/openrtb2/pbs-endpoint-auction.html#targeting) . Default is `false`. | `true` |
| isCoppaEnabled | optional | boolean | ORTB | (Android only) Set this to true if this app is aimed at children. It sets the ORTB COPPA flag. Default is false. | `true` |
| useExternalBrowser | optional | boolean | ORTB | (Android only) TBD? Defaults to `false`. | `true` |
| sendMraidSupportParams | optional | boolean | ORTB | (Android only) TBD If `true`, the SDK sends "af=3,5", indicating support for MRAID. Defaults to `true`. | `false` |

### Global Methods

#### setPrebidServerAccountId() (Android only)

Your Prebid Server team will tell you whether this is required or not and if so, the value. See the initialization page for [Android](/prebid-mobile/pbm-api/android/code-integration-android.html).

#### setPrebidServerHost()  (Android only)

This is where the Prebid SDK will send the auction information.

Signature:

```kotlin
func setPrebidServerHost(host: String)
```

Parameters: 

{: .table .table-bordered .table-striped }
| Parameter | Scope | Type | Description | Example |
| --- | --- | --- | --- | --- |
| host | required | object | Host.APPNEXUS, Host.RUBICON, Host.createCustomHost(PREBID_SERVER_AUCTION_ENDPOINT) | Host.createCustomHost<wbr>("https://prebidserver<wbr>.example.com<wbr>/openrtb2/auction") |

Examples:

```kotlin
PrebidMobile.setPrebidServerHost(Host.APPNEXUS)
PrebidMobile.setPrebidServerHost(Host.RUBICON)
PrebidMobile.setPrebidServerHost(Host.createCustomHost(https://prebidserver.example.com/openrtb2/auction))
```

#### setCustomStatusEndpoint() (Android only)

Signature:

```kotlin
    public static void setCustomStatusEndpoint(String url) {
```

Parameters:

{: .table .table-bordered .table-striped }
| Parameter | Scope | Type | Description | Example |
| --- | --- | --- | --- | --- |
| url | required | string | Use this URL to check the status of Prebid Server. The default status endpoint is the PBS URL appended with '/status'. | "https://prebidserver<wbr>.example<wbr>.com/custom<wbr>/status" |

#### setTimeoutMillis() (Android only)

The Prebid SDK timeout. When this number of milliseconds passes, the Prebid SDK returns control to the ad server SDK to fetch an ad without Prebid bids. See the initialization page for [Android](/prebid-mobile/pbm-api/android/code-integration-android.html).

#### setShareGeoLocation() (Android only)

If this flag is true AND the app collects the user’s geographical location data, Prebid Mobile will send the user’s lat/long geographical location data to the Prebid Server. The default is false.

#### setCustomPrebidServerUrl() (iOS only)

Defines which Prebid Server to connect to. See the initialization pages for [iOS](/prebid-mobile/pbm-api/ios/code-integration-ios.html) or [Android](/prebid-mobile/pbm-api/android/code-integration-android.html).

#### setIncludeWinnersFlag() (Android only)

If `true`, Prebid sdk will add the `includewinners` flag inside the targeting object described in [PBS Documentation](prebid-server/endpoints/openrtb2/pbs-endpoint-auction.html#targeting) . This is needed if you've set up line items in an ad server in "Send Top Bid" mode, as it's what creates the key value pairs like `hb_pb`. 

Signature:

```kotlin
    public static void setIncludeWinnersFlag(boolean includeWinners)
```

Parameters:

{: .table .table-bordered .table-striped }
| Parameter | Scope | Type | Description | Example |
| --- | --- | --- | --- | --- |
| includeWinners | required | boolean | If `true`, Prebid sdk will add `includewinners` flag inside the targeting object described in [PBS Documentation](prebid-server/endpoints/openrtb2/pbs-endpoint-auction.html#targeting) . Default is `false`. | `true` |

#### setIncludeBidderKeysFlag() (Android only)

If `true`, Prebid sdk will add the `includebidderkeys` flag inside the targeting object described in [PBS Documentation](prebid-server/endpoints/openrtb2/pbs-endpoint-auction.html#targeting) . This is needed if you've set up line items in an ad server in "Send All Bids" mode, as it's what creates the key value pairs like `hb_pb_bidderA`. 

Signature:

```kotlin
    public static boolean setIncludeBidderKeysFlag(boolean includeBidderKeys) {
```

Parameters:

{: .table .table-bordered .table-striped }
| Parameter | Scope | Type | Description | Example |
| --- | --- | --- | --- | --- |
| includeBidderKeys | required | boolean | If `true`, Prebid sdk will add `includewinners` flag inside the targeting object described in [PBS Documentation](prebid-server/endpoints/openrtb2/pbs-endpoint-auction.html#targeting) . Default is `false`. | `true` |

#### setStoredAuctionResponse() (Android only)

For testing and debugging. Get this value from your Prebid Server team. It signals Prebid Server to respond with a static response from the Prebid Server Database. 
See [more information on stored auction responses](/prebid-server/endpoints/openrtb2/pbs-endpoint-auction.html#stored-responses).

Signature:

```kotlin
public static void setStoredAuctionResponse(@Nullable String storedAuctionResponse)
```

Parameters:

{: .table .table-bordered .table-striped }
| Parameter | Scope | Type | Description | Example |
| --- | --- | --- | --- | --- |
| storedAuctionResponse | required | string | Key as defined by Prebid Server. Get this value from your Prebid Server team. | "abc123-sar-test-320x50" |

#### addStoredBidResponse()

Stored Bid Responses are for testing and debugging similar to Stored Auction Responses (see the Global Properties above). They signal Prebid Server to respond with a static pre-defined response, except Stored Bid Responses actually exercise the bidder adapter. For more information on how stored bid responses work, refer to the [Prebid Server endpoint doc](/prebid-server/endpoints/openrtb2/pbs-endpoint-auction.html#stored-responses). Your Prebid Server team will help you determine how best to setup test and debug.

Signature:

```swift
func addStoredBidResponse(bidder: String, responseId: String)
```

Parameters:

{: .table .table-bordered .table-striped }
| Parameter | Scope | Type | Description | Example |
| --- | --- | --- | --- | --- |
| bidder | required | string | Bidder name as defined by Prebid Server | "bidderA" |
| responseId | required | string | ID used in the Prebid Server Database. Get this value from your Prebid Server team. | "abc123-sbr-test-300x250" |

#### clearStoredBidResponses()

This method clears any stored bid responses. It doesn’t take any parameters.

Signature:

```swift
func clearStoredBidResponses()
```

Parameters: none.

#### setLogLevel (Android only)

Controls the level of logging output to the console.

Signature:

```kotlin
    public static void setLogLevel(LogLevel logLevel)
```

Parameters:

{: .table .table-bordered .table-striped }
| Parameter | Scope | Type | Description | Example |
| --- | --- | --- | --- | --- |
| logLevel | required | enum | The value can be ERROR, DEBUG, WARN, and NONE. The default is `.NONE`. | `.DEBUG` |

#### setPbsDebug() (Android only)

Adds the debug flag (`test`:1) on the outbound http call to the Prebid Server. The `test` flag signals to the Prebid Server to emit the full resolved request and the full Bid Request and Bid Response to and from each bidder.

Signature:

```kotlin
public static void setPbsDebug(boolean pbsDebug)
```

Parameters:

{: .table .table-bordered .table-striped }
| Parameter | Scope | Type | Description | Example |
| --- | --- | --- | --- | --- |
| pbsDebug | required | boolean | Turn on/off debug mode. Defaults to `false`. | `true` |

#### assignNativeAssetID() (Android only)

Whether to automatically assign an assetID for a Native ad. Default is `false`.

Signature:

```kotlin
    public static void assignNativeAssetID(boolean assignNativeAssetID) {
```

Parameters:

{: .table .table-bordered .table-striped }
| Parameter | Scope | Type | Description | Example |
| --- | --- | --- | --- | --- |
| assignNativeAssetID | required | boolean | Whether to automatically assign an assetID for a Native ad. Defaults to `false`. | `true` |

#### setCreativeFactoryTimeout() (Android only)

Controls how long a banner creative has to load before it is considered a failure.

Signature:

```kotlin
    public static void setCreativeFactoryTimeout(int creativeFactoryTimeout)
```

Parameters:

{: .table .table-bordered .table-striped }
| Parameter | Scope | Type | Description | Example |
| --- | --- | --- | --- | --- |
| creativeFactoryTimeout | required | integer | Controls how long a banner creative has to load before it is considered a failure. (TBD - is this milliseconds? Need the default value) | 2000 |

#### setCreativeFactoryTimeoutPreRenderContent() (Android only)

Controls how much time video and interstitial creatives have to load before it is considered a failure.

Signature:

```kotlin
    public static void setCreativeFactoryTimeoutPreRenderContent(int creativeFactoryTimeoutPreRenderContent)
```

Parameters:

{: .table .table-bordered .table-striped }
| Parameter | Scope | Type | Description | Example |
| --- | --- | --- | --- | --- |
| creativeFactoryTimeoutPreRenderContent | required | integer | Controls how much time video and interstitial creatives have to load before it is considered a failure. (TBD - is this milliseconds? Need the default value) | 2000 |

#### addCustomHeader() (iOS only)

This method enables you to customize the HTTP call to Prebid Server. See [setCustomHeaders()](TBD) for the Android version.

Signature:

```swift
func addCustomHeader(name: String, value: String)
```

Parameters:

{: .table .table-bordered .table-striped }
| Parameter | Scope | Type | Description | Example |
| --- | --- | --- | --- | --- |
| name | required | string | Name of the custom header | "X-mycustomheader" |
| value | required | string | Value for the custom header | "customvalue" |

#### setCustomHeaders() (Android only)

This method enables you to customize the HTTP call to Prebid Server. See [addCustomHeaders()](TBD) for the iOS version

Signature:

```kotlin
    public static void setCustomHeaders(@Nullable HashMap<String, String> customHeaders)
```

Parameters:

{: .table .table-bordered .table-striped }
| Parameter | Scope | Type | Description | Example |
| --- | --- | --- | --- | --- |
| customHeaders | required | hashmap | Hashmap of custom headers | "X-mycustomheader: customvalue" |

#### clearCustomHeaders()

Allows you to clear any custom headers you have previously set.

Signature:

```swift
func clearCustomHeaders()
```

Parameters: none

#### setCustomLogger() (Android only)

TBD

Signature:

```kotlin
    public static void setCustomLogger(@NonNull PrebidLogger logger)
```

Parameters:

{: .table .table-bordered .table-striped }
| Parameter | Scope | Type | Description | Example |
| --- | --- | --- | --- | --- |
| logger | required | ? | TBD | TBD |

## Consent Management

This section describes how app developers can provide info on user consent to the Prebid SDK and how SDK behaves under different kinds of restrictions.

### iOS App Tracking Transparency

You should follow Apple's Guidelines on implementing [App Tracking Transparency](https://developer.apple.com/documentation/apptrackingtransparency). The Prebid SDK automatically sends ATT signals, so no Prebid-specific work is required.

### GDPR

There are two ways to provide information on user consent to the Prebid SDK according to the [IAB's Transparency & Consent Framework (TCF)](https://iabeurope.eu/understanding-the-upcoming-transparency-consent-framework-v2-2/)

- Explicitly via Prebid SDK API: publishers can provide TCF data via Prebid SDK’s Targeting API
- Implicitly set through the CMP: Prebid SDK reads the TCF data stored in the `UserDefaults`. This is the preferred approach.

**Important**: The Prebid SDK explicitly prioritizes the values set through the API over those stored by the CMP.  If the publisher provides TCF data both ways, the values set through the API will be sent to the PBS, and values stored by the CMP will be ignored. 

#### Setting GDPR Values with the API

Prebid SDK provides three properties to set TCF consent values explicitly, though this method is not preferred. Ideally, the Consent Management Platform will set these values – see the next section.

If you need to set the values directly, here's how to indicate that the user is subject to GDPR:

Swift: 

```swift
Targeting.shared.subjectToGDPR = false
```

To provide the consent string:

Swift:

```swift
Targeting.shared.gdprConsentString = "BOMyQRvOMyQRvABABBAAABAAAAAAEA"
```

To set the purpose consent: 

Swift:

```swift
Targeting.shared.purposeConsents = "100000000000000000000000"
```

#### Getting Consent Values from the CMP

Prebid SDK reads the values for the following keys from the `UserDefaults` (iOS) or `SharedPreferences` (Android) object:

- **IABTCF_gdprApplies** - indicates whether the user is subject to GDPR
- **IABTCF_TCString** - full encoded TC string
- **IABTCF_PurposeConsents** - indicates the consent status for the purpose. 

For more detailed information, read the [In-App Details section](https://github.com/InteractiveAdvertisingBureau/GDPR-Transparency-and-Consent-Framework/blob/master/TCFv2/IAB%20Tech%20Lab%20-%20CMP%20API%20v2.md#in-app-details) of the TCF.

**Note**: Publishers shouldn’t explicitly assign values for these keys (unless they have a custom-developed CMP).  It is a privilege of the Consent Management Provider (CMP) SDKs. If the publisher wants to provide this data to the Prebid SDK, they should use the explicit APIs described above.

Here's how Prebid SDK processes CMP values:

- It reads CMP values during the initialization and on each bid request, so the latest value is always used.
- It doesn’t verify or validate CMP values in any way

### CCPA / USP

The California Consumer Protection Act drove the IAB to implement the "US Privacy" protocol.

Prebid SDK reads and sends USP/CCPA signals according to the [US Privacy User Signal Mechanism](https://github.com/InteractiveAdvertisingBureau/USPrivacy/blob/master/CCPA/USP%20API.md) and [OpenRTB extension](https://github.com/InteractiveAdvertisingBureau/USPrivacy/blob/7f4f1b2931cca03bd4d91373bbf440071823f257/CCPA/OpenRTB%20Extension%20for%20USPrivacy.md). 

Prebid SDK reads the value for the `IABUSPrivacy_String` key from the `UserDefaults` (iOS) or `SharedPreferences` (Android) and sends it in the `regs.ext.us_privacy` object of the OpenRTB request.

### COPPA

The Children's Online Privacy Protection Act of the United States is a way for content producers to declare their content is aimed at children, which invokes additional privacy protections.

Prebid SDK follows the OpenRTB 2.6 spec and provides an API to indicate whether the current content falls under COPPA regulation. Publishers can set the respective flag using the targeting API: 

Swift:

```swift
Targeting.shared.subjectToCOPPA = true
```

Prebid SDK passes this flag in the `regs.coppa` object of the bid requests.

If you're app developer setting this COPPA flag, we recommend you also:

- set the `shareGeoLocation` property to false
- avoid passing any sensitive first party data

### GPP

A Consent Management Platform (CMP) utilizing [IAB's Global Privacy Protocol](https://iabtechlab.com/gpp/) is a comprehensive way for apps to manage user consent across multiple regulatory environments.

Since version 2.0.6, Prebid SDK reads and sends GPP signals:

- The GPP string is read from IABGPP_HDR_GppString in `UserDefaults` (iOS) or `SharedPreferences` (Android). It is sent to Prebid Server on `regs.gpp`.
- The GPP Section ID is likewise read from IABGPP_GppSID. It is sent to Prebid Server on `regs.gpp_sid`.

## First Party Data

First Party Data (FPD) is information about the app or user known by the developer that may be of interest to advertisers. 

- User FPD includes details about a specific user like "frequent user" or "job title". This data if often subject to regulatory control, so needs to be specified as user-specific data. Note that some attributes like health status are limited in some regions. App developers are strongly advised to speak with their legal counsel before passing User FPD.
- Inventory FPD includes details about the particular part of the app where the ad will displayed like "sports/basketball" or "editor 5-star rating".

### User FPD

Prebid SDK provides following functions to manage First Party User Data:

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

### Inventory FPD

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

### Controlling Bidder Access to FPD

Prebid Server will let you control which bidders are allowed access to First Party Data. Prebid SDK collects this an Access Control List with the following methods:

```swift
func addBidderToAccessControlList(_ bidderName: String)

func removeBidderFromAccessControlList(_ bidderName: String)

func clearAccessControlList()
```

Example:

```swift
Targeting.shared.addBidderToAccessControlList(Prebid.bidderNameRubiconProject)
```

## User Identity

Mobile apps traditionally rely on IDFA-type device IDs for advertising, but there are other User ID systems available to app developers and more will be made available in the future. Prebid SDK supports two ways to maintain User ID details:

- A global property - in this approach, the app developer sets the IDs while initializing the Prebid SDK. This data persists only for the user session.
- Local storage - the developer can choose to store the IDs persistently in local storage and Prebid SDK will utilize them on each bid request.

Any identity vendor's details in local storage will be sent to Prebid Server unadulterated. If user IDs are set both in the property and entered into local storage, the property data will prevail.

### Storing IDs in a Property

Prebid SDK supports passing an array of UserID(s) at auction time in the Prebid global field `externalUserIdArray`. Setting the `externalUserIdArray` object once per user session is sufficient unless one of the values changes.

```swift
public var externalUserIdArray = [ExternalUserId]()
```

**Examples**

```swift
// User Id from External Third Party Sources
var externalUserIdArray = [ExternalUserId]()

externalUserIdArray.append(ExternalUserId(source: "adserver.org", identifier: "111111111111", ext: ["rtiPartner" : "TDID"]))
externalUserIdArray.append(ExternalUserId(source: "netid.de", identifier: "999888777")) 
externalUserIdArray.append(ExternalUserId(source: "criteo.com", identifier: "_fl7bV96WjZsbiUyQnJlQ3g4ckh5a1N")) 
externalUserIdArray.append(ExternalUserId(source: "liveramp.com", identifier: "AjfowMv4ZHZQJFM8TpiUnYEyA81Vdgg"))
externalUserIdArray.append(ExternalUserId(source: "sharedid.org", identifier: "111111111111", atype: 1))

Prebid.shared.externalUserIdArray = externalUserIdArray
```

```kotlin
setExternalUserIds(List<ExternalUserId> externalUserIds)
```

### Storing IDs in Local Storage

Prebid SDK provides a local storage interface to set, retrieve, or update an array of user IDs with associated identity vendor details. It will then retrieve and pass these User IDs to Prebid Server on each auction, even on the next user session.

Prebid SDK Provides several functions to handle User ID details within the local storage:

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
Targeting.shared.storeExternalUserId(ExternalUserId(source: "sharedid.org", identifier: "111111111111", atype: 1))

//Get External User ID
let externalUserIdSharedId = Targeting.shared.fetchStoredExternalUserId("sharedid.org")

//Get All External User IDs
let externalUserIdsArray = Targeting.shared.fetchStoredExternalUserIds()

//Remove External UserID
Targeting.shared.removeStoredExternalUserId("sharedid.org")

//Remove All External UserID
Targeting.shared.removeStoredExternalUserIds()
```
