# PuP-Pack Reserch

Some reverse engineering of pinup popper pup packs for Visual Pinball Standalone (Linux, macOS, iOS, Android, ...)

* More excellent research done by @jsm174 at https://gist.github.com/jsm174/e5aa4ebe70052b5cf2ef49ab40c35dfb
* Findings by @LegendsUnchained at https://github.com/LegendsUnchained/vpx-standalone-alp4k/wiki/%5B11%5D-PUP-Pack-Table-Configuration

> [!WARNING]
> This is not a manual for making PuP packs. It's research for implementing PuP pack functionality on non-windows systems. Some of the things listed here are probably deprecated and should no longer be used.

## What are Pup Packs and where are they used

PuP Packs are a mix of media and metadata to render animated pinball table backbox screens for the [Visual Pinball](https://github.com/vpinball/vpinball) and [Future Pinball](https://futurepinball.com/) ecosystems.
The Windows PuP-Pack player is closed-source software written by [NailBuster](https://nailbuster.com).

https://nailbuster.com/wikipinup/doku.php?id=pinup_player_tables

## Important files

Pup packs are distributed as directories. Depending on the operating system and the flavour of pinball emulator you have to place them in a specific directory to be activated.

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
* `SN` = screen number (see `screens.pup` or`AddScreen` in script)
* `FN` = function

Functions:
* `2` - ???
  ```vbscript
  Sub pDisableLoopRefresh(PuPID)
     PuPlayer.SendMSG "{ ""mt"":301, ""SN"": "&PuPID& ", ""FN"": 2, ""FF"":0, ""FO"":0 }"   
  end Sub  
  ```
* `3` - hide/show overlay text - `{ "mt":301, "SN": XX, "FN":3, "OT": 0 }` - OT 0/1 overlay text on off bool
  ```vbscript
  sub pAllVisible(lvis)   '0/1 to show hide pup text overlay and HUD
    PuPlayer.SendMSG "{ ""mt"":301, ""SN"": "& "5"& ",""OT"":"&lvis&", ""FN"": 3 }"             'hideoverlay text force
  end Sub
  ```
* `4` - set StayOnTop - `{ "mt":301, "SN": XX, "FN":4, "FS":1/0 }`
* `6` - bring screen to front `{ "mt":301, "SN": XX, "FN":6 }`
* `10` - set ??? volume `{ "mt":301, "SN": XX, "FN":10, "VL":9}`  VL=volume level
* `11` - set screen sound volume 0-100 `"{ ""mt"":301, ""SN"": "&pMusic&", ""FN"":11, ""VL"":"&VolMusic&" }"`
* `12` - STOPSCREEN `{screenNum=18, screenDes=Apron Right, mode=PUP_SCREEN_MODE_FORCE_BACK}, fn=12, szMsg={ "mt":301, "SN": 18, "FN":12 }`
* `15` - set screen custom position (relative positions in %)
  `{ 'mt':301, 'SN':15,'FN':15,'CP':'5,0,0,100,100'} ' show screen`
  `{ 'mt':301, 'SN':15,'FN':15,'CP':'5,0,0,0,0'} 	' Hide the screen `
* `16` - launch executable - WT: Window Title
    `{ ""mt"":301, ""SN"": 2, ""FN"":16, ""EX"": """&PuPMiniGameExe  &""", ""WT"": """&PuPMiniGameTitle&""", ""RS"":1 , ""TO"":15 , ""WZ"":0 , ""SH"": 1 , ""FT"":""Visual Pinball Player"" }`
    `{ ""mt"":301, ""SN"": 2, ""FN"":16, ""EX"": ""Pupinit.bat"", ""WT"": """", ""RS"":1 , ""TO"":15 , ""WZ"":0 , ""SH"": 1 , ""FT"":""Visual Pinball Player"" }`
* `17` { ""mt"":301, ""SN"": ""2"", ""FN"":17, ""WT"":""Visual Pinball Player"", ""WZ"": 1, ""WP"": 1 } -  WT: Window Title / WZ: HWND_BOTTOM=1 / WP: SWP_NOSIZE=1
  ```vbscript
  Sub pResynchLayers()	' For desktop users this will force VPX to the back so pup is in front (Disabled since David has cut through working now)
  	exit sub 
  	if bDesktop then
  		' WT: Window Title 
  		' WZ: HWND_BOTTOM=1
  		' WP: SWP_NOSIZE=1
  		PuPlayer.SendMSG "{ ""mt"":301, ""SN"": ""2"", ""FN"":17, ""WT"":""Visual Pinball Player"", ""WZ"": 1, ""WP"": 1 }"
  	End if 
  End Sub
  ```
* `20` - `{ "mt":301, "SN"": XX, "FN":20, "AM": 1, "AV": 170 }` also related to transparency?
* `22` - set transparency
  `PuPlayer.SendMSG "{ ""mt"":301, ""SN"": " & pTransp &", ""FN"":22, ""AM"":1, ""AV"":255 }"  ' make opaque`
  `PuPlayer.SendMSG "{ ""mt"":301, ""SN"": " & pTransp &", ""FN"":22, ""AM"":1, ""AV"":80 }" ' Make Transparent `
* `30` - PUPDisplayAsJukebox - `{'mt':301, 'SN': XX, 'FN':30, 'PM':1 }`
  ```vbscript
  'jukebox mode will auto advance to next media in playlist and you can use next/prior sub to manuall advance
  'you should really have a specific pupid# display like musictrack that is only used for the playlist
  'sub PUPDisplayAsJukebox(pupid) needs to be called/set prior to sending your first media to that pupdisplay.
  'pupid=pupdiplay# like pMusic
  Sub PUPDisplayAsJukebox(pupid)
  	PuPlayer.SendMSG("{'mt':301, 'SN': " & pupid & ", 'FN':30, 'PM':1 }")
  End Sub
  ```
* `31` - pup jukebox control - `{'mt':301, 'SN': XX, 'FN':31, 'PM':1 }` - PM 1 = next, PM 2 = previous
  ```vbscript
  Sub PuPlayListPrior(pupid)
	PuPlayer.SendMSG("{'mt':301, 'SN': " & pupid & ", 'FN':31, 'PM':1 }")
  End Sub

  Sub PuPlayListNext(pupid)
	PuPlayer.SendMSG("{'mt':301, 'SN': " & pupid & ", 'FN':31, 'PM':2 }")
  End Sub
  ```
* `32` - no antialias on font render `{ "mt":301, "SN": 1, "FN":32, "FQ":3 }` FQ - font quality (3 = no aa?)
* `33` - set pupdmd for mirror and hide behind other pups???  `"{ "mt":301, "SN": XX, "FN":33 } `
* `34` - hideoverlay text during next videoplay on DMD auto return??? `{ "mt":301, "SN": XX, "FN": 34 }`
* `41` - set safeloop mode on current playing media - `{ "mt":301, "SN": XX, "FN":41 }`
  ```vbscript
  'set safeloop mode on current playing media.  Good for background videos that refresh often?  { "mt":301, "SN": XX, "FN":41 }
  Sub pSafeLoopModeCurrentVideo(PuPID)
     PuPlayer.SendMSG "{ ""mt"":301, ""SN"": "&PuPID& ", ""FN"": 41 }"
  end Sub
  ```
* `42` - duck audio volume - `{ "mt":301, "SN": XX, "FN": 42, "DV": duck_volume , "ALL":1 }` ALL 1 = optional
  ```vbscript
  Sub AudioDuckPuP(MasterPuPID,VolLevel)  
    'will temporary volume duck all pups (not masterid) till masterid currently playing video ends.  will auto-return all pups to normal.
    'VolLevel is number,  0 to mute 99 for 99%  
    PuPlayer.SendMSG "{ ""mt"":301, ""SN"": "& MasterPuPID& ", ""FN"": 42, ""DV"": "&VolLevel&" }"             
  end Sub

  Sub AudioDuckPuPAll(MasterPuPID,VolLevel)  
    'will temporary volume duck all pups (not masterid) till masterid currently playing video ends.  will auto-return all pups to normal.
    'VolLevel is number,  0 to mute 99 for 99%  
    PuPlayer.SendMSG "{ ""mt"":301, ""SN"": "& MasterPuPID& ", ""FN"": 42, ""DV"": "&VolLevel&" , ""ALL"":1 }"             
  end Sub
  ```  
* `45` - slow pc mode `{ "mt":301, "SN":XX, "FN":45, "SP":1 }` - SP 0/1 = slow pc mode bool
  ```vbscript
  Sub pSetLowQualityPc  'sets fulldmd to run in lower quality mode (slowpc mode)  AAlevel for text is removed and other performance/quality items.  default is always run quality, 
     PuPlayer.SendMSG "{ ""mt"":301, ""SN"": 5, ""FN"":45, ""SP"":1 }"    'slow pc mode
  end Sub
  ```
* `46` - pad all text `{ "mt":301, "SN": XX, "FN":46, "PA":1 }` - PA 0/1 = padd text bool
  ```vbscript
  Sub pDMDAlwaysPAD  'will pad all text with a space before and after to help with possible text clipping.
     PuPlayer.SendMSG "{ ""mt"":301, ""SN"": XX, ""FN"":46, ""PA"":1 }"    'slow pc mode
  end Sub
  ```  
* `50` - set aspect ratio `{ "mt":301, ""SN": XX, "FN": 50, "WIDTH": arWidth, "HEIGHT": arHeight }`
  ```vbscript
  Sub pSetAspectRatio(PuPID, arWidth, arHeight)
     PuPlayer.SendMSG "{ ""mt"":301, ""SN"": "&PuPID& ", ""FN"": 50, ""WIDTH"": "&arWidth&", ""HEIGHT"": "&arHeight&" }"
  end Sub
  ```
* `51` - set media play position in ms `{ "mt":301, "SN": XX, "FN":51, "SP": 3431}` - SP position in ms
  ```vbscript
  Sub pSetVideoPosMS(mPOS)  'set position of video/audio in ms,  must be playing already or will be ignored.  { "mt":301, "SN": XX, "FN":51, "SP": 3431} 
    PuPlayer.SendMSG "{ ""mt"":301, ""SN"": "& "5"& ", ""FN"": 51, ""SP"":"&mPOS&" }"
  end Sub
  ```
* `52` - set text quality, anti aliassing (similar to 32?) - `{ "mt":301, "SN": XX, "FN":52, "SC": aa_level }` SC 0-4 from low to high
  ```vbscript
  Sub pDMDSetTextQuality(AALevel)  '0 to 4 aa.  4 is sloooooower.  default 1,  perhaps use 2-3 if small desktop view.  only affect text quality.  can set per label too with 'qual' settings.
     PuPlayer.SendMSG "{ ""mt"":301, ""SN"": 5, ""FN"":52, ""SC"": "& AALevel &" }"    'slow pc mode
  end Sub
  ```
* `53` experimental frame rescale -
  ```vbscript
  Sub pForceFrameRescale(PuPID, fWidth, fHeight)   'Experimental,  FORCE higher frame size to autosize and rescale nicer,  like AA and auto-fit.
       PuPlayer.SendMSG "{ ""mt"":301, ""SN"": "&PuPID& ", ""FN"": 53, ""XW"": "&fWidth&", ""YH"": "&fHeight&", ""FR"":1 }"   
  end Sub  
  ```

> [!NOTE]
> Some of these are [implemented in Visual Pinball Standalone](https://github.com/vpinball/vpinball/blob/9443e1f0b54dcdc755a64354fd34836c2ac2bc4b/standalone/inc/pup/PUPPinDisplay.cpp#L408).
