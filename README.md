# OpenSimWorld NPC Controller

This is a full-featured controller for creating interactive NPCs, scripting them (through notecards) and creating waypoints of your region so that the NPCs can roam around. The controller is lightweight (a single script manages all the NPCs) and the NPCs are interactive (i.e. you can give them commands through the local chat).

You can get the latest OSWNPC package from the OpenSimWorld region in Metropolis, where you can also see them in action:

http://opensimworld.com/hop/74730-OpenSimWorld

(move down near the bunker to find the package)


Please visit https://github.com/opensimworld/active-npcs/edit/master/README.md for an up-to-date version of this documentation.

Please send questions and comments here: http://opensimworld.com/forum/viewtopic.php?id=2



# Contents

The controller contains a number of items:

* The controller script (..Controller). This is the single script that runs everything.
* An example extension script (..Extension). This can be used to add commands to the system (explained below).
* The Listener object. All our NPCs must wear this invisible object on their Right Pec. This object listens for commands from the local chat and forwards them to the controller script for processing.
* The __npc_names notecard. This notecard contains the first names of the NPCs, one in each line. The last name is always "(NPC)" . After making changes to this notecard, select "Reconfig" from the controller menu.
* The __initcommands notecard. This is a notecard with the commands that are executed when you click  "InitCmds" on the controller menu.
* The __waypoints notecard. This contains a list of points in your region. The NPCs can walk from one point to the other, if they are connected (via the __links notecard) . *You do not need to edit this by hand. There is an HUD for editing the map*
* The __links notecard. This contains a list of pairs of connected points. I.e. if you want to link point 1 to point 2, there will be a line "1,2" in the notecard. *You do not need to edit this by hand. There is an HUD for editing the map*
* The __config notecard. This contains configuration ootions.
* Command notecards. You can put a list of commands that the NPC will execute in a notecard named [command-name].scr. You can then order your NPCs to execute them in the chat. E.g. if you create a notecard "dance.scr", you can say "Bob dance" to execute it.
* Waypoint notecards. Notecards named _1.scr , _2.scr etc are executed automatically when the NPCs reach the waypoint number 1, number 2 etc. 
* The appearance notecards are stored as APP_[firstname] for each NPC. You can also have multiple appearances per NPC (see below). 
* The Waypoint HUD is used to edit the map and create/update the __waypoints and __links notecards


# Installation

The controller requires OSSL functions to work. Apart from the osNpc*() functions you should also enable: osListenRegex(), osGetNotecard(), osMessageAttachments(), osSetSpeed().   The controller uses channel 68 for all its functions. 

To start using the controller, rez the OSWNpcController object somewhere in your region. 

Before you create a new NPC, you need to add their name to the __npc_names notecard. Edit the __npc_names notecard, and add the *first* name of your NPC, in a new line by itself. The last name of your NPCs will always be (NPC). 

*After making changes in the notecard,  click on the controller and select "ReConfig"*.

To create an NPC appearance notecard, dress yourself as you want the NPC to look, and  wear the "Listener" object on your RIGHT PEC. The Listener is an object that  listens to the local chat for commands and sends them to the NPC controller for processing. When you are happy with your appearance and certain you are wearing the listener, move within 20 meters near the controller,  click on the controller, select SaveNPC, and then NPC you wish to create.  You should see a message "Appearance saved to APP_xxxx" in a few seconds. This means your appearance has been saved in a notecard inside the controller. 

You can now load your NPC to test them. Click the controller, click LoadNPC, then the name of the NPC.  If all has gone well, your NPC should now respond to commands. Try saying "[name] come" or "[name] follow me"  in the local chat near the NPC to test them. If all is going well, the NPC will respond. If not, remove the NPC and try again.

# Supported commands for NPCs

The NPCs respond to commands that you give to them through the local chat. 

Assuming your NPC is named "Bob", you can say the following in the local chat:
```
  Bob come
  Bob leave
  Bob stop
  Bob follow me
  Bob fly with me
  Bob say hi
  ...
  etc.
```
You can put these commands in notecards in order to create complex scenarios. 

For more complex behavior, the following control commands are supported in  notecards:  if, if-not, jump
 
Example of a scenario notecard:
```
  if name-is Bob Alice
   say hi
   say I am either Bob or Alice
   jump outofhere
  end-if
  say i am not bob or alice
  @outofhere
  if-not name-is Bob
    say I am definitely not Bob
  end-if
  say ... and i 'm now leaving
  leave
```
You can add these commands to a notecard named "test.scr" and then ask the Bob to execute them by saying "Bob test" in the local chat.

In order to add a scenario to a waypoint, create a notecard with the name format:  "_[waypoint-number].scr" and drop it in the controller object.
 
For example the "_10.scr" notecard will be executed at waypoint #10, "_11.scr" at waypoint #11 and so on. Waypoints start at #0. You can find the waypoint number on top of the pegs when editing waypoints (Read below).


# List of NPC commands

These commands must be preceded by the name of the NPC. Here we assume our NPC is called "Bob"
```
  Bob come         = "Come here ". Bob will come move close to you

  Bob moveto 23      : walk towards  waypoint #23

  Bob movetov <23,24,25>  :  walk towards point with coordinates <23,24,25> 
  
  Bob runtov <23,24,25>  :  run towards point with coordinates <23,24,25> 

  Bob flytov <23,24,25>  :  fly towards point <23,24,25> in region
  
  Bob movetovr <23,24,25>  <33,34,25>   : walk to a random point between the points   <23,24,25>  <33,34,25>  
  
  Bob runtovr <23,24,25>  <33,34,25>  : same as above, but run

  ** Note: never leave spaces in coordinate vectors, i.e. <23,24,25> NOT <23, 24, 25> **
```

## Sit command

The NPC can sit on objects. The way it works is as follows:
```
Bob use chair         
```
Bob will attempt to find an object named "chair" (Object Name) near him and try to sit on it if its transparency (alpha) is less than 100%. Since by convention poseballs turn transparent when users sit on them, this ensures that Bob will not sit on an already-occupied poseball.

```
Bob stand             : Bob will stand up if he is sitting
```
## Variables
Variables can be used with IF commands for more complex scenarios. Variables are global and can be shared between notecards and NPCs. 
```
Bob setvar foo 13                : set variable foo to be "13". Only string variables are supported. Variables can be used with IF blocks
Bob setvar foo                   :  set variable foo to the empty string. The empty string is the default value if a variable does not exist
Bob say $foo        : Bob says "13"
```

## Flow control with IF commands
There is support for multiple levels of IF blocks. blocks end with "end-if". There is no "else" command, but you can achieve the same effect with "jump" commands

```
if name-is bob alice            : if the npc's name is Bob or alice
      say I am either Bob or Alice
end-if                          : always end IF blocks with end-if.  You  can nest if blocks

if-not name-is Bob              : Example of negative if
  say I am not Bob
end-if
  

if-prob 0.3                     : IF block will be executed with probabilty 0.3 (the if block will be executed 30% of the time)
  say This will happen 30% of the times
end-if

if var-is party 1               : Will execute the IF block if the variable party is "1"
  say We are having a party already!
end-if

if var-is party               : Will execute the IF block if the variable party is empty ("")
  say No party yet, let's start one!
  setvar party 1
end-if
```

Jump command.  You can use the syntax @label to create labels in your notecards. The syntax is:
```
@myLabel       : a label
  say This will loop forever
jump myLabel   :  like "jump" in LSL or "goto" in other languages. the label should be on a line by itself prefixed with '@' 

```


## Useful script commands
```
wait 200           : wait 200 seconds

wait 200  300      : wait between 200 and 300 second before executing the next command

waitvar foo 13     : wait until the variable foo gets the value 13

waitvar foo        : wait until the variable foo is empty.
```


## Pathfinding commands

The NPCs can find the paths between connected waypoints and walk from point A to point B. Because this is computationally intensive, only paths with less than 10 waypoints between them are supported. The following pathfinding commands are supported:

```
Bob go to Bar   : this uses pathfinding  to go to the waypoint with the name "Bar"
Bob go to       : without  an argument, bob will print the names of waypoints he knows
Bob goto 13     : go to waypoint #13 
```

## Other commands
```
Bob say hi                            : Says "hi" on public channel 
Bob saych 90 blablah                  : say "blablah" on channel 90
Bob shout Blah bleh
Bob teleport Bar                            : Teleports bob to the waypoint named "Bar"
Bob teleport <23,30,40>                     : Teleports to a point. REMEMBER to never leave spaces inside the vector string
```

```
Bob msgatt  attachment_command 12 13 14 15  
```
Uses osMessageAttachments to send the message "attachment_command" to attachments at attach points 12 13 14 15. 
This can be useful for scripting NPC attachments. Read the OSSL docs of osMessageAttachments() for more. 

```
Bob lookat me            : attempts to look at you 

Bob lookat <x,y,z>       : look towards point x,y,z

Bob lookat Bar           : look towards the waypoint named "Bar"

Bob anim dance           : play animation "dance" . the animation must be in the inventory of the controller object

Bob sound 1c8a3af2-6e5a-4807-a7a3-a42e5744217c 1.0   : The NPC will play the sound with the given UUID  at the volume specified by the second parameter (1.0 is max volume)

Bob light               :  turn on/off a light the NPCs have on them

Bob follow me

Bob follow  Alice       : follow the avatar whose first name is 'Alice'

Bob fly with me

Bob fly with alice      : fly with another user

Bob stop                : Stops his animation and his movement, and stops following you

Bob leave               : Start wandering  between waypoints

Bob setpath 0:1:3:5:1:0            : Walk through the given waypoints, i.e. walk to waypoint 0 , then to 1, then to 3, then to 5 etc. 

Bob run-notecard my_script.scr     : Execute the contents of the notecard my_script.scr (the notecard must be in the controller inventory

Bob batch say hi ; wait 10; say bye     : Executes multiple commands one after the other. Commands are separated by ";"

Bob stop-script         : stop executing the notecard script

Bob dress               : load the appearance from notecard "APP_bob" (The default appearance notecard)

Bob dress  swimming     : load the appearance from notecard "APP_bob_swimming"
```
You can have multiple appearance notecards per NPC, just rename their appearance notecard from "APP_bob" to "APP_bob_dressName" 


## Custom command notecards

You can create your own commands with a notecard. To create a command notecard, enter the commands for the NPC to execute in a notecard, and save it into the controller's inventory with the name [command-name].scr

E.g. a dance command notecard would be named "dance.scr" and contain the following
```
say I love to dance!
anim RockDance
wait 600 800
say oops, I am  tired...
stop
```
You can then say "Bob dance" to have bob execute the notecard


# Creating waypoints

You can use the controller to create waypoints and links between them in your region. 

Each waypoint has a number (starting from 0) and optionally a name. Waypoints can be connected to other waypoints, thus creating a map (graph) between them. The NPC can then wander between the waypoints (Use "Bob leave" to make them start wandering), and they can also go from waypoint A to waypoint B (if there are appropriate links between the two).

To create a map, wear the WaypointEditor HUD.  If you already have a map, click RezPegs from the HUD menu. This will rez  a peg in each waypoint you have already created.  If you are starting with an empty map, you do not need to do this!

To add a waypoint, move your avatar to the desired position, and select "AddPeg" from the HUD menu. A peg should appear in front of you. To create a second waypoint, move to another position, and repeat. 

To link the two waypoints X and Y, click on the first one (X), then click on the second one (Y). The pop up dialog should say "Current peg: Y, previous: X". To link them, select "LinkPegs" from the popup dialog. To unlink two waypoints, again, click on the first one, then click on the second one and select "UnlinkPegs" from the popup. 


You can give a name to waypoints. Click the peg, select "SetName", and enter the name (E.g. "Bar"). You can use this name to give commands to your NPCs later (e.g. "Bob go to bar").

You can move the waypoints to correct their positions. *Important: After you move pegs to new positions click "ScanPegs" from the editor HUD. If you don't do this your changes will be lost.

To save the changes you have made to the map, click the editor HUD and select "SaveCards". This will update the __waypoints and the __links notecards to match your new map.  To test that all went well, clear the pegs and rez them again. 

Note! Removing waypoints is not supported, as it would break the numbering and would mess up the notecard naming scheme. Although deleting is not supported, you can unlink a peg from all other pegs, in which case the NPCs will never go there when wandering. If you want to start over with an empty map, edit the __waypoints and __links notecards, remove all their contents, and then select "Reconflg" from the controller. 


## Configuration
Configurable options are added in the __config  notecard:
```
AutoLoadOnReset=0
```
Change AutoLoadOnReset=0 to AutoLoadOnReset=1 to make the controller rez the NPCs on region restart or script reset. Please note that autoloading may not always work well. 

The notecard __initcommands contains initial commands to give to the NPCs. These commands are executed automatically after auto-rezzing the NPCs when the controller is configured to rez on reset, or manually by clicking "InitCmds" from the controller menu.  You can use  __initcommands  to add commands that set your NPCs in motion. Typically you would tell them to "leave" so that they start  wandering. An example of __initcommands is : 
```
Bob say I am rezzed!
Bob teleport <64,64,22>
Alice  teleport <128,64,22>
Bob leave
Alice leave
```

## Extensions

You can create your own NPC commands with extension scripts.  Extensions are scripts that are placed inside the controller object.  Commands that are not processed by the NPC controller are sent via link_message to the extension scripts for processing. The extensions can also use link_message (with number parameter >=0) to send back commands to the NPC controller. The default NPC controller object already contains an extension that implements the "help" command (the "..Extension" script). The script in it shows how extensions parse the data sent from the controller (through link_message)  and how they can respond.

The string sent through link_message (or through channel 68) is of the format:
```
! 0000 UUUU Bob help
```
i.e. commands  that you would normally give to the NPC through the chat are prefixed with "! 0000 UUUU "  and sent to channel 68. For commands such as "Bob follow me" which require knowing which avatar sent the command, "0000"  must be replaced with the uuid if the avatar giving the command. 

The 'key'  parameter of link_message is the uuid of the NPC. Look into the ..Extension script for more.

## Sending commands from other objects

You can send commands directly to the NPC controller from any object. The NPC controller listens on channel 68. Remember to send the command using llRegionSay region-wide, as your object will probably be far away from the controller:
```
llRegionSay(68, "! 0000 UUUU Bob say hello");
```

Another way to interact with the controller is the SETVAR command. You can set or change a variable in the controller by 
sending SETVAR to channel 68:

```
llRegionSay(68,  "SETVAR foo 1");
```
(Note we use  SETVAR in capitals. This will set the variable "foo" to "1" . You can then use this variable in an notecard. In this example, the NPC waits for the variable "foo" before dancing:
```
@start
say Waiting for variable ...
waitvar foo 1
say Variable foo is now 1! Let's do something interesting!
anim dance1
wait 20
stop
setvar foo 0
jump start
```


## Extras

The controller checks the number of visitors in your region every 2 minutes. If there are no visitors in the region it will stop executing commands thus avoiding unnecessary load to the region. 

In addition, the controller keeps track of the  latest visitors, giving you a log of the latest visitors:
```
Bob seen all    : will print everyone who visited
Bob seen foo    : will print information about foo
```



# Technical Notes

The controller runs through a single timer, that updates the states of the NPCs every 5 seconds. This allows it to be extremely lightweight, as it does not block processing, but also causes small delays between commands. 


