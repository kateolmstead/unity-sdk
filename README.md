Playnomics PlayRM Unity SDK Integration Guide
=============================================
This guide showcases the features of the PlayRM Unity SDK and shows how to integrate the SDK with your game. Our SDK provides game publishers with tools for tracking player behavior and engagement so that they can:

* Better understand and segment their audience
* Reach out to new like-minded players
* Retain their current audience
* Ultimately generate more revenue for their games

<img src="http://www.playnomics.com/integration/img/60-Day-Plan.png"/>

The Playnomics Unity SDK supports Unity games built for Web Browsers, iOS, Android, PCs and Macs. Integration of the PlayRM SDK into an existing or brand new Unity game involves registering your game with the PlayRM service and properly configuring the SDK. The SDK communicates with the PlayRM RESTful API, and the events are processed and aggregated for your PlayRM Dashboard in the control panel.

The SDK includes several modules which track different player behaviors and actions. The first two modules are initialized at or near the beginning of the play session, and the other modules are event-driven.

* [Engagement Module](#installing-the-sdk) - collects geographic and engagement information
* [User Info Module](#demographics-and-install-attribution) - provides basic user information
* [Monetization Module](#monetization) - tracks various monetization events
* [Viralility Module](#invitations-and-virality) - tracks the social activities of users
* [Milestone Module](#custom-event-tracking) - tracks pre-defined significant events in the game experience

The [engagement module](#installing-the-sdk) is available upon install and will automatically start running.

Core Concepts
=============
* [Prerequisites](#prerequisites)
    * [Signing Up for the PlayRM Service](#signing-up-for-the-playrm-service)
    * [Register Your Game](#register-your-game)
    * [Considerations for Cross-Platform Games](#considerations-for-cross-platform-games)
* [Basic Integration](#basic-integration)
    * [Installing the SDK](#installing-the-sdk)
        * [Interacting with PlayRM in Your Game](interacting-with-playrm-in-your-game)
        * [Starting a Player Session](#starting-a-player-session)
    * [Demographics and Install Attribution](#demographics-and-install-attribution)
    * [Monetization](#monetization)
        * [Purchases of In-Game Currency with Real Currency](#purchases-of-in-game-currency-with-real-currency)
        * [Purchases of Items with Real Currency](#purchases-of-items-with-real-currency)
        * [Purchases of Items with Premium Currency](#purchases-of-items-with-premium-currency)
    * [Invitations and Virality](#invitations-and-virality)
    * [Custom Event Tracking](#custom-event-tracking)
* [Messaging Integration](#messaging-integration)
    * [Setting up a Frame](#setting-up-a-frame)
    * [SDK Integration](#sdk-integration)
    * [Using Code Callbacks](#using-code-callbacks)
* [Support Issues](#support-issues)

Prerequisites
=============
Before you can integrate with the PlayRM SDK you'll need to sign up and register your game.

## Signing Up for the PlayRM Service

Visit <a href="https://controlpanel.playnomics.com/signup" target="_blank">https://controlpanel.playnomics.com/signup</a> to create an account. The control panel is the dashboard to manage all of the PlayRM features once the SDK integration has been completed.

## Register Your Game
After receiving a registration confirmation email, login to the <a href="https://controlpanel.playnomics.com" target="_blank">control panel</a>. Select the "Applications" tab and create a new application. Your application will be granted an Application ID (`<APPID>`) and an API KEY.

**Considerations for Cross-Platform Games**

If you want to deploy your game to multiple platforms (eg: iOS and the Unity Web player), you'll need to create a separate Playnomics Applications in the control panel. Each application will have a separate `<APPID>` along with frames so that creatives are sized appropriately.

Basic Integration
=================

## Installing the SDK
You can download the SDK package from downloading the *PlaynomicsSDK.unitypackage* file included in the *build* folder of this repo, or you can also install the SDK files directly from the <a href="https://www.assetstore.unity3d.com/#/content/6179" target="_blank">Unity Asset Store</a>. If you download the package file from the repo, go to *Assets > Import New Asset ... > Custom Package * to import the package.

![Importing the prefab](http://www.playnomics.com/integration/img/unity/prefab.png)

The package has a prefab with everything you need to work with the SDK. Simply drag the prefab into your first game scene and this will make the SDK available to your game. The `GameObject` for this is called **Playnomics**. You only need to do this once.

### Interacting with PlayRM in Your Game

All calls are made through a single `MonoBehavior` script that persists throughout the game (<a href="http://en.wikipedia.org/wiki/Singleton_pattern" target="_blank">singleton</a>). The static instance of the script is accessible throughout your game using this property `Playnomics.instance`. It is instantiated in the `Awake` event of the `Playnomics` script.

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

**You always need to start a session before 

### Starting a Player Session

To start collecting behavior data, you need to initialize the PlayRM session. You can either provide a dynamic `<USER-ID>` to identify each player:

```csharp
Playnomics.instance.startPlaynomics(<APPID>, <USER-ID>);
```

or have PlayRM, generate a *best-effort* unique-identifier for the player:

```csharp
Playnomics.instance.startPlaynomics(<APPID>);
```

If possible, you should provide your own `<USER-ID>`, especially if your game is cross-platform, since *best-effort* unqiue identifiers are generated differently depending on the Platform. If you do choose to provide a `<USER-ID>`, this value should be a persistent, anonymized, and unique to each player. This is typically discerned dynamically when a player starts the game. Some potential implementations:

* An internal ID (such as a database auto-generated number).
* A hash of the user’s email address.

**You cannot use the user's Facebook ID or any personally identifiable information (plain-text email, name, etc) for the `<USER-ID>`.**

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
    //in this case there is no user id for the game

    var androidApplicationId = <ANDROID-APPID>;
    var iOsApplicationId = <IOS-APPID>;

    var applicationId;
    //this script will get compiled differently based on the platform you are targeting
#if UNITY_ANDROID
    applicationId = androidApplicationId;
#elif UNITY_IOS
    applicationId = iOsApplicationId;
#endif
    Playnomics.instance.startPlaynomics(applicationId);
}

```

Once started, the SDK will automatically start collecting basic user information (including geo-location) and engagement data.

## Demographics and Install Attribution

After the SDK has been loaded, the user info module may be called to collect basic demographic and acquisition information. This data will be used to segment users based on how/where they were acquired and enables improved targeting with basic demographics in addition to the behavioral data collected using other events.

Provide each user's information using this call:

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


| Name             | Type      | Description                        |
| ---------------- | :-------: | ---------------------------------- |
| `country`        | string    | Deprecated, should be left `null`. |
| `subdivision`    | string    | Deprecated, should be left `null`. |
| `birthyear`      | short?    | 4-digit year, such as 1980         |
| `source`         | string    | Source of the user, such as "FacebookAds", "UserReferral", "Playnomics", etc. These are only suggestions, any 16-character or shorter string is acceptable. |
| `sourceCampaign` | string    | If the user was acquired via a UserReferral (i.e., a viral message), the `userId` of the person who initially brought this user into the game |
| `sourceUser`     | string    | If the user was acquired via a UserReferral (i.e., a viral message), the `userId` of the person who initially brought this user into the game |
| `installTime`    | long?     | Unix epoch time in seconds when the user originally installed the game |


Since PlayRM uses the game client’s IP address to determine geographic location, country and subdivision should be set to `null`.

```csharp
void Start()
{
    //after Playnomics.startPlaynomics has been called
    long? installTime = null;
    if(isNewUser){
        //
        DateTime baseDate = new DateTime(1970, 1, 1);
        DateTime now = DateTime.UtcNow;
        installTime = (long)((date - baseDate).TotalSeconds;
    }

    SexEnum playerSex = SexEnum.F;
    short birthYear = 1980;
    ApiResultEnum result = Playnomics.instance.userInfo(null, null, playerSex, birthYear, “AppStore”, “Facebook Ad”, null, installTime);
}
```

## Monetization

PlayRM provides a flexible interface for tracking monetization events. This module should be called every time a player triggers a monetization event. 

This event tracks users that have monetized and the amount they have spent in total, *real* currency:
* FBC (Facebook Credits)
* USD (US Dollars)

or an in-game *virtual* currency.

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
            <td>A unique identifier for this transaction. If you don't have a transaction ID from payments system, you can genenate large random number.</td>
        </tr>
        <tr>
            <td><code>transactionType<code></td>
            <td>TransactionType</td>
            <td>
                The type of transaction occurring:
                <ul>
                    <li>BuyItem: A purchase of virtual item. The <code>quantity</code> is added to the user's inventory</li>
                    <li>
                        SellItem: A sale of a virtual item to another user. The item is removed from the user's inventory. Note: a sale of an item will result in two events with the same <code>transactionId</code>, one for the sale with type SellItem, and one for the receipt of that sale, with type BuyItem.
                    </li>
                    <li>
                        ReturnItem: A return of a virtual item to the store. The item is removed from the user's inventory
                    </li>
                    <li>BuyService: A purchase of a service, e.g., VIP membership </li>
                    <li>SellService: The sale of a service to another user</li>
                    <li>ReturnService:  The return of a service</li>
                    <li>
                        CurrencyConvert: An conversion of currency from one form to another, usually in the form of real currency (e.g., US dollars) to virtual currency.  If the type of a transaction is CurrencyConvert, then there should be at least 2 elements in the <code>transactionCurrencies</code> array
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
            <td>An array of <code>TransactionCurrency</code>, describing the different types of currency used in this transaction.</td>
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

The `TransactionCurrency` class encapsulates the type of currency that was used in the transaction. We provide static constructors for common types of currency.

*Real* currency implementation:
```csharp
TransactionCurrency TransactionCurrency.createReal(double currencyValue, CurrencyType type)
```
`CurrencyType` is an enumeration with either `USD` (US Dollars) or `FBC` (Facebook Credits).

*Virutal* currency implementation:
```csharp
TransactionCurrency TransactionCurrency.createVirtual(double currencyValue, string type)
```
`type` is a short name (up to 16 characters) for the currency, eg: "MonsterBucks."

We hightlight three common use-cases below.
* [Purchases of In-Game Currency with Real Currency](#purchases-of-in-game-currency-with-real-currency)
* [Purchases of Items with Real Currency](#purchases-of-items-with-real-currency)
* [Purchases of Items with In-Game Currency](#purchases-of-items-with-in-game-currency)

### Purchases of In-Game Currency with Real Currency

A very common monetization strategy is to incentivize players to purchase premium, in-game currency with real currency. PlayRM treats this like a currency exchange. This is one of the few cases where multiple `TranactionCurrency` are used in a transaction. `itemId`, `quantity`, and `otherUserId` are left `null`.

```csharp
//player purchases 500 MonsterBucks for 10 USD

TransactionCurrency[] currencies = new TransactionCurrency[2]; 

var quantityCoins = 500;
var gameCurrency = "MonsterBucks";
currencies[0] = TransactionCurrency.createVirtual(quantityCoins, gameCurrency);

var priceInUSD = 10;
currencies[1] = TransactionCurrency.createReal(priceInUSD, CurrencyType.USD);

Playnomics.instance.transaction(transactionId, TransactionType.CurrencyConvert, currencies, null, null, null);
```

### Purchases of Items with Real Currency

```csharp
//player purchases a "Monster Trap" for $.99 USD

var trapItemId = "Monster Trap"
var quantity = 1;
var price = .99;

TransactionCurrency[] currencies = new TransactionCurrency[1]; 
currencies[0] = TransactionCurrency.createReal(price, CurrencyType.USD);

Playnomics.instance.transaction(transactionId, TransactionType.BuyItem, currencies, trapItemId, quantity, null);
```

### Purchases of Items with Premium Currency

This event is used to segment monetized users (and potential future monetizers) by collecting information about how and when they spend their premium currency (an in-game currency that is primarily acquired using a *real* currency). This is one level of information deeper than the previous use-cases.

#### Currency Exchanges

This is a continuation on the first currency exchange example. It showcases how to track each purchase of in-game *attention* currency (non-premium virtual currency) paid for with a *premium*:

```csharp

//In this hypothetical, Energy is an attention currency that is earned over the lifetime of the game. 
//They can also be purchased with the premium MonsterBucks that the player may have purchased earlier.

//player buys 100 Mana with 10 MonsterBucks
//notice that both currencies are virtual
TransactionCurrency[] currencies = new TransactionCurrency[2]; 

var attentionCurrency = "Mana";
var attentionAmount = 100;
currencies[0] = TransactionCurrency.createVirtual(attentionAmount, attentionCurrency);

var premimumCurrency = "MonsterBucks";
var premiumCost = -20;
currencies[1] = TransactionCurrency.createVirtual(premiumCost, premimumCurrency);

Playnomics.instance.transaction(transactionId, TransactionType.CurrencyConvert, currencies, null, null, null);
```
#### Item Purchases

This is a continuation on the first item purchase example, except with premium currency.

```javascript
//player buys 20 light armor, for 5 MonsterBucks

var itemQuantity = 20;
var item = "Light Armor";

var premimumCurrency = "MonsterBucks";
var premiumCost = 5;

TransactionCurrency[] currencies = new TransactionCurrency[1]; 
currencies[0] = TransactionCurrency.createVirtual(premiumCost, premimumCurrency);

Playnomics.instance.transaction(transactionId, TransactionType.BuyItem, currencies, item, itemQuantity, null);
```

## Invitations and Virality

The virality module allows you to track a singular invitation from one user to another (e.g., inviting friends to join a game).

If multiple requests can be sent at the same time, a separate function call should be made for each recipient.

```csharp
ApiResultEnum Playnomics.instance.invitationSent(
  long invitationId,
  string recipientUserId,
  string recipientAddress,
  string method)

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
            <td><code>invitationId</code></td>
            <td>long</td>
            <td>
                A unique 64-bit integer identifier for this invitation. 

                If no identifier is available this could be a hash/MD5/SHA1 of the sender's and neighbor's IDs concatenated. <strong>The resulting identifier can not be personally identifiable.</strong>
            </td>
        </tr>
        <tr>
            <td><code>recipientUserId</code></td>
            <td>string</td>
            <td>This can be a hash/MD5/SHA1 of the recipient's Facebook ID, their Facebook 3rd Party ID or an internal ID. It cannot be a personally identifiable ID.</td>
        </tr>
        <tr>
            <td><code>recipientAddress</code></td>
            <td>string</td>
            <td>
                An optional way to identify the recipient, for example the <strong>hashed email address</strong>. When using <code>recipientUserId</code> this can be <code>null</code>.
            </td>
        </tr>
        <tr>
            <td><code>method</code></td>
            <td>string</td>
            <td>
                The method of the invitation request will include one of the following:
                <ul>
                    <li>facebookRequest</li>
                    <li>email</li>
                    <li>twitter</li>
                </ul>
            </td>
        </tr>
    </tbody>
</table>

You can then track each invitation acceptance. The important thing to note is that you will need to pass the invitationId through the invitation link.

```javascript
ApiResultEnum Playnomics.instance.invitationResponse(
    long invitationId,
    string recipientUserId)
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
            <td>invitationId</td>
            <td>long</td>
            <td>The ID of the corresponding <code>invitationSent</code> event.</td>
        </tr>
        <tr>
            <td>recipientUserId</td>
            <td>string</td>
            <td>The <code>recipientUserID</code> used in the corresponding <code>invitationSent</code> event.</td>
        </tr>
    </tbody>
</table>

Example calls for a user’s invitation and the recipient’s acceptance:

```csharp
var invitationId = 112345675;
var recipientUserId = 10000013;

Playnomics.instance.invitationSent(invitationId, recipientUserId, null, null);

//later on the recipient accepts the invitation

Playnomics.instance.invitationResponse(invitationId, recipientUserId, "accepted");
```

## Custom Event Tracking

Milestones may be defined in a number of ways.  They may be defined at certain key gameplay points like, finishing a tutorial, or may they refer to other important milestones in a player's lifecycle. PlayRM, by default, supports up to five custom milestones.  Players can be segmented based on when and how many times they have achieved a particular milestone.

Each time a user reaches a milestone, track it with this call:

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
            <td>milestoneId</td>
            <td>long</long>
            <td>a unique 64-bit numeric identifier for this milestone occurrence</td>
        </tr>
        <tr>
            <td>milestoneName</td>
            <td>string</td>
            <td>the name of the milestone which should be one of "TUTORIAL" or "CUSTOMn", where n is 1 through 5</td>
        </tr>
    </tbody>
</table>

Example client-side calls for a player reaching a milestone, with generated IDs:

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

//when the player completes the tutorial
var milestoneTutorialId = GetRandomLong();
pnMilestone(milestoneTutorialId, "TUTORIAL");

//when milestone CUSTOM2 is reached
var milestoneCustom2Id = GetRandomLong();
pnMilestone(milestoneCustom2Id, "CUSTOM2");
```

Messaging Integration
=====================
This guide assumes you're already familiar with the concept of frames and messaging, and that you have all of the relevant `frames` setup for your application.

If you are new to PlayRM's messaging feature, please refer to <a href="http://integration.playnomics.com" target="_blank">integration documentation</a>.

Once you have all of your frames created with their associated `<PLAYRM-FRAME-ID>`s, you can start the integration process.

## SDK Integration

Loading frames through the SDK:

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
            <td>frameId</td>
            <td>string</td>
            <td>Unique identifier for the frame.</td>
        </tr>
    </tbody>
</table>

Frames are loaded asynchronously to keep your game responsive. The `initMessagingFrame` call begins the frame loading process. However, until you call `show` on the frame, the frame will not be drawn in the UI. This gives you control over when a frame will appear.

If a frame or its image components cannot be loaded, the SDK will attempt to reloaded the frame.

Frames are destroyed on a scene transition or when closed.

In the example below, we initialize the frame when a behavior script is loaded for the first time. In the update loop, we poll for the frame asking if it can be shown loading and then show it. In practice, a frame can be loaded in a variety of ways.

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

        if(frame.FrameState == FrameStateEnum.Loaded && !shown){
            //the frame is ready and has never been shown

            shown = true;
            //render the frame
            frame.show();
        }
    }
}
```

## Using Code Callbacks

Depending on your configuration, a variety of actions can take place when a frame's message is pressed or clicked:

* Redirect the player to a web URL in the platform's browser application
* Firing a code callback in your game
* Or in the simplest case, just close the frame, provided that the Close Button has been configured correctly.

All of this setup, takes place at the the time of the messaging campaign configuration. However, all code callbacks need to be configured before PlayRM can interact with it. The SDK uses Unity's messaging passing framework for callbacks, so a code callback must be:

* In a script attached to a singular, uniquely-named `GameObject`
* The script method should have no parameters
* The method should return `void`

Here are three common use cases for frames and a messaging campaigns

* [Game Start Frame](#game-start-frame)
* [Event Driven Frame - Open the Store](#event-driven-frame-open-the-store) for instance, when the player is running low on premium currency
* [Event Driven Frame - Level Completion](#event-driven-drame-level-completion)

For each of the examples, we will create a script to handle the code callback, and attach it to the `GameObject` called **ClickHandler**.


### Game Start Frame

In this use-case, we want to configure a frame that is always shown to players when they start playing a new game. The message shown to the player may change based on the desired segments:

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
                Code Callback
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
                In this case, we're be worried that one once-active players are now in danger of leaving the game. We might offer them <strong>50 MonsterBucks</strong> to bring them back.
            </td>
            <td style="width:250px;">
                <img src="http://playnomics.com/integration-dev/img/messaging/50-free-monster-bucks.png"/>
            </td>
        </tr>
        <tr>
            <td>
                Lapsed 7 or more days
            </td>
            <td>2nd</td>
            <td>
                In this case, we want to thank the player from coming back and incentivize these lapsed players to continue doing so. We might offer them <strong>10 MonsterBucks</strong> to increase their engagement and loyalty.
            </td>
            <td style="width:250px;"> 
                <img src="http://playnomics.com/integration-dev/img/messaging/10-free-monster-bucks.png"/>
            </td>
        </tr>
        <tr>
            <td>
                Default - players who don't fall into either segment.
            </td>
            <td>3rd</td>
            <td>
                In this case, we can offer a special item to them for returning to the grame.
            </td>
            <td style="width:250px;">
                <img src="http://playnomics.com/integration-dev/img/messaging/free-bfb.png"/>
            </td>
        </tr>
    </tbody>
</table>

```csharp
using UnityEngine;

public class MessageClickHandler : MonoBehavior {
    
    public void grant10MonsterBucks(){
        //grant 10 MonsterBucks
    }

    public void grant50MonsterBucks(){
        //grant 50 MonsterBucks
    } 

    public void grantBazooka(){
        //grant a bazooka
    }
}
```

### Event Driven Frame - Open the Store

An advantage of a *dynamic* frame, is that it can be triggered based on an in-game event. Another use case, 

In this use-case, we want to configure a frame that is always shown to players when they start playing a new game. The message shown to the player may change based on the configured segments:

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
                Default - all players, because this message is intended for any one playing the game.
            </td>
            <td>1st</td>
            <td>
                You notice that the player's in-game, premium currency drops below a certain threshold, now you can prompt them to re-up with this <strong>message</strong>.
            </td>
            <td>
                <img src="http://playnomics.com/integration-dev/img/messaging/running-out-of-monster-bucks.png"/>
            </td>
        </tr>
    </tbody>
</table>

For this example, the Default message is 

You would create a script to handle the code callback, and attach it to the `GameObject` called **ClickHandler**:

```csharp
using UnityEngine;

public class MessageClickHandler : MonoBehavior {
    
    //...
    //...

    public void openStore(){
        //open the game store after the click has occurred
        store.open();
    }

    //...
    //... 
}
```

The related message would be configured in the Control Panel to use this callback by placing this in the **Target URL** : `pna://ClickHandler.openStore`.

### Event Driven Frame - Level Completion

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
                Non-monetizers, in their 5th day of game play
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


Support Issues
==============
If you have any questions or issues, please contact <a href="mailto:support@playnomics.com">support@playnomics.com</a>.
