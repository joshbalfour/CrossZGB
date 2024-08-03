# CrossZGB

CrossZGB is a multi-platform game engine that lets you write your own games in C or asm for the following platforms:
- Nintendo Game Boy / Nintendo Game Boy Color
- Analogue Pocket .pocket format
- Mega Duck / Coguar Boy
- Sega Master System
- Sega Game Gear

This repo is a fork of the original [ZGB](https://github.com/Zal0/ZGB/) repo, which extends the functionality. It preserves as much backward compatibility as possible with existing projects based on the original ZGB.

It uses [GBDK 2020](https://github.com/gbdk-2020/gbdk-2020/releases/latest) but expands it to give you some common functionallity for writing games such as: a main loop, sprites spawning, sprites life cycle, sprites animations, collison management, easily getting assets into the game, music, fx...

![gif](https://raw.githubusercontent.com/gbdk-2020/CrossZGB/develop/doc%20files/tuto.gif) ![gif](https://github.com/Zal0/bitbitjam2016/blob/develop/bitbit3/res/marketing/screenshots/pretty.gif?raw=true) ![gif](https://github.com/Zal0/gbjam2016/raw/develop/res/marketing/gifs/fly.gif?raw=true)

## Getting started
- Installing CrossZGB
  - Download the latest [release](https://github.com/gbdk-2020/CrossZGB/releases/latest) or the nightly build from [CI](https://github.com/gbdk-2020/CrossZGB/actions/workflows/zgb_build_and_package.yml)
  - Run install.bat (this will create a new environment var ZGB_PATH pointing to %ZGB%/common)
> NOTE: ensure ZGB path doesn't contain any spaces
- Creating a new project
  - Take the CrossZGB-template from the CrossZGB examples subfolder and build it by running `build.bat` or `make` from the `template/src` folder
  - Follow the tutorial on the [wiki](https://github.com/gbdk-2020/CrossZGB/wiki) to understand the basic concepts of the engine
- (**Optional**) Download [***Microsoft Visual Studio Community Edition***](https://www.visualstudio.com/downloads/) or [***Microsoft Visual Studio Code***](https://code.visualstudio.com/download/) and follow [the VSCode setup guide in the template readme](examples/!template/README.md).

## Examples
See the included basic [example projects](/examples) as well as the [Made with CrossZGB](https://github.com/gbdk-2020/CrossZGB/wiki/Made-with-CrossZGB) Wiki page for source code and demonstrations of how to target multiple platforms and use other features of the engine.

## Documentation
Check the [wiki](https://github.com/gbdk-2020/CrossZGB/wiki) and this README

## Support
- discord: [gbdk/zgb discord](https://discord.gg/XCbjCvqnUY)

## Features <a name="features"></a>

(Click on any feature to expand it)

<details>
  <summary><strong>Easy makefile support</strong></summary>

In most cases you just need a small makefile like this:

```
# Project name must not be empty or contain special characters or spaces
PROJECT_NAME = ZGB_TEMPLATE

# Set platforms to build here, space separated. (These are in the separate Makefile.targets)
# They can also be built/cleaned individually: "make gg" and "make gg-clean"
# Possible are: gb gbc pocket megaduck sms gg
TARGETS=gb megaduck gg sms

# Builds all targets sequentially
all: $(TARGETS)

# Set build type to Debug, comment out owr change to Release for the release
BUILD_TYPE = Debug

# Number of banks (must be a power of 2): A (Automatic), 2, 4, 8, 16, 32...
N_BANKS = A

# Default hardware sprites size: SPRITES_8x16(default) or SPRITES_8x8
DEFAULT_SPRITES_SIZE = SPRITES_8x16

include $(subst ',,$(subst \,/,'$(ZGB_PATH)'))/src/MakefileCommon
```
When you make any changes to any of the source files of your project, or any of the assets, only that file will be recompiled. The internal Makefile that comes with CrossZGB creates a list of dependencies and only compiles what is needed saving you a lot of time. It will also help you a lot if you work with a version control system, such as git.

---
</details>

<details>
  <summary><strong>Transparent asset management</strong></summary>
  
CrossZGB will automatically turn all your assets files into C data:
- **Graphics**
  - .gbr from Game Boy Tile Designer
  - .gbm from Game Boy Map Builder
  - .png can be used for backgrounds or sprites
- **Musics**
  - .mod for gbt-player
  - .uge for hUGETracker

In order to use any of these resources in your code you need to declare them first using
```C
IMPORT_MAP(<map_filename_without_extension>)
IMPORT_TILES(<map_filename_without_extension>)
DECLARE_MUSIC(<map_filename_without_extension>)
```

---
</details>

<details>
  <summary><strong>Main Loop</strong></summary>

![gif](/doc%20files/readme/ZGB-loop.png)

---
</details>

<details>
  <summary><strong>States</strong></summary>

All CrossZGB games must contain at least one State. This state must be assigned on ZGBMain.c
```C
UINT8 next_state = StateGame;
```
When CrossZGB starts it will call the **START** function of this State. Then on each frame it will call the **UPDATE** function until **SetState** is called to assign a new State

<details>
  <summary>Creating a new State</summary>

1. Create a new file < YourNewState >.c containing this:
```C
#include "Banks/SetAutoBank.h"

void START(void) {
}

void UPDATE(void) {
}
```
2. Register this new State in ZGBMain.h
```C
#define STATES             \
...
_STATE(<YourNewState>)       \
STATE_DEF_END
````
Now, whenever you want to enter this new state you just need to call **SetState**(< YourNewState >)
</details>

---
</details>

<details>
  <summary><strong>Sprites</strong></summary>

You can manually add Sprites calling **SpriteMangerAdd**(type, x, y). CrossZGB will call the **START** function of this Sprite first and then it will call **UPDATE** on each frame until the Sprite is removed. You can manually remove an Sprite with the function **SpriteManagerRemove** (faster) or **SpriteManagerRemoveSprite** and then the engine will call its **DESTROY** function. 

Sprites will also be removed when getting off screen limits. You can configure how far you allow them to go before the engine disposes them with the fields **lim_x** and **lim_y**

Usually you will create an Sprite in the START function of your State and assing it to scroll_target, so that the camera follows it
```C
void START(void) {
	scroll_target = SpriteManagerAdd(SpritePlayer, 50, 50);
	...
}
```

You can create your sprites with Game Boy Tile Designer or you can use pngs. 
Create your sprites in the res/sprites folder so that the Makefile can identify them as sprites and pass the proper parameters to png2asset.

<details>
  <summary>gbr sprites</summary>
Use Game Boy Tile Designer included in <ZGB_PATH>/env/tools/gbtd22/GBTD.exe to create gbr sprites. Don't worry too much about the TileSize choosen, remember that empty tiles will be discarded

The first time you compile the gbr a .meta will be created with the default params passed to png2asset. You may want to take a look specially at the collision info to adjust the collider
```
-px 0 -py 0 -pw 32 -ph 32
```
Check the png2asset params [here](https://gbdk-2020.github.io/gbdk-2020/docs/api/docs_toolchain_settings.html#png2mtspr-settings)
</details>

<details>
  <summary>png sprites</summary>

GBTD has a few limitations:
- The maximum sprites size is 32x32
- It only lets you choose a palette for the whole metasprite

Luckily you can overcome these limitations by using your preferred pixel art software and then export your data as a png spritesheet

As with gbr sprites a .meta file can be created to pass arguments to png2asset. Unlike gbr sprites this .meta file won't be created automatically so it is important that you create it and at least indicate the sprite dimensions (or the entire spritesheet will be exported as a single sprite).

There should only be one line in .meta files with no line break at the end
```
-sw 32 -sh 16 
```
Check the png2asset params [here](https://gbdk-2020.github.io/gbdk-2020/docs/api/docs_toolchain_settings.html#png2mtspr-settings)
</details>

<details>
  <summary>Creating a new Sprite</summary>

The template already comes with a placeholder Sprite but you surely will need to add more. You do this by following the next 3 steps:
1. Create the sprite image data. 
2. Create a new file < YourNewSprite >.c containing this:
```C
#include "Banks/SetAutoBank.h"

void START(void) {
}

void UPDATE(void) {
}

void DESTROY(void) {
}
```
3. Register this new Sprite in ZGBMain.h
```C
#define SPRITES \
...
_SPRITE_DMG(<YourNewSprite>, <image>)\
SPRITE_DEF_END
```
If you are compiling for some system which does not support hardware sprite flipping like Sega Master System or Sega Game Gear, you should declare which axis sprites may be flipped:
```C
#define SPRITES \
...
_SPRITE_DMG_MIRROR(<YourNewSprite>, <image>, <MirrorFlags>)\
SPRITE_DEF_END
```
Creating mirrored copies of the sprites occupy additional space in VRAM.
</details>

---
</details>

<details>
  <summary><strong>Big maps scroll support</strong></summary>

CrossZGB support maps up to 16384 bytes with a maximum width or height of 255 tiles. The engine will take care of updating the native 32x32 background as the camera moves

![gif](/doc%20files/readme/scroll.gif)

Here is how you create a new map and load it into your game:
- Open Game Boy Tile Designer included in <ZGB_PATH>/env/tools/gbtd22 and create a new tile set
- Open Game Boy Map Builder included in <ZGB_PATH>/env/tools/gbmb18 and create a new map that uses the previous tile set
- In the START function of your state you should manually add the sprite that the camera will follow
- Import your map and Call InitScroll
```C
IMPORT_MAP(map); //This is the name of your map without the extension

void START(void) {
	scroll_target = SpriteManagerAdd(SpritePlayer, 50, 50);
	InitScroll(BANK(map), &map, 0, 0);
}
```
As the scroll updates new rows or columns it will call the function GetTileReplacement located in ZGBMain.c
The default behaviour of this function is to spawn sprites using sprite_tye = 255 - tile_id, but you can customize it for your custom needs

---
</details>

<details>
  <summary><strong>Metasprites</strong></summary>

Metasprites are sprites composed of 8x8 or 8x16 native sprites. 
The tool png2asset from GBDK-2020 is used to create the data that will end up in the final build:
- duplicated tiles will be only added once
- mirrored tiles will also count as duplicated
- empty tiles will be ignored
- palette info will be included

---
</details>

<details>
  <summary><strong>Sprites modes 8x8 and 8x16</strong></summary>

The Game Boy has native support for sprite sizes 8x8 and 8x16. You can use any of them to compose the metasprites in your game.

Choosing 8x8 size will make it easier to duplicate parts of the metasprite and will require less memory to store it but will take longer to render the final metasprite

The default Sprite mode is selected in your makefile
```
# Default hardware sprites size: SPRITES_8x16(default) or SPRITES_8x8
DEFAULT_SPRITES_SIZE = SPRITES_8x16
```

---
</details>


<details>
  <summary><strong>Sprite animations</strong></summary>

Animations in CrossZGB are defined by arrays of frames where the first element is the number of frames
```C
const UINT8 anim_walk[] = VECTOR(0, 1, 2, 1, 0);
```

Setting the current animation is done with **SetSpriteAnim**(sprite, animation, speed)

Instead of setting an animation you can Set the current frame manually by calling **SetFrame**(Sprite, frame_idx)

The Sprite field **anim_frame** contains the animation index if there is an animation running, or the frame index otherwise

---
</details>

<details>
  <summary><strong>Collisions</strong></summary>

All sprites have a rectangle collider that will be used to check collisions. By default it will be defined by the metasprites dimensions but you can adjust it on the sprite .meta file
```
-px 2 -py 0 -pw 12 -ph 19
```
![gif](/doc%20files/readme/collider.png)  ![gif](/doc%20files/readme/mirrors.gif)

This rectangle will remain constant when the sprite is flipped

<details>
  <summary><strong>Sprite vs Background</strong></summary>

First you need to declare an array (terminated in 0) indicating which tiles are considered collidables
```C
UINT8 collision_tiles[] = {1, 2, 3, 4, 8, 10, 0}; //In this case tiles 1, 2, 3, 4, 8 an 10 will be considered collidables
```

Then you need to pass this array when you Init the scroll (you can have several arrays depending on the tileset you use)
```C
InitScroll(BANK(map), &map, collision_tiles, 0);
```

And now, instead of directly modify the X and Y coordinates of your Sprite, you need to call TranslateSprite
```C
TranslateSprite(THIS, -1, 0); //Move the current sprite 1 pixel to the left checking collisions with the background
```
If the Sprite collides then it won't advance and TranslateSprite will return the collision tile (so you can check if there are spikes or other stuff)

You can also declare an array of collision tiles that will be only checked when the Sprite is moving downwards. This is very useful for platform games where the character can jump into a platform from below

![gif](/doc%20files/readme/coll_down.gif)

</details>

<details>
  <summary><strong>Sprite vs Sprite</strong></summary>

To check if two sprites are colliding call the function CheckCollision in "Sprite.h"
```C
if(CheckCollision(THIS, other_sprite))
{
    //Sprites are colliding!
}
```

</details>

---
</details>

<details>
  <summary><strong>Auto Banking</strong></summary>

CrossZGB uses [bankpack](https://bbbbbr.github.io/gbdk-2020/docs/api/docs_toolchain.html#autotoc_md79) so you don't need to worry about where to place your code or resources. Just make sure that:
- **_#include "Banks/SetAutoBank.h"_** is added at the beggining of your States and Sprites
- If you need to call an sprite function from another sprite, declare it **BANKED**
```C
void HitMe(void);        //WRONG!!
void HitMe(void) BANKED; //RIGHT!
```

- Check the png created in the Debug/Release folder of your build to get an overview of your banks usage. For a more detailed information you can use [RomUsage](https://github.com/bbbbbr/romusage)

---
</details>

<details>
  <summary><strong>Fonts</strong></summary>

Fonts in CrossZGB are gbr files of **45** tiles, with uppercase characters **_A-Z 0-9 !'()-.:?_** The ZGB-Template already comes with a default font that you can customize

In order to print some text in your game
1. Import the font using 
```C
#include "Print.h"
IMPORT_TILES(<font filename>);
```
2. Init the font in the START function of your State by calling
```C
INIT_FONT(font, PRINT_BKG); //PRINT_BKG to draw on the background or PRINT_WIN to draw on the Window
```
3. Print some text using 
```C
PRINT(0, 0, "Hello World"); //print Hello World on 0, 0
```

You can also use **Printf** to draw some vars with %d %i &u and %s

You can change the target (background or window) with the var **print_target**

---
</details>

<details>
  <summary><strong>Music</strong></summary>

The Music driver is automatically detected, depending on the assets you actually use. `*.MOD` files enable the [gbt-player](https://github.com/AntonioND/gbt-player) driver, `*.UGE` files enable the [hUGETracker](https://github.com/SuperDisk/hUGETracker) driver, `*.FUR` files enable the [banjo](https://github.com/joffb/banjo) driver for the PSG sound chip.

You can force which music driver is selected in the Makefile, but usually that isn't needed. You cannot mix multiple music drivers in one project.
```
# Music player: HUGETRACKER(default) or GBT_PLAYER
MUSIC_PLAYER = GBT_PLAYER
```

To play some music in your game
- Place the .mod or .uge files in the res/music/<ext>/ folder
- Import the music with
```C
DECLARE_MUSIC(<music_filename>)
```
- Play it with
```C
PlayMusic(<music_filename>, LOOP)
```
- Pause it with
```C
PauseMusic;
```
- And Stop it with
```C
StopMusic;
```

---
</details>

<details>
  <summary><strong>Sound Effects</strong></summary>

To play an FX Sound you just need to call
```C
void PlayFx(SOUND_CHANNEL channel, UINT8 mute_frames, ...); // Add register data from GBSound
```

The channel will be occupied during mute_frames and the music player won't be able to use it

---
</details>

<details>
  <summary><strong>Game Boy Color</strong></summary>

Because CrossZGB uses png2asset, palette data will be always included for each sprite allowing CrossZGB to load the palette automatically when the Sprite is loaded

Just make sure that:
- The total Sprites loaded don't need more than 8 different palettes (if two sprites have the same palettes, they will share them)
- The palette colors are ordered from lighter to darker so that the game will also look good on the original GB


---
</details>

<details>
  <summary><strong>Super Game Boy Borders</strong></summary>

Follow the next steps to create Super Game Boy borders for your game
- create the folder res/borders
- Add a png with these limitations:
   - size must be 256x224
   - there must be a 160x144 transparent rectangle in the center of it
   - maximum number of different tiles is 256
   - each 8x8 tile has a limit of 16 colors
   - there can only be 4 different palettes of 16 colors
   - here is a [template](https://raw.githubusercontent.com/gbdk-2020/gbdk-2020/develop/gbdk-lib/examples/gb/sgb_border/gb_border.png) you can use
- In your code (do this before loading any other bg map)
```C
#include "SGB.h"

IMPORT_MAP(<border_filename>);

void START() {
  LOAD_SGB_BORDER(<border_filename>);
  ...
}
```

---
</details>

<details>
  <summary><strong>Savegame support</strong></summary>

In order to have savegames in your game you must include a couple of files named savegame.h and savegame.c with the following content

```C
//savegame.h
#ifndef SAVEGAME_H
#define SAVEGAME_H

#include <gb/gb.h>
#include "SRAM.h"

typedef struct {
	SAVEGAME_HEADER;

  //Whatever content you want to store in external ram

} Savegame;

extern Savegame savegame;

#endif
```

```C
//savegame.c
#include "savegame.h"
Savegame savegame;
```

Having a file named savegame.c in your project will automatically compile it using MBC5+RAM+BATTERY (otherwise it will use MBC5)

Then before accesing any content you must enable/disable sram access

```C
ENABLE_RAM;
  //Acess savegame content to read/write
DISABLE_RAM;
```

---
</details>


<details>
  <summary><strong>VS Code debugging using Emulicious</strong></summary>

The [ZGB-template](https://github.com/Zal0/ZGB-template) is properly configured for C Debugging under [Emulicious](https://emulicious.net/) in [Visual Studio Code](https://code.visualstudio.com/). If you started your current project with an old version of the template you just need to copy the .vscode folder into the root of your project.

- [Download](https://code.visualstudio.com/Download) Visual Studio Code
- Open the workspace located in the .vscode folder within ZGB-template
- Install the workspace recommended extensions:
  - [C/C++ for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools)
  - [Emulicious Debugger](https://marketplace.visualstudio.com/items?itemName=emulicious.emulicious-debugger)

Now ensure that you have either Debug or DebugColor selected as your current configuration and press F5 to start debugging. Emulicious will be launched automatically and breakpoints will be hit. Enjoy!

---
</details>

## License

Released under the [MIT license](https://opensource.org/licenses/MIT)
