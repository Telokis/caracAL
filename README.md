# caracAL

A Node.js client for [adventure.land](https://adventure.land/ "Adventure Land")

### Changelog for 2022-09-27

Fixes for storage.json sometimes being completely empty.
If caracAL worked for you and you werent getting `Unexpected end of JSON input` you dont need to update.

### Changelog for 2022-09-08

Minor bugfixes with a larger impact.
smart_move should now be usable again.
Also special characters are now respected in login process

## IMPORTANT! Changes from 2021-10-17

This update finally introduces localStorage and sessionStorage. However, in order to provide these technologies we need to bring in additional dependencies. Follow the steps below to upgrade to the most recent version. This version renames the default scripts. The default code file `example.js` now resides in `caracAL/examples/crabs.js`. If you were running the default crab script please edit or regenerate your config file in order to match the changed filename. 

### Upgrading from a git installation

If you have installed caracAL from cloning this repository you can upgrade by entering the following commands into a terminal:

```bash
git pull
npm install
```

### Simpler version

Simply reinstall caracAL by following the steps below. You can keep the config.js file from your old installation for ease of use.

## Installation on Debian/Ubuntu

```bash
#update packages
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install git curl
#install node version manager(nvm)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash
#you need to restart your terminal here
#so the nvm command can be recognized
#install node 14
#latest(16) does not like socket.io for some reason.
nvm install 14
#download caracAL
git clone https://github.com/numbereself/caracAL.git
#switch to directory
cd caracAL
#use npm to download dependencies
npm install
#if you want caracAL to autostart run
#chmod +x ./start_on_boot.sh
#./start_on_boot.sh
#run caracAL
node main.js
```

## Installation on Windows 10

First download Node.js version 14 and npm from

[https://nodejs.org/en/download/](https://nodejs.org/en/download/ "Node.js homepage")

Run the installer you just downloaded. MAKE SURE THAT YOU ENABLE THE "Add to PATH" OPTION DURING INSTALLATION. This option will allow you to refer to the Node.js and npm binarys with a shorthand.

Next hit WINDOWS+R and type "powershell" in the window that opens, and hit enter. You should now be presented with the windows powershell. Copy and paste the following script

```
#download caracAL
wget https://github.com/numbereself/caracAL/archive/refs/heads/main.zip -OutFile caracAL.zip
#unzip archive
tar -xf caracAL.zip caracAL-main
#rename output
ren "caracAL-main" "caracAL"
#switch to directory
cd caracAL
#use npm to download dependencies
npm install
#run caracAL
node main.js
```

## First run and configuration

When you first run caracAL it will ask you for your login credentials to adventure.land. After you have sucessfully entered them it will ask you which characters to start and in which realm. Finally it will ask you if you want to use the bot monitoring panel, and if you answer that with yes also if you want a minimap and what port you want it to be on. Once these questions are answered caracAL will generate a file config.js for you, containing the information you just entered. The characters you specified will immediately be loaded into caracAL and start farming crabs using the script example.js.

### config.js

The session key represents you login into adventure.land. This is the reason why you should not share this file.

caracAL stores versions of the game client on your disk, in the game_files folder.
If set, the cull_versions key makes caracAL delete these versions, except the two most recent ones.
Versions which are not numeric will not be considered for culling.

The web_app section contains configuration that enables caracAL to host a webserver. The port option herein allows to choose which port the webserver should be hosted on.
The enable_bwi option opens a monitoring panel that displays the status of the characters running within caracAL if set to true.
The enable_minimap option configures if caracAL should generate a minimap summarizing your current game state. The minimap is located in the monitoring panel. Therefore, if the minimap is enabled, the monitoring panel will always be served and ignore the previous setting.
The expose_CODE option shares the CODE directory, where your scripts are located, via the webserver. This is useful if you i.e. want to load the scripts you are using in caracAL from the steam client. Scripts shared in this manner will be available i.e. under the URL `localhost:924/CODE/caracAL/tests/cm_test.js`.
If you do not enable either of these options no webserver will be opened. config.js files which do not have the web_app section, i.e. those, which were created before the update, will not open a webserver either.

The characters key contains information about which characters to run. 
Each character has four fields:
 - realm: Which server the character should run on
 - script: Which script the character should run. Scripts are located in the CODE folder.
 - enabled: caracAL will only run characters who have this field set to true
 - version: Which version of the game client this character should run. A value of 0 represents the latest version

### Running your own code
The default code located at `./CODE/example.js`, as specified by the character property `script`. This script makes your characters farm tiny crabs on the beach.
You can create your own scripts in the `./CODE/` directory. Since caracAL runs the same files as the game client you should be able to use the exact same files in caracAL.

## So why should I use caracAL?

There is one big reason and that is

### Parity with the regular game client

caracAL runs the same files as the regular client. You can use the same scripts in caracAL and in the normal client. This means that you can develop your scripts in the game and later deploy them with caracAL. This is the key selling point over competitors like ALclient, who develops a completely new client entirely and ALbot, which has only recently switched to an architecture quite similar to caracAL. 

Keep in mind that some functionality does not make sense in a headless client. This notably concerns the lack of a HTML document and a renderer.
Most HTML routines do not throw an error, but do not expect them to do anything. Calls to PIXI routines should really be mostly avoided.

### It is fast

Compiling the game sources directly into Node.js V8 engine yields performance which is actually serviceable for smaller devices.
The CLI aimed to do the same thing, but it ended up poorly emulating a webpage, which led to atrocious performance.

## Additional features

Some functionality is available through browser-based apis. Notably character deployment and script loading is realized through `<iframe>`, `window.location` and `<script>`. In order to make this functionality available in caracAL we provide the extension `parent.caracAL`. Check out some basic usage here:
```javascript
//check if we are running in caracAL
if(parent.caracAL) {
  if(we_want_to_run_another_char) {
      //runs other char with script farm_snakes.js in current realm
      parent.caracAL.deploy(another_char_name,null,"farm_snakes.js");
  }
  if(we_want_to_switch_this_chars_server) {
      //runs current char with current script in US 2
      parent.caracAL.deploy(null,"USII");
  }
  if(we_want_to_shut_this_char_down) {
      //shuts down current character
      parent.caracAL.shutdown();
  }
  if(we_want_to_know_which_chars_run_in_caracAL) {
      console.log(parent.caracAL.siblings);
  }
  if(we_want_to_load_an_additional_script) {
      //get a promise for loading the script ./CODE/bonus_script.js
      parent.caracAL.load_scripts(["bonus_script.js"])
        .then(()=>console.log("the new script is loaded"));
  }
}
```

code messages(cms) sent from one character to another within the same instance of caracAL are near-instant. cms sent in this manner will have the `caracAL` property set to `true`. This was a lesser known feature in ALbot, which some players used to better coordinate the farming of certain low-hp mobs or in other words ALL HAIL THE CRAB GOD!!

caracAL logs to the file caracAL.log. It also rotates this log when it gets larger than 1.5mb and keeps 3 of these logs. This is helpful when trying to narrow down what your bots did while deployed with caracAL.

The information available in `parent.X` is needed by the coordination part of caracAL. As a result this information is not obtained by each character individually, but for all characters centrally. A nice side effect of this is, that the information in `parent.X` is updated more often than it would be in a normal client.

Characters which disconnect or fail to connect in the first place will be automatically reloaded. If you want to prevent this behaviour call `parent.caracAL.shutdown()` within your code.

If you are not comfortable storing a secret like your auth key plaintext in a file caracAL can read it from the process environment variables instead. Just set the environment var `AL_SESSION` to your session key and caracAL will use that instead.

## localStorage and sessionStorage

Web Storage technology is pretty handy when you want to have data that persists through character reloads. For the longest time, there was no really good possibility to realize these technologys in Node.js. With version 0.2.0 however, caracAL added support for them. `window.localStorage`, as well as `window.sessionStorage` are provided, and behave very much like their web counterparts. This subsequently means that the runner functions `get(key)`, as well as `set(key,value)` also work.

### Technical Details

The implementations are custom made specifically for caracAL. The key value stores are kept in memory, with localStorage additionally being written to file.
Keep in mind that every character process, as well as the coordinator process keeps a copy of localStorage and sessionStorage, so dont put too large data in there.
Writing operations on these stores are comunicated to your characters via IPC messages, and these messages are sent to every character.

## No-Nonsense rewrite the game

Imagine there is an update but there is a null check missing from the client. Actually dont imagine that, just look at the gif:

![funny gif](https://cdn.discordapp.com/attachments/238332476743745536/737426132990820402/cantattack.gif "The day the earth stood still")

Seems like a use case to rewrite the game client. But, that is not so easy, as browsers go great lengths to make sure that the files they run are only from the domain that provides them.

caracAL downloads the game client files to disk. Once they are stored in the game_files directory you can simply edit them.

By default caracAL uses a version culling mechanism. If you want to preserve versions that you have made many edits to you can turn the mechanism off in the config, but there is a better way.

You can rename your custom version with many edits to a name that is not numeric. If you do that then caracAL will never delete it. Now you just have edit your config.js to make your bots actually use the version you renamed in this manner. The version will thus be preserved and your bots will continue running it.

## Bot Monitoring Panel

In the most recent version caracAL added the ability to check up on your bots through a web interface. It can be acessed with a browser, such as firefox or chrome. You need to enable this feature through the config.js file, the details of which are also in this document. If you run caracAL on the same machine as your browser and on the default port you can access the panel under the url `http://localhost:924/`.
It looks somewhat like this:
![BWI Image](https://github.com/numbereself/caracAL/blob/main/presentation/bwi.png?raw=true)

Some people might know this look from ALBot. It is actually the same technology at work. It also comes bundled with a handy minimap, also available in ALBot. Your player character is represented by a light blue dot, other players by dark blue dots, neutral monsters by brown dots and monsters targeting your character are visible as red dots, while walls are grey. The monster your character is currently targeting is additionally marked with a larger red circle.

The panel also does not yet have options to control your bots. Such a feature is not yet planned, but once I have a good idea of how it could look, I might just work on implementing it.

Lastly, if you want to take a look at the game from a spectator standpoint or control your bots by sending commands to them there is always [the Comm Panel](https://adventure.land/comm?scale=1 "The Comm Panel").

An improved version is already in the making.

## Contributing

If you have an idea to improve caracAL that is great!

You can use the issue tracker provided by github to tell me.

Or, even better, you can make a pull request to this repository and submit your changes yourself.

You can get in touch with number_e through [adventure.land on Discord](https://discord.gg/h472pWP9 "Discord Server"). Even if you dont specifically want to help with caracAL you should join the server, there are lots of super nice people there.