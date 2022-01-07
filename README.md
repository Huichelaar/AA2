# AA2
Made by Huichelaar.
I heavily referenced [FEBuilderGBA](https://github.com/FEBuilderGBA/FEBuilderGBA/blob/master/FEBuilderGBA/ImageUtilOAM.cs).
I also referenced circleseverywhere's [Animation Assembler](https://www.dropbox.com/sh/xe3bk2tn87zboif/AACTeniihbt-NQWrTpn6F5OSa?dl=0). This is also where I took lzss.py from. lzss.py is used to compress palettes and spritesheets.

## What is this?
This is an environment that serves as an example use of [AA2](Anims/AA2.py). AA2 is a python script that takes a battle animation script (in the usual format) as input. It'll also need the images referred to in the script to be present in the same directory as the script. AA2 produces an Installer.event file as output. This installer file can be included in a buildfile to insert the battle animation. Make sure to keep lzss.py in the same directory as AA2.py, as AA2 imports lzss.

## Implementation details
The palette of the animation is grabbed from one frame's image. Which frame's palette is used might depend on which version of Python you're running. Frames are kept in a dictionary, which seems to be ordered in later versions of Python, but unordered in earlier versions. AA2 grabs the first frame in the dictionary (which would be the first frame in the script if the dictionary is ordered) and uses its palette to construct the installer's palette. Only the first four palettes (or eight if using 2-palette mode) are used. 

## How to use AA2.
If you want to use/test this environment, apply Main.event to an FE8U ROM using [Event Assembler](https://www.dropbox.com/s/4mql123thxb78kw/Event%20Assembler%20V11.1.3.zip?dl=0). This will install a repaletted version of [Spud & Gamma's bow knight](https://feuniverse.us/t/fire-emblem-resource-repository-battle-animations-portraits-music-etc/3326/1084). This repalette uses 27 colours, to show off AA2's two-palette option. The bow animation is set to overwrite battle animation entry 0x19, which is the Fighter's axe animation. This should be the first enemy you fight in the Prologue of FE8U, so you can see the animation in action as soon as possible (although the enemy won't hit the player, maybe due to the bow script not matching the weapon used).

I've added a [batch script](Anims/5. Bow/AA2.bat) to both weapon folders of the bow knight. Simply drag'n'drop the battle animation script (bow.txt for example) onto the AA2.bat file. It expects AA2.py to be two directories back from the current one, but changing this should be straightforward. If everything works correctly this'll run AA2.py and produce a \<scriptname\>Installer.event file. You'll have to replace the AnimTableEntry(\<scriptname\>) slot with the animation slot you wish to overwrite. You can then include this installer file in your buildfile, and re-build your ROM to see the results.

### Options
There's a few options you can add to the battle animation script to change AA2's output. These options can be added to the script in this format:

\@ \<option\> \<value\>

These options need to be put at the start of the script, before "\/\/\/ \- Mode 1". Only EMPTYTILEPER64 actually uses the \<value\> argument. The other options don't take arguments. These are the possible options:

#### 2PALETTES
Frames will be split in two, each using a different palette. This way it's possible to display two palettes at the cost of two free paletteslots, and more sprites (meaning more sheets, therefore more ROM consumption and more frequent decompression. You'll need to add an "\#include "ASM/ASM.event"" to your buildfile, to enable some asm that allows the second palette to be loaded correctly.

#### HALFSIZESHEETS
Sheets will only be half the usual size (256x32 instead of 256x64). This allows for more battle anims to display at once (pair-up here we go?). The split is horizontal (256x32 instead of 128x64) to avoid having to decompress the sheet elsewhere before then moving it to VRAM. This does mean 64x64 and 32x64 sprites won't fit and will be split into smaller sprites.

#### NOSPLITSPRITES
Sprites will not be split into smaller chunks if there's otherwise no room left on the current sheet. Instead a new sheet will be created. More sheets means more ROMcost and more frequent decompression, but fewer sprites. This option is ignored if 1D sprites are turned on, as it's functionally pointless.

#### EMPTYTILEPER64 <value>
\<value\> Indicates how many empty tiles will be allowed in a 64-tiled sprite. This value is used for both 1D and 2D spritesheets. Divide this value by two to get how many empty tiles will be allowed in a 32-tiled sprite, etc. The more empty tiles are allowed in a sprite, the fewer sprites are necessary to represent a frame. This provides the same advantages and disadvantages as NOSPLITSPRITES. If this option isn't in the script, no empty tiles will be allowed in a sprite.

#### 1DSPRITES
Sprite sheets will be set up to work with one-dimensional OBJ Character VRAM Mapping. There's info on 1D and 2D sprite mapping in [tonc 8.2.1](https://www.coranac.com/tonc/text/regobj.htm#sec-tiles). Normally, either OAM can be conserved at the cost of more sheets, (which costs more ROM and leads to more decompression in equal time) or more OAM will be consumed in exchange for fewer sheets. 1D sprites can use the best of both worlds. They conserve OAM and sheets. However, all vanilla battle animation spritesheets are made to be used with 2-dimensional mapping. These (and other battle sprites such as UI-stuff (item icons), battle & death quote portraits, and spells using OBJ) would need re-doing to not get scrambled in 1D sprite mode. In my, admittedly limited, testing earlier frames on 1D sheets still consume the same amount of sprites, whereas later ones consume fewer (makes sense in theory, I guess).

## TODO
- I'll probably compress palettes after all. I'll have to allocate some 320 bytes of RAM, which isn't too much. As a result, less ROM will be consumed and two-palette animations should preview a little better in [FEBuilderGBA](https://github.com/FEBuilderGBA/FEBuilderGBA/).
- It'd be nice if NOSPLITSPRITES and EMPTYTILEPER64 could be activated and deactived at other points in the script. This way a battle anim's standing motions and dodge frames could focus on using fewer sprites to allow the opposing battle animation to use more sprites when they're attacking, whilst the other modes could still allow more sprites.

## Closing
If there's any bugs or other weirdness, feel free to let me know.