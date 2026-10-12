# INPUT LAG MODS
This is my collection of modifications that improve the responsiveness/lag/delay/latency of button presses, mostly aimed at fighting games. Sometimes the developers cause delays unintentionally with sub-optimal code, other times they inject it on purpose in the misguided attempt of making their bad netcode look less bad by sacrificing offline play.

There's a trivial input lag test anyone with a computer and Cheat Engine can try on almost any game:
1. play the game at 1 FPS (set Speedhack to 0.02 with CE)
2. hold down the attack button, and count the frames it takes for the game to respond

That is my main testing method. Using shortcuts to control the speed (CE -> Settings -> Hotkeys) it becomes quite comfortable to use. Anything that takes longer than 0 frames to respond means you have an unoptimized laggy game on your hands at the mechanical/gameplay level, and no amounts of expensive low latency peripherals will matter. 0F delay means "you hold a button frame 1, and the game responds visually on frame 2", the optimal scenario.

Games optimized so far:
- SC5
- SC6
- TK7
- TTT2
- USF4
- 2XKO
- FF10HD

The "PATCH" sections list the Array of Byte hex patterns you can search & replace to mod the games yourself using an hex editor of your choice.

## Soulcalibur 5
**REDUCTION:**  
2 frames, neutralize the forced network lag simulator option in training mode.  

**LINKS:**  
https://wiki.rpcs3.net/index.php?title=SoulCalibur_V#Patches  
https://github.com/xenia-canary/game-patches/pull/477  

**PATCH:**  
```
7C E9 00 2E 81 2B 03 DC -> 38 E0 00 00 81 2B 03 DC (PS3)
7D 26 58 2E 7D 06 28 2E -> 7D 26 58 2E 39 00 00 00 (X360)
```

## Soulcalibur 6
**REDUCTION:**  
1 frame, neutralize the forced network lag simulator option in training mode.  

**LINKS:**  
https://8wayrun.com/threads/mod-to-reduce-offline-input-lag-on-pc-by-1-frame.20979/  

**PATCH:**  
```
74 06 2B 9F 90 03 00 00 ->  74 06 90 90 90 90 90 90 (Windows SoulcaliburVI.exe)
```

## Tekken 7
**REDUCTION:**  
1 frame, neutralize the forced network lag simulator option in training mode.  

**LINKS:**  
https://archive.supercombo.gg/t/mod-to-reduce-tekken-7-offline-input-delay/184155  
https://web.archive.org/web/20191014113334/https://forums.shoryuken.com/t/mod-to-reduce-tekken-7-offline-input-delay/184155  

**PATCH:**  
```
0C 8B 86 F4 04 00 00 -> 0C 90 90 90 90 90 90 (Windows TekkenGame-Win64-Shipping.exe)
```

## Tekken Tag Tournament 2
**REDUCTION:**  
2 frames, neutralize the forced network lag simulator option in training mode.  

**LINKS:**  
https://wiki.rpcs3.net/index.php?title=Tekken_Tag_Tournament_2#Patches  
https://github.com/xenia-canary/game-patches/pull/452  
https://github.com/cemu-project/cemu_graphic_packs/pull/734  
https://x.com/JustinSGX/status/1994846546643620294  
https://ieoei.github.io/-/1/RPCS3-basic-Game-Patch-creation-tutorial  

**PATCH:**  
```
20 81 23 00 00 81 48 -> 20 39 20 00 00 81 48 (PS3)
90 83 00 00 3D 00 82 00 -> 60 00 00 00 3D 00 82 00 (X360)
90 0A 00 10 80 EB 00 00 -> 90 0A 00 10 38 E0 00 00 (WiiU)
```


## Ultra Street Fighter 4
**REDUCTION:**  
2 frames on controller, 3 frames on keyboard.  
For the gameplay lag mod, it was an off-by-1 bug. Its piece of logic points to `esi+2C` which is a 4-byte frame counter that raises by +1 each frame during a match. Its "neighboring" address at `esi+28` is the same frame counter except smaller by 1F... if we swap `2C` to `28`, the lag decreases by 1F.  
For the keyboard lag mod, it was some kind of bad queue running "one step late" that we reordered for better sync.  
For the ini global edit, we change a flag regarding how the MT Framework engine renders the game, vaguely similar to Unreal Engine's "OneFrameThreadLag" I suppose.  

**LINKS:**  
https://github.com/Confetti3/SF4-Ember-Netplay/issues/77  

**PATCH:**  
```
8B 0F 8B 56 2C -> 8B 0F 8B 56 28 (Windows SSFIV.exe, gameplay lag 1F reduction)
89 45 CC 8B 42 48 6A 01 -> 89 45 CC 8B 42 0C 90 90 (Windows SSFIV.exe, keyboard-only extra lag 1F reduction part 1)
0F 84 9A 02 00 00 8B 8E -> 90 90 90 90 90 90 8B 8E (Windows SSFIV.exe, keyboard-only extra lag 1F reduction part 2)

%USERPROFILE%\Documents\CAPCOM\SUPERSTREETFIGHTERIV\config.ini... RenderingThread=ON -> RenderingThread=OFF
```

## 2XKO
**REDUCTION:**  
3 frames, neutralizing the internal "GetInputDelay" function.  
My theory for their design decision is that despite using rollback netcode made by the creators of GGPO and the official reasons listed in the blog post, the real reason is that the higher-ups decided that the game couldn't be peer-to-peer for anti-cheat and DRM reasons, forcing connections to go through an extra server relay hop, thus almost inevitably increasing the lag, thus resorting to the cheap shortcut of sacrificing offline to mask the online rollbacks.  

**LINKS:**  
https://photon-reddit.com/r/Kappachino/comments/1wwn1hl/2xko_input_lag_reduction_mod_3_frames/  
https://2xko.riotgames.com/en-us/news/dev/how-2xko-handles-online-play/  

**PATCH:**  
```
8B E8 39 44 -> 31 ED 39 6C (Windows Lion-Win64-Shipping.exe, DRM-less version required)
```

## Final Fantasy 10 HD Remaster
**REDUCTION:**  
The battle menus are slow, 350 ms + 400 ms input delays & 200 ms animation delays on every page switch. FF10-2 is unaffected.  

**LINKS:**  
https://www.nexusmods.com/finalfantasyxx2hdremaster/mods/360  
https://gamebanana.com/mods/725233  

**PATCH:**  
```
DA FF D8 66 48 -> DA FF 90 90 90 (Steam FFX.exe, 0.30 single-float/0.45 double-float input)
45 08 DD 05 ?? EB ?? ?? D8 -> 45 08 D9 EE 90 90 90 90 D8 (FFX.exe Steam, 0.20 double-float animation switch)
45 F4 DD 05 ?? EB ?? ?? D8 -> 45 F4 D9 EE 90 90 90 90 D8 (FFX.exe Steam, 0.20 double-float animation overdrive)
03 C1 DD 05 ?? EB ?? ?? 99 -> 03 C1 D9 EE 90 90 90 90 99 (FFX.exe Steam, 0.20 double-float animation main)
45 F8 DD 05 ?? EB ?? ?? D8 -> 45 F8 D9 EE 90 90 90 90 D8 (FFX.exe Steam, 0.20 double-float animation sub)
```
