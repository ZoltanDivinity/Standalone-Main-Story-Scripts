#ZOLTON'S DIVINITY ENGINE STORY EDITOR TOOLS
///////////////////      Version 0.6.0                 /////////////////
/*
TODO
-Game Plan version 0.8.0<
-clean and sort dialogs
-Organize and format!
-double check/ test
completing above achieves Version 1.0.0
-Give definitions for calls and querys
Rearrange/Combine:
Dialogs, _GLOBAL_Dialogs(Rearange!), 
create template/guide:
Compagion/Henchmen, standard quests, shopOwners?, NPCs 

Game Plan
Big Files
Thievery NPC Action Triggers, Thievery OBJECT triggers, Thievery OBJECT_CLASS triggers, Vandalism Triggers
General Look
Test_Bert, GLOBAL_DLC, _GLOBAL_ItemRotationPuzzles
*/


/////Template///////////
/*   Notes

*/
/****Invovled Databases****/

/****Useful Procedures****/

/*   Define Keywords

*/

//////////////   TABLE OF CONTENTS  ////////
/*
---Characters
Move Character
COMBAT

---NPCs
Story NPC
NPC stats
Removing NPC
Trade
Shop Owners
Ownerships
Pets
NPC Groups

---Player-NPC Relationships
Attitude
Relations
Reputation

---Dialogs
On Click Dialogs
Companion Dialog
Threatened
Hostile
Special: Animal Food
Stroy Dialog
Mandatory Dialog
One Shot Dialog
Warning Dialogs
Character Events
Generic Dialogs
Party Dialog

---Companions And Henchmen
Companions and Henchmen

---For Players
Waypoints
Persuasion
Traits
Player Comments
Affection Between Players
Journal & Discription
Tutorial Messages

---Items
General
Doors
Hidden Walls
Shoveling Stuff

---Areas
Triggers
Exploration Bonuses
SubRegion
Ambush
Forbindden Areas
CIRDialog

---Ilegals
Illegal Poop
Sneaking
Guards

---General
Time
Autosave
Generic Quest Reward
Effects
Freezing characters
Object Timers
Counters
Stack
Book Keeping
Useless Stuff
*/

//------------------------Characters------------------------//
/*
This part deals with some very general Character influence scripts
Such as story influenced safe movement and attacking
*/
/////Move Character///////////
/*   Notes
This Section is about moving a Character to somewhere, storing and restoring their normal behaviors
They will also Stop if attacked and resume when they leave combat. Upon death the events and databases will be cleaned up
Larian did good work with this one
*/
/****Invovled Databases****/
/****Useful Procedures****/
ProcFaceCharacter((CHARACTER)_Char,(CHARACTER)_Target); //_Char faces _Target
ProcFaceEachother((CHARACTER)_Char,(CHARACTER)_Target); //Characters face eachother

ProcCharacterMoveToTrigger((CHARACTER)_Char,(TRIGGER)_Point,(INTEGER)_Running,(STRING)_Event);
ProcCharacterMoveToItem((CHARACTER)_Char,(ITEM)_Point,(INTEGER)_Running,(STRING)_Event);
/*
(CHARACTER)_Char 		: Move this Character
(TRIGGER)_Point 		: Move to this Trigger
(ITEM)_Point 			: Move to this item
(INTEGER)_Running 		: is running? 0->False 1->True
(STRING)_Event 			: I believe event upon completion (goes into call CharacterMoveToTrigger)
*/

/////COMBAT///////////
/*   Notes
Combat is mostly generated when adversarial Characters spot eachother, or when a player attacks an NPC
You can change an NPC hostility with certain procedures. 
dbCombat contains all NPCs in combat, see 'Book Keeping' section for more detail
*/
/****Invovled Databases****/
/****Useful Procedures****/
ProcMakeNPCHostile((CHARACTER)_Player,(CHARACTER)_Npc); //NPC is hostile towards player
Attack((CHARACTER)_Npc,(CHARACTER)_Player); //NPC attacks player, mostly used internally

/*   Define Keywords
(CHARACTER)_Character 	: Character in combat (generated)
(INTERGER)_ID 			: Combat interger ID (generated)
*/
/* Edits
I removed the PROC Attack that EnableDefaultBehavior, replaced by DisableDefaultBehavior generally
*/

//---------------------------NPCs---------------------------//
/*
This Part is about handling and keeping track of NPCs
*/
/////Story NPC///////////
/*   Notes

*/
/****Invovled Databases****/
IsStoryNpc((CHARACTER)_Npc);  	//NPC involved in story (doesn't attack? automatic?)

/****Useful Procedures****/
SetStoryNpc((CHARACTER)_Npc[,(INTEGER)_State]); //Set a NPC to story (State=1, default) or not (State=0), changes IsStoryNPC
SetStoryNpcStatus((CHARACTER)_Npc);			//Depending on if character is in IsStoryNpc database sets as a story NPC 

/*   Define Keywords

*/

/////NPC stats///////////
/*   Notes
This Section deals with general NPCs and their actions towards the player
*/
/****Invovled Databases****/
NoDefaultBehavior((CHARACTER)_NPC);
DB_CustomAttackDialog((CHARACTER)_Npc,(STRING)_Dialog); 
Unintelligent((CHARACTER)_Npc); //Database of NPCs that don't react to much
NoStealingReaction((CHARACTER)_Npc); //NPCs that just don't care if an item is stolen

EvilDude((CHARACTER)_Npc);  	//Pretty much makes a NPC evil, is checked by many things (can't trade, changes default dialogs, relations, ...)
Amoral((CHARACTER)_Npc); 		//Characters in EvilDude are Amoral unless they are in the Moral Database (has influence on theft)
MoralNpc((CHARACTER)_Npc);		//Not used in main game but its there 
IsNotMessingAround((CHARACTER)_Npc);	//Won't stop and chat if you attack them

CharacterStats((CHARACTER)_npc,(INTEGER)_tolerance_thievery,(INTEGER)_value_cheap,(INTEGER)_value_expensive);
//Generated at game start of game, but can be set in scripting
IsRich((CHARACTER)_Npc); //Influences CharacterStats, not actually used in main game
IsPoor((CHARACTER)_Npc); //Influences CharacterStats, not actually used in main game

/****Useful Procedures****/
SetCharacterStats(_Npc,_tolerance_thievery,_value_cheap,_value_expensive);
DecreaseToleranceThievery((CHARACTER)_Npc,(INTEGER)_I);
DoRevenge((CHARACTER)_Player,(CHARACTER)_Npc,(INTEGER)_Reason); //Mostly Internal, can be used to start a conflict (use reason 5)
//The general idea, this is the same thing as a reaction to criminal players
//If _Npc dares, _Npc will attack player, calling friends (not implemeted).
//If _Npc dares not to attack himself, _Npc will call for help (guards).
//If no help available and npc does not dare to attack himself, he runs off. (might say a few choice words)

/*   Define Keywords
(INTEGER)_State 	: boolean 1->make story, 0->make not sotry. Default to 1
(INTEGER)_Tolerance_Thievery: -100: (lawful good) even theft of the most insignificant thing has high consequences
							   0: (neutral) a cheap item: don't talk about it, a famous or expensive thing or something of my own...
						   +100: (chaotic evil) if you can steal from somebody. You are smart, the other one is stupid!
		Important note: Tolerance_Thievery is NEVER applied when it concerns my own items
		(but ONLY if MY OWN items. My wife's items is something different.)
(INTEGER)_Value_Cheap: I consider items with this price or lower as "cheap"
(INTEGER)_Value_Expensive: I consider items with this price or lower as "expensive"
Notes:
(INTEGER)_Value_Cheap: if you steal items that are "cheap", you might get away with it
(INTEGER)_Value_Expensive: if you offer goods or trade a LOT, you gain 5 attitude points
	for every time the Npc has a net trade result with you = _Value_Expensive
	Example: Otho's _Value_Expensive == 1000. I trade with objects having
			real value 3000 but I only charge 2000, Otho will gain 1000 on it,
			so I get 5 attitude points extra. Given the default relative trade
			factors, any npc you visit frequently to trade with will get your
			best friend sooner or later.
Thanks Larian for some documentaton*/

/////Removing NPC///////////
/*   Notes
Removes and places NPC

*/
/****Invovled Databases****/
RemoveNpc((CHARACTER)_Npc); 			//Removes an NPC if the NPC is not seen by players
RemoveNpc((CHARACTER)_Player,(CHARACTER)_Npc,(TRIGGER)_Region); //removes an NPC if player(s) are not in trigger region (maybe only one player)
//I would suggest against using this proc)

/****Useful Procedures****/
DoRemoveNpc((CHARACTER)_Npc); //
SetNpcAtLocation((CHARACTER)_Npc,(TRIGGER)_Location);		//Sets NPC on stage and teleports to that location
NpcRemoved((CHARACTER)_Npc); //you may define this proc so that when a specific NPC is removed, something happens (only works with [Do]RemoveNpc)

Poof((CHARACTER)_Character); //Disappears character in a puff of smoke
Foop((CHARACTER)_Character); //Character Appears in a puff of smoke

/*****Character Event*****/
// Character event "NPC_TeleportedAway" removes character from stage

/*   Define Keywords
(INTEGER)_Teleport 			: 1->Teleport Animation, 0->no animation
(TRIGGER)_Location			: Teleports to this location
*/

/////Trade///////////
/*   Notes
Trade items are generated based on player's level, if player levels up, items regenerate
Regenartes Items every 48 hours in world. According to Time(_,_,_)
You can't trade with a character with attitude >= -45 towards player
*/
/****Invovled Databases****/
DBMarketStands((STRING)_Region,(TRIGGER)_StallLocation,(CHARACTER)_NPC); //Sets up a communal Market in _Region (ex: Cyseal stalls)
Trader(_Npc); //On game start everyone that isn't in character group "animal" or the databases EvilDude or NoInitialTrade
IsTrader(_Npc); //Generated database 
/* From Larian
Assertion of the fact CannotTrade(_Npc) prevents startup of the trade window.
This prevents the npc from interacting via the trade window, hence from repairing, healing or identifying as well.
By default, CannotTrade(_Npc) is not asserted, so trade is possible.
*/
 DB_CustomTradeTreasure((CHARACTER)_Npc,(STRING)_Treasure);

/****Useful Procedures****/
ProcDisablePlayerTrade((CHARACTER)_Npc); //Disables a NPCs trading
ProcEnablePlayerTrade((CHARACTER)_Npc);  //Re-enabled a characters trading
ProcClearTradeFacts((CHARACTER)_Trader); //

/*   Define Keywords

*/

/////Shop Owners///////////
/*   Notes
This sets up standard warnings and threatened message for shopkeepers
*/
/****Invovled Databases****/
ShopRegion((STRING)_Level,(TRIGGER)_Region,(CHARACTER)_ShopKeeper,(STRING)_?);
ItemOwnerShipTriggers((STRING)_Level,(TRIGGER)_Region,(CHARACTER)_ShopKeeper)

/****Useful Procedures****/


/****Special*****/
SetupShopTriggers();
//Put your initilizing of ShopRegion in this procedure or they will execute before the engine is ready, I put a null shop in there to not be empty

/*   Define Keywords
(STRING)_Level 			: Name of level shop is on (ex:"Cyseal")
(TRIGGER)_Region 		: Trigger region of Shop
(CHARACTER)_ShopKeeper 	: Shopkepper
(STRING)_? 				: Unused
*/

/////Ownerships///////////
/*   Notes
Helps with ownership of items and areas
*/
/****Invovled Databases****/
ItemOwnerShipTriggers((STRING)_Region,(TRIGGER)_Trigger,(CHARACTER)_Owner); //Trigger area is owned by _Owner, 
//Changing ItemOwnerShipTriggers database will update all ownerships
ItemOwnerShipClearItem((STRING)_Region,(ITEM)_Item); 		//Ensures Item is not owned by anyone

/****Useful Procedures****/

/****Character Varariables****/
/*
"InOwnedArea" Interger/boolean 1->True, 0->Flase
"OwnedAreaOwner" Character that owns area
*/

/*   Define Keywords

*/

/////Pets///////////
/*   Notes
A small section about NPCs that are pets, Generally Unintelligent and loved by their owner
*/
/****Invovled Databases****/
DB_Pets((CHARACTER)_Npc,(CHARACTER)_Owner); //The NPC are owned by an owner

/****Useful Procedures****/

/****Default Dialog****/
/*
If you attack a pet and the owner can see you they will start "Default_AttackPet" Dialog with
you become hostile and try to call the gaurds in the region
*/

/*   Define Keywords

*/

/////NPC Groups///////////
/*   Notes
Needs to be looked at, mostly works with guards
*/
/****Invovled Databases****/

/****Useful Procedures****/
RemoveNpcGroupComplete((CHARACTER)_Player,(INTEGER)_ID);

/*   Define Keywords

*/

//----------------Player-NPC Relationships------------------//
/*
This part is about dealing with the different relations a Player Character has with NPCs and
their general influence on the world
*/
/////Attitude///////////
/*   Notes
If you attack someone that the player has greater or equal to then 25 reputation towards you, they will start warning dialog

*/
/****Invovled Databases****/
/****Useful Procedures****/
SetAttitudeToAtLeast((CHARACTER)_Player,(CHARACTER)_Npc,(INTEGER)_MinAtt);
SetAttitudeToAtMax((CHARACTER)_Player,(CHARACTER)_Npc,(INTEGER)_MaxAtt);
SetRepAttitudeToAtLeast((CHARACTER)_Player,(CHARACTER)_Npc,(INTEGER)_MinRepAtt);
DecreaseAttitude((CHARACTER)_Player,(CHARACTER)_Npc,(INTEGER)_Delta);
IncreaseAttitude((CHARACTER)_Player,(CHARACTER)_Npc,(INTEGER)_Delta);
SetHostileAtt((CHARACTER)_Npc,(INTEGER)_Att);	//NPC becomes hostile when at at or below the HostileAtt, becomes unhostile when above. Not used in Game

/*****Character Dialog Events*****/
//Dialog Character Event "EVENT_npc_attitude_up" increases attitude of players in dialog by 10
//Dialog Character Event "EVENT_npc_attitude_down" decreases attitude of players in dialog by 10
//Dialog Character Event "EVENT_npc_attitude_up_25" increases attitude of players in dialog by 25
//Dialog Character Event "EVENT_npc_attitude_down_25" decreases attitude of players in dialog by 25
//Dialog Character Event "EVENT_npc_attitude_up_50" increases attitude of players in dialog by 50
//Dialog Character Event "EVENT_npc_attitude_down_50" decreases attitude of players in dialog by 50

/*   Define Keywords

*

/***TODO***/
//check MustNotBecomeHostile_UnlessAttacked used correctly?

/////Relations///////////
/*   Notes
Relations affect NPC reacts to certain things, mostly crime
If Player has good relations they are likely only get warned when they commit a crime
*/
/****Invovled Databases****/

/****Useful Procedures****/
SetRelationFactionToPlayers((STRING)_Faction,(INTEGER)_Relation);
SetRelationIndivFactionToPlayers((CHARACTER)_char,(INTEGER)_Relation);
ProcSetRelationToPlayers((CHARACTER)_Character,(INTEGER)_Relation);
ChangeAttitude((CHARACTER)_NPC,(CHARACTER)_Player,(INTEGER)_Value);

/*   Define Keywords

*/

/////Reputation///////////
/*   Notes
Characters general reputation scores ie: how famous Player is

UpdateRepAttitude_Handled(_Player,_Npc), ClearReputationFacts((CHARACTER)_Player) might be useless
*/
/****Invovled Databases****/

/****Useful Procedures****/
DecreaseReputation((CHARACTER)_Player,(INTEGER)_Delta);
IncreaseReputation((CHARACTER)_Player,(INTEGER)_Delta);

/*   Define Keywords
(INTEGER)_Delta : You can increase or decrease by a negative amount and it will do the negative effect
*/

/***Dialog Falgs
"EVENT_reputation_infamous" 	       Rep < -25
"EVENT_reputation_negative"		-25 <= Rep < 0
"EVENT_reputation_low"			  0 <= Rep < 20
"EVENT_reputation_good"			 20 <= Rep < 40
"EVENT_reputation_famous"		       Rep > 40
*/

//-------------------------Dialogs--------------------------//
/*
This part contains the tools for dialog. Larian created a lot of these to handle many different
situations and to make dialog as safe as possible. They made many checks for death and being otherwise 
occupied and buffers so that dialogs wont be forgotten if they are told to begin by the story.
*/
/////On Click Dialogs//////////
/*   Notes
This section sets up dialogs when you click on the global character
"AnimalEmpathy" Talent will allow you to speak to characters in group "Animal"
If player has it weapons out and tries to talk to a Unintelligent, Non-Story Animal (see NPC stats), Player will attack
If your compaion talks to a character see 'Companion Dialog'
If the player has his weapon out character will be threatened look at Threatened section
NPC must have attitude > -45 to start default dialog, responds as Hotile (See 'Hostile' section)
If the NPC has default dialog it will play it (unbuffered)
*/
/****Invovled Databases****/
//main way of adding dialog to global units
DB_Dialogs((CHARACTER)_Npc,[(CHARACTER)_Npc2,(CHARACTER)_Npc3,(CHARACTER)_Npc4,](STRING)_Dialog);
	//start a dialog with one or more NPC when you click on the NPC(s)

//doesn't follow most of the usual checks, will just start and say a line
DB_AD_Dialog((CHARACTER)_Character,(STRING)_Dialog); //NPC One Speaker Dialog

OverrideDialog((CHARACTER)_Npc,(STRING)_Dialog); //for debugging, overrides ignores most checks
	//goes to SetAndStartDialog, see Story Dialog

/****Useful Procedures****/
ProcRemoveDialogEntryForSpeaker((CHARACTER)_NPC,(STRING)_Dialog);
ProcRemoveAllDialogEntriesForSpeaker((CHARACTER)_NPC); //Remove all default dialogs
DB_ItemGivesDualDialog((ITEM)_Item,(STRING)_Dialog); //Give an item dialog on use (one time use?)
ProcRemoveNPCADs((CHARACTER)_Npc); 			//Removes all Automatic dialogs

//used Once in main game, adding items to DB_Dialogs should work
ClearDefaultDialog((CHARACTER)_Npc);
SetDefaultDialog((CHARACTER)_Npc,(STRING)_Dialog,(INTEGER)_CloseButtonEnabled);

/*  Unused
OverrideMonologue(_Npc,(STRING)_Monologue) (unused)
StartMonologue(_Monologue) (undefined)
*/

/////Companion Dialog///////////
/*   Notes
If you talk to a NPC with your companion
Compaions don't threaten or are hostile towards unless NPC is in DB_NoCharacterCompanionReplace
Will be finished with version 0.8.0
*/
/****Invovled Databases****/
DB_NoCharacterCompanionReplace(_Npc); //Compaion acts like a player in this conversation

/****Useful Procedures****/
DB_CustomCompanionDialog((CHARACTER)_Npc,(STRING)_Dialog);
DB_CustomCompanionGroupDialog(_Group,_Dialog);

//Tutorial: "TUT_CompanionTalk"

/*   Default
"Default_Companion" the normal conversation a companion has with a character
*/

/////Threatened///////////
/*   Notes
This is a part for dealing with threatened dialog, in other words a player with their weapons out.
NPC in Character Group "Animals" or people in combat are never threatened
I would suggest against using SetDefaultThreatenedDialog, it skips several checks. Used by ShopOwners and non-story NPCs
*/
/****Invovled Databases****/
NeverThreatenedDialog((CHARACTER)_Npc);//Database for characters that are never threatened
	//players will automaticly put weapons away

//Character must be in IsStoryNpc (use proc SetStoryNpc, see Story NPC for details)
DB_CustomThreatenDialog((CHARACTER)_Npc,(STRING)_Dialog); //Custom dialog for specific Character
DB_CustomThreatenFactionDialog((STRING)_Faction,(STRING)_Dialog); //Custom dialog for specific faction, overridden by Character
DB_CustomThreatenGroupDialog((STRING)_Group,(STRING)_Dialog); //Custom dialog for specific Group, overridden by faction

/****Useful Procedures****/
SetDefaultThreatenedDialog((CHARACTER)_Npc,(STRING)_Dialog); 	//Sets a default threatened dialog

/*   Defaults
An Evil NPC will default to "Default_Threatened_EVIL" (Evil => in EvilDude database)
A non-Evil NPC will default to "Default_Threatened"  
*/
/* 

*/

/////Hostile///////////
/*   Notes
Hostile is Attitude >= -45
See 'Special: Animal Food' for a special Animal reaction
*/
/****Invovled Databases****/
HostileDialog((CHARACTER)_Npc,(STRING)_Dialog); //Give a character a default hostile dialog

/****Useful Procedures****/
//Procedure once used in main game to create a special reaction to hostility
StartHostileDialog((CHARACTER)_Player,(CHARACTER)_Npc);

/* Default Dialogs
"Default_Out" Default Dialog for Evil NPCs (in EvilDude, See NPC Stats)
	Attitude will decrease if NoDecreaseAttitudeFor(_Player,_Npc,"Default_Out") is not set
	TODO: Also used in theft?
"Default_Hostile" Default Dialog for non-evil NPCs
*/

/////Special: Animal Food///////////
/*   Notes
Isn't fully functional but its nifty
A player can give food to a hostile "Animal" if he has food for a 5 point attitude bonus
If the player has food that is defined in the animals character script he can feed
It seems that they have the scripts for creatures to run to the closest food, just not used
*/
/****Invovled Databases****/
//String value of a food template, by default FoodTemplate[1-6]
DB_AnimalFoodVars((STRING)_Food); 

/****Useful Procedures****/
//see these for a bit more detail
ProcSetAnimalFoodEvents((CHARACTER)_Player,(CHARACTER)_Npc); 
ProcGiveAnimalFood((CHARACTER)_Player,(CHARACTER)_Npc);
/* 
"GEN_HasAnimalFood" is a var interger that is set if the player has food the hostile creature enjoys
"GEN_PlayerGivesFood" Dialog event that removes a food from inventory and gives small attitude bonus
*/

/////Stroy Dialog///////////
/*   Notes
This is what one would use in the story editor to start a conversation
SetAndStartDialog is what an Editor would normally use, it queues dialog, disables trade if necessary, and
initilizes character dialog events
*/
/****Invovled Databases****/
/****Useful Procedures****/
DoStartDialog((CHARACTER)_Player,(CHARACTER)_Npc,(STRING)_Dialog); 		//Starts a dialog by destroying the queue and ended current dialogs
SetAndStartDialog((CHARACTER)_Player,(CHARACTER)_Npc,(STRING)_Dialog[,(INTEGER)_EnableCloseButton]);	//Starts a dialog safely
ProcStartCompanionDialog((CHARACTER)_Player,(CHARACTER)_Npc);			//This one just has an extra tutorial as far as I can tell, can do custom or default

/*   Define Keywords
(CHARACTER)_Player 				: 
(CHARACTER)_Npc 				: 
(STRING)_Dialog 				: 
(INTEGER)_EnableCloseButton 	: Default to 1
*/

/*****Dialog Events******/
/**Put away weapons during dialog
set the Character "EVENT_npc_player_puts_down_weapons_for"
***NPC Attacks after Dialog
Set the Character Event :"EVENT_npc_player_attacks"
*/
/*****Stored Data*****/
TalkedToPlayer(_Player,_Npc);	//Remembers if Player talked to an NPC
RanDialog(_Player,_Dialog);		//Remembers if Player ran a dialog


/*********Special*********/
//When you start a conversation you can preload an event for the upcoming conversation by adding another definition of the PreStartDialog Procedure 
/* Simple Example 
PROC
PreStartDialog((CHARACTER)_Player,(CHARACTER)_Npc,"MyDialogName")
//AND Conditions
THEN
SetAndRememberDialogEvent(_Player,"MyEventName",0);

*/

/////Mandatory Dialog///////////
/*   Notes
It's spelled wrong by Larian...
Does whatever it can to make NPC talk to players
Checks if there are any other players to talk to, then Queues the dialog
*/
/****Invovled Databases****/
PROC_MendatoryDialog((STRING)_Dialog,(CHARACTER)_Npc,(CHARACTER)_Player);
PROC_MendatoryDialogThreeSpeaker((STRING)_Dialog,(CHARACTER)_Npc,(CHARACTER)_Npc2,(CHARACTER)_Player);

//NPCs resorts to this if attacked, define it yourself. Not sure why this is a database they use it like a proc
DB_NoMoreQueued((CHARACTER)_Npc, (STRING)_Dialog);
//example of how to use: IF DB_NoMoreQueued(CHARACTER_SpecificNpc, "Name_of_Dialog") THEN ....

/****Useful Procedures****/

/*   Define Keywords

*/

/////One Shot Dialog///////////
/*   Notes
Basicly all the dialogs that only happen once, luckly larin named these relatively well.
There are brackets around certain databases you don't nessassarily need (you can have that many NPCs conversing, might be a problem with having 4).
*/
/****Invovled Databases****/
DB_OneShot_PlayerOnlyDialogTrigger((TRIGGER)_Trigger,(STRING)_Dialog,(CHARACTER)_NPC); //
DB_OneShot_DialogTrigger((TRIGGER)_Trigger, (STRING)_Dialog,(CHARACTER)_NPC [, (CHARACTER)_NPC2, (CHARACTER)_NPC3, (CHARACTER)_NPC4]);
DB_OneShot_DialogItem((ITEM)_Item, (STRING)_Dialog [, (CHARACTER)_NPC, (CHARACTER)_NPC2, (CHARACTER)_NPC3, (CHARACTER)_NPC4]); //happens upon use of item
DB_OneShot_ADTrigger((TRIGGER)_Trigger,(STRING)_Dialog,(CHARACTER)_NPC[,(CHARACTER)_NPC2]); //not sure differncebetween this and DB_OneShot_DialogTrigger
DB_OneShot_DialogTrigger_NewSystem(_Trigger,_Dialog,_Spotter1[,_Spotter2,_Spotter3]);//Not sure about the pros and cons of each oneshot system. 
	//This one is named as if to be more for spotting sneakers (?)

/****Useful Procedures****/

/*   Define Keywords
(TRIGGER)_Trigger		: Dialog occurs at this trigger
(STRING)_Dialog			: 
(CHARACTER)_NPC(s)		: NPC included in dialog
*/

/////Warning Dialogs///////////
/*   Notes
Combat Warnings are entering combat shoutouts if the player is already in combat
Warnings are dialogs 
If the player attacks someone they get a couple chances before NPC DoRevengeOnAttack
EvilDude : 0
Companion: 3
Good standing (>=0 attitude) : 3
Bad Standing (<0 attitude) : 2
*/
/****Invovled Databases****/
dbCustomCombatWarningDialog((STRING)_Faction,_Dialog);	//If player is in comabt and is joined by NPC
DB_CustomWarningDialog(_Npc,(STRING)_Dialog);			//For NPC player interaction upon illegal activity, overrides faction dialogs
DB_CustomFactionWarningDialog(_Faction,(STRING)_Dialog);	//For NPC player interaction upon illegal activity, overrides group dialogs
DB_CustomGroupWarningDialog(_Group,_Dialog);		//For NPC player interaction upon illegal activity, overrides default dialog
WarningDialog((CHARACTER)_Npc,(STRING)_Dialog); //Custom Warning Dialog for NPC, unused

/****Useful Procedures****/

/******Default Dialogs******/
//Default combat warning Dialog: "please_dont_attack_combat" 
//Default Warning Dialog: "please_dont_attack"

/*   Define Keywords

*

/////Character Events/////////
/*
The following events are set and cleared if the conditions are met
*/

/****Invovled Databases****/
DialogConditionTrue((CHARACTER)_Character,(STRING)_Event); //Character dialog Event storage 

/****Useful Procedures****/
SetAndRememberDialogEvent((CHARACTER)_Character,(STRING)_Event[, (INTERGER)_Set]);

/*****Always Set or Cleared*****/

"EVENT_in_guarded_area" 	//Set if Player is in a guarded aarea

//Time of day Events, Uses Time (see 'Time' section)
"EVENT_time_night"			//      Hour < 6
"EVENT_time_morning"		// 6 <= Hour < 12
"EVENT_time_afternoon"		//12 <= Hour < 18
"EVENT_time_evening"		//18 <= Hour

//Randomly sets of flags
"EVENT_dialog_random1_1"
"EVENT_dialog_random1_2"	//Flag randomly set between these 3
"EVENT_dialog_random1_3"

"EVENT_dialog_random2_1"
"EVENT_dialog_random2_2"	//Flag randomly set between these 3
"EVENT_dialog_random2_3"

"EVENT_dialog_random3_1"	//Flag randomly set between these 2
"EVENT_dialog_random3_2"

//Based on (CHARACTER)_NPC.EvilDude() (might be shortcut for checking if in EvilDude database)
"EVENT_npc_is_evil_dude"
//Based on Database IsHostile((CHARACTER)_Player,(CHARACTER)_Npc)
"EVENT_npc_is_hostile"
//Based on (CHARACTER)_Npc.DoesNotWantToTrade()
"EVENT_npc_does_not_want_to_trade"
//Based on database RanDialog(_Player,_Dialog), set after first conversation
"EVENT_npc_talks_second_time"
//Based on database IsStoryNpc(_Npc)
"EVENT_npc_is_story_npc"

//Based on Attidute 
"EVENT_npc_is_best_friend"	//	     Attidute >= 90
"EVENT_npc_is_helpful"		//  90 > Attidute >= 70
"EVENT_npc_is_positive"		//  70 > Attidute >= 0
"EVENT_npc_is_negative"		//   0 > Attidute > -75
"EVENT_npc_is_real_angry"	//-50 => Attidute

/****Set and cleared when not Hostile****/

//Based on Database CanHeal((CHARACTER)_NPC) 
"EVENT_npc_can_heal"

/////Generic Dialogs///////////
/*   Notes
Disables some other generic dialogs for 2 seconds after the use of SetAndStartDialog_DisableDialogsCausedByGenericRules
Used in calling a default dialog to avoid alot of defaults happening simultaneously
*/
/****Invovled Databases****/
GenericDialogTimers((CHARACTER)_Player,(STRING)_TimerName); //Have one declared for each player and a unique TimerName

/****Useful Procedures****/

/****Dialog Names*****/
/* TODO sort this out
"I_attack_you" 			general attacking for a reason
"I_attack_you_EVIL" 	general attacking for a reason and NPC is in EvilDude database
"Lets_teach_himher"	 	Revenge with friends Not Evil
"Guards_Lets_teach_himher" And guards have been summoned 
"Lets_teach_himher_EVIL" Revenge with friends Not Evil and NPC is in EvilDude database
"Hey_thief_I_will_teach_you" attacking because they are stealing
"Nobody_steals_from_me" 	attacking because they are stealing and NPC is in EvilDude database
"Catch_the_trespasser"
"Save_my_property"
"Guard_save_my_property"
"Save_me"
"Runaway" 	
*******disgusted*******
"YouScum" 	NPC can't do anything to player but still says something
*******Theft of own object*******
"Encourage_thievery_warn_for_guards" NPC dialog that doesn't really react other then to say, go for it
"Npc_accepts_theft_of_own_objects" NPC dialog that doesn't like it but doesn't do anything, because attitude >= 0
"Catch_the_thief" If guards are coming dialog
"Dont_steal_from_me_Please_Leave"  NPC stolen from and doesn't like player (Attitude < 0)
******Theft of Others Objects*****
"Thievery_is_unwise" 	Character like you
"You_are_a_thief_Please_leave" Character dislikes you (Attitude < 0)
*****Vandalism*****
"Npc_accepts_destruction_of_own_objects" If player 
Will be disgusted if no help is coming
"Catch_the_vandal" if help is comeing
"Dont_destroy_my_stuff_Please_Leave" 
****Trespass*****
"Catch_the_trespasser" If guards are comming
"Trespassing" said to a trespasser if not helped, (NPC might attack?)


*/

/****Creating your own****/
//Start your own default dialogs with this to avoid simultaneous conversations
SetAndStartDialog_DisableDialogsCausedByGenericRules((CHARACTER)_Player,(CHARACTER)_Npc,(STRING)_Dialog);
//TODO example

/*   Define Keywords

*/

/////Party Dialog///////////
/*   Notes

*/
/****Invovled Databases****/
DB_KillCounterGivesPartyDialog((STRING)_Name, (INTEGER)_Count, (STRING)_Dialog);
DB_ItemGivesPartyDialog((ITEM)_Item,(STRING)_Dialog);
DB_EventGivesPartyDialog((STRING)_Event,(STRING)_Dialog);
DB_TriggerGivesPartyDialog((TRIGGER)_Trigger,(STRING)_Dialog);
DB_CharacterEventGivesPartyDialog((CHARACTER)_Character,(STRING)_Event,(STRING)_Dialog);

/****Useful Procedures****/
ProcDefinePartyDialog((STRING)_Dialog);			//Used in Infected Dog Mission, and more...
ProcCancelPartyDialog((STRING)_Dialog);			//Cancel Specific party dialog
ProcCancelDualDialogs(); 						//Cancels all party dialogs

/*   Define Keywords
(STRING)_Dialog 		: Active Party Dialog ID
(INTEGER)_Count 		: Count target for kill counter
(ITEM)_Item 			: Item that upon use gives party dialog
(STRING)_Event 			: Global Event that gives party dialog
(TRIGGER)_Trigger 		: Trigger that upon trip gives party dialog (one time)
*/

//------------------Companions and Henchmen-----------------//
/*
This part deals with NPCs that are in a players party but are not main characters 
so people react differently to them. Companions have back story and are recruited in the 
course of the story, they have some of their own interactions and special dialogs.
Henchmen are just people that help and have no extra personality.
This part is incomplete and will be furthur updated with version 0.8.0
*/

/////General///////////
/*   Notes
This section deals with general companion settings
*/
/****Invovled Databases****/
DB_CompMax((INTERGER)_MaxCompanions);	//Determines the maximum number of compaions and henchmen you can have
DB_Compcount((INTERGER)_NumCompainons); //The current number of compnions your group has

/****Useful Procedures****/
Proc_Z_SetMaxCompanion((INTEGER)_Max); //Made by me for a safe way to change companion number size

/****Lone Wolf****/
//The LoneWolf Talent reduces the number of Max compaions by 1. 

/*   Define Keywords

*/

/////Companions///////////
/*   Notes
General procedures and databases for each companion
Most of the companion information is created in their own pages (TODO companion template)
*/
/****Invovled Databases****/
DB_Companion(_Character); //Database that contains companions, players that are not source hunters
DB_Companion_Default_Dialog((CHARACTER)_Companion, (STRING)_Dialog);
DB_AttackDialog(_companion, _dialog); //custom dialog for when limit reached of being attacked
Proc_LimitAbilitiesForCompanion((CHARACTER)_Companion); //This database disables compaions abilities in Charisma and Blackrock
DB_DismissEvent(_Npc,_Event); //When companion get dismissed it sets this event

/****Useful Procedures****/
Proc_ResetCompanions(); //used to send compaions back to their start locations, ment for main game init
	//TODO change so works for any companions
Proc_CompanionDialog((CHARACTER)_Companion,(STRING)_Dialog,(CHARACTER)_Anchor);
	//Set the compaion to have a special dialog and an exclamation mark over his head
	//Companion must be within a distance of 30 from anchor character
Proc_CompanionComment((CHARACTER)_Companion,(STRING)_Comment,(CHARACTER)_Anchor);
	//Companion will make a comment about something if they are within a distance of 30 from anchor character
ProcCancelAllCompanionDialogsForAllCompanions();
ProcDismissAllCompanions();

/*   Define Keywords

*

/////Henchmen///////////
/*   Notes
Characters without a back story or special dialog
also includes the henchmen trader
	TODO make this an viable for stand alone or remove it completely
*/
/****Invovled Databases****/
DB_HenchmanTrader(_Npc,_Trigger);
DB_HenchmanPool((CHARACTER)_Char);

/****Useful Procedures****/
ProcDismissHenchman((CHARACTER)_Char);  //Normally Will Teleport back to player home in Main Game (TRIGGER_HOM_HenchmenDismiss)
ProcHireHenchman((CHARACTER)_Char);		//Adds a henchmen to your group
ProcStartHenchmanInterfaceForTrader((CHARACTER)_Npc,(CHARACTER)_Player); //only is set to work with the player home henchmentrader

/******Default Dialogs******/
//Dialog if a henchmen leaves the party: "Henchman_LeavesParty" on attack

/****Dialog Events****/
"GEN_DismissHenchman" //If this character event is set the Henchmen will be dismissed
"GEN_SetupHenchman"   //With this character event Henchmen will appear in front of trader
"GEN_StartHenchmenInterface" //this will start up the UI?

/****Global Event*****/
"GEN_LastHenchMan" //one henchmen left in database
"GEN_NoMoreHenchmanAvailable" //no henchmen left in database


/*   Define Keywords

*/

//-----------------------For Players------------------------//
/*
This Part contains stuff almost soly about players and how they can be helped or their special
abilities. Tutorials are here, have a player make a comment as a clue, Journal is handled in this section

*/

/////Waypoints///////////
/*   Notes
This Section helps deal with waypoints, teleport using a UI to select destination
Warps certain calls: OpenWaypointUI,UnlockWaypoint,RegisterWaypoint
*/
/****Invovled Databases****/
DB_WaypointInfo((ITEM)_Waypoint,(TRIGGER)_Trigger,(STRING)_CurrentWP);

/****Useful Procedures****/
PROC_UnlockAllWP(); //Unlocks all waypoints in DB_WaypointInfo

/****Strings****/
"wp" //Will unlock all waypoints and open the UI
"GLO_WaypointDiscovered" //Shows this notification upon discovering Waypoint
"GLO_Waypoint" //Unlocking a waypoint will cause this event as defined in DB_ExplorationEvents

/*   Define Keywords
(ITEM)_Waypoint 	: Using this item opens menu
(TRIGGER)_Trigger 	: Teleport to Point
(ITEM)_Waypoint 	: Unique name of Waypoint
*/

/////Traits///////////
/*   Notes
Traits are things like "Timid" and "Independent". I left the standard ones in
Uses the calls CharacterAddTrait and CharacterIncreaseSocialStat. 
Not 100% sure how the CharacterIncreaseSocialStat works because it takes the Event not the trait
Traits are not resricted to players
*/
/****Invovled Databases****/
DB_StatIncreases((STRING)_CharEvent,(STRING)_Trait);
//Upon revceiving a dialog character event _CharEvent (must be in a dialog). Add 1 point to _Trait

/****Useful Procedures****/
ProcCheckPartyForTrait((STRING)_Trait,(CHARACTER)_Player,(STRING)_OnTrueEvent);

/*   Define Keywords
(STRING)_Trait 		: Name of Trait
(CHARACTER)_Player 	: Will check this player and nearby characters for 
(STRING)_OnTrueEvent: Will set this global flag if any players nearby have this trait
*/

/****Special***/
//The "GLO_Yield" Dialog Character Event also acts like the "GLO_IncreaseObedient" Event

/////Sneaking///////////
/*   Notes
This spots player characters in a target area 
every 0.1 seconds checks CharacterCanSee
*/
/****Invovled Databases****/
//Character is on the look out for players in this area, If they gain sight of them the later procedures will occur
SneakTriggerSpotter((TRIGGER)_Trigger, (CHARACTER)_Char); 

/****Useful Procedures****/
//These are special procedures that occur when a player is spotted
ProcCharInTriggerSpotted(_Player,_Trigger);
ProcCharInTriggerSpottedByChar(_Player,_Trigger,_Char);
ProcHandleSneakSpotted(_Char);

/****Character Scripts****/
//"RestartSpotting" : if the character stopped (due to combat or someother reason) spotting restarting spotting with this character event
//

/*   Define Keywords

*/

/////Persuasion///////////
/*   Notes
This section creates the persuasion possiblities. The standard are Intimidate, Charm, and Logic. 
I am unsure I you can add more options to this list, one would need to look at a couple calls
All the information boils down into StartDialogConflict call.
It aquires information from the ability information from CharacterGetAbility query

Network? - kinda confused on this one
if a "Set_Persuasion_Flags" is set in a dialog and one or both player characters have higher then 10 perception,
it will call DialogSetNodeFlag (?) and flag it something in Dialog.

This all exist in the file _Glo_IntimidateCharmLogic
*/
/****Invovled Databases****/
dbDialogPersuasionEvents((STRING)_Event,(STRING)_LocalEvent,(STRING)_Text);
dbDialogPersuasionNodes((INTERGER)_?, (STRING)_?);

/****Useful Procedures****/

/*   Define Keywords
(STRING)_Event 			: Name of persusion event (ex: "ApplyReason", or "ApplyIntimidate") can be anything vent named here
(STRING)_LocalEvent 	: Name of event localy (ex: "Reason", or "Charm") can be anything vent named here
(STRING)_Text 			: Name of String to be shown
*/

/////Player Comments////////////////
/*   Notes
If you want A Player Character to make a comment about somthing
May Require 2 players 
*/
/****Invovled Databases****/
DB_PlayerComments((STRING)_Comment, (STRING)_Text, (INTERGER)_PlayerAmount, (INTERGER)_Current);
DB_PlayerComment_Trigger((TRIGGER)_Location,(STRING)_Comment);
DB_PlayerComment_Event((STRING)_Event, (STRING)_Comment);											//Not used in main game
PROC_Z_CompanionComment((STRING)_ID, (CHARACTER)_Companion, (STRING)_Dialog);						//Non-Larin I made this one

/****Useful Procedures****/
Launch_PlayerComment((CHARACTER)_Player,(STRING)_Comment); //_Player will make a comment

/*   Define Keywords
(STRING)_Comment		: Unique ID for each comment 
(STRING)_Text			: Name of Comment 
(CHARACTER)_Player 		: The player that makes the comment
(INTERGER)_PlayerAmount	: Ammount of players required to be present
(INTERGER)_Current  	: On current comment
(CHARACTER)_Companion 	: Campanion that comments
*/

/////Affection Between Players///////////
/*   Notes
This section deals with 2 players interacting when certain conditions are met
It turns on exclamation points over players heads and preparing a party dialog
Affections include leveling up, and gaining traits, friendly fire, etc...
This can only happen a certain number of times per level
Upon certain triggers Party dialog will happen. The default list is in the INIT section of Global_AffinitySystem
The Triggers list (and triggers) are in the REGION Trigger of KB in Global_AffinitySystem
*/
/****Invovled Databases****/
DB_AffectionTriggers((STRING)_AffecionDialog); //Name of dialog/affection triggers name
DB_AffectionTraitTrigger((STRING)_Trait,(INTEGER)_Trigger,(STRING)_AffecionDialog); //When a _Trait reaches _Trigger begin _AffecionDialog
DB_MaxAffectionDialogsPerLevel((INTERGER)_Max); //max number of affection dialogs per level
DB_AffectionTypeDefs((STRING)_Flag,(STRING)_Type,(STRING)_DomFlag); //keeps a counter of how many times _Flag (Dialog Character Flag)
//is set. 
//generated
DB_Affinity((INTEGER)_Number); //initilize at 0, if two players have the same idea +1, other otherwise -1 to this
DB_Affection((CHARACTER)_Player, (CHARACTER)_OtherPlayer, (INTEGER)_Number); //initilize for each player
//Total Affection _OtherPlayers have for _Player, modified by Dialog Character Event "GLO_IncreaseAffection" and "GLO_DecreaseAffection"

/****Useful Procedures****/
ProcStopAffectionDialog(); //Disables all current affection dialogs, (will do this on player death)
ProcTriggerAffectionDialog((CHARACTER)_SrcChar,(STRING)_Dialog); //use this to run _Dialog (from _SrcChar) after your event triggers
ProcCalculateAffinity(); //If characters have the same level in 5 or more traits Global event "GLO_PlayersAffine" is set (used in endgame)
ProcCalculateDominantAffectionType((CHARACTER)_Player); //Works with Sets DB_AffectionTypeDefs, whichever has the highest count during
//the check, its _DomFlag character event is set, (DOES NOT CLEAR OLD EVENTS, used in endgame)

/*   Define Keywords

*/

/////Journal & Discription///////////
/*   Notes
Recipe in the journal are unlocked by an item template (usually a book) and unlock
a Journal ID (not really sure how that works). Item descriptions are text that appear when
the Player clicks on said item.

ItemDisplayText is used with DB_Global_ItemDescriptions
UnlockJournalRecipe is used with DB_RecipeBook
*/
/****Invovled Databases****/
DB_Global_ItemDescriptions((ITEM)_item, (STRING)_description);
DB_RecipeBook((STRING)_Template, (STRING)_ID);

/****Useful Procedures****/

/*   Define Keywords
(ITEM)_item 			: Item to be descriped
(STRING)_description 	: description of that item
((STRING)_Template 		: template of an item
(STRING)_ID) 			: Journal unlock ID
*/

/////Tutorial Messages///////////
/*   Notes
Tutorial messages are displayed once
*/
/****Invovled Databases****/
DB_TutorialInfo((STRING)_Message,(INTEGER)_Key,(STRING)_Catagory);
DB_TutPlayed((INTERGER)_ID,(STRING)_Message);		//Used in checks for tutorial arleady played

/****Useful Procedures****/
PROC_CheckPlayTut((STRING)_Message);	//I Commented this out because it was ment for 2 players
	//it calls the following procedure once for each player
PROC_CheckPlayTut((CHARACTER)_Player,(STRING)_Message);	//Use to play tutorial for a character
PROC_CheckPlayTutWithDelay((CHARACTER)_Player,(STRING)_Message,(INTEGER)_Delay); //play a tutorial message for each player
ProcPlayTut((CHARACTER)_Char,(STRING)_Message); 	//Plays tutorial if not shown, need to double check which is better

/*   Define Keywords
(INTERGER)_ID 			: Reserved Peer ID, aquire with query 'CharacterGetReservedPeerID((CHARACTER)_Player,(INTERGER)_ID)'
(STRING)_Message 		: Internal name of tutorial you show
(INTEGER)_Delay 		: Delay before tutorial plays (in ms)
(INTEGER)_Key 			: Goes into ButtonID of show tutorial... wha? earlier the lower?
*/

//--------------------------Items---------------------------//
/*
This part is for objects in the world that are not Characters and a few things
one can do with them
*/
/////General///////////
/*   Notes

*/
/****Invovled Databases****/
EquippedArmorDB((STRING)_Item, (STRING)_Flag); /*Upon Equipping and Unequiping _Item, will set and clear the _Flag as a Character Dialog Event
 	If both have it equiped it will set and clear _Flag as a global Event*/

/****Useful Procedures****/

/*   Define Keywords

*/


/////Doors///////////
/*   Notes
When you want to open a door in story script or lock one, look no furthur!
*/
/****Invovled Databases****/
/****Useful Procedures****/
ItemCloseAndLock((ITEM)_Item,(STRING)_Key); 
ItemUnlockAndOpen((ITEM)_Item);

/*   Define Keywords
(ITEM)_Item  : Door to close and lock
(STRING)_Key : Name of key, seemingly not used (use ItemUnlockAndOpen)
*/

/////Hidden Walls///////////
/*   Notes
Hidden walls disappear in a puff of smoke when certain contions are met

To create a hidden wall and the way to open it with usage of an item one must run a procedure to register it
you can then create your own Procedure to then assign a way to open it. An example is provided
*/
/****Invovled Databases****/
HiddenWallItemDB((ITEM)_Item, (INTEGER)_WallIndex);
//PROC_CommentHiddenEffect is a procedure that occurs when a HiddenWallItemDB is used and plays one of 4 lines, I have disabled it
HiddenWallEventDB((STRING)_flag, (INTEGER)_wallIndex);	//When the global event _flag is set the hidden wall opens
HiddenWallTriggerDB((TRIGGER)_Trigger, (INTEGER)_wallIndex); //Opens wall when someone is in the trigger area, closes up when no one is :)

HiddenWallDB((INTEGER)_WallIndex, (ITEM)_Wall); //Generated by PROC_RegisterHiddenWall

/****Useful Procedures****/
PROC_RegisterHiddenWall((ITEM)_Wall);

/* example of a register, run it in init section
PROC PROC_RegisterWaysToOpenHiddenWall()
AND HiddenWallDB(_WallIndex, (ITEM)_Wall)
THEN
HiddenWallItemDB((ITEM)_Item,_WallIndex);	
*/

/****Special****/
//Item Event "Open" on a hidden wall will open the wall

/*   Define Keywords
(INTEGER)_WallIndex 	: generated number
(ITEM)_Wall 			: The hidden wall to be opened
(ITEM)_Item 			: Item upon usage opens wall
(TRIGGER)_Trigger 		: Trigger Zone to activate wall
(STRING)_flag 			: Global flag that upon set opens wall
*/

///////Shoveling Stuff/////////////
/*   Notes
Script for shoveling
If its in the database the reward(s) spawn upon completing the dig
*/
/****Invovled Databases****/
DB_ShovelArea((TRIGGER)_Area, (STRING)_Reward, (ITEM)_Dirt);
DB_ShovelRewardComment((STRING)_Reward, (STRING)_Comment);										//Test with Test_Greever
DB_ShovelRewardItemAppear((STRING)_Reward, (ITEM)_Item,(TRIGGER)_Spawn);
DB_ShovelRewardItemAdd((STRING)_Reward, (ITEM)_Item);
DB_ShovelRewardCharacterAppear((STRING)_Reward,(CHARACTER)_Character);
DB_ShovelRewardItemSpawn((STRING)_Reward,(ITEM)_Item);
DB_ShovelRewardItemTemplate((STRING)_Reward,(STRING)_ItemTemplate,(INTEGER)_Amount);
DB_ShovelRewardEvent((STRING)_Reward,(STRING)_Event);
DB_ShovelRewardSurface((STRING)_Reward,(TRIGGER)_Trigger, (STRING)_Type, (REAL)_Radius);

/****Useful Procedures****/

/*   Define Keywords
(TRIGGER)_Area 		: Player must be in this trigger area
(ITEM)_Dirt			: The dirt mount that appears at end of dig
(STRING)_Reward		: Name of Reward
(ITEM)_Item			: Item given upon completion
(TRIGGER)_Spawn)	: Reward is moved to this trigger
*/


//--------------------------Areas---------------------------//
/*
This part deals with players entering trigger zones and minor thing you can do with
them.
*/
/////Triggers///////////
/*   Notes
How to register Triggers so they are cleanly added and removed from registering upon entering
*/
/****Invovled Databases****/
OneShotPlayerOnlyTrigger((TRIGGER)_Trigger);	//Cleanly register a trigger for players only, only fires once

/****Useful Procedures****/
ProcTriggerRegisterForPlayers((TRIGGER)_Trig);		//cleanly register trigger for players and companions
ProcTriggerUnregisterForPlayers((TRIGGER)_Trig);	//cleanly unregister trigger for players and companions
ProcRegisterForCompanions((TRIGGER)_Trig);			//cleanly register trigger for companions only
ProcUnRegisterForCompanions((TRIGGER)_Trig);		//cleanly unregister trigger for companions only
ProcRegisterPlayerTriggers((CHARACTER)_Char);		//cleanly register character for triggers
ProcUnRegisterPlayerTriggers((CHARACTER)_Char);		//cleanly unregister character for triggers

ProcOneShotTriggerEntered((CHARACTER)_Player,(TRIGGER)_Trigger);
//Define your own procedure so that it runs when a player walks into it for the first time
/* example
PROC ProcOneShotTriggerEntered(_Player, TRIGGER_MyTrigger) THEN .... 
*/

/*   Define Keywords

*/

/////Exploration Bonuses//////////////
/*   Notes
Gain experience when you reach a location or Event occurs
*/
/****Invovled Databases****/
DB_ExplorationZones((TRIGGER)_Location,(INTEGER)_Act,(INTEGER)_ActPArt,(INTEGER)_Gain);
DB_ExplorationEvents((STRING)_String,(INTEGER)_Act,(INTEGER)_ActPart,(INTEGER)_Gain);

/****Useful Procedures****/
Proc_AddExplorationEvent((STRING)_String);

/*   Define Keywords
(TRIGGER)_Location	: Gain Exirence upon reaching location
(INTEGER)_Act		: Always 1 (in Main game)
(INTEGER)_ActPart	: Higher the later in game it is (not sure why)
(INTEGER)_Gain		: Level of Reward (not exp) (1-5 in main game)
*/

/////SubRegion///////////
/*   Notes

*/
/****Invovled Databases****/
DB_Subregion((TRIGGER)_Trigger,(STRING)_Message,(INTERGER)_ShowMarker);
DB_MarkerDB((STRING)_Marker);

/****Useful Procedures****/

/*   Define Keywords
(TRIGGER)_Trigger 		: When entering this trigger shows subRegion message
(STRING)_Message 		: label for message to be displayed
(INTERGER)_ShowMarker 	: bool, doesn't matter what goes here. Always Shows
(STRING)_Marker 		: ID for marker
*/

/////Ambush///////////
/*   Notes
Create an Ambush with this database and procedure
characters won't trigger it if they are invisible or sneaking (but will if they stop doing so while inside)
helper item will be removed from stage when the ambush occurs and AmbushTrigger will be cleared. If you want
to return the item i suggest storing it elsewhere as well
*/
/****Invovled Databases****/
AmbushTrigger((TRIGGER)_Trigger, (ITEM)_Helper);

/****Useful Procedures****/
//Add a new definition for this so something happens when player enters the 
ProcLaunchAmbush((TRIGGER)_Trigger, (CHARACTER)_Player);

DB_Event2DisplayText((STRING)_Event,(STRING)_PlayerText,(STRING)_TextAlt);
/* Upon occurance of CharacterEvent _Event the _PlayerText DisplayText will appear
You can set up an alternative character event to display the _TextAlt, see TrapReactions
*/

/*   Define Keywords
(TRIGGER)_Trigger 	: Area where the trigger occurs
(ITEM)_Helper 		: Item to help lure the players in. can be NULL
(CHARACTER)_Player 	: Player in the ambush
*/

/////Forbindden Areas///////////
/*   Notes
Makes NPCs react when Player(s) go where they are not wanted
*/
/****Invovled Databases****/
DB_ForbiddenArea((STRING)_ID,(CHARACTER)_NPC);
DB_ForbiddenAreaTriggers((STRING)_ID,(TRIGGER)_Trigger,(TRIGGER)_Exit);
DB_ForbiddenDoors((STRING)_ID,(ITEM)_Door);

/****Useful Procedures****/
ProcClearForbiddenArea((STRING)_ID);

/****Default Dialog****/
"default_forbidden_door"

/*   Define Keywords
(STRING)_ID 		: ID of forbindden area
(CHARACTER)_NPC		: Owner of forbindden area (There can be multiple per forbidden area)
(TRIGGER)_Trigger	: Area of trespassing
(TRIGGER)_Exit		: If kicked out of area end up at this trigger
(ITEM)_Door			: Door that is considered trespassing if used
*/

//////CIRDialog////////////
/*   Notes
Something to do with charisma experience
gain expirence when an event occurs?
*/
/****Invovled Databases****/
DB_CIRDialog((STRING)_Event, (INTERGER)_Act, (INTERGER)_ActPart, (INTERGER)_Gain);

/****Useful Procedures****/

/*   Define Keywords
(STRING)_Event		: Global Event its triggered on
(INTERGER)_Act 		: Always 1 (in Main game)
(INTERGER)_ActPart 	: Higher the later in game it is (not sure why)
(INTERGER)_Gain		: Reletive to Exp Gain? Main Game: 1-10
*/


//-------------------------Ilegals--------------------------//
/*
Everything has its consequences, here is how those Illegal activities effect the world

These scripts are very complicated and many thousands of lines, But mostly handled automatically once set up.

Non EvilDude, non guard NPCs wont attack for certain illegal activity unless they are at least equally leveled with player
You can't pickpocket Characters in group: "Animal", "Elemental", or "Ghost". They will attack instead

		Reason
1 : Stealing from me 
2 : Stealing, not from me (Evil NPCs don't care)
4 : Destroying property, vandalism
5 : attack
6 : trespass
*/

/////Illegal///////////
/*   Notes
*/
/****Invovled Databases****/

/****Useful Procedures****/

/*   Define Keywords

*/
/* Crimes
"Vandalism"

*/

/////Prison///////////
/*   Notes
The lock-up
*/
/****Invovled Databases****/
PrisonData((CHARACTER)_Player,(STRING)_Prison, (TRIGGER)_Cell,(TRIGGER)_Exit, (ITEM)_CellDoor,(ITEM)_PrisonChest);
DB_PrisonComments(_Prison,_Comment,_Nr);
PrisonEscapeTimers(_Char,_Tim,_Escape); //Initialized by Zolton's Tools
PrisonEscapeData((STRING)_Prison,(TRIGGER)_Prohib,(CHARACTER)_Guard,(STRING)_Leak,(TRIGGER)_Jail);
DB_CurrentPrisonComment(_Prison,_Player,_Nr);
DB_CompanionChest(_Prison,_Chest,_Char);
PrisonDataDoorKeys(_Door,_Key);
NoPrisonEscapeCalls(_Player,1); //useful if you want to bribe a guard will be unset if when player leaves prison

/****Useful Procedures****/
ImprisonPlayer((CHARACTER)_Player, (STRING)_Prison); //If you want to send a player to prison
	//Also sets the Interger Variable on player "Player_WasIn_[Prison]" to 1
FreePlayer((CHARACTER)_Player);
	//Frees Character from prison with all his stuff, also decreases reputation by 1

/****Generic Dialog****/
"Default_Prisoner_Escapes"

/****Character Events****/
//If during a conversation one of the Events is set, the player goes to prison. (no difference)
"EVENT_thievery_player_to_prison"
"EVENT_trespassing_player_to_prison"
"EVENT_vandalism_player_to_prison"

//If a player is in prison
"EVENT_player_in_prison"

/*   Define Keywords

*/

/////Fines///////////
/*   Notes
Sometimes a player needs to pay their way out of the problem
*/
/****Invovled Databases****/
ThieveryFine((STRING)_Reason, (INTEGER)_Fine);
VandalismFine((STRING)_Reason, (INTEGER)_Fine);
TrespassingFine((STRING)_Reason, (INTEGER)_Fine);

/****Useful Procedures****/
/****Fine Names****/
//These are by default in [Crime]Fine database
//The fine is higher then the value of items witnessed to be stolen
"EVENT_thievery_fine_low"    	//by default 250 gold
"EVENT_thievery_fine_medium" 	//by default 1000 gold fine
"EVENT_thievery_fine_high"   	//by default 5000 gold fine

"EVENT_vandalism_fine_low"   	//by default 250 gold fine
"EVENT_vandalism_fine_medium"	//by default 1000 gold fine
"EVENT_vandalism_fine_high"  	//by default 5000 gold fine

"EVENT_trespassing_fine_low" 	//by default 250 gold fine
"EVENT_trespassing_fine_medium" //by default 1000 gold fine
"EVENT_trespassing_fine_high" 	//by default 5000 gold fine

/*   Define Keywords
(STRING)_Reason : See begining notes of part for details
*/

/////Guards///////////
/*   Notes
Guards are called by Non-Evil NPCs if they need help (against the player)
Guards always have a theivery tolerance of at most -50
Guards will resurrect if they are killed when they send the player to prison (and leave the area?)
*/
/****Invovled Databases****/
GuardedRegion((TRIGGER)_Region,(TRIGGER)_QuitChaseRegion,(STRING)_Type,(STRING)_Prison);
//		_GuardedRegion: region in which guards of _GuardType operate.
//		If they imprison you (in case of thievery behaviour), it'll be in the prison with ID _Prison.
//		_QuitChaseRegion can be = _GuardedRegion but can be a bigger region to. When the player runs outside
//		this region, the guards quit following him. (Useful to prevent the guards to be lured in monster regions.)

DB_GuardDialogs((STRING)_Region,(STRING)_Dialog); //Default Dialog for a guards in that Region (uses DB_Dialogs)

GuardTypeNpc((STRING)_Region,(CHARACTER)_Guard,(STRING)_Script,(INTEGER)_Index); //Links guards as a group
	//Region indicates where they are from. Guard is the actual NPC. Script is unused. 
	//Index each guard (starting at 1), mostly unused (have one for each area that is 1).
	//These guards might level up with players
IsGuardAlignment((STRING)_Alignment); //probably useless, used with NpcFollowsAlignment (never used)
IsGuard((CHARACTER)_Npc); //Basic database for all guards, GuardTypeNpc and IsGuardAlignment populates this database

DB_NumGuards((STRING)_Region,(INTEGER)_Number); //not actually used as far as I can tell

//For crimes, given default values. See default dialog
GuardDialog((STRING)_Dialog,(INTEGER)_Crime); //name of dialog for Crime (see Reason in begining notes)
ArrestThiefDialog((CHARACTER)_Guard,(STRING)_Dialog); //name of dialog on be played when a guard catches 
	//the player theieving. Doesn't need to be defined, defaults to "Thievery_PayFine"

NpcCannotCallGuards(_NpcCallingGuards); //If you dont want an NPC to be able to call guards
NoGuards(_Player,1); //never actually used in main game but could be used if you want to bride guards in your story

//Initilized with players and 0
DB_PlayerSummonID((CHARACTER)_Player, (INTERGER)_Number);

/****Useful Procedures****/
MakeGuardsFriendlyOnMap((CHARACTER)_Player,(STRING)_Region); //Makes guards leave

/****Default Dialog****/
//These are the default dialogs from guards and specific crimes player has commited
"Thievery_PayFine"
"Vandalism_PayFine"
"Trespassing_PayFine"

/****Dialog Events****/
"EVENT_npc_is_a_guard" //set if an NPC is a guard (in IsGuard), cleared if not
//This dialog Event is set, Player starts a fight, guards are summoned
"EVENT_thievery_player_fights"
"EVENT_vandalism_player_fights"
"EVENT_trespassing_player_fights"
//If these dialog events guards leave the area, even during conversation
"EVENT_trespassing_guards_leave_peacefully"
"EVENT_thievery_guards_leave_peacefully" //also player gives back stolen stuff
"EVENT_vandalism_guards_leave_peacefully"
//Subtracts the fine from a player
"EVENT_thievery_player_pays_fine" 
"EVENT_vandalism_player_pays_fine"
"EVENT_trespassing_player_pays_fine"

/*   Define Keywords

*/

/////Story Call Guards///////////
/*   Notes
This Section tells you how to call guards within the story editor, Thanks Larian for this explanation
Story_CallGuards can be used to call the guards bypassing the usual system of thievery events
or attitudes. There are three versions
*/
Story_CallGuards((NPC)_NpcCallingGuards,(INTEGER)_Reason,(DIALOG)_CallGuardsDialog);
Story_CallGuards((NPC)_NpcCallingGuards,(INTEGER)_Reason);
Story_CallGuards((INTEGER)_Reason);
//	 All versions will check IF the guards can come to aid, and if so, they'll do. 
// In this case, the version providing a dialog as 3rd parameter will first run this dialog
// and summon the guards only after finishing the dialog. Version 2 and 3 skip the dialog.
// The difference between the first two and the last version is that the first two prevent
// than one npc calls the guards twice if they are still around. The 3rd version will always
// bring on the guards if you are in a guarded region.
//
// Note that if you want to have the guards react properly on thievery, you have to:
// 1) call DeleteThieveryFacts(1);
// 2) assert:
//					ValueStolen((INTEGER)_Value)
//					OwnerStolenGoods((NPC)_Npc)
//	 3) if an object is stolen assert as well:
//					ObjectStolen((OBJECT)_Object)
//	 4) if an object class is stolen assert as well:
//					ObjectClassStolen((OBJECT_CLASS)_ObjectClass,(INTEGER)_Amount)

/*   Define Keywords
(INTEGER)_Reason : see General for detail
*/

//-------------------------General--------------------------//
/*
General Game influencing stuff
*/
/////Time///////////
/*   Notes
Larian actually implemented a '24-hour' clock with a day counter. Didnt use it as far as I can tell
*/
/****Invovled Databases****/
Time((ITERGER)_Day,(ITERGER)_Hour,(ITERGER)_TotalHours);	//Atmosphere has no reaction, i think

/****Useful Procedures****/
SetGameClock((INTEGER)_NewHour);		//Moves foward to that time (may advance a day)

/*   Define Keywords

*/

/////Autosave///////////
/*   Notes
Have an auto save happen at a Trigger, only happens once
uses the AutoSave() call
*/
/****Invovled Databases****/
DB_AutoSaveTrigger((TRIGGER)_Trigger);

/****Useful Procedures****/

/*   Define Keywords
(TRIGGER)_Trigger  	: Autosave at Trigger
*/

/////Generic Quest Reward///////////
/*   Notes
In a poof of smoke creates an item at the trigger location
*/
/****Invovled Databases****/

/****Useful Procedures****/
ProcRewardQuestMedium((TRIGGER)_Trigger); //generates "Quest_Reward_Small_94e12298-5e59-405a-9e93-95833e648ce2"
ProcRewardQuestBig((TRIGGER)_Trigger);	 //generates "Quest_Reward_Big_b67596ec-18ca-4790-9273-8af23d8a7a43"

/////Effects///////////
/*   Notes
Upon Entering an Region All triggers for that region will begin and when leaving they will all turn off
Use the PROC_LoopEffectAt[Object] to add loops to 
*/

/****Invovled Databases****/

/****Useful Procedures****/
PROC_LoopEffectAtTrigger((STRING)_effect, (TRIGGER)_trigger,(STRING)_ID,(STRING)_Region);
PROC_LoopEffectAtCharacter((STRING)_effect, (CHARACTER)_character,(STRING)_ID,(STRING)_Region);
PROC_LoopEffectAtItem((STRING)_effect, (ITEM)_item,(STRING)_ID,(STRING)_Region);
PROC_LoopBeamEffectAtItem((STRING)_effect, (ITEM)_item,(CHARACTER)_Char,(STRING)_SrcBone,(STRING)_TargetBone,(STRING)_ID,(STRING)_Region);
//to deactivate a specific effect
PROC_StopEffectAtTrigger((TRIGGER)_trigger,(STRING)_ID);
PROC_StopEffectAtCharacter((CHARACTER)_character,(STRING)_ID);
PROC_StopEffectAtItem((ITEM)_item,(STRING)_ID);
PROC_StopLoopBeamEffectAtItem((ITEM)_item,(STRING)_ID);
//Zolton's Remove 
Proc_Z_RemoveEffectID((STRING)_ID); //Stops all Effects with _ID

/*   Define Keywords
(STRING)_effect 		: Name of the effect
(STRING)_Region 		: Name of the region you want effect to start in, can be marked as "__ANY__" to always play
(TRIGGER)_trigger 		: Trigger to play effect on
(ITEM)_item 			: item to play effect on
(CHARACTER)_Char 		: character to play effect on
(INTEGER)_fxHandle 		: Generated ID for effect, -1 in database means not playing
(STRING)_ID 			: how you want to ID this effect, multiple effects can share ID
(STRING)_SrcBone 		: 
(STRING)_TargetBone 	: 
*/

/////Freezing characters///////////
/*   Notes
This is an editor resourse useful for cutscenes.
Will freeze a character
uses CharacterFreeze, CameraLockOnNpc
The description needs work and some actual testing
*/
/****Invovled Databases****/
DB_FreezeLines((INTERGER)_Num, (STRING)_Text);

/****Useful Procedures****/
Hold((CHARACTER)_Npc[, (INTERGER)_Bool]);		//Aslo Displays a random text
ControlToNpc((CHARACTER)_Player,(CHARACTER)_Npc);
ShiftToNpc((CHARACTER)_Player,1);
ReturnControlToPlayer((CHARACTER)_Player,(INTEGER)_?);
ProcFreezePlayers();
ProcUnfreezePlayers();

/****Default Text names****/
/*
"GEN_ArrestFreeze_1"
"GEN_ArrestFreeze_2"
"GEN_ArrestFreeze_3"
"__LAST__"		Not actually a text name, is a cap to make it easier to add more Text
*/

/*   Define Keywords
(CHARACTER)_Npc 	: Character to be frozen
(STRING)_Text		: name of text, last number should be name "__LAST__"
(INTERGER)_Num 		: line number starting with (start at 0)
(INTERGER)_Bool		: 1->freezes (default), 0->unfreezes
*/

/////Object Timers///////////
/*   Notes
These are a bit funky, they use Database entries as events that pass objects
*/
/****Invovled Databases****/
ItemTimerFinished((ITEM)_Item,(STRING)_TimerName);
CharTimerFinished((CHARACTER)_Character,(STRING)_TimerName);
//Use as an event that passes an object ex:"IF ItemTimerFinished(_Item, (STRING)_TimerName) THEN" ex:CYS_Church_Door, KB(76)

/****Useful Procedures****/
ItemTimer((ITEM)_Item,(STRING)_TimerName,(INTEGER)_Time);
ItemTimerCancel((ITEM)_Item,(STRING)_TimerName);
CharTimer((CHARACTER)_Character,(STRING)_TimerName,(INTEGER)_Time);
CharTimerCancel((CHARACTER)_Character,(STRING)_TimerName);

/*   Define Keywords
(STRING)_TimerName 		: ID of timer
(INTEGER)_Time 			: Length of timer (ms)
(ITEM)_Item 			: Item passed into and out of Timer Finished Database
(CHARACTER)_Character	: Character passed into and out of Timer Finished Database
*/

/////Counters///////////
/*   Notes
I did a little cleaning, there was a random interger multiplication
*/
/****Invovled Databases****/
KillCounterCounts(_Name, _Count);
dbReflectionDialogs((STRING)_Type,(STRING)_Dialog);
dbTargetCounts((STRING)_Type,(INTERGER)_Target);
//Item destruction counter is never used in the game but its there, not quite finished from the looks of it
ItemDestroyCounter((STRING)_CounterDB,(INTEGER)_TargetCount); 
//when the number of destroyed items related to _CounterDB reaches the _TargetCount
//It cleans itself up and does nothing else. I added a GlobalEvent see special section
ItemDestroyCounterDB(_Item,_CounterDB); //If specified item is destroyed, counts down on the counter

/****Useful Procedures****/
ProcDeclareCounter((STRING)_Name);
ProcIncreaseCounter((TRIGGER)_Trigger[, (INTERGER)_Amount]);
ProcDecreaseCounter((STRING)_Name[,(INTEGER)_Amount]);

/*****Special****/
/* Since the item destruction counter did nothing I added a global event named in relation to the counter
"Event_[NameOfItemCounter]" bracketed section indicates the name of the counter
*/

/*   Define Keywords
(STRING)_Type 		: Name of counter
(INTERGER)_Target 	: target quantity of envents
(STRING)_Dialog 	: name of dialog to be started when counter reaches its target
(INTERGER)_Amount 	: Change counter by amount (default to 1)
*/

/////Stack///////////
/*   Notes
This is useful if you want to have certain things happen in order
FILO
*/
/****Invovled Databases****/
TopOfStack((STRING)_ID,(STRING)_Value);							//Top item of a stack
TopOfStack[Item|Character|Trigger]((STRING)_ID,(STRING)_Value);	//If you want the top Item, Character, or Trigger of a stack

/****Useful Procedures****/
Stack_Internal((STRING)_ID,(STRING)_Value);		//Push Items onto or create a new stack with this proc 
PopStack((STRING)_ID);							//Remove the top item from the stack and put the new top in TopOfStack

/*   Define Keywords
(STRING)_ID 		: name and ID of stack
(STRING)_Value		: UUID of item, example is in DAF_Druid_Room2
*/

/////Book Keeping///////////
/*   Notes
These aremostly generated databases
Book Keeping databases and procedures that are good to be aware of
Managed in AAA_FirstGoal
*/
/****Invovled Databases****/
DB_DialogPlayers((ITERGER)_DialogInst,(CHARACTER)_Player,(ITERGER)_Index);	//Player in a dialog and their index number
DB_DialogNPCs((ITERGER)_DialogInst,(CHARACTER)_Player,(ITERGER)_Index); 	//NPC in a dialog and their index number
DB_DialogNumPlayers((ITERGER)_DialogInst,(ITERGER)_NumPlayers);	//Number of players in a Dialog
DB_DialogNumNPCs((ITERGER)_DialogInst,(ITERGER)_NumNPCs);		//Number of NPCs in a Dialog
DB_OtherPlayersSee((CHARACTER)_Npc);				//Can another Player see this NPC
Sees((CHARACTER)_OtherPlayer,(CHARACTER)_Npc);		//List of who PC sees and which of those the sees the player character (invisible PC aren't seen)
DB_OtherPlayersInRegion((TRIGGER)_CheckTrigger,1);	//Is there another player in the Trigger area
DB_PlayerTriggers((TRIGGER)_Trig);					//Player triggers
InRegion((CHARACTER)_Npc, (STRING)_Region);			//NPC is in Region
WasInRegion((CHARACTER)_Npc,(STRING)_Region);		//NPC was in Region
Dead((CHARACTER)_Npc);								//List of dead NPCs
CurrentLevel((STRING)_Region);						//Current Level String
ObjectState((ITEM)_Object,(STRING)_State);			//Objects and current state ("in the world", "unreachable", "in npc backpack") 
OneShotPlayerTrigger((TRIGGER)_Trig);				//List of unactivated one time triggers include companions
OneShotPlayerOnlyTrigger((TRIGGER)_Trig);			//List of unactivated one time triggers for only players
DB_GlobalEvent((STRING)_String); 					//Stores all currently set global events
dbCombat((CHARACTER)_Character,(INTERGER)_ID);	/*database containing all characters currently in combat,
	Useful as a check for if a characcher is in combat ex: NOT dbCombat(_Character,_ID)*/

/****Useful Procedures****/

/****Character Status Text*****/
/**
Default Forrbidden item text: "GLO_ForbiddenItem"
*/

/*   Define Keywords

*/

/////Useless Stuff///////////
/*   Notes
Not sure why I have this section, I'm just putting those useless/unfinshed scripts I found in here
*/
/****Invovled Databases****/
DeadlockTimer((INTEGER)_ID,(STRING)_Timer);
TriggerDB((TRIGGER)_Trigger,(CHARACTER)_Character); //Seems to be an old trigger database, unsupported
DB_ZappedCounter((INTEGER)_Num); //Unused database I commented it out
DialogEvent_triggers_Attack(_DialogEvent,(CHARACTER)_Attacker);//Never set anywhere commented out event occurances that used it as a condition
Script(_Npc,"outside",(STRING)_,(INTEGER)_);//Seems to be unused databases
SelectScript(_Npc,"outside"); 		//looks like it was suppose to be a proc, no deffinition
DoRunaway_1Done(_Player,1); //also means this database is unnecessary
CanHeal((CHARACTER)_Npc); 		//Never seems to be set
NoIdentify((CHARACTER)_Npc); 	//Never seems to be set
NoRepair((CHARACTER)_Npc); 		//Never seems to be set

//Unfortunatly not much reacts to these and its never used
FamousObject(object);		// necessary because IF... NOT FamousObject(object,_,_) is not allowed
FamousObject(object, fame);		// fame: between 0 and 100
Relation(npc1,npc2);		// necessary because IF... NOT Relation(A,B,_) is not supported
Relation(npc1,npc2,R);	// R = 100: married couple, R>=90: best friend, R>75: good friends
							// R = 0: no relation (equivalent to no Relation/2,/3 facts asserted
							// R < 0... R == -100: sworn enemies.
PlayerHasThievesRelationWith(_Npc); //_Npc will allow you to steal stuff if the object is not too important to him.
BlackMarketTrader(_Npc);
ItemDB((ITEM)_Item, (STRING)_PlayerKnowsAboutItem, (STRING)_PlayerHasItem, (STRING)_Event); /*If you want to keep track of an item
(STRING)_PlayerKnowsAboutItem global event is set if a player ever picks up the item
(STRING)_PlayerHasItem character var interger(bool) is set if player has it, cleared if he loses it
(STRING)_Event if this global event is set it will give the item to a character in 
the database ItemToCharacterDB((ITEM)_Item, (CHARACTER)_Character, (INTEGER)_xp) and doesn't do anything
This was never used in the main game. 
 */

/****Useless Procedures****/
StartScriptFrameWhenPlayerOutsideRegion((CHARACTER)_Player,(CHARACTER)_Npc,(STRING)_ScriptFrame,(TRIGGER)_Region);
//Runs a script frame if and when a player is out of a Trigger area. Not sure exactly what this is it seems 
//to want to run StartScriptFrame(_Npc,_ScriptFrame) but its never implemented, so its a database?
DeadlockBreak((INTEGER)_ID,(INTEGER)_Timeout,(STRING)_Condition);
//I found DeadlockBreak that is partially commented out but I left it in there it seems to want to help break out
//of a camera grab after a certain ammount of time
BeginFight((CHARACTER)_Npc1,(CHARACTER)_Npc2);
//Sounded cool but its completely unimplemented. doesn't have any clean up or make anyone attack
TryCallHelp((CHARACTER)_Player,(CHARACTER)_Npc,(INTEGER)_Reason);
//It seems they tried to set up something to call friends and guards to aid NPCs, might be an old version
StopTradingAndGiveWarning((CHARACTER)_Player,(CHARACTER)_Npc);
//proc that us this proc are commented out, I commented it out too

/*   Define Keywords

*/


/////////////////////   Made by Zolton     //////////////////////////////