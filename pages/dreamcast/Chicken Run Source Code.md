---
layout: post
tags: 
- dreamcast
- dc
- ps1
- pc
- games
- sourcecode
title: Chicken Run Source Code
thumbnail: /public/consoles/Sega Dreamcast.png
image: http://img.youtube.com/vi/thRXO3YwOCg/0.jpg
permalink: /Chicken-Run-Source-Code
breadcrumbs:
  - name: Home
    url: /
  - name: Sega Dreamcast
    url: /dc
  - name: Chicken Run Source Code
    url: #
videocarousel:
  - title: Blitz Game Studios
    image: http://img.youtube.com/vi/XVSNxQbQFJw/0.jpg
    youtube: 'XVSNxQbQFJw'
  - title: Chicken Run
    image: http://img.youtube.com/vi/thRXO3YwOCg/0.jpg
    youtube: 'thRXO3YwOCg'
recommend: dreamcast
references:
- Dreamcast Hub
editlink: dreamcast/Chicken Run Source Code.md
---

Developed by Blitz Games (formerly known as Interactive Studios Limited) in 2000 as a cross platform action adventure game based on the movie with the same name.

Throughout the engine you will find referenced to the acronym isl (Interactive Studios Limited) e.g islfile, islfont etc.

# Act (Level) Data
On Dreamcast each act has its own folder each with its own .BFF and .SPT and some containing region specific .SBB/.SBH files.
On PC each act also has its own folder plus the addition of .WAB files accompaning the .BFF and .SPT files.
On PS1 the layout is completely different, presumably the data is compressed in CRTEST.DAT. it also contains a file called DUMMY which just contains padding 00 bytes.

## BFF files (Big file format?)
There is a very helpful comment at the top of the `bff_load.c` file which explains what BFF files are:
```
Quickie guide to BFF files:-

 A BFF is, simply, a collection of data segments, prefixed by a TYPE, a SIZE, and an ID (the CRC of a name)
Data of many different types can just be shoved together into one file.
After loading, the program scans through the list, checks the type of each segment, and calls a "link" routine
for that type, which basically fills in any required pointers.

 Then there's a little routine saying "find a bff segment of "such&such type, with this name-CRC"
 ```
 In the header file it also mentions:
 ```
 // All the types of object that you stick into a BFF need to start with a standard header
// which allows the BFF routines to seek out specific objects on request.

// "id" is long something like "PSA0" or "COL1" for PSA and collision data,
// for identifying the TYPE of the block it is.

// "len" is the length of the block, including the header.

// "crc" is the CRC of the name of this specific entry, to distinguish it from any
// other blocks of the same type.

typedef struct BFF_HeaderTag
{
	unsigned long id;
	unsigned long len;
	unsigned long crc;
}BFF_Header;
```

## SPT Files
Used for File streaming, seems to be used for both texture and sound streaming.


## CRTEST.DAT (Playstation 1)
This is where the Chicken Run game data is stored on the PS1 version instead of the folder based layout of the PC and Dreamcast.
It has a magic number of FLA2, not sure why, did they ever have a FLA1, what does FLA even stand for?
```
static char FILEIO_CDINDEX[64] = "\\CRTEST.DAT;1"; //	"\\MYGAME.DAT;1"
#define FLA_MAGIC			0x32414c46			// Magic number 'FLA2' INTEL
```
---

# Games Referenced in the Source code
Developers tend to reuse the same engine or parts of the same engine for multiple games across a wide range of platforms (one of the beenfits of writing in cross-patform C), which means there are other games that will share many of the same source files used in chicken run. 
Here is a list of games that have been referenced in the source code:
* Glover
* Action Man 2
* Frogger 2

---

# Dreamcast Engine

Most of the dreamcast specific code has a DCK_ prefix, presumable short for Dreamcast Katana (Katana was the code name for the dreamcast).
Some example:
* DCK_System
* DCK_Texture

Presumably there were versions of this code with N64 and PS1 prefix used at ISL but they weren't released with this source code unfortunetly. As most of the game code is cross platform but these files contain the very platform specific code for the Dreamcast.

---

# Compiling for Playstation 1 with PSYQ
So although it was originally compiled for PS1, all the sourcecode has been modified for the dreamcast version so it will need some modifications in order to compile for the playstation.

The following files won't compile to PS1 without modification:
* types.c

However the root/cr/source folder contails a makefile.mak which is for psyq and build the PS1 version.

## PSY-Q Linker Files
The PS1 linker files are available for both the release and the demo version: `crtest.lnk` and `crdemo.lnk` respectivly. 
You can pass a .lnk file as a @ parameter to the C compiler (CCPSX.EXE) like so:
```c
$(PSYLINK) /l $(LIBISL) /psx /wo /v /c /strip /nostriplib @crdemo.lnk,crdemo.cpe,crdemo.sym,crdemo.map
```

The use of the @ operator is explained in the PSX developer documentation:
```
Long command lines can be stored in control files.  By using an
'@' sign in front of the control file name the contents of the
control file can be embedded in the command line.

The contents of the control file can be split across several lines
without the need to use any special characters.  An end of line in
a control file is treated as a space.

You can specify as many control files as you want on the command
line and a control file can even reference another control file.
```

### ORG directive
The ORG directive is normally used is assemblers/linkers to choose a specific location for certain pieces of code. For the PS1 the executable is normally started at location 0x00018000 but for demos it seems to be 0x80018000, not sure why.
```asm
;        org     $00018000
; 18000, this is a demo disk...
        org     $80018000
```

### Group directive
You can also specify a section group such as text or bss here. In this executable we only have text for code and bss for small static variable init. Also it looks like they used to have a grouo called dcache but it was commented out.
```asm
text    group

; I've removed the dcache area so that it's definitely all available
; for the model plotter. So just be careful to use it for leaf-routine
; optimisations only, rather than the placement of moderately commonly used
; globals (as was the case in the shell)  - Fred

;dcache  group   obj(0x1f800000),size(0x400)

bss     group   bss	
```

The rest of the contents of crdemo.lnk is:
```c
		regs    pc=__SN_ENTRY_POINT							   

;overlay1                group   file("overlay1.bin")                            ; Test 1
;overlay2                group   over(overlay1),file("overlay2.bin")     ; Test 2
;vid_ovl                 group   over(overlay1),file("vid_ovl.bin")    ; video play

;vid_ovl                 group   file("vid_ovl.bin")      ; video
;sub1_ovl                group   over(vid_ovl),file("sub1_ovl.bin")     ; subgames
;sub2_ovl                group   over(vid_ovl),file("sub2_ovl.bin")     ; subgames
;sub4_ovl                group   over(vid_ovl),file("sub4_ovl.bin")     ; subgames
;deb_ovl                 group   over(vid_ovl),file("deb_ovl.bin")      ; debug menu

; exp_ovl			group	over(overlay1),file("exp_ovl.bin")	; explore game

;        section.4096 align4k.text,text
;        section align4k.*,text
;        section align4k.bss,bss
;        section align4k.sbss,bss


	section .*,text
	section	.sdata,text
	section	.sbss,bss
	section	.bss,bss

;	section vid_ovl.*, vid_ovl
;	section sub1_ovl.*, sub1_ovl
;	section sub2_ovl.*, sub2_ovl
;	section sub4_ovl.*, sub4_ovl
;	section deb_ovl.*, deb_ovl


;include "fixed.obj"
;include "powerbar.obj"
;include "fx.obj"
;include "map_asm.obj"
;"Particle" was being initialised, but never used for anything
;include "particle.obj"

        include "demostub.obj"

        include "dualshock.obj"
        include "pad.obj"
;        include "options.obj"
;       include "gallery.obj"
;        include "cntrscn.obj"

        include "fmademo.obj"
        include "inv_demo.obj"
        include "startup.obj"
        include "credits.obj"
        include "timer.obj"
;        include "main.obj"
        include "maindemo.obj"
        include "actor.obj"

;        include "lev_flow.obj"
        include "map_view.obj"
        include "map_play.obj"

        include "bff_load.obj"
        include "map_draw.obj"
        include "snapshot.obj"
        include "debris.obj"
        include "lights.obj"
        include "puzzles.obj"
        include "collide.obj"
        include "maths.obj"
        include "camera.obj"

        include "enemies.obj"

        include "platform.obj"
        include "platcoll.obj"


        include "overlays.obj"
        include "scenics.obj"
        include "sound.obj"

        include "cr_asm.obj"
        include "sptstream.obj"
;        include "card.obj"

        include "loadsnd.obj"
        include "help.obj"
        include "deth.obj"
        include "demo.obj"

        include "nme_dogs.obj"	;,exp_ovl
        include "nme_mrst.obj"	;,exp_ovl
        include "route.obj"     ;,exp_ovl
        include "sprouts.obj"	;,exp_ovl
        include "charactr.obj"	;,exp_ovl
;        include "curtains.obj"	;,exp_ovl

        include "language.obj"
        include "map.obj"
        include "pause.obj"


;include "stilts.obj"
;include "station.obj"
;include "fxtest.obj"

        include "subgame.obj"

;        include "menu.obj",deb_ovl
;        include "viewer.obj",deb_ovl
;        include "cus_dyn.obj",deb_ovl
;        include "cus_full.obj",deb_ovl


;        include "dogchase.obj",sub1_ovl
        include "fireworks.obj"
;        include "catapult.obj",sub2_ovl
;        include "seesaw.obj",sub2_ovl
;        include "engine.obj",sub4_ovl
;        include "construct.obj",sub4_ovl
;        include "wings.obj",sub4_ovl
;        include "egglaying.obj",sub4_ovl
;        include "crate.obj",sub4_ovl

;include "overlay1.obj", overlay1
;include "overlay2.obj", overlay2




; Things that have been adapted from the ISL libs, rather than
; actually using them

        include "custom.obj"
        include "custom2.obj"
        include "cus_anim.obj"

;        include "islvideo.obj",vid_ovl
        include "islutil.obj"
        include "incbin.obj"
        include "islxa.obj"
        include "islfile.obj"
		include "islfont.obj"


; The ISL libraries
; "Debug" or "Release" versions are switched betwen in the makefile
; by cunning use of environment variables.

        inclib  "islmem.lib"
        inclib  "isltex.lib"
        inclib  "isllocal.lib"
        inclib  "islpad.lib"
        inclib  "islsfx2.lib"
        inclib  "islcard.lib"
        inclib  "islcard.lib"

; -- REMOVED ISL LIBS --
; -- video has the multi-language stuff --
; -- psi is now in custom.c --
; -- util has had it's un-flat-packer rewritten --
; -- sound is now islsfx2 --
; -- xa has a fix for "current sector" establishment
; -- file has an assert check for XA playing
; -- islfont now copes with "OE" and "oe" for French use

;        inclib  "islvideo.lib"
;        inclib  "islutil.lib"
;        inclib  "islpsi.lib"
;        inclib  "islsound.lib"
;        inclib  "islxa.lib"
;        inclib  "islfile.lib"
;        inclib  "islfont.lib"

; -- Sony Libs --

       ;inclib  "libsn.lib"
       ;inclib "c:\psx\crtest\source\none2.lib"

; Fred's cunning plan:
; PSX\ISL\LIB\DEBUG\NONE2.LIB is actually LIBSN.LIB
; PSX\ISL\LIB\RELEASE\NONE2.LIB is the real thing
	   inclib none2.lib

	inclib  "libapi.lib"
	inclib  "libpad.lib"
	inclib	"libmcrd.lib"
	inclib	"libcard.lib"
	inclib  "libgpu.lib"
	inclib  "libgs.lib"
	inclib  "libgte.lib"
	inclib  "libcd.lib"
	inclib  "libetc.lib"
	inclib  "libspu.lib"
	inclib  "libc2.lib"
        inclib  "libds.lib"
;        inclib  "libpress.lib",vid_ovl
```

---

# Missing Files
As with most leaked source code there are some missing files these are:
* shinobi.h (Dreamcast SDK)
* sg_syhw.h (Dreamcast SDK)
* km*.h (Dreamcast SDK)

---

# C Game Source Code
Some of the source code for the game Chicken Run has been available to the internet as part of a "Dreamcast source code" bundle. After extracting the archive we are presented with two folders, "simple model shell" and "cr". 

Presumably cr stands for chicken run and the simple model shell is a tool to display some of the 3D models as they would appear in the game engine. 

## Simple Model Shell
This tool would have been very useful for artists when exporting their models from their modelling software such as 3ds max or maya, as it will show what optimizations need to be made to the model.

Filename | Description | Category
--- | --- | ---
DCK_Maths.c | 
DCK_Maths.h | 
DCK_System.c | 
DCK_System.h | 
DCK_Texture.c | Texture handler rountines
DCK_Texture.h | 
DCK_Types.h | 
actor2.c | 
actor2.h | 
bff_load.c | 
bff_load.h | 
crc32.c | 
crc32.h | 
fixed.c | 
fixed.h | 
gte.c | Gte playstation emulaution routines and structures
include.h | 
islfile.c  | 
islfont.c | font support - AM2 PS   (c) 1999-2001 ISL (AM2 = Action Man 2 Destruction X for PS1)
isltex.c | Texture and VRAM management
islutil.c | ISL PSX LIBRARY	(c) 1999 Interactive Studios Ltd.
islutil.h | 
layout.c | 
layout.h | 
main.c | Main core routine and system initialisation - This file is part of Frogger II, Copyright 2000 Interactive Studios Ltd
main.h | 
makefile | 
maths.c | This file is part of Frogger2, (c) 1999 Interactive Studios Ltd.
maths.h | 
newpsx.c | 
newpsx.h | 
psi.c | Playstation Model (i) Handler - PSX CORE (c) 1999 ISL
psi.h | 
psiactor.c | 
psiactor.h | Skinned model control routines
psitypes.h | ISL PSX LIBRARY	(c) 1999 Interactive Studios Ltd.
quatern.c | 
quatern.h | PSX CORE (c) 1999 ISL
sbinit.c | Copyright (C) 2000, Sega of America Dreamcast
shell.c | 
shell.h | 
sonylibs.h | 
sprite.c | This file is part of Frogger2, (c) 1999 Interactive Studios Ltd.
sprite.h | 
types.c | 
types.h | 
ultra64.h | n64 header (Copyright (C) 1994, Silicon Graphics, Inc.)


# CR (Chicken run source)

We actually have multiple versions of the chicken run source code:
* the main cr folder
* source folder under cr (ps1 makefile so guessing ps1)
* "Copy of Source" folder under cr (?)

This was very common before good version control software such as git came along, so developers would work in seperate folders and merge changes, or keep different console ports in different folders.

The main files in cr are:

Filename | Category | Description
--- | --- | --- 
DCK_Maths.c | 
DCK_Maths.h | 
DCK_System.c | 
DCK_System.h | 
DCK_Texture.c | 
DCK_Texture.h | 
DCK_Types.h | 
DC_TIM.c | 
DC_TIM.h | 
DISPSTR.C | 
DISPSTR.H | 
PSX_PC.h | 
USR.H | 
Usr.c | 
actor.c | 
actor.h | 
actor2.c | 
actor2.h | 
adxtest.c | 
adxtest.h | 
anim.c | 
anim.h | 
backup.c | 
backup.h | 
bff_load.c | 
bff_load.h | 
bpacsetup.c | 
bpacsetup.h | 
bpamsetup.c | 
bpamsetup.h | 
camera.c | 
camera.h | 
card.c | 
card.h | 
catapult.c | 
catapult.h | 
charactr.c | 
charactr.h | 
chase.c | 
chase.h | 
chicken.scr | 
chickenrun.cpj | 
cntrscn.c | 
cntrscreen.c | 
collide.c | 
collide.h | 
construct.c | 
construct.h | 
cr | 
cr.dsp | 
cr.dsw | 
cr.ncb | 
cr.opt | 
crate.c | Animation | Chase sequence featuring Mrs Tweedy hanging from the airplane
crate.h | 
crc32.c | 
crc32.h | 
credits.c | 
credits.h | 
curtains.c | 
curtains.h | 
cus_anim.c | 
cus_dyn.c | 
cus_full.c | 
custom.c | 
custom.h | 
custom2.c | 
dc_timer.c | 
dc_timer.h | 
dctext.xls | 
debris.c | 
debris.h | 
demo.c | 
demo.h | 
deth.c | 
deth.h | 
dogchase.c | 
dogchase.h | 
dualshock.c | 
dualshock.h | 
ectsmenu.c | 
ectsmenu.h | 
egglaying.c | 
egglaying.h | 
empty.c | 
empty.h | 
enemies.c | 
enemies.h | 
engine.c | 
engine.h | 
error.h | 
fireworks.c | 
fireworks.h | 
fixed.c | 
fixed.h | 
fma.c | 
fma.h | 
fx.c | 
fx.h | 
fxtest.c | 
fxtest.h | 
gallery.c | 
gallery.h | 
gametext.h | 
gametext.txt | 
global.h | 
gte.c | 
help.c | 
help.h | 
incbn | 
include.h | 
includeps | 
inventory.c | 
inventory.h | 
islcard.c | 
islfile.c | 
islfont.c | 
isllocal.c | 
islpad.c | 
islsfx2.c | 
isltex.c | 
islutil.c | 
islvideo.c | 
islxa.c | 
language.c | 
language.h | 
layout.c | 
layout.h | 
lcdicons.h | 
lev_flow.c | 
lev_flow.h | 
lights.c | 
lights.h | 
loading.h | 
loadlnd.h | 
loadsnd.c | 
loadsnd.h | 
main.c | 
main.h | 
makefile | 
map.c | 
map.h | 
map_draw.c | 
map_draw.h | 
map_play.c | 
map_play.h | 
map_view.c | 
map_view.h | 
maths.c | 
maths.h | 
menu.c | 
menu.h | 
mssccprj.scc | 
newpsx.c | 
newpsx.h | 
nme_dogs.c | 
nme_mrst.c | 
ok.h | 
options.c | 
options.h | 
overlay.h | 
overlay1.c | 
overlay2.c | 
overlays.c | 
overlays.h | 
pad.c | 
pad.h | 
particle.c | 
particle.h | 
pause.c | 
pause.h | 
platcoll.c | 
platform.c | 
platform.h | 
powerbar.c | 
powerbar.h | 
psi.c | 
psi.h | 
psiactor.c | 
psiactor.h | 
psitypes.h | 
puzzles.c | 
puzzles.h | 
quatern.c | 
quatern.h | 
ranges.h | 
route.c | 
route.h | 
saveicon.h | 
saving.h | 
sbinit.c | 
scenics.c | 
scenics.h | 
seesaw.c | 
seesaw.h | 
sfxtest.c | 
sfxtest.h | 
shell.c | 
shell.h | 
snapshot.c | 
snapshot.h | 
sonylibs.h | 
sound.c | 
sound.h | 
source | 
sprite.c | 
sprite.h | 
sprouts.c | 
sprouts.h | 
sptstream.c | 
sptstream.h | 
startup.c | 
startup.h | 
station.c | 
station.h | 
stilts.c | 
stilts.h | 
subgame.c | 
subgame.h | 
test.c | 
text_ids.h | 
timer.c | 
timer.h | 
types.c | 
types.h | 
ultra64.h | 
viewer.c | 
viewer.h | 
vmucheck.c |
vmucheck.h | 
vssver.scc | 
wings.c | 
wings.h | 

# Data Structures

Name | Category | Description
-- | -- | --
A2TS_STATEREF | | 
A4TS_STATEREF | | 
ACTORLIST | | 
ACTOR_SHADOW | | 
AM2_STREETSTR | | 
ANIMATION | | 
ANIM_ANIMATION | | 
ANIM_DATA | | 
ANIM_SEGMENT | | 
BACKUPINFO | | 
BFF_HeaderTag | | 
BMP_HeaderInfoType | | 
BMP_HeaderType | | 
BMP_RGBType | | 
BOUNDINGBOX | | 
BOX | | 
BYTEVECTOR | | 
CARDSTR | | 
CARD_HEADERSTR | | 
CARD_SAVE_STR | | 
CASEDATA | | 
CASERECORD | | 
CHARACTERINFO | | 
CHARACTERVARSTR | | 
COLLIDEBOUNDSTR | | 
COLLSPHERE | | 
COLL_PLAT_CACHE | | 
COLOUR | | 
CONSTRUCT_DATA | | 
CONTROLMAP | | 
CON_CAMERA | | 
CON_CHICKEN | | 
CON_MRTWEEDY_CAM | | 
CON_RYTHM_DISPLAY | | 
CricketScore | | 
CurrentData | | 
D2M_TMD_P_FG3I | | 
D2M_TMD_P_FG4I | | 
D2M_TMD_P_FT3I | | 
D2M_TMD_P_FT4I | | 
D2M_TMD_P_GT3I | | 
D2M_TMD_P_GT4I | | 
D2M_TMD_P_SP4I | | 
DCKBYTEVECTOR | | 
DCKFLOAT2DVECTOR | | 
DCKSBYTEVECTOR | | 
DCKSHORT2DVECTOR | | 
DCKSHORTVECTOR | | 
DCKVECTOR | | 
DCTIMER | | 
DEBRISSTR | | 
DIRECT2MESH | | 
DISPLAYABLETEXT | | 
DUELVECTOR | | 
EGGLAYING_CTRL | | 
EL_CHICKEN | | 
EL_EGG | | 
EL_PANEL | | 
ENEMYBFFLIST | | 
ENEMYDEF | | 
ENEMYGLOBALS | | 
ENGINECTRL | | 
FIREWORK_DATA | | 
FIREWORK_SPARKS | | 
FMA4_GT3 | | 
FMA4_GT4 | | 
FMACTRL | | 
FMADATA | | 
FMAOBJECT | | 
FMAPLAYEDSTATUS | | 
FMASTATUS | | 
FMA_G3 | | 
FMA_G4 | | 
FMA_GT3 | | 
FMA_GT4 | | 
FMA_MESH_HEADER | | 
FMA_WORLD | | 
FMA_WORLD2 | | 
FOG | | 
FRAMELIST | | 
FVECTOR | | 
FX_PARTICLE | | 
FX_PARTICLE_SYS | | 
GETMATSTR | | 
GamePlayDataType | | 
GameSaveDataType | | 
IQUATERNION | | 
KEYFRAME | | 
KEYFRAMESHORT | | 
LCOORD | | 
LEVINFO | | 
LIGHT_MOBILESTR | | 
LIGHT_MOVEPOINT | | 
LVERT | | 
MAPDRAW_CLIPCHECKSTR | | 
MATRIXI | | 
MENUDATAFMA | | 
MOVEDATA | | 
MRT_TORCH_STR | | 
NAMEVALUE | | 
OBJECTSPRITE | | 
OPTIONDATASTR | | 
PATROLDOGSTR | | 
PLANE | | 
PLATFORM_BIG_DUMMY_DEFSTR | | 
PLATFORM_CONVEY_DEFSTR | | 
PLATFORM_CONVEY_WORKSTR | | 
PLATFORM_POINTTAG | | 
PLATFORM_POINTX_TAG | | 
PLATFORM_PUSHABLE_DEFSTR | | 
PLATFORM_PUSHABLE_WORKSTR | | 
PLATFORM_ROTATING_DEFSTR | | 
PLATFORM_SAVESTR | | 
POINT2D | | 
POSMARK | | 
PSITEXTURES | | 
PUZZLEVARSSTR | | 
QUATERNION | | 
QUATERNIONSHORT | | 
RAILDATA | | 
RAILRECORD | | 
RGBCD | | 
ROUTEBOX | | 
ROUTEBOX_VECTOR | | 
ROUTEFIND_STR | | 
ROUTE_BOXISECT_STR | | 
SCENICSTR | | 
SCENIC_BFF | | 
SFX | | 
SHORT2DVECTOR | | 
SHORTQUAT | | 
SHORTXY | | 
SKEYFRAME | | 
SKINVTX | | 
SLERPSTR | | 
SOUNDTRIGGERLIST | | 
SOUND_BFF | | 
SOUND_TRIGGER | | 
SPECIALLEVEL_SWITCHSTR | | 
SPOT_SFX | | 
SPRITELIST | | 
SPRITE_ANIMATION | | 
SPRITE_ANIMATION_TEMPLATE | | 
SPROUTMANAGER | | 
SPROUTSTR | | 
TAGACTION | | 
TAGACTION_FIVESHLONGS | | 
TAGACTION_FOURSHORTS | | 
TAGACTION_GOTO_LEVEL | | 
TAGACTION_LEAD_ACTION | | 
TAGACTION_OBJECT_USABLE | | 
TAGACTION_PLAY_SAMPLE | | 
TAGACTION_SETOBJECTPARAMETER | | 
TAGACTION_SETVAR | | 
TAGACTION_SET_SCENIC_ANIM | | 
TAGACTION_TASK_COMPLETED | | 
TAGACTION_VECTOR | | 
TAGACTOR2 | | 
TAGCONDITION | | 
TAGCONDITION_BUTTON_PRESSED | | 
TAGCONDITION_IN_BOX | | 
TAGCONDITION_NEAR_POINT | | 
TAGCONDITION_OBJECTPARAMETER | | 
TAGCONDITION_TASK_COMPLETED | | 
TAGCONDITION_VARIABLE | | 
TAGLEVEL_VISUAL_DATA | | 
TAGPLANE2 | | 
TAGPOLY | | 
TAGPUZZLE | | 
TAGPUZZLIST | | 
TAGRGBCD | | 
TAGSPRITE | | 
TAGWORLD_VISUAL_DATA | | 
TAG_MALLOC_LIST_TYPE | | 
TAG_MALLOC_TYPE | | 
TEXANIM | | 
TEXANIMFRAME | | 
TEXANIMHEADER | | 
TEXANIMLIST | | 
TEXINSTANCE | | 
TEXTCTRL | | 
TEXTURE_ANIMATION | | 
TMD_P_FG3I | | 
TMD_P_FG4I | | 
TMD_P_FT3I | | 
TMD_P_FT4I | | 
TMD_P_GT3I | | 
TMD_P_GT4I | | 
TPOINT | | 
TVECTOR | | 
UBYTEVECTOR | | 
VECTOR2D | | 
VKEYFRAME | | 
WINGCTRL | | 
WM_BARS | | 
WM_CHICKEN | | 
WM_PROGRESS | | 
WORLD_MESH_INSTANCE | | 
XA_INDEX_STR | | 
XYSHORT | | 
_ACTOR | | 
_CAMVARS | | 
_DCKMATRIX | | 
_DCKOBJECT | | 
_DCKPOLYGON | | 
_DCKQUATERNION | | 
_DCKVERTEX | | 
_DEBRIS_FLAG_SUB_TYPE | | 
_DEBRIS_FLAG_TYPE | | 
_DSHOCK_EVENT | | 
_EXLIGHT_DATA_TYPE | | 
_EXLIGHT_RGB | | 
_EXLIGHT_TYPE | | 
_FileIODataType | | 
_LanguageDataType | | 
_PadDataType2 | | 
_PadPacket | | 
_Point2DType | | 
_Point3DType | | 
_RAILDATA | | 
_TIM_HEADER | | 
_TIM_PIXELDATA | | 
_VMUCHECK | | 
_XADataType | | 
_displayPageType | | 
animation | | 
catDeviceStr | | 
catDroneStr | | 
catFloaterStr | | 
catStr | | 
controlType | | 
countDownTimerStr | | 
deviceStr | | 
dogType | | 
dogchaseChickenType | | 
doorType | | 
dustType | | 
fireDrone | | 
fireFlap | | 
gametimerType | | 
inventoryType | | 
light | | 
link | | 
mattressStr | | 
optionsScreenType | | 
seesawDroneStr | | 
seesawStr | | 
soundType | | 
spotliteDrawScratch | | 
stiltsStr | | 
tagBMPHeaderType | | 
tagCAMINFO | | 
tagCHARACTER | | 
tagCHARACTERLIST | | 
tagCOLLBOX | | 
tagCOLLDATA | | 
tagCOLL_BFF | | 
tagDEMOFRAME | | 
tagENEMY | | 
tagENEMYINFO | | 
tagENEMYLIST | | 
tagENEMYPOINT | | 
tagFMAINFO | | 
tagMAPCAMDATATYPE | | 
tagMAPDATA | | 
tagOVERRIDE | | 
tagPARTICLE | | 
tagPARTICLEFX | | 
tagPARTICLEFXLIST | | 
tagPLATFORMDEF | | 
tagPLATFORMWORK | | 
tagRAMPHEADER | | 
tagREBDATA | | 
tagTXTATTRIB | | 
tag_VENT | | 
u162DVECTOR | | 
u16VECTOR | | 


<div class="mermaid">
graph LR
        A-->B
        B-->C
        C-->A
        D-->C
</div>

# Playstation 1

## PS1 Gameshark cheats

Name | Code
--- | ---
Max Eggs |	800AA1D0 0063 [^1]
Have All Map Pieces (GS 2.2 or Higher Needed) |	50000901 0000 300AA1F1 0001[^1]

# Game Credits
* Fred Williams (Lead Programmer) - https://twitter.com/RFredW
  - ```c
    // I'll do these things for platforms, with a view to copying 'em into the main collision code
    // when it's free. Luv, Fred.
    ```
* Tom Drummond (Programmer)
* David Harries (Programmer) - https://www.linkedin.com/in/david-harries-92a56213/
* Barry Peterson (Programmer)
* Chris Wilson (Programmer)
* James Steele (Tools Programming) - https://www.linkedin.com/in/james-steele-0a62791/
  - ```c
    // Since Fred insits on being such a clever bugger and using in-line assembler, this file contains a load of macros so that the custom.c,     custom2.c and map_draw.c compile and work on the PC as well as the PSX without me having to write a whole load of PC draw routines.
    // James Steele```
* Chris Wilson (Tools Programming)
  - `
  /*	Curtains.c - by Chris Wilson
  It's curtains for you, my lad!*/
  `
* Ian Bird (Additional Programming) - https://www.linkedin.com/in/ian-bird-19358780/
* Steve Bond (Additional Programming) - https://www.linkedin.com/in/steve-bond-b21ba9/
* Scott Lamb (Additional Programming) - https://www.linkedin.com/in/scottjameslamb/ (SL: in comments)
* Andy Sidwell (Additional Programming)
* John Whigham (Additional Programming) - https://www.linkedin.com/in/john-whigham-b8524834/
* Richard Hackett (Compression Technology) - https://www.linkedin.com/in/richard-hackett-4324531

Also thanks to Phillip and Andrew Oliver who founded Blitz Games - http://www.olivertwins.com

# Sources
[^1]: http://www.cheatcc.com/psx/codes/chickenr.html#ixzz5RAbHltbW 