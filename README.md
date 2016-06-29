<!--
@Author: Ranjithkumar Matheswaran <ranjithkumar>
@Date:   2016-06-28T18:21:03+05:30
@Last modified by:   ranjithkumar
@Last modified time: 2016-06-29T02:46:31+05:30
-->


# WOMI Game Template - Beta
Added all the necessary generic codes like splash screen, back key management for the android etc. and integrations like Flurry, OneSignal, GameCenter, Google Play are integrated. So developers just need to start the game from the template.

**INTEGRATIONS LIST**

```
CODE
    - Utility File
    - Game Manager  (Manages game data in local database)
    - Womi Splash screen
    - Back Key Handling in Android
    - Keyboard disappear when screenshots are shared from notification center in Android
    -  ATS disable for iOS
    - Constants to get the device dimensions
    - Notification icons specially for One Signal - Android
    - Modified Config file
    - Reachability Test function
PLUGINS
    - One Signal
    - Rate Me
    - Womi More Apps
    - Flurry Analytics
    - Apple Game Centre for iOS
    - Google Play for Android
```

# Usage
1. Copy the folder `womi_game_template_root` which is your project directory.
2. Set the keys for `oneSignalAppId`,`googleProjectNumber` and `flurryAnalyticsKey` in `helpers/utility.lua`
3. Change the `dbName` variable value from `WOMIGAME_LOCAL_DATA_CONTAINER.json` if you want a special name for your local data file.
4. In `build.setting` set the following values.
    1. `version` - version for your app. Eg. `1.0.0`
    2. `build` - Build Number Eg. `45`
    3. `iOSBundleId` - iOS Bundle id (`com.womistudios.appName.iphone`)
    4. `androidAppId` - App Id (`com.womistudios.appName.aphone`)
    5. `CFBundleDisplayName` -  Display name for iOS
    6. `googlePlayGamesAppId` - Google Play Id for Leaderboards
5. In the `config.lua`, the values `w` and `h` are set according to the device as default by dynamically calculating the height and width. If you dont want this settings then you can uncomment the other lines having fixed width and height.
5. Fill out the following variables in `coronaRateMe/rateMeUtilities.lua`

Variable  |  Usage
------  | ---------
`firstAlertAfter`  |  no. of launches after the first time alert rate us alert comes.
`laterAlertAfter`  |  no. of launches gap between the alerts
`appIconUrl`  |  App icon local url 60 x 60
`appStoreId`  |  App Store Id. Get this from iTunes Connect.
`androidPackageName`  |  Android Package name
`appName` |   App Name to be displayed.


# App Structure
The app is structured so that the scenes, utility files, image files are separated and categorized into separate folders.

Copy the folder `womi_game_template_root` folder and this is your project folders.



- Root Project Folder
    - assets/
        - splashscreen/
            - ipad_splash.png
            - womi_splash.png
    - build.settings
    - config.lua
    - coronaRateMe/
        - rateMe.lua
        - rateMeUtilities.lua
        - Image Files (14 files)
    - coronawomiapps/
        - cancel.png
        - coronamoreapps.lua
    - helpers/
        - corona_composer_template.lua
        - gamemanager.lua
        - utility.lua
    - Icon**.png (23 files)
    - IconNotificationDefault**.png (11 files)
    - main.lua
    - scenes/
        - menu.lua



Make sure you have the above project structure.

# Documentation :

### **`utility.lua`**
utility.lua will contain all the contants and function that are needed for most of the places. List of functions that are available for the developers to access.

FUNCTIONS  |  USAGE
--  | --
`getNewHeight(image , newWidth)`  |  To resize the Height of the `image` for the given `newWidth`
`getNewWidth(image, newHeight)`  |  To resize the Width of the `image` for the given `newHeight`
`osType()`  |  Return `ios` for Apple Devices and returns `android` for all android devices.
`colorsRGB(red, green, blue)`  |  Takes the color in RGB value in range 0 to 255 and returns in 0 to 1 value.
`getOneSignalAppId()`  |  Gives One signal App id.
`getGoogleProjectNumber()`  |  Gives Google Project Number.
`getFlurryAnalyticsKey()`  |  Gives the Flurry Analytics Key
`analyticsTrackEvent(eventName)`  |  Call this one to track the event for Analytics. Please use this function so that, changing the Analytics provider will be so easy.
`isRechable()`  |   returns `true` if network is available or `false` when not available based on the network condition. Handled both for the iOS and Android.


### **`gamemanager.lua`**
`gamemanager.lua` provides interface to save the game data in the app in local documents directory in `json` format.

FUNCTIONS  |  USAGE
----  | ----
`loadTable(filename)`    |   Used to retrieve the saved data.
`function saveTable(t, filename)`   |   Used to save the data in form of json
`checkForDBData()`  |   Sanity check for the json file save and read. Also declare all the default values for the game data.

You can save some data in the `gamemanager` using the setter and getter methods as showin in the example commented out.

```lua
-- SETTER
function M.setHighScore(highscore)
    checkForDBData()
    local data = loadTable(dbName)
    data.highScore = highscore
    saveTable(data,dbName)
end

-- GETTER
function M.getHighScore()
    checkForDBData()
    local data = loadTable(dbName)
    return data.highscore
end
```

### **BACK KEY HANDLING**
For the Android devices the back keys are handled. Current handling is, when there is any overlay,the overlays are closed.

Handle all other scenes in the function `onKeyEvent` like this.

```lua
local currentSceneName = composer.getSceneName( "current" )
if currentSceneName == "howtoplay" then
    composer.gotoScene("menu")
    return true
elseif currentSceneName == "tutorial" then
    composer.gotoScene("menu")
    return true
else
    -- SHOW THE CUSTOM EXIT POPUP
end
```

### **KEYBOARD FOCUS**
Consider this scenario. You are taking a screenshot, then sharing the screen shot in mail, right from the notification bar. At this scenario, the keyboard appears on the screen. To handle this, added a runtime `system` event listener, ans closing the keyboard when the app resumes.

```lua
function systemEvent(event)
    if event.type == "applicationResume" then
        native.setKeyboardFocus(nil)
    end
end

Runtime:addEventListener("system",systemEvent)
```

### **ATS for iOS**
Application Transport Security from iOS 9, allows only the `https`. To allow the `http` calls, we need to disable this manually in `build.setting`.

```
iphone = {
    NSAppTransportSecurity =
    {
        NSAllowsArbitraryLoads = true,
    }
}
```

### **GLOBAL CONSTANTS & FUNCTIONS**
Few constants and functions are declared at `main.lua` for the convinient usage in the app.
**CONSTANTS**
```lua
_Wi = display.pixelWidth * display.contentScaleX  -- width will be equal to the Width of the device
_He = display.pixelHeight * display.contentScaleY  --height will be equal to the height of the device
_cX = display.contentCenterX
_cY = display.contentCenterY
_cH = display.contentHeight
_cW = display.conetentWidth
```

**FUNCTIONS**
This function can be accessed anywhere in the app.

```lua
mainInitLeaderBoards()  --Init the leaderboard from anywhere in the app
mainSendScoreToLeaderboard() -- Send scores to the leaderBoard
mainShowLeaderboard() --Show the leaderBoard
```

### **NETWORK REACHABILITY**
A function is added in the `utility.lua` to check the reachability. Returns in `Boolean`

```lua
if utility.isReachable() then
    --Network is Available
else
    --Network is not Available. Probably show an alert that network is not available.
end
```

### **ANDROID ONESIGNAL NOTIFICATION ICONS**
Android needs seperate icons in the folder for the Notifications. It will appear in the status bar.

![image](IconNotificationDefault-xxxhdpi.png)

The above icon will appear in the status bars of Android. The following files are added in the project.
```
IconNotificationDefault-hdpi-v11.png
IconNotificationDefault-hdpi.png
IconNotificationDefault-ldpi.png
IconNotificationDefault-mdpi-v11.png
IconNotificationDefault-mdpi.png
IconNotificationDefault-xhdpi-v11.png
IconNotificationDefault-xhdpi.png
IconNotificationDefault-xxhdpi-v11.png
IconNotificationDefault-xxhdpi.png
IconNotificationDefault-xxxhdpi-v11.png
IconNotificationDefault-xxxhdpi.png
```


## SDK INTEGRATIONS

### **GAMECENTER AND LEADERBOARD**
Both GameCenter and Google Play are integrated. All codes are added up in `main.lua`.

Game Network for iOS and Android can be initialised by calling the following method anywhere in the app.

```lua
mainInitLeaderBoards()
```

Send the scores to the leaderboard by calling, where `score` is the score in Int or Float. and `leaderBoard` is the leaderboard name for either Apple Game Centre or Google Play.

```lua
mainSendScoreToLeaderboard( score, leaderBoard )
```

When you want to show the leaderboard at any scene, just call by passing the leaderboard you want to open.

```lua
mainShowLeaderboard(leaderboardId)
```

### **ONE SIGNAL**
One Signal is integarated and receives notification. You can handle the inApp notification and the push notification at `DidReceiveRemoteNotification` function in the `main.lua`
```lua
oneSignal.Init(utility.getOneSignalAppId(), utility.getGoogleProjectNumber(), DidReceiveRemoteNotification)
```

### **RATEME**
Rate me integrated for automatic alert in `menu.lua` in the `scene:show` function.If you want to call manually you can call by following syntax.

```lua
local rateMeUtilities = require "coronaRateMe.rateMeUtilities"
if rateMeUtilities.checkForAlert() then
    local options =
    {
      isModal = true,
      effect = "fade",
      time = 200,
      params = {
        text = "Would you like to rate the App ?",
      }
    }
  composer.showOverlay("coronaRateMe.rateMe",options)
  ```

### **WOMI MORE APPS**
Womi more and its files are integrated. Call them by using,
```lua
if utility.isRechable() == true then
    local options =
  {
    isModal = true,
    effect = "fade",
    time = 200,
  }
    composer.showOverlay("coronawomiapps.coronamoreapps",options)
else
    native.showAlert( "No Internet","It seems internet is not Available. Please connect to internet.", { "OK" })
end
```

### **FLURRY ANALYTICS**
Flurry Analytics is integrated. Dont forget to add the Flurry key in `utilities.lua`

To track an event, use

```lua
utility.analyticsTrackEvent("event_name")
```

Please use this function instead of the original function, as in order to change the analytics network it is very easy to change in once place rather than changing all over the game.

### **SPLASH SCREEN**

Splash screen is integrated in the `main.lua` scene. Once the splash screen loads for `2000 ms`, it calls the listener `splashListener` which opens up the menu screen.

Start coding from the menu screen, or do a custom navigation. But make sure you add the `rateme` accordingly as it is added in the `menu.lua` scene.


## **CONTRIBUTORS**
- Ranjithkumar Matheswaran <ranjithkumar@perk.com> / [iranjith4](http://twitter.com/iranjith4)
