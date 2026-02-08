# RPCS3 BASIC GAME PATCH CREATION TUTORIAL
This is a basic step-by-step guide on **how to create and share a RPCS3 Game Patch** from scratch. This was motivated by the fact that documentation about this procedure is quite scarce. The exact steps for your own mod may differ, this is just a working example.

Our test subject will be *Tekken Tag Tournament 2*, and the modification we're aiming for is the removal of its 2 frames of forced offline input lag.
In case you're wondering what that means, certain Bandai Namco fighting games (TTT2,TK7,SC5,SC6) share the... "questionable" game design decision of injecting some fake input delay to your button presses offline as a cheap trick to make online play look less laggy by comparison.

# TL;DR - JUST SHOW ME A VIDEO
```
<video width="100%" height="100%" preload="metadata" controls>
  <source src="RPCS3.address.finding.tutorial.mp4" type="video/mp4" />
  <p>Your browser doesn't support HTML video. Here is a <a href="RPCS3.address.finding.tutorial.mp4" download="RPCS3.address.finding.tutorial.mp4">link to the video</a> instead.</p>
</video>
```

# PREPARATIONS
## Cheat Engine settings to change:
*SETTINGS YOU NEED TO APPLY ONLY ONCE:*
```
Settings -> Debugger Options -> Use VEH Debugger
Settings -> Extra Custom Types -> check all
Settings -> Scan Settings -> check "All Custom Types"
Settings -> Scan Settings -> Scan the following memory region types: MEM_PRIVATE + MEM_IMAGE + MEM_MAPPED
```
*SETTINGS YOU NEED TO APPLY EVERY STARTUP:*
```
Start:300000000 Stop:400000000
Writable/Executable/CopyOnWrite checkboxes = all Gray
```

## RPCS3 files to edit (while RPCS3 is closed):
```
  FILE: RPCS3\config\config.yml
PPU Decoder: Interpreter (static)
Assume External Debugger: true

  FILE: RPCS3\GuiConfigs\CurrentSettings.ini
debuggerVisible=true
```

# FINDING THE LAG ADDRESS IN CE
Let's enter Practice Mode because luckily, it has a setting that lets us simulate various levels of lag, which makes our search trivial. Spoiler: `OFF/5/4/3/2/1` ping bars translate to `2/2/5/8/11/14` frames. Then,
- in-game: set the lag to 1 bar, unpause.
- in CE: New Scan 14 (Value Type = 4 Byte Big Endian)
- in-game: set the lag to 2 bars, unpause.
- in CE: Next Scan 11
- in-game: set the lag to 3 bars, unpause.
- in CE: Next Scan 8
Repeat as necessary until we filter the results down to just our input lag address. We can tell if it's the correct one by changing it to a big number such as 120, throwing a punch, and noticing how it takes 2 whole seconds to come out.

# FINDING THE OPCODE THAT ACCESSES THE LAG ADDRESS IN RPCS3
But we can't simply set that value to 0 just once because it resets back to 2 every match, and we can't freeze it either because RPCS3 doesn't support freezing. So we need to find what opcode inside the main executable accesses it, and neutralize it.
Here's how:
- with the game unpaused, in CE *right click the input lag address -> Browse this memory region -> right click the first byte of the Memory Viewer -> Data Breakpoint -> Break on Access*. The game will freeze.
- in the RPCS3 debugger, select `PPU[0x1000000] main_thread` -> copy the `CIA: 0x????` line (it stands for *Current Instruction Address*).
`0x1c310`, that's where our relevant opcode is located! Let's jump there in the RPCS3 debugger with `Ctrl+G` -> paste `0x1c310` -> OK. The instruction there will say `lwz r9,0(r3)` (`81 23 00 00`)

By the way: the "current input lag" value gets accessed every game tick while not in menus, that's why the breakpoint hits immediately.

# PATCHING THE OPCODE
- Press `E` to edit the instruction with `39 20 00 00` (represents `li r9, 0`).
Naturally you'll be on your own when it comes to editing the relevant assembly code logic correctly. Basic points of reference:
https://jimkatz.github.io/powerpc_for_dummies
https://ret.futo.org/ppc/
Normally, when doing basic modding/reversing you can get a lot of mileage out of simply replacing things with the NOP/no-operation instruction (`60 00 00 00`), but in my case NOPing would disable inputs entirely. One method is to ask some AI to spoonfeed you the solution, for example [Qwen3-Max (Thinking)](<https://chat.qwen.ai/>) was able to answer correctly right away:
    > I have this PowerPC assembly instruction: “lwz r9,0(r3)”. I want to change it so that it writes a 0. The instruction has to fit within 4 bytes.
    > […]
    > li r9, 0 #encodes to 39 20 00 00
- In CE, *right click the byte where you previously set your Breakpoint on -> Delete Breakpoint -> click Run (or F9)* to resume the game.
- In-game, throw some punches and notice how the game will feel more responsive, as if the delay was permanently set to 0. We have successfully disabled the whole "Input Delay" system!

# CREATING AND SHARING THE PATCH
Now you'll want to write a proper RPCS3 Game Patch.
Format documentation: https://wiki.rpcs3.net/index.php?title=Help:Game_Patches
The RPCS3 menu `Utilities -> Patch Creator` can also help, but taking a pre-existing patch to use as a template is honestly the best way to learn. You should test your patch by saving it in `patches\imported_patch.yml` first, enabling it from `Manage -> Game Patches`, and fresh Booting the game (without savestates). Here's a minimal working patch of what we've learned so far (see the [wiki](<https://wiki.rpcs3.net/index.php?title=Tekken_Tag_Tournament_2#Patches>) for the full version):
```yml
Version: 1.2

PPU-8645cfa235c9980ff74b5e06388b29e97fdbbed9:
  "Reduce input lag":
    Games:
      "TEKKEN TAG TOURNAMENT 2":
        NPUB30899: [ 01.01 ]
    Author: "WAZAAAAA"
    Notes: "Reduce the delay of your button presses by 2 frames in all offline modes."
    Patch Version: 1.0
    Patch:
      - [be32, 0x0001c310, 0x39200000]
```

Once tested thoroughly (beware of compatibility with multiple regions/Game Updates!), you can share it with the public.
You do that by editing the corresponding game page on the RPCS3 wiki, and once approved by a staff member it should show up automatically in the [master list](<https://rpcs3.net/compatibility?patch&api=v1&v=1.2>) within a few minutes.

If you want to create cheats that work on modded PS3 hardware instead, look into [ArtemisPS3](https://bucanero.github.io/ArtemisPS3/).
