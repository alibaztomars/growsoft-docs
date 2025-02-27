# growsoft-docs
Since there is no official docs for Growsoft's lua scriptig, i tried to make it with AI. Don't trust this docs %100 it was made by AI. If you find anymistakes you can report me.

# GrowSoft Scripting API Documentation

**Introduction**

The GrowSoft scripting API allows you to extend and customize the Growtopia Private Server (GTPS) experience using the [Lua](https://www.lua.org/) programming language.  You can create custom commands, modify block behavior, add new items and effects, manage player data, and much more.  Scripts are placed in the `scripts/` directory of your GrowSoft server, which are automatically loaded and executed.

**Table of Contents**

*   [Core Concepts](#core-concepts)
*   [API Reference](#api-reference)
    *   [Global Functions](#1-global-functions)
    *   [Callback Functions](#2-callback-functions)
    *   [Object Reference](#3-object-reference)
        *   [`player` Object](#player-object)
        *   [`world` Object](#world-object)
        *   [`tile` Object](#tile-object)
        *   [`item` Object](#item-object)
        *   [`mod` Object](#mod-object)
    *   [Enums](#4-enums)
        *   [`Roles`](#roles)
        *   [`StateFlags`](#stateflags)
        *   [`PlayerStats`](#playerstats)
        *   [`PlayerClothes`](#playerclothes)
        *   [`PlayerSubscriptions`](#playersubscriptions)
        *   [`PlayerStatus`](#playerstatus)
        *   [Other Enums](#other-enums)
    *   [Utility Functions](#5-utility-functions)
*   [Example: A Simple "Hello" Command](#example-a-simple-hello-command)
*   [Important Notes](#important-notes)

## Core Concepts

*   **Callbacks:** The API is event-driven.  You register *callback functions* that are executed when specific events occur in the game (e.g., a player chats, a block is broken, a player logs in). These callbacks are your primary way to interact with the game.  See [Callback Functions](#2-callback-functions).
*   **Objects:** The API provides access to various game objects, such as `world`, `player`, `tile`, `item`, and more. These objects have methods (functions) and properties (data) that you can use to get information and perform actions. See [Object Reference](#3-object-reference).
*   **Enums:** Many functions use enums (enumerations) to represent predefined values, like item IDs, roles, or state flags. This makes the code more readable and avoids using "magic numbers."  See [Enums](#4-enums).
*   **Data Storage:** GrowSoft has a storage system to save and load your data using `loadDataFromServer` and `saveDataToServer`. See [Global Functions](#1-global-functions).
*   **Playmods:** Playmods are special effects that can be added to players, modifying their appearance, abilities, and behavior. See [`registerLuaPlaymod`](#registerluaplaymodmoddata) and the [`mod` Object](#mod-object).

## API Reference

### 1. Global Functions

These functions are available globally within your Lua scripts.

*   `print(message)`: Prints a message to the server console. Essential for debugging.

    ```lua
    print("(Loaded) My Awesome Script")
    ```

*   `registerLuaCommand(commandData)`: Registers a custom chat command.

    ```lua
    local myCommandData = {
        command = "mycommand",
        roleRequired = Roles.ROLE_ADMIN, -- See Roles enum
        description = "This is my custom command!"
    }
    registerLuaCommand(myCommandData)
    ```

    *   `command`: The command name (e.g., "mycommand" for `/mycommand`).
    *   `roleRequired`: The minimum player role required to use the command (see [`Roles`](#roles) enum).
    *   `description`: A description of the command, used in help messages.

*   `registerLuaPlaymod(modData)`: Registers a custom playmod. Returns the mod ID.  See also the [`mod` Object](#mod-object).

    ```lua
    local myModData = {
        modID = -1000, -- Must be a unique negative integer
        modName = "My Cool Mod",
        onAddMessage = "You are now cool!",
        onRemoveMessage = "You are no longer cool.",
        iconID = 660,
        changeSkin = {255, 0, 0, 255},  -- RGBA
        modState = {StateFlags.STATE_DOUBLE_JUMP} -- See StateFlags enum
    }
    local myModID = registerLuaPlaymod(myModData)
    ```

    *   `modID`: A *negative* integer ID for the mod. Must be unique.
    *   `modName`: The name of the mod.
    *   `onAddMessage`: Message displayed to the player when the mod is added.
    *   `onRemoveMessage`: Message displayed when the mod is removed.
    *   `iconID`: The item ID to use as the mod's icon.
    *   `changeSkin`: Optional table. Changes skin color `{red, green, blue, alpha}`.
    *   `modState`: Optional table. List of the player state flags to apply (see [`StateFlags`](#stateflags)).
    *  `changeMovementSpeed`, `changeAcceleration`, `changeGravity`, `changePunchStrength`, `changeBuildRange`, `changePunchRange`, `changeWaterMovementSpeed` : Optional. Changes a player's stat.

*   `reloadScripts()`: Reloads all Lua scripts. Useful for testing changes without restarting the server. Requires developer role.

*   `getItem(itemID)`: Returns an [`item`](#item-object) object representing the item with the given ID.

    ```lua
    local dirtItem = getItem(2)
    print(dirtItem:getName()) -- Output: Dirt
    ```

*   `getItemsCount()`: Get the total item count.

*   `getPlayerByName(partialName)`: Returns a table of [`player`](#player-object) objects whose names start with the given partial name. Case-insensitive.

    ```lua
    local players = getPlayerByName("test")
    if #players > 0 then
        print("Found player: " .. players[1]:getName())
    end
    ```
*  `getServerPlayers()`: Get a list of players online.

*   `getEnumItem(itemName)`: Get enum item.

*   `getServerName()`: Returns server name.

*  `getNewsBanner()`, `getNewsBannerDimensions()`, `getTodaysDate()`, `getTodaysEvents()`, `getCurrentEventDescription()`, `getCurrentDailyEventDescription()`, `getCurrentRoleDayDescription()`: Returns data from config.

* `getTopWorldByVisitors()`: Returns the top world.

*   `loadDataFromServer(uniqueName)`: Loads data from the server's persistent storage. Returns the data (usually a table) or `nil` if no data is found.

    ```lua
    local myData = loadDataFromServer("my_script_data") or {}
    ```

*   `saveDataToServer(uniqueName, data)`: Saves data to the server's persistent storage. `data` is typically a table.

    ```lua
    myData.playerCount = myData.playerCount + 1
    saveDataToServer("my_script_data", myData)
    ```

*    `getEasterBuyTime(userID)`, `isDailyOfferPurchased(userID, itemID)`, `addDailyOfferPurchased(userID, itemID)`: Functions to get easter event data, check daily offers and add one.
* `getEasterEggs(userID)`, `getCorruptedSouls(userID)`: Get the user id easter eggs, get user id corrupted souls.
*   `getStoreItems()`, `getEventOffers()`, `getActiveDailyOffers()`: Returns the store items.
*  `getRealGTItemsCount()`: Get the original GT items count.

* `getTopPlayerByBalance()`: Returns the top player by total wls.
* `getIOTMItem(itemID)`: Returns the IOTM item object if it exists.

### 2. Callback Functions

These are the functions you define to respond to game events. You *must* use the correct function signature (name and parameters).

*   `onPlayerChatCallback(function(world, player, message))`

    *   `world`: The [`world`](#world-object) object where the chat occurred.
    *   `player`: The [`player`](#player-object) object who sent the message.
    *   `message`: The chat message string.
    *   **Return Value:** `true` to prevent the message from being displayed, `false` to allow it.

    ```lua
    onPlayerChatCallback(function(world, player, message)
        if string.lower(message) == "hello" then
            player:onConsoleMessage("Greetings!")
            return true -- Prevent the "hello" message from appearing
        end
        return false
    end)
    ```

*   `onTileBreakCallback(function(world, player, tile))`

    *   `world`: The [`world`](#world-object) object.
    *   `player`: The [`player`](#player-object) object.
    *   `tile`: The [`tile`](#tile-object) object representing the broken block.
    *   **Return Value:** `true` to prevent the default block breaking behavior, `false` to allow it.

    ```lua
    onTileBreakCallback(function(world, player, tile)
        if tile:getTileID() == 2 then -- Dirt
            if math.random(1, 100) <= 50 then
                world:spawnItem(tile:getPosX(), tile:getPosY(), 10, 1) -- Spawn a rock
                return true -- Prevent the dirt from dropping
            end
        end
        return false
    end)
    ```

*   `onPlayerCommandCallback(function(world, player, fullCommand))`

    *   `world`: The [`world`](#world-object) object.
    *   `player`: The [`player`](#player-object) object.
    *   `fullCommand`: The full command string, including the command name and any arguments.
    *   **Return Value:** `true` if the command was handled by your script, `false` otherwise.

    ```lua
    onPlayerCommandCallback(function(world, player, fullCommand)
        local command, args = fullCommand:match("^(%S+)%s*(.*)")
        if command == "warp" then
            -- Handle the /warp command
            return true
        end
        return false
    end)
    ```

*   `onPlayerDialogCallback(function(world, player, data))`

    *   `world`: The [`world`](#world-object) object.
    *   `player`: The [`player`](#player-object) object.
    *   `data`: A table containing information about the dialog interaction. Key fields include:
        *   `dialog_name`: The name of the dialog.
        *   `buttonClicked`: The ID of the button that was clicked.
        *   Other fields depending on the dialog type (e.g., text input values).
    *   **Return Value:** `true` if the dialog was handled, `false` otherwise.

    ```lua
    onPlayerDialogCallback(function(world, player, data)
        if data.dialog_name == "my_dialog" then
            if data.buttonClicked == "ok_button" then
                -- Do something
                return true
            end
        end
        return false
    end)
    ```

*   `onPlayerLoginCallback(function(player))`

    *   `player`: The [`player`](#player-object) object.

    ```lua
    onPlayerLoginCallback(function(player)
        player:onConsoleMessage("Welcome to the server!")
    end)
    ```
*   `onPlayerRegisterCallback(function(world, player))`:  Called when a player registers into the server.

    *   `world`:  The [`world`](#world-object) object.
    *   `player`:  The [`player`](#player-object) object.
*   `onPlayerConsumableCallback(function(world, player, tile, clickedPlayer, itemID))`
    * `world`: The [`world`](#world-object) object.
    * `player`: The [`player`](#player-object) object who used the item.
    * `tile`: The [`tile`](#tile-object) object the item was used on (may be `nil`).
    * `clickedPlayer`: The [`player`](#player-object) object that was clicked on (may be `nil`).
    * `itemID`: The ID of the consumable item.
    * **Return Value:** `true` to prevent the default consumable behavior, `false` to allow it.
*  `onAutoSaveRequest(function())`: Called each X minutes and on server shutdown, used to save data.
*  `onPlayerProfileRequest(function(world, player, tabID, flags))`: Called when a player requests to open another player profile, it opens their own player profile.
    *   `world`:  The [`world`](#world-object) object.
    *   `player`:  The [`player`](#player-object) object.
    *   `tabID`: The tab id that should be opened.
    *   `flags`: Extra flags.
    *   **Return Value:** `true` if you handled, `false` otherwise.
*   `onTileWrenchCallback(function(world, player, tile))`: Called when a player wrenches a tile.
    *   `world`: The [`world`](#world-object) object.
    *   `player`: The [`player`](#player-object) object who used the wrench.
    *   `tile`: The [`tile`](#tile-object) object that was wrenched.
    *   **Return Value:** `true` to prevent the default wrench behavior, `false` to allow it.
*   `onStoreRequest(function(world, player))`: Called when a player requests to open the store.
    *   `world`: The [`world`](#world-object) object.
    *   `player`: The [`player`](#player-object) object who requested.
    * **Return Value:** `true` to prevent the default opening, `false` to allow it.
*   `onPlayerActionCallback(function(world, player, data))`: Called when player does some actions such as opening dialogs, purchasing items, etc.
      *   `world`:  The [`world`](#world-object) object.
    *   `player`:  The [`player`](#player-object) object.
    *   `data`:  A table containing information about the action.  Key fields include:
        *   `action`: The name of the action.
        *   Other fields depending on the action type.
    *   **Return Value:** `true` if was handled, `false` otherwise.

### 3. Object Reference

#### `player` Object

*   `player:hasMod(modID)`: Checks if the player has the specified mod.
*   `player:addMod(modID, duration)`: Adds a mod to the player.
    *   `modID`: The ID of the mod.
    *   `duration`: The duration of the mod in seconds (0 for permanent).
*   `player:removeMod(modID)`: Removes a mod from the player.
    *   `modID`: The ID of the mod.
*   `player:getMod(modID)`: Returns a [`mod`](#mod-object) object representing the player's mod, or `nil` if the player doesn't have that mod.
*   `player:getMods()`: Returns a table of [`mod`](#mod-object) objects representing all of the player's active mods.
*   `player:onConsoleMessage(message)`: Sends a console message to the player.
*   `player:onTalkBubble(netID, message, type)`: Displays a talk bubble above the player.
    *   `netID`: The NetID of the player to show the bubble above (usually `player:getNetID()`).
    * `type`: 0 for chat, 1 for error.
*   `player:onTextOverlay(text, delayMS)`: Shows text overlay to the player. `delayMS` is ms delay.
*   `player:getUserID()`: Returns the player's unique user ID.
*   `player:getNetID()`: Returns the player's network ID.
*   `player:getPosX()`, `player:getPosY()`: Returns the player's X and Y coordinates (in pixels).
*   `player:getBlockPosX()`, `player:getBlockPosY()`: Returns player's X and Y coordinates (in blocks).
*   `player:getMiddlePosX()`, `player:getMiddlePosY()`: Returns the middle X, Y coordinates of the player sprite (in pixels).
*   `player:getGems()`: Returns the player's current gem count.
*   `player:addGems(amount, instant, countTowardsQuests)`: Adds gems to the player.
    *   `amount`: The number of gems to add.
    *   `instant`: `1` to add instantly, `0` to show the gem animation.
    *  `countTowardsQuests`: 1 to count for quests, 0 if not.
*   `player:removeGems(amount, instant, countTowardsQuests)`: Removes gems from the player. Returns `true` if successful, `false` if the player doesn't have enough gems.
*   `player:changeItem(itemID, amount, backpack)`: Adds or removes items from the player's inventory. Returns `true` on success.
    *   `itemID`: The ID of the item.
    *   `amount`: The number of items to add (positive) or remove (negative).
    *   `backpack`: `0` to add to inventory, `1` to add to backpack.
*   `player:hasRole(role)`: Checks if the player has the specified role (see [`Roles`](#roles) enum).
*   `player:setRole(role)`: Sets the player's role (see [`Roles`](#roles) enum).
*   `player:getItemAmount(itemID)`: Returns the number of items of the specified ID the player has.
*   `player:getAutofarm()`: Returns the autofarm object.
    *   `autofarm:getSlots()`: Returns amount of autofarm slots.
    *   `autofarm:setSlots(amount)`: Sets amount of autofarm slots.
*   `player:onDialogRequest(dialogContent)`: Sends a dialog to the player. `dialogContent` is a string containing the dialog definition.
*   `player:playAudio(name)`: Play audio to the player. `name` is the audio file name.
*   `player:updateStats(world, statID, value)`: Updates a player stat.
    *   `world`: The [`world`](#world-object) object.
    *   `statID`: The ID of the stat (see [`PlayerStats`](#playerstats)).
    *   `value`: The amount to increase.
* `player:updateClothing(world)`: Updates clothing.
     *   `world`: The [`world`](#world-object) object.
*   `player:getCleanName()`: Returns a name without any color.
*   `player:setNextDialogRGBA(r, g, b, a)`, `player:setNextDialogBorderRGBA(r, g, b, a)`, `player:resetDialogColor()`: Change color of dialogs.
*   `player:getWorldName()`: Returns the name of the world the player is currently in.
*   `player:enterWorld(worldName, text)`: Sends the player to the specified world.
*   `player:getInventorySize()`: Returns total inventory size.
*   `player:getBackpackUsedSize()`: Returns backpack used size.
*   `player:isMaxInventorySpace()`: Returns if the player reached max inventory size.
*   `player:upgradeInventorySpace(amount)`: Upgrades a user's inventory by the amount of slots.
*   `player:canFit(items)`: Checks if a player can fit items in their inventory. `items` is a table.
*   `player:getLevel()`: Returns player's level.
*   `player:getXP()`, `player:getRequiredXP()`: Returns player's XP and required XP.
*   `player:onAddNotification(texture, message, audio, flags, delay)`: Shows a notification. `delay` is delay ms.
*   `player:onParticleEffect(effectID, x, y, xSpeed, ySpeed)`: Shows a particle effect.
*   `player:onProfileUI(world, tabID)`: Shows the wrench/profile UI. `tabID` starts from 1.
    *   `world`:  The [`world`](#world-object) object.
*   `player:onTradeScanUI()`, `player:onGrow4GoodUI()`, `player:onGuildNotebookUI()`, `player:onGrowmojiUI()`, `player:onGrowpassUI()`, `player:onNotebookUI()`, `player:onBillboardUI()`, `player:onPersonalizeWrenchUI()`, `player:onOnlineStatusUI()`, `player:onFavItemsUI()`, `player:onCoinsBankUI()`, `player:onUnlinkDiscordUI()`, `player:onLinkDiscordUI()`, `player:onClothesUI(targetPlayer)`, `player:onAchievementsUI(targetPlayer)`, `player:onTitlesUI(targetPlayer)`, `player:onWrenchIconsUI(targetPlayer)`, `player:onNameIconsUI(targetPlayer)`, `player:onVouchersUI()`, `player:onMentorshipUI()`, `player:onBackpackUI(targetPlayer)`: Opens the UIs. If `targetPlayer` is nil, it opens it for own player, otherwise shows a target player data.
*   `player:getProfileGuildInfo()`, `player:getProfileAccessButton()`, `player:getProfileGuildJoinButton()`, `player:getTransformProfileButtons()`: Returns custom strings for profile UI.
* `player:getDiscordID()`: Returns a player's Discord ID.
*   `player:getPlaytime()`: Returns the player's playtime in seconds.
*   `player:getAccountCreationDateStr()`: Returns how much days ago an account was created.
*   `player:getOnlineStatus()`: Returns player online status (See [`PlayerStatus`](#playerstatus)).
*   `player:getHomeWorldID()`: Returns the player's home world ID.
* `player:getSubscription(subscriptionID)`: Returns if a player has a subscription (See [`PlayerSubscriptions`](#playersubscriptions)).
* `player:getTotalWorldLocks()`: Returns total wls balance of a player.
* `player:getUnlockedAchievementsCount()`: Returns unlocked achievements count.
*   `player:onStorePurchaseResult(message)`: Shows a message about purchasing.
*  `player:getStats(statID)`: Returns value of the stat (see [`PlayerStats`](#playerstats)).
*  `player:getClothingItemID(itemID)`: Returns item id of clothing (See [`PlayerClothes`](#playerclothes)).
*  `player:getClassicProfileContent(tab, flags)`: Returns a string of classic profile content.
*  `player:progressQuests(itemID, count)`: Progress quests, required for achievements.

#### `world` Object

*   `world:spawnItem(x, y, itemID, count)`: Spawns an item in the world.
    *   `x`, `y`: Coordinates (in pixels).
    *   `itemID`: The ID of the item.
    *   `count`: The number of items to spawn.
*   `world:spawnGems(x, y, amount)`: Spawns gems.
*   `world:onCreateChatBubble(x, y, message, type)`: Creates a chat bubble at the specified location.
*   `world:onCreateExplosion(x, y, radius, power)`: Creates an explosion.
*   `world:getPlayers()`: Returns a table of all [`player`](#player-object) objects in the world.
*   `world:getTile(x, y)`: Returns the [`tile`](#tile-object) object at the specified coordinates (in blocks).
*   `world:getVisiblePlayersCount()`: Returns players that dont have `player_invisible` mod.
* `world:getOwner()`: Returns owner of the world, it can return nil.
    *   `player:getName()`: Returns world owner name.
    * `player:getUserID()`: Returns owner user ID.
*   `world:useItemEffect(netID, itemID, targetNetID, delayMS)`: Plays an item use effect.
*   `world:updateClothing(player)`: Calls `player:updateClothing`.
*   `world:getName()`: Returns world name.
*   `world:isGameActive()`:  Returns `true` if a world is a game, otherwise `false`.
*   `world:onGameWinHighestScore()`: Ends game.

#### `tile` Object

*   `tile:getTileID()`: Returns the ID of the tile.
*   `tile:getPosX()`, `tile:getPosY()`: Returns the tile's X and Y coordinates (in pixels).
*   `tile:getNote()`: Returns the note if the tile has any.

#### `item` Object

*   `item:getID()`: Returns the item's ID.
*   `item:getName()`: Returns the item's name.
*   `item:getPrice()`: Returns the item's price (real-gt item price).
*   `item:setPrice(price)`: Sets a price.
*   `item:isObtainable()`: Returns `true` if the item is obtainable (from splicing, etc.) `false` if it can only be bought.
*   `item:getDescription()`: Returns item description.
*   `item:getInfo()`: Returns item info as a table.

#### `mod` Object

*   `mod:getItemID()`: Returns the item ID of a playmod.
*   `mod:getName(player)`: Returns the name of a playmod.
     *   `player`:  The [`player`](#player-object) object.
*   `mod:getExpireTime()`: Returns the expiration time of the mod (in seconds since the epoch).
*   `mod:getDescription(player)`: Returns a mod description.
    *   `player`:  The [`player`](#player-object) object.

### 4. Enums

#### `Roles`

```lua
Roles = {
    ROLE_NONE = 0,
    ROLE_VIP = 1,
    ROLE_SUPER_VIP = 2,
    ROLE_MODERATOR = 3,
    ROLE_ADMIN = 4,
    ROLE_COMMUNITY_MANAGER = 5,
    ROLE_CREATOR = 6,
    ROLE_GOD = 7,
    ROLE_DEVELOPER = 51
}
```

#### `StateFlags`

```lua
StateFlags = {
    STATE_NO_CLIP = 0,
    STATE_DOUBLE_JUMP = 1,
    STATE_INVISIBLE = 2,
    STATE_NO_HAND = 3,
    STATE_NO_EYE = 4,
    STATE_NO_BODY = 5,
    STATE_DEVIL_HORNS = 6,
    STATE_GOLDEN_HALO = 7,
    STATE_FROZEN = 11,
    STATE_CURSED = 12,
    STATE_DUCT_TAPED = 13,
    STATE_CIGAR = 14,
    STATE_SHINING = 15,
    STATE_ZOMBIE = 16,
    STATE_RED_BODY = 17,
    STATE_HAUNTED_SHADOWS = 18,
    STATE_GEIGER_RADIATION = 19,
    STATE_SPOTLIGHT = 20,
    STATE_YELLOW_BODY = 21,
    STATE_PINEAPPLE_FLAG = 22,
    STATE_FLYING_PINEAPPLE = 23,
    STATE_SUPER_SUPPORTER_NAME = 24,
    STATE_SUPER_PINEAPPLE = 25,
    STATE_BUBBLE = 26,
    STATE_SOAKED = 27
};
```

#### `PlayerStats`

```lua
PlayerStats = {
    PlacedBlocks = 0,
    HarvestedTrees = 1,
    SmashedBlocks = 2,
    GemsSpent = 3,
    ItemsDisposed = 4,
    ConsumablesUsed = 5,
    ProviderCollected = 6,
    MixedItems = 7,
    FishRevived = 8,
    StarshipFall = 9,
    GhostsCaptured = 10,
    MindGhostsCaptured = 11,
    AnomalizersBroken = 12,
    AnomHammerBroken = 13,
    AnomScytheBroken = 14,
    AnomBonesawBroken = 15,
    AnomAnomarodBroken = 16,
    AnomTrowelBroken = 17,
    AnomCultivatorBroken = 18,
    AnomScannerBroken = 19,
    AnomRollingPinsBroken = 20,
    SurgeriesDone = 21,
    GeigerFinds = 22,
    VillainsDefeated = 23,
    StartopianItemsFound = 24,
    FuelUsed = 25,
    FishTrained = 26,
    RoleUPItemsCrafted = 27,
    CookedItems = 28,
    FiresPutout = 29,
    AncestralUpgraded = 30,
    ChemsynthCreated = 31,
    MaladyCured = 32,
    GhostBossDefeated = 33,
    StarshipsLanded = 34,
    MagicEggsCollected = 35,
    EasterEggsFound = 36,
    UltraPinatasSmashed = 37,
    GrowganothFeed = 38,
    GrowchGifted = 39,
    RarityDonated = 40
}
```

#### `PlayerClothes`
```lua
PlayerClothes = {
    HAIR_ITEM = 0,
    SHIRT_ITEM = 1,
    PANTS_ITEM = 2,
    FEET_ITEM = 3,
    FACE_ITEM = 4,
    HAND_ITEM = 5,
    BACK_ITEM = 6,
    MASK_ITEM = 7,
    NECK_ITEM = 8,
    ANCES_ITEM = 9
}
```

#### `PlayerSubscriptions`
```lua
PlayerSubscriptions = {
    TYPE_SUPPORTER = 0,
    TYPE_SUPER_SUPPORTER = 1,
    TYPE_YEAR_SUBSCRIPTION = 2,
    TYPE_MONTH_SUBSCRIPTION = 3,
    TYPE_GROWPASS = 4,
    TYPE_TIKTOK = 5,
    TYPE_BOOST = 6,
    TYPE_STAFF = 7
}
```

#### `PlayerStatus`
```lua
PlayerStatus = {
	PLAYER_ONLINE = 0,
	PLAYER_BUSY = 1,
	PLAYER_AWAY = 2
}
```

#### Other Enums
`ServerEvents`, `DailyEvents`, `StoreCat`, `ProfileCat`, `MissionsCat`, `MissionsTypes`: Other enums for various parts of the server. These are not fully documented here, but you can discover them by examining the server code and through experimentation.

### 5. Utility Functions

*   `startsWith(str, start)`: Checks if a string starts with another string.

    ```lua
    if startsWith("hello world", "hello") then
        print("It starts with hello!")
    end
    ```

*   `formatNum(num)`: Formats a number with commas (e.g., 1234567 becomes "1,234,567").

    ```lua
    print(formatNum(1234567)) -- Output: 1,234,567
    ```
*   `formatTime(targetTime, currentTime)`: Formats time.
*   `resetColor(text)`: Resets color.

## Example: A Simple "Hello" Command

```lua
-- hello_command.lua

-- Assuming Roles enum is defined elsewhere or globally
Roles = {
    ROLE_NONE = 0,
    -- ... other roles ...
}

local helloCommandData = {
    command = "hello",
    roleRequired = Roles.ROLE_NONE,
    description = "Says hello to you!"
}
registerLuaCommand(helloCommandData)

onPlayerCommandCallback(function(world, player, command)
    if command == "hello" then
        player:onConsoleMessage("Hello, " .. player:getName() .. "!")
        return true -- Command handled
    end
    return false -- Command not handled
end)
```

## Important Notes

*   **Error Handling:** The provided code doesn't include much explicit error handling. In a production environment, you should add checks for `nil` values and potential errors to prevent your scripts from crashing the server.  Always check return values of functions that might fail.
*   **Security:** Be mindful of security when writing scripts, especially when dealing with player input and data. Avoid giving players excessive power or access to sensitive information. Sanitize user inputs.
*   **Performance:** Optimize your code for performance. Avoid unnecessary loops or calculations, especially within frequently called callbacks.  Use local variables whenever possible.
*   **Naming Conventions:** Use descriptive names for variables, functions, and files.  This improves readability and maintainability.  Follow a consistent style (e.g., `snake_case` for variables and functions).
*   **Documentation:** Add comments to your code to explain what it does.  This is crucial for collaboration and long-term maintenance.  Keep this README.md file up-to-date with any changes to your API.
*   **Testing:** Test your scripts thoroughly in a controlled environment before deploying them to a live server. Use `print` statements liberally for debugging. Consider setting up a separate test server.
* **Modularity:** Break down large scripts into smaller, more manageable modules. This makes your code easier to understand, test, and reuse.

This documentation covers the key elements revealed in the provided code snippets. It provides a solid foundation for getting started with GrowSoft scripting. Remember to experiment, test thoroughly, and refer back to this documentation as you develop your own scripts. Good luck!
