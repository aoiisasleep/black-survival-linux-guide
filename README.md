# black-survival-linux-guide 
This guide explains how to run Black Survival: Project Lumia on Linux using Wine, along with Discord PTB Portable so the can detect Discord proprely. Everything here is based on a fully working setup i tested myself. This guide will walk you through:

-Preparing your Wine prefix

-Installing the Project Lumia Launcher

-Setting up Discord PTB Portable

-Applying registry fix

-Lauching both together

# Requierments
### 1. Wine & Winetricks
   The game itself and Discord PTB Portable run best on normal Wine.

   Install them (Arch example):
   ```bash
   sudo pacman -S wine winetrickss
   ```
   on Debian/Fedora/etc., Install via your distro's package manager.


### 2. Proton-GE (only needed for Project Lumia Installer)
   
   The Project Lumia Installer sometimes fails under normal Wine but works reliably with Proton-GE in my testing.

   You can install Proton-GE here -> [protongerelease](https://github.com/GloriousEggroll/proton-ge-custom/releases)


### 4. Project Lumia Installer

   Download from their Discord server here -> https://discord.gg/projectlumia

   
### 5. Discord PTB Portable

   In my testing normal Discord has issues inside Wine, so i used Discord PTB Portable that works properly and reliably.

   Download it from the Github here -> [discordptbportable](https://github.com/portapps/discord-ptb-portable/releases)


# Preparing Your Wine Prefixes
Because the Lumia installer currently requires Proton‑GE’s improved WOW64 handling, while Discord PTB does not run correctly under Proton‑GE, we will be creating two separate Wine prefixes:

Prefix A – Proton‑GE Prefix → used only for running the Project Lumia installer

Prefix B – Normal Wine Prefix → used to actually run Project Lumia + Discord PTB Portable

This keeps everything stable and avoids the problems we ran into (WOW64 mode issues + Discord failing to launch under Proton‑GE).


### 1. Create the Installer Prefix

  This prefix will run the Project Lumia installer using Proton‑GE later, but for now it should be created as a normal Wine prefix.
  ```bash
  WINEPREFIX="$HOME/.bs-installer" wineboot --init
```

 ### Install the dependecies required by the installer
 These are the only ones we need here:
 ```bash
WINEPREFIX"/path/to/prefix" winetricks -q corefonts dotnet48 gdiplus wmp9
```

### 2. Create the Game Prefix
This is where we're actually running Project Lumia and Discord PTB Portable
```bash
WINEPREFIX="$HOME/.black-survival" wineboot --init
```

### Install the dependencies needed by the game prefix:
```bash
WINEPREFIX="$HOME/.black-survival" winetricks -q corefonts dotnet48 vcrun2015 d3dcompiler_47

# Installing Black Survival Project Lumia
The Project Lumia installer requires Proton‑GE’s improved WOW64 support to render the UI properly.
A normal Wine prefix cannot run the installer.

### Run the installer
Navigate to the folder where your Project Lumia installer EXE is located, and run with Proton-GE like this:
```bash
WINEPREFIX="/path/to/installer-prefix" "/path/to/GE-Proton/files/bin/wine" BlackSurvivalInstaller.exe
```
>**Note:** The installer prefix is temporary — its only purpose is to run the game installer with Proton‑GE and the needed dependencies.
You can delete this prefix after installation, because the actual game files will be placed in whatever folder you choose during the installer setup. To avoid confusion, when the installer asks where to install the game, you should choose a different folder outside the installer prefix; "/home/username/Games/BlackSurvival"


# Installing Discord PTB Portable
Installing the Discord PTB is pretty straight forward just navigate to where the installer is located and run:
```bash
WINEPREFIX="/path/to/game-prefix" wine discord-ptb-portable-win64-<version>-setup
```
Once the Discord PTB installer finishes:

-Run Discord PTB once so it can finish setting itself up.

-Log in to your account

### At this point:

If you run Black Survival’s launcher, it should open normally and download any updates.

But if you try launching the actual game right now, you will almost definitely get this error:

“No app to open this type of file.”

this happens because the registry entries for .exe handling in this prefix aren’t fixed yet.
You’ll fix this in the Registry Fix section next.

# Registery Fix
Some versions of Wine/Proton can’t correctly open external links using the default handler (winebrowser).
This causes Black Survival to fail when trying to open web URLs, resulting in errors like "No app to open this type of file".

To fix this, we manually set Firefox as the handler for http and https links.

### Before continuing:

Make sure you have Firefox installed inside your game prefix.
You can install it with:
```bash
WINEPREFIX="/path/to/game-prefix" winetricks firefox
```
>**Note:** You can use diffrent browser if you want.

### 1. Open regedit on your game prefix
```bash
WINEPREFIX="/path/to/game-prefix" wine regedit
```

### 2. Navigate to
```arduino
HKEY_CLASSES_ROOT\http\shell\open\command
HKEY_CLASSES_ROOT\https\shell\open\command
```

### 3. Replace the default value with:
```perl
"C:\Program Files (x86)\Mozilla Firefox\firefox.exe" "%1"
```
Adjust the path if your Firefox is installed elsewhere, or you're using a diffrent browser.

with this done you should be able to run Black Survival and the game will detect the Discord running.

# Running Black Survival + Discord Together (Launch Script)
Once everything is installed and your registry/browser fix is done, you can make the game and Discord PTB start together using a small script.
This also makes sure Discord closes automatically after you exit the game.

### 1. Example Launch Script
```bash
#!/bin/bash

# Change this to your actual Wine prefix
PREFIX="/path/to/your/bs-prefix"

# Path to Discord PTB portable EXE (inside the prefix)
DISCORD="$PREFIX/drive_c/portapps/discord-ptb-portable/discord-ptb-portable.exe"

# Path to the Project Lumia launcher (your game folder)
GAME_LAUNCHER="/path/to/your/BlackSurvival/Project Lumia Launcher.exe"

export WINEPREFIX="$PREFIX"

# Make sure the script runs from the Black Survival game folder
cd "$(dirname "$GAME_LAUNCHER")"

# Start Discord PTB in the background (only if it's not already running)
if ! pgrep -f "DiscordPTB.exe" >/dev/null; then
    wine "$DISCORD" &>/dev/null &
    sleep 4
fi

# Start the game
wine "$GAME_LAUNCHER"

# When the game closes, shut down the Discord PTB process
pkill -f "DiscordPTB.exe"
```
With this script you should be able run the game and it will kill Discord in the background, if for some reason you don't want that you can delete this line 
`pkill -f "DiscordPTB.exe"`

### 2. Why we `cd` into the game folder
Project Lumia’s launcher only works if the current working directory is the game folder.
If you try launching it from another place (like an alias, a .desktop file, or a script located somewhere else), the launcher crashes.

That’s why the script uses:

```bash
cd "$(dirname "$GAME_LAUNCHER")"
```
This makes sure the launcher is running from the correct folder, no matter where the script itself is stored.

### 3.  Using it with aliases or a .desktop file
Because the script fixes the working directory internally, you can place it anywhere:

`~/bin/bs.sh`

`~/Games/bs-launch.sh`

Or even launch it from a desktop shortcut

As long as the script contains the `cd` part, it will work.

# Common mistake
1. Running the game/installer without changing into its folder first

   When running Windows applications using Wine, you might encounter issues if you launch the executable without first navigating to the directory where the program and its supporting files are located. The application may crash or fail to launch because it cannot find its necessary data.
```bash
WINEPREFIX="/path/to/prefix" wine /path/to/executable
```

Fixing it is very simple just navigate to where the executable is located with `cd` and run the executable
```bash
WINEPREFIX="/path/to/prefix" wine executable.exe
```

2. Not changing the Windows version in winecfg
   Project Lumia behaves best when the Wine prefix is set to Windows 10.
   Older versions (like Win7) might work, but it’s not guaranteed, and I can’t say for sure—so just set it to Win10 to avoid weird issues.

   To change it:
   ```bash
   WINEPREFIX="/path/to/prefix" winecfg
   ```
   and simply change the default to Windows 10.
