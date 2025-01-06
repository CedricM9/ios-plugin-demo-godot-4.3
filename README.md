# iOS-Plugin-Demo for Godot 4.3 #

This Readme and the Demo Project are meant to work together to help people understand how to get iOS Plugins to work with Godot.

There are A LOT of potential pitfalls someone can fall into trying to get this working. It's not entirely at the fault of Godot. It's a complicated procedure.

This is forked from [this excellent guide](https://github.com/LettucePie/ios-plugin-integrate-demo) and follows most of the same instructions. Trying this with Godot 4.3 leads to some errors however, so an extra step is required for it to work.
All of the instructions worked for me as of godot 4.3-stable, and the pre-built plugins are included. The demo project has not yet been ported over to 4.3 and will most likely not work with the updated plugins

## Requirements ##

* MacOS and XCODE (or a Linux workaround)
* git
* scons and python
	* I used brew for this
* MoltenVK 2.1.6 (New in 4.3)
* Godot Export Templates
	* https://docs.godotengine.org/en/stable/tutorials/export/exporting_projects.html

## Guide ##
Note that the demo project probably wont work with the updated plugins as the plugins are built for 4.3 and the project is built on 4.0

1. Run this Project in Editor at least once.
2. Export to XCODE / iOS.
	1. Download Export Templates if you haven't already.
	2. Make your iOS Preset.
	3. Setup your IDs and Export Mode in the Application section of your Preset.
		1. This could not be more vague, I'm sorry. In short, you need to know your Appstore connect Team ID, and your Provisioning Profile UUID. I would recommend keeping the Export Method Debug as Development, and the Release as App Store.
3. Build/Run the Project from XCODE once without any plugins.
	1. If you encounter an Error here, then your Application section is probably wrong, or there is something wrong with your XCODE installation. Follow along the Output Console at the bottom of the editor to diagnose.
	2. Open the IPA in XCODE.
	3. Configure your Device to XCODE (Plug it in and enable developer settings)
	4. Run the Project by hitting the Play Icon near the top. Once the App opens on your device you should see a red square.
4. Return to your export preset and enable the In App Store Plugin, then Export again.
	1. If you encounter an Error here, then something is wrong with the Plugin. This Demo was made on Godot 4.0.2, and the Plugin Headers were compiled on that same source. Consider running this demo on 4.0.2, or compiling new headers on your target version... More on that Later.
	2. Open XCODE normally from the Applications Menu, it should now show your project on the side.
	3. Build the Project to your iPhone again, and you should hopefully now see a green square.

## Setting up other Plugins ##

This Demo comes with the inappstore plugin compiled for 4.3 by me. Logistically you have no reason to trust me, and can/should replace my plugin with your own compiled version. You can also download the plugins released here (https://github.com/godotengine/godot-ios-plugins/releases)

At the time of making this the last release is for Godot 3.5; so if you're reading this, using Godot 4.x, and they still haven't released updated versions, continue reading. __Otherwise, please follow the official installation instructions.__ I'm writing this guide with the hope and expectation of it becoming obsolete.

### Sites ###

https://github.com/godotengine/godot-ios-plugins

https://docs.godotengine.org/en/stable/tutorials/platform/ios/ios_plugin.html

- - - -

### Additional Steps for 4.3 ###

An issue I encountered was that when trying to build using scons in the following steps, i got the error: `MoltenVK SDK installation directory not found, use 'vulkan_sdk_path' SCons parameter to specify SDK path.` which apparently happens since a [newer version moves the required sdk](https://github.com/godotengine/godot/pull/87305). The error is pretty straightforward but I thought it important to include nevertheless.

1. Download the MoltenVK [release 1.2.6](https://github.com/KhronosGroup/MoltenVK/releases/tag/v1.2.6) from github. Downloading the file `MoltenVK-macos.tar` is enough.
2. Navigate to the downloads and extract the tar.
3. Make a note of the path of the extracted folder. The path of the folder is what we will be using for the compilation process in the next steps (There is another folder `MoltenVK` inside of the top level folder. Don't use this path but rather the most top-level folder called `MoltenVK`)

- - - -

### Normal Steps ###

1. Clone the plugin repository and submodules. This grabs the source code of Godot Engine as well. If you don't need to do that again then clone without `--recursive` around your godot folder or symlink/alias it in.

```bash
git clone --recursive https://github.com/godotengine/godot-ios-plugins.git
```

2. Match the godot repo within to your desired version. As of right now the godot version within is 3.5. Since this guide is intended for 4.3 I imagine you will want to change this. We start by hopping into the godot directory, updating the HEAD with a `fetch` command. After we are certain on the target version, we `checkout` to it. Replace `git checkout 4.3-stable` with your desired version. 

```bash
cd godot
git fetch
git checkout 4.3-stable
```

3. Now we compile the header files, which apparently get compiled first. Run the following command, then once you start seeing [numbers%] blah blah blah, you can cancel the compilation with Control + C. On macOS default terminal, it truly is the Control Key and not the Command Key. I encountered another error here on my first attempt. I got the error: `ERROR: Failed to find SDK path while running xcrun --sdk iphoneos --show-sdk-pat` which I was able to fix by going into xCode->Settings->Locations->Command Line Tools and selecting it in the drop-down menu. This was probably an issue because the development environment was not yet set-up correctly and shouldnt affect most people but I thought to mention it anyways.

```bash
scons platform=ios target=editor
```

4. Next we generate a static `.a` library within the godot folder. Replace `plugin=inappstore` with your desired plugin. It can be `apn, arkit, camera, gamecenter, inappstore,` or `photo_picker`. Also replace version with your desired version. Do not interrupt this process like before. This is where we need to specifiy the location of our SDK which we prepared earlier. replace `<path_to_extracted_moltenVK_2.1.6>` with the path of your extracted release folder, for me this was `~/Downloads/MoltenVK`. This will have to be added for each of the following compilations, and so I have added it to each instruction.

```bash
scons target=editor arch=arm64 simulator=no plugin=inappstore version=4.3 vulkan_sdk_path=<path_to_extracted_moltenVK_2.1.6>
```

5. Leave the godot directory then access the scripts from the repo root. Replace `inappstore` with your desired plugin, but keep `4.0` as is. Currently the script doesn't support any other input.

```bash
cd ..
./scripts/generate_static_library.sh inappstore debug 4.0
```

then

```bash
./scripts/generate_xcframework.sh inappstore debug 4.0
```

6. We now start the process again, but for the other target types. It's awkward because the scons target types in godot expect `editor, template_release, or template_debug`, whereas the provided scripts for making the .a and xcframework libraries expect `debug, release, or release_debug`. I don't think you need to recompile the Godot Engine (Step 3) for each target as well, just the plugin and the two scripts. Your terminal should something like this for compiling the `template_debug`:

```bash
cd godot
scons target=template_debug arch=arm64 simulator=no plugin=inappstore version=4.3 vulkan_sdk_path=<path_to_extracted_moltenVK_2.1.6>
cd ..
./scripts/generate_static_library.sh inappstore release_debug 4.0
./scripts/generate_xcframework.sh inappstore release_debug 4.0
```

7. Repeat the steps for the `template_release`:

```
cd godot 
scons target=template_release arch=arm64 simulator=no plugin=inappstore version=4.3 vulkan_sdk_path=<path_to_extracted_moltenVK_2.1.6>
cd ..
./scripts/generate_static_library.sh inappstore release 4.0
./scripts/generate_xcframework.sh inappstore release 4.0
```

8. Prepare your personal project for the plugins. On the root folder of your project make a folder called `ios` and a folder within that called `plugins`. It should look similar to the file structure of the demo project.
9. Return to the root folder of the godot-ios-plugins repo. Open the `bin` folder and try to ignore all the individual files. Select and copy just the folders to the clip board. The folder name scheme is plugin.target.xcframework.

```
inappstore.debug.xcframework
inappstore.release_debug.xcframework
inappstore.release.xcframework
```

10. Navigate back or up, then go into the `plugins` folder. You should see all the folders for each plugin. Each of these contains a Readme on __How to Use__ each plugin in code, as well as a `.gdip` file and a bunch of other stuff. Hop into the `inappstore` folder or your desired plugin. Then paste in the folders from clipboard.
11. Now select the `.gdip` file, the Readme, and the folders we just copied in and copy those to your clipboard. Return to the `ios/plugins` folder you made in Step 8, make a new folder named for your plugin, then paste within it.

If you encounter "Invalid Plugin Config File plugin.gdip", then you too now share the confusing experience that consumed a large part of my day. Initially when I followed along the official instructions [Here](https://github.com/godotengine/godot-ios-plugins#instructions) I encountered a lot of issues, and had great difficulty finding answers on this error code. Eventually I tried using the Released 3.5 plugins, and the error went away, but then I couldn't export to XCODE with the plugin enabled. Something about it failing to run Ld /blah/blah/blah/blah. What finally worked was the instructions I have provided you now. I at first assumed I would only need the `template_debug` and `release_debug` targets. It wasn't until I went ahead and just compiled all the different targets out of desperation that it finally worked. 

If you encounter an issue when running the scripts that it failed to copy `A` to `B` because `B` exists, you have to delete `B` then try again. This will probably happen if you, trying to get the plugin working, are checking out the godot repo to different versions and going through the steps again.

I am not 100% certain that this works for everyone or that the additional steps are even necessary for others. This is what allowed me to compile the plugin and get everything working
