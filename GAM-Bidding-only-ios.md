# iOS GAM Bidding-Only Integration iOS

## What this How-To-Guide covers

This page explains how to integrate the Prebid SDK into your app with the GMA SDK. In this integration scenario:

- **Prebid SDK** and the **Prebid server** handle the bidding and auction process.
- **GAM** and the **GMA SDK** manage the ad inventory, select the winning ad to display based on various factors, render the ad content in the app.  

## Prerequisites

The GAM Bidding Only Integration recipe assumes that you have the following ingredients:

- **Google Ad Manager Account** - Your GAM account allows you to manage and serve ads within your mobile app. Within this account you'll need to configure your ad inventory and create orders for serving ads within your app. This involves defining ad units (spaces within your app where ads will be displayed) and setting up orders and line items to deliver ads to those units. See [Prebid's AdOps Guide](/adops/before-you-start.html) for more information.
- **Google Mobile Ads (GMA) SDK** - This refers to the software development kit provided by your ad server (in this case, Google Ad Manager). You need to ensure that you have the latest version of the Ad Server SDK supported by Prebid SDK. This SDK integration is necessary to communicate with the ad server and display ads in your app.
- **Prebid SDK** - You will need the latest version of the Prebid Mobile SDK for either [Android](/prebid/prebid-mobile-android) or [iOS](/prebid/prebid-mobile-ios). This page will help you integrate the Prebid SDK into your app.
- **Prebid Server** - You will need a server running the [Prebid Server software](/prebid-server/use-cases/pbs-sdk.html). You can set up your own Prebid Server or work with [Prebid Server managed service](https://prebid.org/managed-services/). Prebid Server will provide you with the following:
    - Stores configuration - rather than hardcoding all the details of your current business arrangements in the app, Prebid Server stores which bidders you're currently working with, their inventory details, and other settings that can be changed without updating your app.
    - Server-side auction - the server will make the connections to multiple auction bidding partners so the app doesn't have to.
    - Privacy regulation tools - the server can help your legal team meet different regulatory needs in different jurisdictions by configuring various protocols and anonyimization activities.

## High-level overview

The following is a high-level explanation of how the ad bidding-auction-rendering process works in this integration scenario.

![GAM Bidding Only Integration Overview](https://docs.prebid.org/assets/images/prebid-mobile/prebid-in-app-bidding-overview-prebid-original.png)

### Step 1: Initial bid request

When your mobile app wants to show an ad, the Prebid SDK sends a basic request to the Prebid Server. This request includes details about the ad space (like size and location) but not specific bidders.

### Step 2: Full bid request

The Prebid Server takes the incomplete information received from the Prebid SDK and adds more details to create a full request for each bidder configured by the app developer's business team. It then sends this request to these various demand partners. Each bidder evaluates the request and decides if they want to bid on it. If they do, they send back their bid, including the price they're willing to pay and the ad creative - the actual content like image or video to be displayed if they win.

### Step 3: Bid Response and caching

After validating the received bids, Prebid Server caches them for a limited time and sends them back to the Prebid SDK with [ad server key-value pairs](/adops/key-values.html) attached.

### Step 4: Bid details passed GMA SDK

The Prebid SDK passes the bid details to the adunit created in the Google Mobile Ads (GMA) SDK.

### Step 5: Ad server request

The GMA SDK makes a request to Google Ad Manager (GAM) for an ad to be chosen. It includes a number of targeting key/value pairs correspondinging to bid price, format, size, and other details.

### Step 6: GAM ad source resolution

GAM resolves the competing ad sources including the special line items created for Prebid bids. If the Prebid line item associated with the winning bid is selected, GAM returns the creative associated with it to the GMA SDK. 

### Step 7: Ad Rendering

Depending on the ad type (banner, native, or video), the rendering process begins. For banners and native ads, the GMA SDK renders the ad markup using Prebid Universal Creative (PUC). For video ads, the VAST URL (a format for serving video ads) is handed off to the video player on the mobile device.

### Step 8: Prebid Cache lookup

Before displaying the ad, the PUC or the video player may check the Prebid Cache Server to load the creative content associated with the winning bid has been cached. If it has, the content is fetched and rendered.

## Tradeoffs and alternative approaches

Another way to integrate GAM into your app is with the [GAM Auto-Rendering integation](TBD). 

(TBD: I don't recall why auto-rendering exists - asking Yuriy.)

The main advantage of this integration approach is that it is more flexible. As a developer, you have greater control over how you handle Prebid bids.

The major drawback however is that this approach requires more lines of code compared to alternatives like the rendering approach. 

## Initialization and General Parameters

Assuming your app is already built with GMA AdUnits, the technical implementation of the Prebid SDK into your app will involve 4 major steps:

1. Initialize the Prebid SDK - create a connection to your Prebid Server.
2. Set Global Parameters - let bidders know important data about the page, privacy consent, and other settings.
3. Link Prebid AdUnit code to your GMA AdUnits - for any adunits that your business team wants to connect to the Prebid auction.
4. Test

[links to be added once pages are written]

## AdUnit-Specific instructions

This section describes the integration details for different ad formats. In each scenario, you'll be asked for a `configId` - this is a key worked out with your Prebid Server provider. It's used at runtime to pull in the bidders and parameters specific to this adunit. Depending on your Prebid Server partner, it may be a UUID or constructed out of parts like your account number and adunit name.

### Format: HTML Banner

To integrate HTML banner ads into the app you should use the `BannerAdUnit` class. It makes bid requests to Prebid Server and provide targeting keywords of the winning bid to the GMA SDK.

**Integration Example (Swift):**

```swift
func createAd() {
        // 1. Create a BannerAdUnit
        adUnit = BannerAdUnit(configId: CONFIG_ID, size: adSize)
        adUnit.setAutoRefreshMillis(time: 30000)
        
        // 2. Configure banner parameters
        let parameters = BannerParameters()
        parameters.api = [Signals.Api.MRAID_2, Signals.Api.OMID_1]
        adUnit.parameters = parameters
        
        // 3. Create a GAMBannerView
        gamBanner = GAMBannerView(adSize: GADAdSizeFromCGSize(adSize))
        gamBanner.adUnitID = AD_UNIT_ID
        gamBanner.rootViewController = self
        gamBanner.delegate = self
        
        // Add GMA SDK banner view to the app UI
        bannerView?.addSubview(gamBanner)
        
        // 4. Make a bid request to Prebid Server
        let gamRequest = GAMRequest()
        adUnit.fetchDemand(adObject: gamRequest) { [weak self] resultCode in
            DemoLogger.shared.info("Prebid demand fetch for GAM \(resultCode.name())")
            
            // 5. Load GAM Ad
            self?.gamBanner.load(gamRequest)
        }
    }
```

If you want to support several ad sizes, you also need to implement `GADBannerViewDelegate` to adjust banner view size according to the creative size

```swift
func bannerViewDidReceiveAd(_ bannerView: GADBannerView) {

    // 6. Resize ad view if needed
    AdViewUtils.findPrebidCreativeSize(bannerView, success: { size in
        guard let bannerView = bannerView as? GAMBannerView else { return }
        bannerView.resize(GADAdSizeFromCGSize(size))
    }, failure: { (error) in
       // The received ad is not Prebid’s one 
    })
}
```

> [!NOTE]
>  in case you use a single-size banner (as opposed to multi-size), i.e. 300x250 - you don’t need to make a call to the `AdViewUtils.findPrebidCreativeSize` routine - because you already know the size of the creative, however you still need to make a call to `bannerView.resize` because the creative in GAM has the 1x1 size by default and without this call it will be rendered, but as a pixel. 

> [!NOTE]
> Make sure you process all possible cases in the  `AdViewUtils.findPrebidCreativeSize` callbacks (both success and failure).  Sometimes you might not get the size of the creative (or a failure callback) - it simply means that this is not a Prebid creative.  It means that you still need to render the creative, but you most likely don’t need to resize it.

#### Step 1: Create a `BannerAdUnit`

Initialize the `BannerAdUnit` with the properties:

- `configId` - an ID of the AdUnit Level Stored Request in the Prebid Server
- `adSize` - the size of the AdUnit which will be used in the bid request.

If you need to bid on other ad sizes as well use `addAdditionalSize()` method to provide more relevant sizes. All of them will be added to the bid request. 

#### Step 2: Configure banner parameters

Use `BannerParameter` to customize bid request. 

The `api` property is dedicated to signal the supported creative frameworks to bidder. The selected values will be added to the bid request according to the [OpenRTB 2.6](https://github.com/InteractiveAdvertisingBureau/openrtb2.x/blob/main/2.6.md) spec. GMA SDK supports creatives using the following api frameworks:

- **5** or **Signals.Api.MRAID_2** : MRAID-2 support signal
- **7** or **Signals.Api.OMID_1** : signals OMSDK support

> _Note_: MRAID 3 is in beta so far. 

#### Step 3: Create a GAMBannerView

Follow the [GMA SDK documentation](https://developers.google.com/ad-manager/mobile-ads-sdk/ios/banner) to integrate a banner ad unit. 

#### Step 4: Make the bid request

The _fetchDemand_ method makes a bid request to the Prebid Server. The `GAMRequest` object provided to this method must be the one used in the next step to make the GAM ad request.

When Prebid Server responds, Prebid SDK will set the targeting keywords of the winning bid into provided object.

#### Step 5: Call the ad server

Next, request the ad from GAM. If the `GAMRequest` object contains targeting keywords, the respective Prebid line item may be returned from GAM, and GMA SDK will render its creative. 

Ensure that you call the _load_ method with the same `GAMRequest` object that you passed to the _fetchDemand_ method on the previous step. Otherwise, the ad request won't contain targeting keywords, and Prebid bids won't be displayed.

#### Step 6: Adjust the ad view size

Once an app receives a signal that an ad is loaded, you should use the method `AdViewUtils.findPrebidCreativeSize` to verify whether it's Prebid Server’s ad and resize the ad slot respectively to the creative's properties. 

### Format: Video Banner (aka Outstream Video)

This section describes integration of a video creative that serves into a display AdUnit. This differs from a video ad that will display inside a dedicated video player, also called [Instream Video](link TBD).

To integrate video banner ads into the app you need to use the `VideoAdUnit` class. It is dedicated to configure and make bid requests to Prebid Server and provide targeting keywords of the winning bid to the GMA SDK.

Integration Example (Swift):

```swift
    func createAd() {
        // 1. Create a VideoAdUnit
        adUnit = VideoAdUnit(configId: CONFIG_ID, size: adSize)
        
        // 2. Configure video parameters
        let parameters = VideoParameters()
        parameters.mimes = ["video/mp4"]
        parameters.protocols = [Signals.Protocols.VAST_2_0]
        parameters.playbackMethod = [Signals.PlaybackMethod.AutoPlaySoundOff]
        parameters.placement = Signals.Placement.InBanner
        adUnit.parameters = parameters
        
        // 3. Create a GAMBannerView
        gamBanner = GAMBannerView(adSize: GADAdSizeFromCGSize(adSize))
        gamBanner.adUnitID = AD_UNIT_ID
        gamBanner.rootViewController = self
        gamBanner.delegate = self
        
        // Add GMA SDK banner view to the app UI
        bannerView.addSubview(gamBanner)
        bannerView.backgroundColor = .clear
        
        // 4. Make a bid request to Prebid Server
        let gamRequest = GAMRequest()
        adUnit.fetchDemand(adObject: gamRequest) { [weak self] resultCode in
            DemoLogger.shared.info("Prebid demand fetch for GAM \(resultCode.name())")
            
            // 5. Load GAM Ad
            self?.gamBanner.load(gamRequest)
        }
    }
```

#### Step 1: Create a VideoAdUnit

Initialize the `VideoAdUnit` with properties:

- `configId` - an ID of the Ad Unit Level Stored Request on Prebid Server
- `adSize` - the size of the ad unit which will be used in the bid request.

#### Step 2: Configure video parameters

Using the `VideoParameters` you can customize the bid request for `VideoAdUnit`. 

##### plcmt and/or placement
{:.no_toc}

The [OpenRTB 2.6](https://github.com/InteractiveAdvertisingBureau/openrtb2.x/blob/main/2.6.md) protocol defines the `plcmt` and `placement` (deprecated) as an integer array. You may use an enum for easier readability: 

- **1** or **InStream** : In-Stream Played before, during or after the streaming video content that the consumer has requested (e.g., Pre-roll, Mid-roll, Post-roll).
- **2** or **InBanner** : In-Banner placement exists within a web banner that leverages the banner space to deliver a video experience as opposed to another static or rich media format. The format relies on the existence of display ad inventory on the page for its delivery.
- **3** or **InArticle** : In-Article placement loads and plays dynamically between paragraphs of editorial content; existing as a standalone branded message.
- **4** or **InFeed** : In-Feed placement is found in content, social, or product feeds.
- **5** or **Slider**, **Floating** or **Interstitial**: Open RTB supports one of three values for option 5 as either Slider, Floating or Interstitial. If an enum value is supplied in placement, bidders will receive value 5 for placement type and assume to be interstitial with the instl flag set to 1.

##### api
{:.no_toc}

The _api_ property may be used to add values for which API Frameworks may be used in the bid response as defined in [OpenRTB 2.6](https://github.com/InteractiveAdvertisingBureau/openrtb2.x/blob/main/2.6.md). The supported values for GMA SDK integration are:

- **5** or **Signals.Api.MRAID_2** : MRAID-2 support signal
- **7** or **Signals.Api.OMID_1**  : signals OMSDK support

##### maxBitrate
{:.no_toc}

Integer representing the OpenRTB 2.6 maximum bit rate in Kbps.

##### minBitrate
{:.no_toc}

Integer representing the OpenRTB 2.6 minimum bit rate in Kbps.

##### maxDuration
{:.no_toc}

Integer representing the OpenRTB 2.6 maximum video ad duration in seconds.

##### minDuration
{:.no_toc}

Integer representing the OpenRTB 2.6 minimum video ad duration in seconds.

##### mimes
{:.no_toc}

Array of strings representing the supported OpenRTB 2.6 content MIME types (e.g., “video/x-ms-wmv”, “video/mp4”).

##### playbackMethod
{:.no_toc}

Array of OpenRTB 2.6 playback methods. If none are specified, any method may be used. Only one method is typically used in practice. It is advised to use only the first element of the array. 

- **1** or **Signals.PlaybackMethod.AutoPlaySoundOn** : Initiates on Page Load with Sound On
- **2** or **Signals.PlaybackMethod.AutoPlaySoundOff** : Initiates on Page Load with Sound Off by Default
- **3** or **Signals.PlaybackMethod.ClickToPlay** : Initiates on Click with Sound On
- **4** or **Signals.PlaybackMethod.MouseOver** : Initiates on Mouse-Over with Sound On
- **5** or **Signals.PlaybackMethod.EnterSoundOn** : Initiates on Entering Viewport with Sound On
- **6** or **Signals.PlaybackMethod.EnterSoundOff**: Initiates on Entering Viewport with Sound Off by Default

##### protocols
{:.no_toc}

Array or enum of OpenRTB 2.6 supported Protocols. GMA SDK supports all VAST versions, but not all VAST features ([details](https://developers.google.com/interactive-media-ads/docs/sdks/ios/client-side/compatibility)). So the set of protocols mostly depends on the demand partner supported formats and expected values in the request. Developers can add the following values to the request:   

- **1** or **Signals.Protocols.VAST_1_0** : VAST 1.0
- **2** or **Signals.Protocols.VAST_2_0** : VAST 2.0
- **3** or **Signals.Protocols.VAST_3_0** : VAST 3.0
- **4** or **Signals.Protocols.VAST_1_0_Wrapper** : VAST 1.0 Wrapper
- **5** or **Signals.Protocols.VAST_2_0_Wrapper** : VAST 2.0 Wrapper
- **6** or **Signals.Protocols.VAST_3_0_Wrapper** : VAST 3.0 Wrapper
- **7** or **Signals.Protocols.VAST_4_0** : VAST 4.0
- **8** or **Signals.Protocols.VAST_4_0_Wrapper** : VAST 4.0 Wrapper

#### Step 3: Create a GAMBannerView

Follow the [GMA SDK documentation](https://developers.google.com/ad-manager/mobile-ads-sdk/ios/banner) to integrate the banner ad unit. 

#### Step 4: Make the bid request

The _fetchDemand_ method makes a bid request to the Prebid Server. The `GAMRequest` object provided to this method must be the one used in the next step to make the GAM ad request.

When Prebid Server responds, Prebid SDK will set the targeting keywords of the winning bid into provided object.

#### Step 5: Call the ad server

Next, request the ad from GAM. If the `GAMRequest` object contains targeting keywords, the respective Prebid line item may be returned from GAM, and GMA SDK will render its creative. 

Ensure that you call the _load_ method with the same `GAMRequest` object that you passed to the _fetchDemand_ method on the previous step. Otherwise, the ad request won't contain targeting keywords, and Prebid bids won't be displayed.

### Format: Interstitial Banner

To integrate an interstitial banner ad into the app you use the Prebid SDK `InterstitialAdUnit` class. It makes bid requests to Prebid Server and provides targeting keywords of the winning bid to the GMA SDK.

**Integration example(Swift):**

```swift
   func createAd() {
        // 1. Create an InterstitialAdUnit
        adUnit = InterstitialAdUnit(configId: CONFIG_ID)
        
        // 2. Make a bid request to Prebid Server
        let gamRequest = GAMRequest()
        adUnit.fetchDemand(adObject: gamRequest) { [weak self] resultCode in
            DemoLogger.shared.info("Prebid demand fetch for GAM \(resultCode.name())")
            
            // 3. Load a GAM interstitial ad
            GAMInterstitialAd.load(withAdManagerAdUnitID: AD_UNIT_ID, request: gamRequest) { ad, error in
                guard let self = self else { return }
                
                if let error = error {
                    DemoLogger.shared.error("Failed to load interstitial ad with error: \(error.localizedDescription)")
                } else if let ad = ad {
                    // 4. Render the interstitial ad
                    ad.fullScreenContentDelegate = self
                    ad.present(fromRootViewController: self)
                }
            }
        }
    }
```

#### Step 1: Create an Ad Unit

Initialize the Interstitial Ad Unit with properties:

- `configId` - an ID of the Ad Unit Level Stored Request on Prebid Server
- `minWidthPerc` - Optional parameter to specify the minimum width percent an ad may occupy of a device's real estate. For example, a screen width of 1170 and "minWidthperc": 60 would allow ads with widths from 702 to 1170 pixels inclusive.
- `minHeightPerc` - Optional parameter to specify the minimum height percent an ad may occupy of a device's real estate. For example, a screen height of 2532 and "minHeightPerc": 60 would allow ads with widths from 1519 to 2532 pixels inclusive.

{: .alert.alert-info :}
Here's how min size percentage work: Prebid Server takes the screen width/height and the minWidthPerc/minHeightPerc as a size range, generating a list of ad sizes from a [predefined list](https://github.com/prebid/prebid-server/blob/master/config/interstitial.go). It selects the first 10 sizes that fall within the max size and minimum percentage size. All the interstitial parameters will still be passed to the bidders, allowing them to use their own size matching algorithms if they prefer.

#### Step 2: Make the bid request

The _fetchDemand_ method makes a bid request to the Prebid Server. The `GAMRequest` object provided to this method must be the one used in the next step to make the GAM ad request.

When Prebid Server responds, Prebid SDK will set the targeting keywords of the winning bid into provided object.

#### Step 3: Load a GAM interstitial ad

After receiving a bid it's time to load the ad from GAM. If the `GAMRequest` contains targeting keywords, the respective Prebid line item may be returned from GAM, and GMA SDK will render its creative. 

#### Step 4: Render the interstitial ad

Follow the [GMA SDK guide](https://developers.google.com/ad-manager/mobile-ads-sdk/ios/interstitial#display_the_ad) to display the interstitial ad. Note that you'll need to decide whether it's going to be rendered immediately after receiving it or rendered later in the flow of an app.

### Format: Video Interstitial

To integrate Video Interstitial ads into the app you should use the Prebid SDK `VideoInterstitialAdUnit` class. It makes bid requests to Prebid Server and provides targeting keywords of the winning bid to the GMA SDK.

**Integration Example:**

```swift
    func createAd() {
        // 1. Create an VideoInterstitialAdUnit
        adUnit = VideoInterstitialAdUnit(configId: CONFIG_ID)
        
        // 2. Configure video parameters
        let parameters = VideoParameters()
        parameters.mimes = ["video/mp4"]
        parameters.protocols = [Signals.Protocols.VAST_2_0]
        parameters.playbackMethod = [Signals.PlaybackMethod.AutoPlaySoundOn]
        adUnit.parameters = parameters
        
        // 3. Make a bid request to Prebid Server
        let gamRequest = GAMRequest()
        adUnit.fetchDemand(adObject: gamRequest) { [weak self] resultCode in
            DemoLogger.shared.info("Prebid demand fetch for GAM \(resultCode.name())")
            
            // 4. Load a GAM interstitial ad
            GAMInterstitialAd.load(withAdManagerAdUnitID: AD_UNIT_ID, request: gamRequest) { ad, error in
                guard let self = self else { return }
                if let error = error {
                    DemoLogger.shared.error("Failed to load interstitial ad with error: \(error.localizedDescription)")
                } else if let ad = ad {
                    // 5. Present the interstitial ad
                    ad.present(fromRootViewController: self)
                    ad.fullScreenContentDelegate = self
                }
            }
        }
    }
```

#### Step 1: Create an Ad Unit

Initialize the `VideoInterstitialAdUnit` with properties:

- `configId` - an ID of the Ad Unit Level Stored Request on Prebid Server

#### Step 2: Configure video parameters

Provide configuration properties for the video ad using the [VideoParameters](TBD) object.

#### Step 3: Make the bid request

The _fetchDemand_ method makes a bid request to the Prebid Server. The `GAMRequest` object provided to this method must be the one used in the next step to make the GAM ad request.

When Prebid Server responds, Prebid SDK will set the targeting keywords of the winning bid into provided object.

#### Step 4: Load a GAM interstitial ad

After receiving a bid it's time to load the ad from GAM. If the `GAMRequest` contains targeting keywords, the respective Prebid line item may be returned from GAM, and GMA SDK will render its creative. 

#### Step 5: Render the interstitial ad

Follow the [GMA SDK guide](https://developers.google.com/ad-manager/mobile-ads-sdk/ios/interstitial#display_the_ad) to display the interstitial ad. Note that you'll need to decide whether it's going to be rendered immediately after receiving it or rendered later in the flow of an app.

### Format: Rewarded Video Ad

To integrate Rewarded Video ads into the app you should use the Prebid SDK `RewardedVideoAdUnit` class. It makes bid requests to Prebid Server and provides targeting keywords of the winning bid to the GMA SDK.

**Integration Example**

```swift
    func createAd() {
        // 1. Create an RewardedVideoAdUnit
        adUnit = RewardedVideoAdUnit(configId: CONFIG_ID)
        
        // 2. Configure video parameters
        let parameters = VideoParameters()
        parameters.mimes = ["video/mp4"]
        parameters.protocols = [Signals.Protocols.VAST_2_0]
        parameters.playbackMethod = [Signals.PlaybackMethod.AutoPlaySoundOn]
        adUnit.parameters = parameters
        
        // 3. Make a bid request to Prebid Server
        let gamRequest = GAMRequest()
        adUnit.fetchDemand(adObject: gamRequest) { [weak self] resultCode in
            DemoLogger.shared.info("Prebid demand fetch for GAM \(resultCode.name())")
            
            // 4. Load the GAM rewarded ad
            GADRewardedAd.load(withAdUnitID: AD_UNIT_ID, request: gamRequest) { [weak self] ad, error in
                guard let self = self else { return }
                if let error = error {
                    DemoLogger.shared.error("Failed to load rewarded ad with error: \(error.localizedDescription)")
                } else if let ad = ad {
                    // 5. Present the interstitial ad
                    ad.fullScreenContentDelegate = self
                    ad.present(fromRootViewController: self, userDidEarnRewardHandler: {
                        _ = ad.adReward
                    })
                }
            }
        }
    }
```

#### Step 1: Create an Ad Unit

Initialize the `RewardedVideoAdUnit` with properties:

- `configId` - an ID of Stored Impression on Prebid Server

#### Step 2: Configure video parameters

Provide configuration properties for the video ad using the [VideoParameters](TBD) object.

#### Step 3: Make the bid request

The _fetchDemand_ method makes a bid request to the Prebid Server. The `GAMRequest` object provided to this method must be the one used in the next step to make the GAM ad request.

When Prebid Server responds, Prebid SDK will set the targeting keywords of the winning bid into provided object.

#### **Step 4: Load a GAM Rewarded Ad**

After receiving a bid it's time to load the ad from GAM. If the `GAMRequest` contains targeting keywords, the respective Prebid line item may be returned from GAM, and GMA SDK will render its creative. 

#### Step 5: Present the Rewarded Ad

Follow the [GMA SDK guide](https://developers.google.com/ad-manager/mobile-ads-sdk/ios/rewarded#show_the_ad) to display the rewarded ad.

### Format: Video Instream

To integrate instream video ads into the app you should use the Prebid SDK `VideoAdUnit` class. It makes bid requests to Prebid Server and provides targeting keywords to the [Google IMA SDK](https://developers.google.com/interactive-media-ads/docs/sdks/ios/client-side).

**Integration Example (Swift):**

```swift
// 1. Create VideoAdUnit
adUnit = VideoAdUnit(configId: CONFIG_ID, size: CGSize(width: 1,height: 1))

// 2. Configure Video Parameters
let parameters = VideoParameters()
parameters.mimes = ["video/mp4"]
parameters.protocols = [Signals.Protocols.VAST_2_0]
parameters.playbackMethod = [Signals.PlaybackMethod.AutoPlaySoundOn]
adUnit.parameters = parameters

// 3. Prepare IMAAdsLoader
adsLoader = IMAAdsLoader(settings: nil)
adsLoader.delegate = self

// 4. Make the Prebid bid request
adUnit.fetchDemand { [weak self] (resultCode, prebidKeys: [String: String]?) in
    guard let self = self else { return }
    if resultCode == .prebidDemandFetchSuccess {
        do {
            
            // 5. Generate GAM Instream ad tag
            let adServerTag = try IMAUtils.shared.generateInstreamUriForGAM(adUnitID: GAM_AD_UNIT_ID, adSlotSizes: [.Size320x480], customKeywords: prebidKeys!)
            
            // 6. Make IMA ad request
            let adDisplayContainer = IMAAdDisplayContainer(adContainer: self.instreamView, viewController: self)
            let request = IMAAdsRequest(adTagUrl: adServerTag, adDisplayContainer: adDisplayContainer, contentPlayhead: nil, userContext: nil)
            self.adsLoader.requestAds(with: request)
        } catch {
            PrebidDemoLogger.shared.error("\(error.localizedDescription)")
            self.contentPlayer?.play()
        }
    } else {
        PrebidDemoLogger.shared.error("Error constructing IMA Tag")
        self.contentPlayer?.play()
    }
}
```

Implement  `IMAAdsLoaderDelegate`:

```swift
// 7. ads loader delegate
func adsLoader(_ loader: IMAAdsLoader, adsLoadedWith adsLoadedData: IMAAdsLoadedData) {
    // Grab the instance of the IMAAdsManager and set ourselves as the delegate.
    adsManager = adsLoadedData.adsManager
    adsManager?.delegate = self
    
    // Initialize the ads manager.
    adsManager?.initialize(with: nil)
}

func adsLoader(_ loader: IMAAdsLoader, failedWith adErrorData: IMAAdLoadingErrorData) {
    PrebidDemoLogger.shared.error("IMA did fail with error: \(adErrorData.adError)")
    contentPlayer?.play()
}
```

Implement `IMAAdsManagerDelegate`:

```swift
// 8. ads manager delegate
func adsManager(_ adsManager: IMAAdsManager, didReceive event: IMAAdEvent) {
    if event.type == IMAAdEventType.LOADED {
        // When the SDK notifies us that ads have been loaded, play them.
        adsManager.start()
    }
}

func adsManager(_ adsManager: IMAAdsManager, didReceive error: IMAAdError) {
    PrebidDemoLogger.shared.error("AdsManager error: \(error.message ?? "nil")")
    contentPlayer?.play()
}

func adsManagerDidRequestContentPause(_ adsManager: IMAAdsManager) {
    // The SDK is going to play ads, so pause the content.
    contentPlayer?.pause()
}

func adsManagerDidRequestContentResume(_ adsManager: IMAAdsManager) {
    // The SDK is done playing ads (at least for now), so resume the content.
    contentPlayer?.play()
}
```

#### Step 1: Create an Ad Unit

Initialize the Video Ad Unit with properties:

- `configId` - an ID of Stored Impression on the Prebid Server
- `size` - Width and height of the video ad unit.

#### Step 2: Configure video parameters

Provide configuration properties for the video ad using the [VideoParameters](TBD) object.

#### Step 3: Prepare IMAAdsLoader

Prepare the instream setup according to [Google's docs](https://developers.google.com/interactive-media-ads/docs/sdks/ios/client-side).

#### Step 4: Make the bid request

The _fetchDemand_ method makes a bid request to the Prebid Server. The `GAMRequest` object provided to this method must be the one used in the next step to make the GAM ad request. Later you will construct the IMA ad request using these keywords. 

#### Step 5: Generate GAM Instream ad tag

Using Prebid SDK `IMAUtils.shared.generateInstreamUriForGAM` method, generate Google IMA ad tag for downloading the cached creative from the winning bid.

#### Step 6: Invoke IMA ad request

First, create an ad display container for ad rendering. Then create an ad request with the ad tag, display container, and optional user context. Finally, call IMA with the ad request. See the [in-stream video guide](https://developers.google.com/interactive-media-ads/docs/sdks/ios/client-side#6_initialize_the_ads_loader_and_make_an_ads_request) for additional details.  

#### Step 7: Set up an ads loader delegate

On a successful load event, the `IMAAdsLoader` calls the _adsLoadedWithData_ method of its assigned delegate, passing it an instance of `IMAAdsManager`.

#### Step 8: Set up an ads manager delegate

Lastly, to manage events and state changes during the ad playback, the ads manager needs a delegate of its own. The `IMAAdManagerDelegate` has methods to handle ad events and errors, as well as methods to trigger play and pause on your video content.

### Format: Multiformat (Banner + In-App Native) 

A *multiformat* slot is able to display different ad formats. This scenario covers an adunit that can receive bids for both HTML Banner (WebView) and In-App Native (i.e. publisher-defined layout of native views like text, images, buttons). 

Different display mechanisms are used for these formats: 

- **banner** is rendered by the GMA SDK
- **native** is rendered in the publisher’s custom view

From the high level perspective the integration consist of three steps:

1. [Configure the Native Bid Request](TBD) 
2. [Configure and perform the ad request](TBD)
3. [Manage the Ad Response](TBD)

The following sections describe these steps in details. 

#### Configure the Native Bid Request

According to the [OpenRTB protocol](https://iabtechlab.com/wp-content/uploads/2022/04/OpenRTB-2-6_FINAL.pdf) the native object in the Bid Request should contain request payload complying with the [Native Ad Specification v1.2](https://www.iab.com/wp-content/uploads/2018/03/OpenRTB-Native-Ads-Specification-Final-1.2.pdf). For this purpose, publishers should provide the set of native assets that confirms the ad’s layout in the app. 

{: .alert.alert-info :}
While the IAB Native spec v1.2 is the most recent, you may want to check with the rest of your tech stack for compatibility. Older versions are still in use by some bidders.

The Prebid SDK provides API classes to register needed native assets:    

```swift
private var nativeRequestAssets: [NativeAsset] {
    let title = NativeAssetTitle(length: 90, required: true)
    let body = NativeAssetData(type: DataAsset.description, required: true)
        
    let image = NativeAssetImage(minimumWidth: 120, minimumHeight: 100, required: true)
    image.type = ImageAsset.Main     
    let sponsored = NativeAssetData(type: DataAsset.sponsored, required: true)
        
    return [title, body, image, sponsored]
}
```
#### Prepare the event trackers (Optional)

Both the [Native Ad v1.2 Specification](https://www.iab.com/wp-content/uploads/2018/03/OpenRTB-Native-Ads-Specification-Final-1.2.pdf) and Prebid SDK assume that custom event trackers are added to the native ad. 

```swift
private var eventTrackers: [NativeEventTracker] {
        [NativeEventTracker(event: EventType.Impression, methods: [EventTracking.Image,EventTracking.js])]
}
```

#### Integrate the multiformat ad unit 

To integrate multiformat ads into the app you should use the `PrebidAdUnit` and `PrebidRequest` classes. They'll be used to make multiformat bid requests.

The following code snippets show the integration approach. The description for each integration step is provided after the code examples.

Integration example(Swift):

```swift
 func createAd() {
        // 1. Enable adding an id for each asset in the assets array
        Prebid.shared.shouldAssignNativeAssetID = true
        
        // 2. Setup a PrebidAdUnit
        adUnit = PrebidAdUnit(configId: CONFIG_ID)
        adUnit.setAutoRefreshMillis(time: 30000)
        
        // 3. Setup the parameters
        let bannerParameters = BannerParameters()
        bannerParameters.api = [Signals.Api.MRAID_2, Signals.Api.OMID_1]
        bannerParameters.adSizes = [adSize]
        
        let nativeParameters = NativeParameters()
        nativeParameters.assets = nativeAssets
        nativeParameters.context = ContextType.Social
        nativeParameters.placementType = PlacementType.FeedContent
        nativeParameters.contextSubType = ContextSubType.Social
        nativeParameters.eventtrackers = eventTrackers
        
        // 4. Configure the PrebidRequest
        let prebidRequest = PrebidRequest(bannerParameters: bannerParameters, nativeParameters: nativeParameters)
        
        // 5. Make the bid request
        let gamRequest = GAMRequest()
        adUnit.fetchDemand(adObject: gamRequest, request: prebidRequest) { [weak self] bidInfo in
            guard let self = self else { return }
            // 6. Configure and make a GAM ad request
            self.adLoader = GADAdLoader(adUnitID: AD_UNIT_ID, rootViewController: self, adTypes: [.customNative, .gamBanner], options: [])
            self.adLoader?.delegate = self
            self.adLoader?.load(gamRequest)
        }
    }
```

##### Step 1: Enable adding an id for each asset in the assets array

Set the `Prebid.shared.shouldAssignNativeAssetID` to `true`. This will cause SDK to add IDs for each registered native asset.

##### Step 2: Setup a PrebidAdUnit 

Initialize the `PrebidAdUnit` with the following properties:

- **configId** - an ID of the Ad Unit Level Stored Request in Prebid Server

##### Step 3: Setup the parameters 

Next, for each ad format you set the respective configuration parameters:

- [BannerParameters](TBD)
- [VideoParameters](TBD)
- [NativeParameters](TBD)

Using the `NativeParameters` you can customize the bid request for native ads:

- **assets:** the array of requested asset objects. Prebid SDK supports all kinds of assets according to the IAB spec except video;
- **eventtrackers:** the array of requested native trackers. Prebid SDK supports only image trackers according to the [IAB spec](https://iabtechlab.com/wp-content/uploads/2016/07/OpenRTB-Native-Ads-Specification-Final-1.2.pdf);
- **version:** version of the Native Markup version in use. The default value is 1.2;
- **context:** the context in which the ad appears;
- **contextSubType:** a more detailed context in which the ad appears;
- **placementType:** the design/format/layout of the ad unit being offered;
- **placementCount:** the number of identical placements in this layout;
- **sequence:** 0 for the first ad, 1 for the second ad, and so on;
- **asseturlsupport:** whether the supply source/impression supports returning an assetsurl instead of an asset object. 0 or the absence of the field indicates no such support;
- **durlsupport:** whether the supply source / impression supports returning a dco url instead of an asset object. 0 or the absence of the field indicates no such support;
- **privacy:** set to 1 when the native ad supports buyer-specific privacy notice. Set to 0 (or field absent) when the native ad doesn’t support custom privacy links or if support is unknown;
- **ext:** this object is a placeholder that may contain custom JSON agreed to by the parties to support flexibility beyond the standard defined in this specification.

##### Step 4: Configure the PrebidRequest 

Create the instance of `PrebidRequest` initializing it with all needed ad format parameters.

##### Step 5: Make the bid request 

The _fetchDemand_ method makes a bid request to the Prebid Server. The `GAMRequest` object provided to this method must be the one used in the next step to make the GAM ad request.

##### Step 6: Configure and make a GAM ad request 

Finally it's time to request the ad from GAM. If the a Prebid line item is triggered, the GMA SDK will render the creative.

See the [GMA SDK documentation](https://developers.google.com/ad-manager/mobile-ads-sdk/ios/native-banner) for more information on how to integrate multiformat ads. 
