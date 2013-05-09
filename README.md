Playnomics PlayRM Unity SDK Integration Guide
=============================================
This guide showcases the features of the PlayRM Unity SDK and shows how to integrate the SDK with your game. Our SDK provides game developers with tools for tracking player behavior and engagement so that they can:

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
    * [Enabling Click-to-JS](#enabling-click-to-js)
* [Support Issues](#support-issues)
    * [Troubleshooting](#troubleshooting)


Prerequisites
=============
Before you can integrate with the PlayRM SDK you'll need to sign up and register your game.

## Signing Up for the PlayRM Service

Visit <a href="https://controlpanel.playnomics.com/signup" target="_blank">https://controlpanel.playnomics.com/signup</a> to create an account. The control panel is the dashboard to manage all of the PlayRM features once the SDK integration has been completed.

## Register Your Game
After receiving a registration confirmation email, login to the <a href="https://controlpanel.playnomics.com" target="_blank">control panel</a>. Select the "Applications" tab and create a new application. Your application will be granted an Application ID (`<APPID>`) and an API KEY.

## Considerations for Cross-Platform Games


Basic Integration
=================

## Installing the SDK
You can download the SDK package from downloading the *PlaynomicsSDK.unitypackage* file included in the *build* folder of this repo, or you can also install the SDK files directly from the <a href="https://www.assetstore.unity3d.com/#/content/6179" target="_blank">Unity Asset Store</a>. If you download the package file from the repo, go to *Assets > Import New Asset ... > Custom Package * to import the package.

<img style="margin-left:auto;margin-right:auto;display:block;" src="http://www.playnomics.com/integration/img/unity/prefab.png"/>

The package has a prefab with everything you need to work with the SDK. Simply drag the prefab into your first game scene and this will make the SDK available to your game. The `GameObject` for this is called "Playnomics." You only need to do this once.

### Interacting with PlayRM in Your Game

All calls are made through a single `MonoBehavior` script that persists throughout the game (<a href="http://en.wikipedia.org/wiki/Singleton_pattern" target="_blank">singleton</a>). The static instance of the script is accessible throughout your game using this property `Playnomics.instance`. It is instantiated in the `Awake` event.

When working with the SDK, you'll need to import the SDK namespace to work with the PlayRM SDK in your scripts:

```csharp
//C#
using PlaynomicsPlugin;
```
```javascript
//JavaScript
import PlaynomicsPlugin;
```

All public methods return an enumeration `ApiResultEnum` with a value of `NotStarted` or `Success`. `NotStarted` indicates that the PlayRM session has not been started, covered in the next section.

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
            <td></td>
        </tr>
        <tr>
            <td><code>subdivision</code></td>
            <td>string</td>
            <td></td>
        </tr>
        <tr>
            <td><code>birthyear</code></td>
            <td>short?</td>
            <td>4-digit year, such as 1980</td>
        </tr>
        <tr>
            <td><code>source</code></td>
            <td>string</td>
            <td>source of the user, such as "FacebookAds", "UserReferral", "Playnomics", etc. These are only suggestions, any 16-character or shorter string is acceptable.</td>
        </tr>
        <tr>
            <td><code>sourceCampaign</code></td>
            <td>string</td>
            <td>any 16-character or shorter string to help identify specific campaigns</td>
        </tr>
        <tr>
            <td><code>sourceUser</code></td>
            <td>strings</td>
            <td>if the user was acquired via a UserReferral (i.e., a viral message), the `userId` of the person who initially brought this user into the game</td>
        </tr>
        <tr>
            <td><code>installTime</code></td>
            <td>long?</td>
            <td>unix epoch time in seconds when the user originally installed the game</td>
        </tr>
    </tbody>
</table>

Since PlayRM uses the game client’s IP address to determine geographic location, country and subdivision are often set to null.

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
`type` is a short name (up to 16 characters) for the currency, eg: "Gold Coins."

We hightlight three common use-cases below.
* [Purchases of In-Game Currency with Real Currency](#purchases-of-in-game-currency-with-real-currency)
* [Purchases of Items with Real Currency](#purchases-of-items-with-real-currency)
* [Purchases of Items with In-Game Currency](#purchases-of-items-with-in-game-currency)

### Purchases of In-Game Currency with Real Currency

A very common monetization strategy is to incentivize players to purchase premium, in-game currency with real currency. PlayRM treats this like a currency exchange. This is one of the few cases where multiple `TranactionCurrency` are used in a transaction. `itemId`, `quantity`, and `otherUserId` are left `null`.

```csharp
//player purchases 500 Gold Coins for 10 USD

TransactionCurrency[] currencies = new TransactionCurrency[2]; 

var quantityCoins = 500;
var gameCurrency = "Gold Coins";
currencies[0] = TransactionCurrency.createVirtual(quantityCoins, gameCurrency);

var priceInUSD = 10;
currencies[1] = TransactionCurrency.createReal(priceInUSD, CurrencyType.USD);

Playnomics.instance.transaction(transactionId, TransactionType.CurrencyConvert, currencies, null, null, null);
```

### Purchases of Items with Real Currency

```csharp
//player purchases a "Sword" for $.99 USD

var swordItemId = "Sword"
var quantitySword = 1;
var priceOfSword = .99;

TransactionCurrency[] currencies = new TransactionCurrency[1]; 
currencies[0] = TransactionCurrency.createReal(priceOfSword, CurrencyType.USD);

Playnomics.instance.transaction(transactionId, TransactionType.BuyItem, currencies, swordItemId, quantitySword, null);
```

### Purchases of Items with Premium Currency

This event is used to segment monetized users (and potential future monetizers) by collecting information about how and when they spend their premium currency (an in-game currency that is primarily acquired using a *real* currency). This is one level of information deeper than the previous use-cases.

#### Currency Exchanges

This is a continuation on the first currency exchange example. It showcases how to track each purchase of in-game *attention* currency (non-premium virtual currency) paid for with a *premium*:

```csharp

//In this hypothetical, Energy is an attention currency that is earned over the lifetime of the game. 
//They can also be purchased with the premium Gold Coins that the player may have purchased earlier.

//player buys 100 Energy with 10 Gold Coins
//notice that both currencies are virtual
TransactionCurrency[] currencies = new TransactionCurrency[2]; 

var attentionCurrency = "Energy";
var attentionAmount = 100;
currencies[0] = TransactionCurrency.createVirtual(attentionAmount, attentionCurrency);

var premimumCurrency = "Gold Coins";
var premiumCost = -20;
currencies[1] = TransactionCurrency.createVirtual(premiumCost, premimumCurrency);

Playnomics.instance.transaction(transactionId, TransactionType.CurrencyConvert, currencies, null, null, null);
```
#### Item Purchases

This is a continuation on the first item purchase example, except with premium currency.

```javascript
//player buys 20 light armor, for 5 Gold Coins

var itemQuantity = 20;
var item = "Light Armor";

var premimumCurrency = "Gold Coins";
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

```javascript
ApiResultEnum Playnomics.instance.milestone(
    long milestoneId,
    string milestoneName);
```

These parameters should be replaced:
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
Before you start working with messaging, you'll need to complete the installation process. PlayRM messaging real estate is called a **frame** and its responsible for delivering segment-based messages to players.

The coordinate system for drawing a frame is 2D plane the the origin at the top-left of the screen, *x* going positive right, *y* going positive down.

<img src="http://www.playnomics.com/integration/img/mobile-ad-layout.png"/>

Each frame can be positioned at a *fixed* location, based on a static *x* and *y* location, or *dynamic* location, based on a 3x3 grid defined by horizontal and vertical justification parameters. If the location is *dynamic*, PlayRM is able to calculate the origin (top-left corner starting point) of the real-estate based on the screen size.

<table>
    <thead>
        <tr>
            <th>Justification</th>
            <th>Axis</th>
            <th>Frame Effect</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Left</td>
            <td>Horizontal</td>
            <td>The left-edge of the frame will be at 0 along the x-axis.</td>
        </tr>
        <tr>
            <td>Center</td>
            <td>Horizontal</td>
            <td>The frame will be centered along the x-axis.</td>
        </tr>
        <tr>
            <td>Right</td>
            <td>Horizontal</td>
            <td>The right-edge of the frame will be at the maximum of the x-axis.</td>
        </tr>
        <tr>
            <td>Top</td>
            <td>Vertical</td>
            <td>The top-edge of the frame will be at 0 of the y-axis.</td>
        </tr>
        <tr>
            <td>Center</td>
            <td>Vertical</td>
            <td>The frame will be centered along the y-axis</td>
        </tr>
        <tr>
            <td>Bottom</td>
            <td>Vertical</td>
            <td>The bottom-edge of the frame will be at the maximum of the y-axis.</td>
        </tr>
    </tbody>
</table>

**Important!**
<hr/>
Before releasing the messaging integration to production, you will need to log into the <a href="https://controlpanel.playnomics.com/signin/" target="_blank">control panel</a> and ensure that you have uploaded the creatives/messages or placeholders. **A frame always needs to have a default creative before it can be launched.**

## Setting up a Frame
Each opportunity for unique messaging, should have its own **frame** configuration.

PlayRM Messaging for Unity supports images:
* PNG or JPG image format
* Size must be less than 512kB
* Conditional images are not supported

To configure a **frame**, email <a href="mailto:support@playnomics.com">support@playnomics.com</a> the following information for each **frame**:

<table>
    <thead>
        <tr>
            <th>
                Variable
            </th>
            <th>
                Explanation
            </th>
            <th>
                Default Value
            </th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Name of the application</td>
            <td>

            </td>
            <td>
                N/A
            </td>
        </tr>
        <tr>
            <td>Name of the frame</td>
            <td></td>
            <td></td>
        </tr>
        <tr>
            <td></td>
            <td></td>
            <td>N/A</td>
        </tr>
    </tbody>
</table>

Name of app
Name of frame (i.e. "Top Banner", "Sidebar", or "Box1")
* Height in pixels (eg "90")
* Width in pixels (eg "760")


Once the frame has been configured, Playnomics will provide you with a `<PLAYRM-FRAMEID>`.

## SDK Integration

Support Issues
==============
If you have any questions or issues, please contact <a href="mailto:support@playnomics.com">support@playnomics.com</a>.
