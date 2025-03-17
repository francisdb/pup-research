# PuP-Pack Reserch

Some reverse engineering of pinup popper pup packs

## What are Pup Packs and where are they used

PuP Packs are a mix of media and metadata to render animated pinball table backbox screens for the [Visual Pinball](https://github.com/vpinball/vpinball) and [Future Pinball](https://futurepinball.com/) ecosystems.
The Windows PuP-Pack player is closed-source software written by [NailBuster](https://nailbuster.com).

https://nailbuster.com/wikipinup/doku.php?id=pinup_player_tables

## Important files

Pup packs are distributed as directories. Depending on the operating system and the flavour of pinball emulater you have to place them in a specific directory to be activated.

There are 3 main files that set up the PuP-Pack:
* `screens.pup` - Lists all screens and where they are displayed
* `playlists.pup` - Media playlists
* `triggers.pup` - Controls what is loaded where and when. (can react to [PinMAME](https://github.com/vpinball/pinmame) events)

_Older PuP Packs don't have these files and define this data at runtime in the table script._

> [!NOTE]
> Most pup packs come with different `.pup` files for different screen configurations. Check the `.bat` files in the root of the pup pack to see what they do. Most of the time it's copying over .pup files from the `PuP-Pack_Options/chosen_option` directory to the root of the pup pack.


## Scripting API

Virtual pinball tables also interact with the PuP-Pack player in their script. All examples below use `VBScript` which is the scripting language in use by Visual Pinball.

```vbscript
Dim PuPlayer
Set PuPlayer = CreateObject("PinUpPlayer.PinDisplay")

PuPlayer.LabelNew pBackglass,"Play2score",numberfont,	5,0  ,0,0,1,55,85,1,0

PuPlayer.B2SData "E"&EventNum,1  'send event to Pup-Pack
...
```

### LabelNew

TODO

### SendMSG

> NailBuster thinks nobody should be using these calls directly https://www.vpforums.org/index.php?showtopic=47811&p=487964

Example:
```vbscript
PuPlayer.SendMSG "{ 'mt':301, 'SN': 2, 'FN':15, 'CP':'99,15,15,70,70' }"  'custompos
```

JSON body but extended where properties can also be quoted with single quotes.

* `MT` = ??? (always 301?)
* `SN` = screen number (see screens.pup)
* `FN` = function

Functions:
* `3` - hide(/show?) screen - `"{ ""mt"":301, ""SN"": 13, ""FN"":3, ""OT"": 0 }"`
* `4` - set StayOnTop - `{ "mt":301, "SN": XX, "FN":4, "FS":1/0 }`
* `6` - bring screen to front ` "{ ""mt"":301, ""SN"": " & pDMDText & ", ""FN"":6 }"`
* `10` - set ??? volume `{ "mt":301, "SN": XX, "FN":10, "VL":9}`  VL=volume level
* `11` - set screen sound volume 0-100 `"{ ""mt"":301, ""SN"": "&pMusic&", ""FN"":11, ""VL"":"&VolMusic&" }"`
* `12` - STOPSCREEN `{screenNum=18, screenDes=Apron Right, mode=PUP_SCREEN_MODE_FORCE_BACK}, fn=12, szMsg={ "mt":301, "SN": 18, "FN":12 }`
* `15` - set screen custom position (relative positions in %)
  `{ 'mt':301, 'SN':15,'FN':15,'CP':'5,0,0,100,100'} ' show screen`
  `{ 'mt':301, 'SN':15,'FN':15,'CP':'5,0,0,0,0'} 	' Hide the screen `
* `16` - launch executable `"{ ""mt"":301, ""SN"": 2, ""FN"":16, ""EX"": """&PuPMiniGameExe  &""", ""WT"": """&PuPMiniGameTitle&""", ""RS"":1 , ""TO"":15 , ""WZ"":0 , ""SH"": 1 , ""FT"":""Visual Pinball Player"" }" `
* `22` - set transparency
  `PuPlayer.SendMSG "{ ""mt"":301, ""SN"": " & pTransp &", ""FN"":22, ""AM"":1, ""AV"":255 }"  ' make opaque`
  `PuPlayer.SendMSG "{ ""mt"":301, ""SN"": " & pTransp &", ""FN"":22, ""AM"":1, ""AV"":80 }" ' Make Transparent `
* `30` - PUPDisplayAsJukebox - `"{'mt':301, 'SN': " & pupid & ", 'FN':30, 'PM':1 }"`
* `31` - pup jukebox control - `"{'mt':301, 'SN': " & pupid & ", 'FN':31, 'PM':1 }"` - PM 1 = next, PM 2 = previous
* `32` - no antialias on font render ` "{ ""mt"":301, ""SN"": 1, ""FN"":32, ""FQ"":3 }" `
* `33` - set pupdmd for mirror and hide behind other pups???  `"{ ""mt"":301, ""SN"": 2, ""FN"":33 }" `
* `34` - hideoverlay text during next videoplay on DMD auto return??? `"{ ""mt"":301, ""SN"": "& pDisp &", ""FN"": 34 }"`

TODO: Some more functions documented here:
https://github.com/mpcarr/aztec-quest/blob/682d81fc78cd66c83d2038246be44a65f062d2f3/scripts/dest/mpf/tablescript.vbs#L7123-L7253
(these seem to be part of the `PUPDMD FRAMEWORK v3.0 BETA` txt file, not sure where to get that)

> [!NOTE]
> Some of these are [implemented in Visual Pinball Standalone](https://github.com/vpinball/vpinball/blob/9443e1f0b54dcdc755a64354fd34836c2ac2bc4b/standalone/inc/pup/PUPPinDisplay.cpp#L408).
