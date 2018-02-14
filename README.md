# BaseMod #
BaseMod provides a number of hooks and a console.

Currently supported version: `[EARLY_ACCESS_015]` (non beta)

## Requirements ##
#### General Use ####
* Java 8+
* ModTheSpire v2.2.1+ (https://github.com/kiooeht/ModTheSpire/releases)

#### Development ####
* Java 8
* Maven
* CFR 124 (run this with Java 8, doesn't work well with 9)
* ModTheSpireLib (https://github.com/kiooeht/ModTheSpireLib)

## Building ##
1. (If you haven't already) `mvn install` ModTheSpireLib. Altenatively, modify pom.xml to point to a local copy of the JAR.
2. Copy `desktop-1.0.jar` from your Slay the Spire folder into `../_lib` relative to the repo.
3. Decompile `desktop-1.0.jar` with `java -jar "cfr_0_124.jar" --comments false --showversion false --caseinsensitivefs true --outputdir "decompiled" --jarfilter com.megacrit.cardcrawl.* "desktop-1.0.jar"`
4. Run `_patch.bat` to automatically apply diffs
5. Run `mvn package`

## Installation ##
1. Copy `target/BaseMod.jar` to your ModTheSpire mods directory. Maven will automatically do this after packaging if your mods directory is located at `../_ModTheSpire/mods` relative to the repo.

## Console ##
Default hotkey is `` ` ``, can be changed from BaseMod's settings screen.
* `deck add [id] {upgrades}` add card to deck (optional: integer # of upgrades)
* `deck r [id]` remove card from deck
* `draw [num]` draw cards
* `energy add [amount]` gain energy
* `energy inf` toggles infinite energy
* `energy r [amount]` lose energy
* `fight [name]` enter combat with the specified encounter
* `gold add [amount]` gain gold
* `gold r [amount]` lose gold
* `hand add [id] {upgrades}` add card to hand with (optional: integer # of upgrades)
* `hand r all` exhaust entire hand
* `hand r [id]` exhaust card from hand
* `info` toggle Settings.isInfo
* `kill all` kills all enemies in the current combat
* `kill self` kills your character
* `potion [pos] [id]` gain specified potion in specified slot
* `relic add [id]` generate relic
* `relic list` logs all relic pools
* `relic r [id]` lose relic

## For Modders ##
### Hooks ###
#### Subscription handling ####
* `BaseMod.subscribeTo...(this)`
* `BaseMod.unsubscribeFrom...(this)`

#### Subscriptions ####
Implement the appropriate interface (ex. `basemod.interfaces.PostInitializeSubscriber`)
All interfaces are in the package `basemod.interfaces` (also **receive** is probably misspelled somewhere so sorry about that)
* `boolean receiveStartAct()` - After a new act is started.
* `boolean receivePostCampfire()` - After a campfire action is performed. Returning false will allow another action to be performed.
* `void receivePostDraw(AbstractCard)` - After a card is drawn.
* `void receivePostDungeonInitialize()` - After dungeon initialization completes.
* `void receivePostEnergyRecharge()` - At the start of every player turn, after energy has recharged.
* `void receivePostInitialize()` - One time only, at the end of `CardCrawlGame.initialize()`.
* `boolean receivePreMonsterTurn(AbstractMonster)` - Before each monster takes its turn. Returning false will skip the monsters turn.
* `void receiveRender(SpriteBatch)` - Under tips and the cursor, above everything else.
* `void receivePostRender(SpriteBatch)` - Above everything.
* `void receivePreStartGame()` - When starting a new game, before generating/loading the player.
* `void receiveStartGame()` - When starting a new game or continuing, after generating/loading the player and before dungeon generation.
* `void receivePreUpdate()` - Immediately after input is read.
* `void receivePostUpdate()` - Immediately before input is disposed.
* `boolean receivePostCreateStartingDeck(cardsToAdd)` - Immediately after the character's starting deck is created. Returning true will remove all the cards from the default starting deck. Add the cards you want to be in the starting deck to `cardsToAdd`.
* `boolean receievePostCreateStartingRelics(relicsToAdd)` - Immediately after the character's starting relics are created. Returning true will remove all the default relics from the player. Add the relics you want to be in the starting deck to `relicsToAdd`.
* `void receivePostCreateShopRelics(relics, sceenInstance)` - Immediately after the shop generates its relics. Modifying `relics` will change the relics in the shop. `screenInstance` is an instance of the `ShopScreen`.
* `void receivePostCreateShopPotions(potions, screenInstance)` - Immediately after the shop generates its potions. Modifying `potions` will change the potions in the shop. `screenInstance` is an instance of the `ShopScreen`.
* `void receiveEditCards` - When you should register any cards to add or remove with `BaseMod.addCard` and `BaseMod.removeCard`. Do **NOT** initialize any cards or register any cards to add or remove outside of this handler. Slay the Spire needs some things to be done in certain orders and this handler ensures that happens correctly. Note that removing any cards involved in game events is **undefined behavior** currently.
* `void receiveEditRelics` - When you should register any relics to add or remove with `BaseMod.addRelic` and `BaseMod.removeRelic`. Do **NOT** initialize any relics or register any relics to add or remove outside of this handler. Slay the Spire needs some things to be done in certain orders and this handler ensures that happens correctly. Note that removing any relics involved in game events is **undefined behavior** currently.
* `void receiveEditCharacters` - When you should register any characters to add or remove with `BaseMod.addCharacter` and `BaseMod.removeCharacter`. Do **NOT** initialize any characters or register any relics to add or remove outside of this handler. Slay the Spire needs some things to be done in certain orders and this handler ensures that happens correctly. Note that removing the default characters **IS NOT** supported at this time.

### Mod Badges ###
32x32 images that display under the title on the main menu. Clicking one opens that mods settings menu.
* `BaseMod.registerModBadge(Texture texture, String modName, String author, String description, ModPanel settingsPanel)`
* `ModPanel.addButton(float x, float y, Consumer<ModButton> clickEvent)`
* `ModPanel.addLabel(String text, float x, float y, Consumer<ModLabel> updateEvent)`
* There is more here, but it is a big mess and is going to be cleaned up soon.

Example of setting up a basic mod badge with settings panel:

```java
ModPanel settingsPanel = new ModPanel();
settingsPanel.addLabel("", 475.0f, 700.0f, (me) -> {
    if (me.parent.waitingOnEvent) {
        me.text = "Press key";
    } else {
        me.text = "Change console hotkey (" + Keys.toString(DevConsole.toggleKey) + ")";
    }
});

settingsPanel.addButton(350.0f, 650.0f, (me) -> {
    me.parent.waitingOnEvent = true;
    oldInputProcessor = Gdx.input.getInputProcessor();
    Gdx.input.setInputProcessor(new InputAdapter() {
        @Override
        public boolean keyUp(int keycode) {
            DevConsole.toggleKey = keycode;
            me.parent.waitingOnEvent = false;
            Gdx.input.setInputProcessor(oldInputProcessor);
            return true;
        }
    });
});

Texture badgeTexture = new Texture(Gdx.files.internal("img/BaseModBadge.png"));
registerModBadge(badgeTexture, MODNAME, AUTHOR, DESCRIPTION, settingsPanel);
```
Take a look at `basemod.BaseModInit` to see the code used to create the `ModBadge` for BaseMod.

### Custom Relics ###
* `CustomRelic(String id, Texture texture, AbstractRelic.RelicTier tier, AbstractRelic.LandingSound sfx)`
* `BaseMod.addRelic(AbstractRelic relic, RelicType type)` (note: `CustomRelic` extends `AbstractRelic`) and `RelicType` indicates if this relic is shared between both characters or `RED` only or `GREEN` only.
* `BaseMod.removeRelic(AbstractRelic relic, RelicType type)` removes a relic from the game (note: removing a relic used by an event is currently untested/undefined behavior)
* `BaseMod.removeRelic(AbstractRelic relic)` removes a relic from the game without having to know its `RelicType`

### Custom Cards ###
* `CustomCard(String id, String name, String img, int cost, String rawDescription, CardType type, CardColor color, CardRarity rarity, CardTarget target, int cardPool)`
* `BaseMod.addCard(AbstractCard card)` (note: `CustomCard` extends `AbstractCard`).
* `BaseMod.removeCard(AbstractCard card)` removes a card from the game (note: removing a card used by an event is currently untested/undefined behavior)

### Custom Player Characters ###
The process for creating custom player characters is fairly involved but still not too complex. It is detailed below:

1. To add a custom character there are two major steps. You need to register both a new color and a new character.  Basically the Ironclad is RED, the Silent is GREEN, the unused content Crowbot is BLUE, etc... In making a custom character you would want to make a new color like maybe YELLOW or PURPLE or ORANGE.
2. Since the base game represents colors using an enum we need to use ModTheSpire's enum patching feature. To do this create any class and add to it the following code:
```
	@SpireEnum
	public static AbstractCard.CardColor MY_NEW_COLOR;
```
3. The base game also reprents players using an enum so we must do the same thing for the player enum. Use this code:
```
	@SpireEnum
	public static AbstractPlayer.PlayerClass MY_NEW_PLAYER_CHARACTER;
```
3. To create a new color use `BaseMod.addColor`. This should be called in your mod's `initialize` method. It does not need a special handler to work. The parameters are as follows: `String color` (this should be `MY_NEW_COLOR.toString()`), `Color bgColor` (the background color for the card color), `Color backColor` (the back color for the card color), `Color frameColor` (the frame color for the card color), `Color frameOutineColor` (the frame outline color for the card color), `Color descBoxColor` (the description box color), `Color trailVfxColor` (the trail vfx color), `Color glowColor` (the glow color), `String attackBg` (path to your attack bg texture for the card color, path starts relative to your `SlayTheSpire` folder), `String skillBg` (path to your skill bg texture for the card color, path starts relative to your `SlayTheSpire` folder), `String powerBg` (path to your power bg texture for the card color, path starts relative to your `SlayTheSpire` folder), `String energyOrb` (path to your energy orb texture for the card color, path starts relative to your `SlayTheSpire` folder)
4. To create a new player character make a EditCharacterSubscriber and in the `receiveEditCharacters` method go ahead and call `BaseMod.addCharacter`. The parameters are as follows: `Class characterClass` (the actual java *Class* of your character, e.g. `MyCharacterClass.class`), `String titleString` (title string for the character), `String classString` (class string for the character), `String color`, (the color for this character; should be `My_New_Color.toString()`), `String selectText` (select text for the character), `String selectButton` (path to select button texture starting relative to the `SlayTheSpire` folder), `String portrait` (path to portrait texture starting relative to the `SlayTheSpire` folder), `String characterID` (this should be `MY_NEW_PLAYER_CHARACTER.toString()`)
5. Now just be sure to add some cards for your custom character! When defining cards for your custom character rather than using `AbstractCard.CardColor.WHATEVER` go ahead and use `MY_NEW_COLOR` instead.
6. Note that there are likely to be bugs with this feature since it is complex and new so use at your own risk and also please submit bug reports as issues on this repository.

## Changelog ##
#### v1.0.0 ####
* Initial release

#### v1.0.1 ####
* Scale console by Settings.scale
* Prevent game hotkeys from activating while console is visible

#### v1.1.0 ####
* Add mod badges
* Add initial support for mod settings screens
* Add `relic` console command
* Add option to change console keybind on BaseMod settings screen

#### v1.1.1 #####
* Scale mod badges by Settings.scale
* Scale mod settings screens by Settings.scale

#### v1.1.2 #####
* Fix bug with IDs which contain spaces (FlipskiZ)
* Add `card` console command (FlipskiZ)
* Add `kill all` console command

#### v1.1.3 #####
* Initial support for each mod badge being tied to its own settings panel

#### v1.1.4 #####
* Add `gold` command
* Add `energy` command
* Remove bundled font and use one from the base game instead

#### v1.1.5 ####
* Add `energy inf` command
* Add `PostEnergyRechargeSubscriber` interface and related code

#### v1.2.0 ####
* Add `PostDrawSubscriber` interface and related code
* Add `CustomRelic` extension of AbstractRelic
* Add support for loading custom RelicStrings
* Fix a bug that prevented the character `D` from being input into the console
* Rename `card` command to `hand`
* Add `hand r all` command
* Add `deck` command 
* Add `draw` command
* Add `float BaseMod.pathDensityMultiplier` property which can be used to modify map generation

#### v1.2.1 ####
* Add `PreStartGameSubscriber` interface and related code

#### v1.2.2 ####
* Add `AbstractDungeon` cleanup to diff, nothing changed yet.
* Add `relic list` command
* Fix crash when attempting to `deck add` an invalid card id
* Add upgrade support to `deck add` and `hand add` 
* Add `PreMonsterTurnSubscriber` interface and related code

#### v1.2.3 ####
* Add `PostCampfireSubscriber` interface and related code
* Add proper support for `IntangiblePower` to be applied to players

#### v1.3.0 ####
* Update to support `[EARLY_ACCESS_013]`

#### v1.3.1 ####
* Fix bug that prevented the `deck r` command from working

#### v1.3.1a ####
* Pull in changes from `[EARLY_ACCESS_013]` hotfix 2

#### v1.3.2 ####
* A lot of behind the scenes work on settings pages. 
* MODDERS: You shouldn't try to build complex settings pages right now, its probably going to get significantly cleaned up and reworked to be more simple soon.

#### v1.3.3 ####
* Fix non-id parameters on commands being cases sensitive
* Change backspace behavior on console
* Fix scaling for new settings panel elements introduced in v1.3.2

#### v1.3.4 ####
* Add `StartActSubscriber` interface and related code
* Add `PostDungeonInitializeSubscriber` interface and related code
* Add `StartGameSubscriber` interface and related code
* Fix bug that would prevent `publishPreStartGame()` from occuring in some cases
* Cleanup part 1
* Add IDEA project files
* Remove `_build.bat` since this can be automated in IDEA

#### v1.3.5 ####
* Add support for CustomRelic outline images

#### v1.3.6 ####
* Prevent Girya from stacking above 3 if `receivePostCampfire()` returns false

#### v1.4.0 ####
* Update to support `[EARLY_ACCESS_014]`
* Mostly switch to using SpirePatch for inserting hooks
* Speed up `kill all` command

#### v1.4.1 ####
* Add `kill self` command

#### v1.4.2 ####
* Add support for adding custom localization strings of any type
* Code cleanup

#### v1.4.3 ####
* Add `potion` command
* Add `fight` command
* Speed up `kill self` command

#### v1.5.0 ####
* Update to support `[EARLY_ACCESS_015]`

#### v1.5.1 (daviscook477) ####
* Code cleanup + bugfixes
* Add hooks for modifying the player's initial deck and initial relics
* Add hooks for changing the relics and potions sold by the shop
* Add methods for abstracting the manipulation of RelicLibrary
* Initial support for custom cards

#### v1.5.2 ####
* More code cleanup
* Add hooks for adding custom player characters and the corresponding hooks for adding a new colors for custom cards


## Contributors ##
* t-larson - Original author
* FlipskiZ - `hand` command, bug fixes
* daviscook477 - Custom players, custom colors, custom cards, more API hooks, code cleanup, bugfixes