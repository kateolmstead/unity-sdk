Playnomics PlayRM Unity SDK Integration Guide
=============================================
If you're new to PlayRM and/or don't have a PlayRM account and would like to get started using PlayRM please visit   <a href="https://controlpanel.playnomics.com/signup">https://controlpanel.playnomics.com/signup</a> to sign up. Soon after creating an account you will receive a registration confirmation email permitting you access to your PlayRM control panel.

Within the control panel, click the <strong>applications</strong> tab and add your app. Upon doing so, you will receive an <strong>Application ID</strong> and an <strong>API KEY</strong>. These two components will enable you to begin the integration process.

Our integration has been optimized to be as straight forward and user friendly as possible. If you're feeling unsure or would like to better understand the process before beginning integration, please take a moment to check out the <a href="http://integration.playnomics.com/technical/#integration-getting-started">getting started</a> page. Here you can find an overview of our integration process, and platform specific features, to help you better understand the PlayRM integration process.

## Considerations for Cross-Platform Applications

If you want to deploy your app to multiple platforms (eg: Android and the Unity Web player), you'll need to create separate applications in the control panel. Each application must incorporate a separate `<APPID>` particular to that application. In addition, placements and their respective creative uploads will be particular to that app in order to ensure that they are sized appropriately - proportionate to your app screen size.

Basic Integration
=================

You can install the SDK package by downloading the *PlaynomicsSDK.unitypackage* file directly from <a href="https://github.com/playnomics/unity-sdk/raw/master/build/PlaynomicsSDK.unitypackage"><strong>this link</strong></a>, or you can install the SDK files directly from the <a href="https://www.assetstore.unity3d.com/#/content/6179" target="_blank">Unity Asset Store</a>. If you download the package file from the repo, go to *Assets > Import New Asset ... > Custom Package * to import the package.

![Importing the prefab](http://integration.playnomics.com/img/unity/prefab.png)

The package has a prefab with everything you need to work with the SDK. Simply drag the prefab into your first app scene and this will make the SDK available to your app. The `GameObject` for this is called **Playnomics**. You only need to do this once.

### Interacting with PlayRM in Your Application

All calls are made through a single `MonoBehavior` script that persists throughout the app (<a href="http://en.wikipedia.org/wiki/Singleton_pattern" target="_blank">singleton</a>). The static instance of the script is accessible throughout your application using this property `Playnomics.instance`. It is instantiated in the `Awake` event of the `Playnomics` script.

When working with the SDK, you'll need to import the SDK namespace to work with the PlayRM SDK in your scripts:

```csharp
//C#
using PlaynomicsPlugin;
```
```javascript
//JavaScript
import PlaynomicsPlugin;
```

All public methods, except for messaging specific calls, return an enumeration `ApiResultEnum` with a value of `NotStarted` or `Success`. `NotStarted` indicates that the PlayRM session has not been started, covered in the next section. 

**You always need to start a session before making any other SDK calls.**

### Starting a User Session

To start collecting behavior data, you need to initialize the PlayRM session. You can either provide a dynamic `<USER-ID>` to identify each user:

```csharp
Playnomics.instance.startPlaynomics(<APPID>, <USER-ID>);
```

or have PlayRM, generate a *best-effort* unique-identifier for the user:

```csharp
Playnomics.instance.startPlaynomics(<APPID>);
```

If possible, you should provide your own `<USER-ID>`, especially if your app is cross-platform, since *best-effort* unique identifiers are generated differently depending on the Platform. If you do choose to provide a `<USER-ID>`, this value should be persistent, anonymized, and unique to each user. This is typically discerned dynamically when a user starts the application. Some potential implementations:

* An internal ID (such as a database auto-generated number).
* A hash of the user’s email address.

**You cannot use the user’s Facebook ID or any personally identifiable information (plain-text email, name, etc) for the `<USER-ID>`.**

You **MUST** make the initialization call before working with any other PlayRM modules. You only need to call this method once.

```csharp
//C#
using UnityEngine;
using PlaynomicsPlugin;
 
public class PlaynomicsInit : MonoBehaviour
{
    void Start() {
        
        const long androidApplicationId = <ANDROID-APPID>;
        const long iOsApplicationId = <IOS-APPID>;
        
        long applicationId;
        //this script will get compiled differently based on the platform you are targeting

#if UNITY_ANDROID
        applicationId = androidApplicationId;
#elif UNITY_IOS
        applicationId = iOsApplicationId;
#endif
        
        string hashedUserId;
    
        Playnomics.instance.TestMode = true;
    
        //set the hashedUserId from the authenticated user
        //this value should not include personally identifiable information
    
        Playnomics.instance.start(applicationId, hashedUserId);
    }
}

```
```javascript
//JavaScript
import UnityEngine;
import PlaynomicsPlugin;

import PlaynomicsPlugin;
function Start(){
    //in this case there is no user id for the application

    var androidApplicationId = <ANDROID-APPID>;
    var iOsApplicationId = <IOS-APPID>;

    var applicationId;
    //this script will get compiled differently based on the platform you are targeting
#if UNITY_ANDROID
    applicationId = androidApplicationId;
#elif UNITY_IOS
    applicationId = iOsApplicationId;
#endif
    Playnomics.instance.TestMode = true;
    Playnomics.instance.startPlaynomics(applicationId);
}

```

Once started, the SDK will automatically begin collecting basic user information (including geo-location) and engagement data in **test mode** (be sure to switch to [production mode](#switch-sdk-to-production-mode) before deploying your application).


Congratulations! You've completed our basic integration. You will now be able to track engagement behaviors (having incorporated the Engagement Module) from the PlayRM dashboard. At this point we recommend that you use our integration validation tool to test your integration of our SDK in order insure that it has been properly incorporated into your app. 


PlayRM is currently operating in test mode. Be sure you switch to [production mode](#switch-sdk-to-production-mode), by implementing the code call outlined in our Basic Integration before deploying your app on the web or in an app store.


# Full Integration



<div class="outline">
<ul>
    <li>
        <a href="#full-integration">Full Integration</a>
        <ul>
            <li><a href="#demographics-and-install-attribution">Demographics and Install Attribution</a></li>
            <li>
                <a href="#monetization">Monetization</a>
                <ul>
                    <li>
                        <a href="#purchases-of-in-app-currency-with-real-currency">Purchases of In-App Currency with Real Currency</a>
                    </li>
                    <li>
                        <a href="#purchases-of-items-with-real-currency">Purchases of Items with Real Currency</a>
                    </li>
                    <li>
                        <a href="#purchases-of-items-with-premium-currency">Purchases of Items with Premium Currency</a>
                    </li>
                </ul>
            </li>
<li><a href="#custom-event-tracking">Custom Event Tracking</a></li>
            <li><a href="#validate-integration">Validate Integration</a></li>
            <li><a href="#switch-sdk-to-production-mode">Switch SDK to Production Mode</a></li>
        </ul>
    </li>
    <li>
        <a href="#messaging-integration">Messaging Integration</a>
        <ul>
            <li><a href="#sdk-integration">SDK Integration</a></li>
            <li><a href="#using-rich-data-callbacks">Using Rich Data Callbacks</a></li>
        </ul>
    </li>
    <ul>
        <li><a href="#push-notifications">Push Notifications</a></li>
    </ul>
    <ul>
        <li><a href="#support-issues">Support Issues</a></li>
        <li><a href="#change-log">Change Log</a></li>
    </ul>
</ul>
</div>


If you're reading this it's likely that you've integrated our SDK and are interested in tailoring PlayRM to suit your particular segmentation needs.

The index on the right provides a holistic overview of the <strong>full integration</strong> process. From it, you can jump to specific points in this document depending on what you're looking to learn and do.

To clarify where you are in the timeline of our integration process, you've completed our basic integration. Doing so will enable you to track engagement behaviors from the PlayRM dashboard (having incorporated the Engagement Module). The following documentation will provide succinct information on how to incorporate additional and more in-depth segmentation functionality by integrating any, or all of the following into your application:

<ul>
    <li><strong>User Info Module:</strong> - provides basic user information</li>
    <li><strong>Monetization Module:</strong> - tracks various monetization events and transactions</li>
    <li><strong>Custom Event Module:</strong> - tracks significant user events customized to your app</li>
</ul>


Along with integration instructions for our various modules, you will also find integration information pertaining to messaging setup, and push setup within this documentation.


## Demographics and Install Attribution

After the SDK is loaded, the user info module may be called to collect basic demographic and acquisition information. This data is used to segment users based on how/where they were acquired and enables improved targeting with basic demographics, in addition to the behavioral data collected using other events.

Provide each user’s information using this call:

```javascript
ApiResultEnum Playnomics.instance.userInfo(
    string country,
    string subdivision,
    SexEnum? sex,
    short? birthyear,
    string source,
    string sourceCampaign,
    string sourceUser,
    long? installTime)
```
If any of the parameters are not available, you should pass `null`.

<table>
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><code>country</code></td>
            <td>string</td>
            <td>This has been deprecated. Just pass <code>null</code>.</td>
        </tr>
        <tr>
            <td><code>subdivision</code></td>
            <td>string</td>
            <td>This has been deprecated. Just pass <code>null</code>.</td>
        </tr>
        <tr>
            <td><code>birthyear</code></td>
            <td>short?</td>
            <td>4-digit year, such as 1980</td>
        </tr>
        <tr>
            <td><code>source</code></td>
            <td>string</td>
            <td>
                Source of the user, such as "FacebookAds", "UserReferral", "Playnomics", etc. These are only suggestions; any 16-character or shorter string is acceptable.
            </td>
        </tr>
        <tr>
            <td><code>sourceCampaign</code></td>
            <td>string</td>
            <td>any 16-character or shorter string to help identify specific campaigns</td>
        </tr>
        <tr>
            <td><code>sourceUser</code></td>
            <td>strings</td>
            <td>
                If the user was acquired via a UserReferral (i.e., a viral message), the `userId` of the person who initially brought this user into the app.
            </td>
        </tr>
        <tr>
            <td><code>installTime</code></td>
            <td>long?</td>
            <td>Unix epoch time in seconds when the user originally installed the app.</td>
        </tr>
    </tbody>
</table>

Since PlayRM uses the application client's IP address to determine geographic location, country and subdivision should be set to `null`.

```csharp
void Start()
{
    //after Playnomics.startPlaynomics has been called
    long? installTime = null;
    if(isNewUser){
        DateTime baseDate = new DateTime(1970, 1, 1);
        DateTime now = DateTime.UtcNow;
        installTime = (long)((now - baseDate).TotalSeconds;
    }

    SexEnum playerSex = SexEnum.F;
    short birthYear = 1980;
    ApiResultEnum result = Playnomics.instance.userInfo(null, null, playerSex, birthYear, "AppStore", "Facebook Ad", null, installTime);
}
```

## Monetization

PlayRM provides a flexible interface for tracking monetization events. This module should be called every time a user triggers a monetization event. 

This event tracks users that have monetized and the amount they have spent in total, *real* currency:
* FBC (Facebook Credits)
* USD (US Dollars)

or an in-app, *virtual* currency.

```csharp
ApiResultEnum Playnomics.instance.transaction(
    long transactionId,
    TransactionType transactionType,
    TransactionCurrency[] transactionCurrencies,
    string itemId,
    int? quantity,
    string otherUserId)
```
<table>
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><code>transactionId</code></td>
            <td>long</td>
            <td>
                A unique identifier for this transaction. If you don't have a transaction ID from payments system, you can generate a large random number.
            </td>
        </tr>
        <tr>
            <td><code>transactionType<code></td>
            <td>TransactionType</td>
            <td>
                The type of transaction occurring:
                <ul>
                    <li>BuyItem: A purchase of virtual item. The <code>quantity</code> is added to the user’s inventory</li>
                    <li>
                        SellItem: A sale of a virtual item to another user. The item is removed from the user’s inventory. Note: a sale of an item will result in two events with the same <code>transactionId</code>, one for the sale with type SellItem, and one for the receipt of that sale, with type BuyItem.
                    </li>
                    <li>
                        ReturnItem: A return of a virtual item to the store. The item is removed from the user’s inventory
                    </li>
                    <li>BuyService: A purchase of a service, e.g., VIP membership </li>
                    <li>SellService: The sale of a service to another user</li>
                    <li>ReturnService: The return of a service</li>
                    <li>
                        CurrencyConvert: A conversion of currency from one form to another, usually in the form of real currency (e.g., US dollars) to virtual currency.  If the type of a transaction is CurrencyConvert, then there should be at least 2 elements in the <code>transactionCurrencies</code> array
                    </li>
                    <li>Initial: An initial allocation of currency and/or virtual items to a new user</li>
                    <li>Free: Free currency or item given to a user by the application</li>
                    <li>
                        Reward: Currency or virtual item given by the application as a reward for some action by the user
                    </li>
                    <li>
                        GiftSend: A virtual item sent from one user to another. Note: a virtual gift should result in two transaction events with the same <code>transactionId</code>, one with the type GiftSend, and another with the type GiftReceive
                    </li>
                    <li>GiftReceive: A virtual good received by a user. See note for GiftSend type</li>
                </ul>
            </td>
        </tr>
        <tr>
            <td><code>transactionCurrencies</code></td>
            <td>TransactionCurrency[]</td>
            <td>
                An array of <code>TransactionCurrency</code>, describing the different types of currency used in this transaction.
            </td>
        </tr>
        <tr>
            <td><code>itemId</code></td>
            <td>string</td>
            <td>If applicable, an identifier for the item. The identifier should be consistent.</td>
        </tr>
        <tr>
            <td><code>quantity</code></td>
            <td>int?</td>
            <td>If applicable, the number of items being purchased.</td>
        </tr>
        <tr>
            <td><code>otherUserId</code></td>
            <td>string</td>
            <td>
               If applicable, the other user involved in the transaction. A contextual example is a user sending a gift to another user.
            </td>
        </tr>
    </tbody>
</table>

The `TransactionCurrency` class encapsulates the type of currency that was used in the transaction. We provide static constructors for common currency types.

*Real* currency implementation:
```csharp
TransactionCurrency TransactionCurrency.createReal(double currencyValue, CurrencyType type)
```
`CurrencyType` is an enumeration with either `USD` (US Dollars) or `FBC` (Facebook Credits).

*Virtual* currency implementation:
```csharp
TransactionCurrency TransactionCurrency.createVirtual(double currencyValue, string type)
```
`type` is a short name (up to 16 characters) for the currency, e.g.: "MonsterBucks."

We highlight three common use-cases below.
* [Purchases of In-App Currency with Real Currency](#purchases-of-in-app-currency-with-real-currency)
* [Purchases of Items with Real Currency](#purchases-of-items-with-real-currency)
* [Purchases of Items with In-App Currency](#purchases-of-items-with-in-app-currency)

### Purchases of In-App Currency with Real Currency

A very common monetization strategy is to incentivize users to purchase premium, in-app  currency with real currency. PlayRM treats this like a currency exchange. This is one of the few cases where multiple `TranactionCurrency` are used in a transaction. `itemId`, `quantity`, and `otherUserId` are left `null`.

```csharp
//user purchases 500 MonsterBucks for 10 USD

TransactionCurrency[] currencies = new TransactionCurrency[2]; 

var quantityCoins = 500;
var gameCurrency = "MonsterBucks";
currencies[0] = TransactionCurrency.createVirtual(quantityCoins, gameCurrency);

var priceInUSD = -10;
currencies[1] = TransactionCurrency.createReal(priceInUSD, CurrencyType.USD);

Playnomics.instance.transaction(transactionId, TransactionType.CurrencyConvert, currencies, null, null, null);
```

### Purchases of Items with Real Currency

```csharp
//user purchases a "Monster Trap" for $.99 USD

var trapItemId = "Monster Trap"
var quantity = 1;
var price = .99;

TransactionCurrency[] currencies = new TransactionCurrency[1]; 
currencies[0] = TransactionCurrency.createReal(price, CurrencyType.USD);

Playnomics.instance.transaction(transactionId, TransactionType.BuyItem, currencies, trapItemId, quantity, null);
```

### Purchases of Items with Premium Currency

This event is used to segment monetized users (and potential future monetizers) by collecting information about how and when they spend their premium currency (an in-app currency that is primarily acquired using a *real* currency). This is one level of information deeper than the previous use-cases.

#### Currency Exchanges

This is a continuation on the first currency exchange example. It showcases how to track each purchase of in-app *attention* currency (non-premium virtual currency) paid for with a *premium*:

```csharp

//In this hypothetical, Energy is an attention currency that is earned over the lifetime of the app. 
//They can also be purchased with the premium MonsterBucks that the user may have purchased earlier.

//user buys 100 Mana with 10 MonsterBucks
//notice that both currencies are virtual
TransactionCurrency[] currencies = new TransactionCurrency[2]; 

var attentionCurrency = "Mana";
var attentionAmount = 100;
currencies[0] = TransactionCurrency.createVirtual(attentionAmount, attentionCurrency);

var premimumCurrency = "MonsterBucks";
var premiumCost = -10;
currencies[1] = TransactionCurrency.createVirtual(premiumCost, premimumCurrency);

Playnomics.instance.transaction(transactionId, TransactionType.CurrencyConvert, currencies, null, null, null);
```
#### Item Purchases

This is a continuation on the first item purchase example, except with premium currency.

```javascript
//user buys 20 light armor, for 5 MonsterBucks

var itemQuantity = 20;
var item = "Light Armor";

var premimumCurrency = "MonsterBucks";
var premiumCost = 5;

TransactionCurrency[] currencies = new TransactionCurrency[1]; 
currencies[0] = TransactionCurrency.createVirtual(premiumCost, premimumCurrency);

Playnomics.instance.transaction(transactionId, TransactionType.BuyItem, currencies, item, itemQuantity, null);
```

## Custom Event Tracking

Custom Events may be used in a number of ways.  They can be used to track certain key in-app events such as finishing a tutorial or receiving a high score. They may also be used to track other important lifecycle events such as level up, zone unlocked, etc.  PlayRM, by default, supports up to five custom events.  You can then use these custom events to create more targeted custom segments.

Each time a user completes a certain event, track it with this call:

```csharp
ApiResultEnum Playnomics.instance.milestone(
    long milestoneId,
    string milestoneName);
```
<table>
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><code>milestoneId</code></td>
            <td>long</long>
            <td>A unique 64-bit numeric identifier for this custom event occurrence</td>
        </tr>
        <tr>
            <td><code>milestoneName</code></td>
            <td>string</td>
            <td>
                The name of the custom event should be "CUSTOMn", where n is 1 through 5.  The name is case-sensitive.
            </td>
        </tr>
    </tbody>
</table>

Example client-side calls for users completing events, with generated IDs:

```csharp
long GetRandomLong(){
  var rnd = new System.Random();
    var buffer = new byte[8];
    rnd.NextBytes(buffer);
    return BitConverter.ToInt64(buffer, 0);
}

//...
//...
//...

//when custom event CUSTOM1 is completed
var milestoneCustom1Id = GetRandomLong();
Playnomics.instance.milestone(milestoneCustom1Id, "CUSTOM1");
```
## Validate Integration
After configuring your selected PlayRM modules, you should verify your application's correct integration with the self-check validation service.

Simply visit the self-check page for your application: **`https://controlpanel.playnomics.com/validation/<APPID>`**

You can now see the most recent event data sent by the SDK, with any errors flagged. Visit the <a href="http://integration.playnomics.com/technical/#self-check">self-check validation guide</a> for more information.

We strongly recommend running the self-check validator before deploying your newly integrated application to production.

## Switch SDK to Production Mode
Once you have [validated](#validate-integration) your integration, switch the SDK from **test** to **production** mode by simply 
setting the `Playnomics.instance.TestMode` field to `false` (or by removing/commenting out the call entirely) in the initialization block:

```csharp
//...
public class PlaynomicsInit : MonoBehaviour
{
    void Start() {
        
        //...
    
        Playnomics.instance.TestMode = false;
    
        //set the hashedUserId from the authenticated user
        //this value should not include personally identifiable information
    
        Playnomics.instance.start(applicationId, hashedUserId);
    }
}

```
```javascript
//JavaScript
//...
function Start(){
    //...
    Playnomics.instance.TestMode = false;
    Playnomics.instance.startPlaynomics(applicationId);
}

```

If you ever wish to test or troubleshoot your integration later on, simply set `Playnomics.instance.TestMode` back to `true` and revisit the self-check validation tool for your application:

**`https://controlpanel.playnomics.com/applications/<APPID>`**

Messaging Integration
=====================
This guide assumes you're already familiar with the concept of placements and messaging, and that you have all of the relevant `placements` setup for your application.

If you are new to PlayRM's messaging feature, please refer to <a href="http://integration.playnomics.com" target="_blank">integration documentation</a>.

Once you have all of your placements created with their associated `<PLAYRM-FRAME-ID>`s, you can start the integration process.

## SDK Integration

Loading placements through the SDK:

```csharp
MessagingFrame Playnomics.instance.initMessagingFrame(string frameId);
```
<table>
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><code>frameId</code></td>
            <td>string</td>
            <td>Unique identifier for the placement, the <code>&lt;PLAYRM-FRAME-ID&gt;</code></td>
        </tr>
    </tbody>
</table>

Optionally, associate an implementation of IFrameDelegate to process rich data callbacks. See [Using Rich Data Callbacks](#using-rich-data-callbacks) for more information.

```csharp
MessagingFrame Playnomics.instance.initMessagingFrame(string frameId, IFrameDelegate frameDelegate);
```
<table>
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><code>frameId</code></td>
            <td>string</td>
            <td>Unique identifier for the placement, the <code>&lt;PLAYRM-FRAME-ID&gt;</code></td>
        </tr>
    </tbody>
    <tbody>
        <tr>
            <td><code>frameDelegate</code></td>
            <td>IFrameDelegate</td>
            <td> Processes rich data callbacks, see <a href="#using-rich-data-callbacks">Using Rich Data Callbacks</a></td>
        </tr>
    </tbody>
</table>

Placements are loaded asynchronously to keep your application responsive. The `initMessagingFrame` call begins the loading process. However, until you call `show` on the placement, the placement will not be drawn in the UI. This gives you control over when a placement will appear.

Placements are destroyed on a scene transition or when closed.

In the example below, we initialize the placement when a behavior script is loaded for the first time. In the update loop, we poll for the placement asking if it can be shown loading and then show it. In practice, a placement can be loaded in a variety of ways.

```csharp
using PlaynomicsPlugin;
using UnityEngine;
 
public class Scene : MonoBehavior {
    private MessagingFrame frame;
    private bool shown;  
 
    private void Awake(){
        //Playnomics.instance.start has already been called before we call this method
        const string frameId = "<PLAYRM-FRAME-ID>";
        MessagingFrame frame = Playnomics.instance.initMessagingFrame(frameId);
        //enabled code callbacks, eg PNA   
        frame.EnableAdCode = true;  
    }

    private void Update(){

        if(frame.FrameState == MessagingFrame.FrameStateEnum.Loaded && !shown){
            //the placement is ready and has never been shown

            shown = true;
            //render the frame
            frame.show();
        }
    }
}
```

## Using Rich Data Callbacks

Depending on your configuration, a variety of actions can take place when a placement’s message is pressed or clicked:

* Redirect the user to a web URL in the platform's browser application
* Firing a Rich Data callback in your app
* Or in the simplest case, just close the placement, provided that the **Close Button** has been configured correctly.

Rich Data is a JSON message that you associate with your message creative. When the user presses the message, the PlayRM SDK bubbles-up the associated JSON object to an implementation of the interface, `IFrameDelegate` associated with the placement.

```csharp
public interface IFrameDelegate
{
    void onClick(LitJson.JsonData data);
}
```

The actual contents of your message can be delayed until the time of the messaging campaign configuration. However, the structure of your message needs to be decided before you can process it in your app. 

**The Rich Data callback will not fire if the Close button is pressed.**

Here are three common use cases for placements and messaging campaigns:

* [App Start Placement](#app-start-placement)
* [Currency Balance Low Placement](#currency-balance-low-placement) - for instance, when the user is running low on premium currency
* [Level Complete Placement](#level-complete-placement)

### App Start Placement

In this use-case, we want to configure a placement that is always shown to users when they start a new session. The message shown to the user may change based on the desired segments:

<table>
    <thead>
        <tr>
            <th >
                Segment
            </th>
            <th>
                Priority
            </th>
            <th>
                Callback Behavior
            </th>
            <th style="width:250px;">
                Creative
            </th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                At-Risk
            </td>
            <td>1st</td>
            <td>
                In this case, we're worried that once-active users are now in danger of leaving the app. We might offer them <strong>50 MonsterBucks</strong> to bring them back.
            </td>
            <td>
                <img src="http://playnomics.com/integration-dev/img/messaging/50-free-monster-bucks.png"/>
            </td>
        </tr>
        <tr>
            <td>
                Lapsed 7 or more days
            </td>
            <td>2nd</td>
            <td>
                In this case, we want to thank the user for coming back and incentivize these lapsed users to continue doing so. We might offer them <strong>10 MonsterBucks</strong> to increase their engagement and loyalty.
            </td>
            <td> 
                <img src="http://playnomics.com/integration-dev/img/messaging/10-free-monster-bucks.png"/>
            </td>
        </tr>
        <tr>
            <td>
                Default - user who don't fall into either segment.
            </td>
            <td>3rd</td>
            <td>
                In this case, we can offer a special item to them for returning to the app.
            </td>
            <td>
                <img src="http://playnomics.com/integration-dev/img/messaging/free-bfb.png"/>
            </td>
        </tr>
    </tbody>
</table>

We want our app to process messages for awarding items to users. We process this data with an implementation of the `IFrameDelegate` interface.

```csharp
public class AwardFrameDelegate : IFrameDelegate
{
    public void onClick(LitJson.JsonData data)
    {
        IDictionary dataDict = data as IDictionary;
        if(dataDict == null)
        {
            return;
        }

        if(dataDict.ContainsKey("type") &&
            dataDict["type"].Equals("award"))
        {
            if(dataDict.ContainsKey("award"))
            {
                string item = dataDict["award"]["item"];
                int quantity = dataDict["award"]["quantity"];

                //call your own inventory object
                Inventory.addItem(item, quantity);
            }
        }
    }
}
```
And then attaching this AwardFrameDelegate class to the frame shown in the first app scene:

```csharp
public class FirstGameScene : MonoBehavior
{
    //...
    //...
    void Start()
    {
        const string frameId = "<PLAYRM-FRAME-ID>";
        IFrameDelegate awardDelegate = new AwardFrameDelegate();
        MessagingFrame frame = Playnomics.instance.initMessageFrame(frameId, awardDelegate);
        frame.start();
    }
    //...
    //...
}

```

The related messages would be configured in the Control Panel to use this callback by placing this in the **Target Data** for each message:

Grant 10 Monster Bucks
```json
{
    "type" : "award",
    "award" : 
    {
        "item" : "MonsterBucks",
        "quantity" : 10
    }
}
```

Grant 50 Monster Bucks
```json
{
    "type" : "award",
    "award" : 
    {
        "item" : "MonsterBucks",
        "quantity" : 50
    }
}
```

Grant Bazooka
```json
{
    "type" : "award",
    "award" :
    {
        "item" : "Bazooka",
        "quantity" : 1
    }
}
```

### Currency Balance Low Placement

An advantage of a *dynamic* placement is that it can be triggered by in-app events. For each in-app event you would configure a separate placement. While segmentation may be helpful in deciding what message you show, it may be sufficient to show the same message to all users.

For example, a user may deplete their premium currency and you want to remind them that they can re-up through your store. In this context, we display the same message to all users.

<table>
    <thead>
        <tr>
            <th>
                Segment
            </th>
            <th>
                Priority
            </th>
            <th>
                Callback Behavior
            </th>
            <th>
                Creative
            </th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                Default - all users, because this message is intended for anyone using the app.
            </td>
            <td>1st</td>
            <td>
                You notice that the user’s in-app, premium currency drops below a certain threshold, now you can prompt them to re-up with this <strong>message</strong>.
            </td>
            <td>
                <img src="http://playnomics.com/integration-dev/img/messaging/running-out-of-monster-bucks.png"/>
            </td>
        </tr>
    </tbody>
</table>

Related delegate callback code:

```csharp
using UnityEngine;

public class StoreFrameDelegate : IFrameDelegate
{
    public void onClick(LitJson.JsonData data)
    {
        IDictionary dataDict = data as IDictionary;
        if(dataDict == null){
            return;
        }

        if(dataDict.ContainsKey("type") &&
            data["type"] == "action")
        {
            if(dataDict.ContainsKey("actionType") &&
                data["actionType"] == "openStore")
            {
                //opens the store in our app
                Store.open();
            }
        }
    }
}
```

The Default message would be configured in the Control Panel to use this callback by placing this in the **Target Data** for the message:

```json
{
    "type" : "action",
    "action" : "openStore"
}
```

### Level Complete Placement

In the following example, we wish to generate third-party revenue from users unlikely to monetize by showing them a segmented message after completing a level or challenge:

<table>
    <thead>
        <tr>
            <th>
                Segment
            </th>
            <th>
                Priority
            </th>
            <th>
                Code Callback
            </th>
            <th>
                Creative
            </th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                Non-Monetizers, in their 5th day of app usage
            </td>
            <td>1st</td>
            <td>Show them a 3rd party ad, because they are unlikely to monetize.</td>
            <td>
                <img src="http://playnomics.com/integration-dev/img/messaging/third-party-ad.png"/>
            </td>
        </tr>
        <tr>
            <td>
                Default: everyone else
            </td>
            <td>2nd</td>
            <td>
                You simply congratulate them on completing the level and grant them some attention currency, "Mana" for completeing the level.
            </td>
            <td>
                <img src="http://playnomics.com/integration-dev/img/messaging/darn-good-job.png"/>
            </td>
        </tr>
    </tbody>
</table>

This is another continuation on the `AwardFrameDelegate`, with some different data. The related messages would be configured in the Control Panel:

* **Non-Monetizers, in their 5th day of app usage**, a Target URL: `HTTP URL for Third Party Ad`
* **Default**, Target Data:

```json
{
    "type" : "award",
    "award" :
    {
        "item" : "Mana",
        "quantity" : 20
    }
}
```

Push Notifications
==================

## Registering for PlayRM Push Messaging

Push Notifications are currently only supported for iOS devices. Toggle the iOS Push Notifications on the Playnomics prefab.

## Push Messaging Impression and Click Tracking

There are 3 situations in which an iOS device can receive a Push Notification

<table>
    <thead>
        <tr>
            <th>Sitatuation</th>
            <th>Push Message Shown?</th>
            <th>Delegate Handler</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>App is not running</td>
            <td rowspan="2">Yes</td>
            <td>didFinishLaunchingWithOptions:(NSDictionary*)launchOptions</td>
        </tr>
        <tr>
            <td>App is running in the background</td>
            <td rowspan="2">
                didReceiveRemoteNotification:(NSDictionary*)userInfo
            </td>
        </tr>
        <tr>
            <td>App is running in the foreground</td>
            <td>No</td>
        </tr>
    </tbody>
</table>

By default, iOS does not show push notifications when your app is already in the foreground. Consequently, PlayRM does NOT track these push notifications as impressions nor clicks.

## Clearing Push Badge Numbers

When you send push notifications, you can configure a badge number that will be set on your application. iOS defers the responsibility of resetting the badge number to the developer.

We have taken care of the step for you, so that the badge number is cleared when your app goes into the background.

Support Issues
==============
If you have any questions or issues, please contact <a href="mailto:support@playnomics.com">support@playnomics.com</a>.

Change Log
==========
#### Version 3.2
* Added support for Messaging with Rich Data Callbacks.

#### Version 3.1.3
* Bug fixes related to rendering messaging placements in multiple orientations on mobile platforms.

#### Version 3.1.2
* Simplified the package file structure.

#### Version 3.1.1
* Caching performance improvements

#### Version 3.1
* Adding Push Notifications support for iOS devices.
* Improve performance and exception handling for messaging placements.

#### Version 3.03
* Making the SDK compliant with iOS6 IDFA requirements.

#### Version 3.02
* Messages can now support messages with no target URL.

#### Version 3.01
* Fix related to Unity Player editor crashes.
* Fix related to engagement module logging.

#### Version 3
* Added support for messaging.
* Added support for milestones.

#### Version 2
* Updated Playnomics server URLs

#### Version 1
* First release.

View version tags <a href="https://github.com/playnomics/unity-sdk/tags">here</a>
