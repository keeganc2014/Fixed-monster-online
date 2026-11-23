# Gtag-style map test (offline)

Open this folder in Unity Hub with **Unity 6 (6000.0 LTS)** (add the Android Build Support module if you want a Quest build).

1. First open: Unity downloads the XR packages and creates `Assets/Scenes/Main.unity` by itself.
2. Press Play. The scene builds itself at runtime: map, lighting, gorilla rig, menu. Nothing to drag into the scene.
   - No headset: right-mouse-drag = look, WASD = move, Space = jump, Shift = fly, Q/E = platforms, Tab = menu (aim at a button + left click).
3. Quest build (local): menu **Gtag Test > Configure for Quest (Android)** (also enables the Oculus XR loader), then File > Build Settings > Build.
   Or headless: `Unity -batchmode -quit -projectPath . -executeMethod BuildSetup.BuildAndroid` -> `Builds/GtagMapTest.apk`.

## Unity Build Automation (Cloud Build) -> APK
1. Push this folder's *contents* to the root of a Git repo (or set "Project subdirectory" in the build target). `.gitignore` is included.
2. Create a build target: platform **Android**, Unity version **6000.0.23f1** (or the closest 6000.0 LTS offered).
3. Advanced settings -> **Pre-export method name**: `BuildSetup.PreExport`
   (creates the scene, applies IL2CPP/ARM64/Vulkan/Linear color space, assigns the Oculus loader).
4. Build. Download the APK and sideload it to the Quest (`adb install GtagMapTest.apk`).
   No keystore is needed for sideloading; add one in the target's credentials for a Store/release build.

## In VR
- Left X: open/close the **Main Menu**. It sits on your left hand and turns to face you. Poke a button with your right hand + right trigger.
  - **Player**: name and color. **Settings**: turn speed, monster on/off + speed, mic mute, reconnect, reset board.
  - **Mods** and **Moderator** are locked: press them (or "Owner Code"), type the owner code on the keypad. That also gives you the gold **OWNER** title above your head for everyone else.
  - Mods: Speed, Gravity, Long Arms (1.5x/2x/3x), Fly, Noclip, Platforms, Invincible, Freeze Monster, Flashlight, Darkness, Random Spot, Save/Load Pos, Reset Mods.
  - Moderator: pick a player, then Kick, Bring To Me, Go To Player, Freeze 10s, Send To Stump.
- **Monster**: stands on the forest floor near the gazebo (the roofed camp structure) and scans around. It chases only when it sees you or you get close, and gives up a few seconds after losing you. If it catches you: "YOU DIED", back to the stump.
- **Leaderboard**: on the map's scoreboard, readable from both sides. Longest time alive, with name + color.
- **Online**: everybody joins ONE public lobby automatically ("gtag-public-lobby", up to 12 players). You see the others, with name tags, and hear them with positional voice chat. The status line at the bottom of the menu shows what is happening (Online: 3 in lobby | Voice: on).

## One-time setup for online + voice (Unity Gaming Services)
1. Open the project in the Editor once: Edit > Project Settings > Services, sign in and **link it to your Unity project**. Commit the changed ProjectSettings so Cloud Build uses the same project.
2. In the Unity Dashboard for that project, make sure **Multiplayer (Relay + Lobby)** and **Vivox** are enabled.
3. Build as usual (BuildSetup.PreExport does the rest). Quest asks for the microphone permission the first time.
If it says "Offline: ..." or "Voice: ..." at the bottom of the menu, that text is the reason.

## Security note
The owner code (123) is checked inside the app, so it keeps casual players out but is NOT real security: anyone who unpacks the APK can find it. Moderator commands are checked by the host with the same code. For real protection use a server check (e.g. Unity Cloud Code) or an allow-list of your player ID.

## Player rig (your Blender model)
Unity Cloud has no Blender, so convert once on a PC with Blender:
`blender -b _Originals/Gorilla_IK_Rig.blend -P Tools/export_rig_fbx.py` -> writes `Assets/Resources/Rig/GorillaRig.fbx`.
The game loads that FBX automatically. Until it exists, a built-in gorilla (body + stretchy arms) is used.

## Files
- `Assets/Scripts/GorillaLocomotion/` - your Player.cs + Surface.cs (unchanged)
- `Assets/Scripts/GtagBootstrap.cs` - builds the scene, loads the map, spawns the rig with the original prefab's tuning values
- `Assets/Editor/BuildSetup.cs` - Quest/XR build configuration used by the menu, the batch build and Cloud Build
- `Assets/Scripts/XRRigInput.cs` - headset/controller input (+ desktop fallback)
- `Assets/Scripts/MainMenu.cs` - combined main menu (mods + settings + player)
- `Assets/Scripts/OnlineManager.cs`, `RemoteAvatar.cs` - public lobby, voice, other players, moderator commands
- `Assets/Scripts/Monster.cs`, `Leaderboard.cs`, `PlayerRigModel.cs`, `PlayerProfile.cs` - monster, board, gorilla model, saved settings
- `Tools/export_rig_fbx.py` - Blender -> FBX export for the rig
- `Assets/Resources/Map/` - map FBX + textures (30 textures in your zip; the FBX references a few more that weren't included, those materials stay plain)
- `_Originals/` - your original unitypackage (outside Assets, ignored by Unity)

Poison_Menu.dll is intentionally not in the project: it hooks into the real game's code and can't load in a standalone Unity project.
