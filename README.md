# A Corrupted Adventure
An RE CTF challenge based on the data structures of Gen 3 Pokémon save files

# Challenge:
```
Our trainer, Carrot, has has accidentally corrupted his entire Pokémon Emerald adventure!
He managed to get it exported into a save dump though. 
Can you help him find his beloved starter?  He knows it was definitely in his party when it was corrupted and was sound asleep.  
```
### Flag:  Nickname of Carrot's Starter Pokémon (It's Nickname, not the species name)


######
######



# WARNING: Spoilers Below 



# Walkthrough:

### Useful information to know:
- Pokémon Emerald is a game from Generation 3 of the Pokémon series of games.  

- Emerald save files are primarily in hex with a little bit of binary.

- Due to the extensive modding community around the game, this data structure has been entirely reverse-engineered and very well documented. The primary resource I use is: https://bulbapedia.bulbagarden.net/wiki/Save_data_structure_(Generation_III)

- A "starter" Pokémon is one given to the player at the beginning of the game.  In most save files, there can only be one starter Pokémon.  In Emerald, these are: Treeko, Grovyle, Sceptile, Torchic, Combusken, Blaziken, Mudkip, Marshtomp, and Swampert.

- The "party" is the (up to) six Pokémon that a player has with them as they play the game and run around the map.  

- Pokémon can be affected by a "status condition" such as "asleep", "poisoned", "burned", etc. that affects gameplay.

### Steps to complete:
- First thing's first, the file needs to be opened in some form of a hex viewer.  This could be as simple as `xxd | less` or an actual tool.

With the info about this data structure, there are a few ways to go about finding this Pokémon.  You could:
- Calculate the offset of Carrot's party (which, according to the challenge description, is where the Pokémon is),then look through the party for either the species of each Pokémon in the party, or the status condition (odds are only one is asleep) to find it. 
- Look up the data values for each starter and search through the file in whatever tool you're using to find the starter.  (Part of this datais encrypted, and while definitely possible to decrypt as all the data needed is in the file, it is really involved.)
- Find the trainer's ID from the save file and search for that through the file to find all Pokémon belonging to Carrot and check the status condition of each (asleep).

Personally, I like a combination of a few methods.

- To find the party, I searched through the file for Carrot's name in hex.
- Initially, this results in nothing found.  This is because Emerald uses a proprietary character encoding format, which can be found here: https://bulbapedia.bulbagarden.net/wiki/Character_encoding_(Generation_III)
- Converting "Carrot" to this format results in: `BD D5 E7 E7 E3 E8`
- Searching for this comes back with a lot of results.  
- As I tabbed through the results, I noticed that six entries were grouped close together, with a large offset between the next time that hex value shows up. This must be the party.
- I looked up the Pokémon data structure for each individual Pokémon at: https://bulbapedia.bulbagarden.net/wiki/Pok%C3%A9mon_data_structure_(Generation_III)to find that the Trainer's Name is located at an offset of 20 bytes.  
- Another data in each Pokémon's structure is their status condition.  Since Iknow that the Pokémon in question is asleep, I can look at this value for each entry in the party.

Starting with the first one, which is:
```
00001230            :           |69 2C 15 74:40 F4 32 46
00001240 CA BF C6 C3:CA CA BF CC|FF 00 02 02:BD D5 E6 E6
00001250 E3 E8 FF 00:16 1C 00 00|1F D9 27 32:C3 A8 27 32
00001260 29 B7 27 32:38 D8 34 32|10 D8 91 32:0A D7 28 38
00001270 29 FF BC 2B:F5 76 15 12|29 D8 27 32:28 DF 24 35
00001280 2D DB 27 32:29 D8 27 32|00 00 00 00:1E FF 54 00
00001290 54 00 29 00
```
- I know that the status condition is at an offset of 80.  So I went to position 80 and find the value `00`.  Looking this up on the status conditions table indicates no condition present.

- I took the next 100 bytes and looked again.  No luck.  I tried each Pokémon in the party until... AHA!  I found one with the status condition value `02`!  
```
000013C0            :           |92 36 6E EC:40 F4 32 46
000013D0 D7 E8 DA C2:A4 D2 CE A2|C7 A4 02 02:BD D5 E6 E6
000013E0 E3 E8 FF 00:00 00 00 00|00 00 00 00:00 00 00 00
000013F0 00 00 00 00:00 00 00 00|00 00 00 00:00 00 00 00
00001400 00 00 00 00:00 00 00 00|00 00 00 00:00 00 00 00
00001410 00 00 00 00:00 00 00 00|02 00 00 00:00 FF 7E 00
00001420 7E 00 76 00:50 00 55 00|73 00 4C 00
```
- The hex value `02` isn't on the table, so we have to convert it to binary, which gives us: `00000010`  The table indicates that the second bit is the "asleep" bit!  This means that we found the only sleeping Pokémon in the party!  The name of the Pokémon is at offset 8 with a length of 10, which means it's name is...

Drumroll please!

`D7 E8 DA C2 A4 D2 CE A2 C7 A4 !!!`

Wait, that doesn't mean anything...  In ASCII?

`×èÚÂ¤ÒÎ¢Ç¤`

Oh that's even worse...

WAIT!  Pokémon Emerald uses that other encoding method! If we plug these hex values into that, we arrive at...

`ctfH3XT1M3` !!!

And man, we certainly have had some time with hex! 

Thanks for playing!

