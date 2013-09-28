NPC Framework for Minetest 0.4.8
--------------------------------
This mod adds some, hopefully useful, non-player characters to the minetest game.
The mod also provides a framework for others to create and manage their own custom NPCs.

Features currently include overhead name tags, formspec handling, ownership and management,
file based backups and automatic duplicate entity removal. (experimental)

These NPC's are not craftable although by default will be available in the creative menu.
Servers would be encouraged to override this default and allocate the NPCs on /give basis.

### Info NPC

The Info NPC is a simple information serving character. You could think of them as a
human book providing information about a particular server location, or whatever else you like.
Supports multiple pages of text. 12 lines per page, ~50 chars per line.

### Deco NPC

A purely decorative NPC, can be set to roam freely and/or follow random players it encounters.

### Guard NPC

Protect yourself and your property against other players and mobs. Features 3d weapon and armor.
Can be left to guard a certain area or set to follow their owner.

### Trader NPC

Provides a quantity based exchange system. The owner can set a given number of exchanges.
This would likely be best used in conjunction with one of the physical currency mods.

	Buy [default:mese] Qty [1] - Sell [default:gold_ingot] Qty [10]
	Buy [default:gold_ingot] Qty [20] - Sell [default:mese] Qty [1]

Note that the NPC's owner cannot trade with their own NPC, that would be rather pointless anyway.

### Builder NPC

Not really much point to this atm other than it's really fun to watch. By default, it be can only
build a basic hut, however this is compatible (or so it seems) with all the schematics provided by
Dan Duncombe's instabuild mod. These should be automatically available for selection if you have
the instabuild mod installed.

Server Commands
---------------

Server commands are issued as follows.

	/npcf <command> [npc_name] [args]

### list

List all registered NPCs and display online status.

### clearobjects

Clear all loaded NPCs. (requires server priv)

### loadobjects

Reload all currently unloaded NPCs. (requires server priv)

### getpos <npc_name>

Display the integer position of the named NPC.

### reload <npc_name>

Reload an unloaded NPC. (requires ownership or server priv)

### clear <npc_name>

Clear (unload) named NPC. (requires ownership or server priv)

### delete <npc_name>

Permanently unload and delete named NPC.  (requires server priv)

### setskin <npc_name> <skin_filename|random>

Set the skin texture of the named NPC.

	/npcf setskin npc_name character.png

If you have Zeg9's skins mod installed you can select a random texture from said mod.

	/npcf setskin npc_name random

### setpos <npc_name> <pos>

Set named NPC location. (x, y, z)

	/npcf setpos npc_name 0, 5, 0


API Reference
-------------
Use the global npcf api to create your own NPC.

	npcf:register_npc("my_mod:my_cool_npc" ,{
		description = "My Cool NPC",
	})

This is a minimal example, see the NPCs included for more complex usage examples.

### npcf

The global NPC framework namespace. Other global variables include.

	NPCF_ANIM_STAND = 1
	NPCF_ANIM_SIT = 2
	NPCF_ANIM_LAY = 3
	NPCF_ANIM_WALK = 4
	NPCF_ANIM_WALK_MINE = 5
	NPCF_ANIM_MINE = 6
	NPCF_ANIM_RUN = 7
	NPCF_SHOW_IN_CREATIVE = true
	NPCF_SHOW_NAMETAGS = true
	NPCF_BUILDER_REQ_MATERIALS = false
	NPCF_DECO_FREE_ROAMING = true
	NPCF_GUARD_ATTACK_PLAYERS = true
	NPCF_DUPLICATE_REMOVAL_TIME = 10

All of the above can be overridden by including a npcf.conf file in the npcf directory.
See: npcf.conf.example

### npcf:register_npc(name, def)

Register a non-player character. Used as a replacement for minetest.register_entity, it includes
all the callbacks and properties (at the time of writing) available there. The only exceptions
are the get_staticdata callback (used internally) and you are currently not able to create arbitrary
property variables, instead the framework provides 'metadata' and 'var' tables for those purposes.
The metadata table is persistent following a reload and automatically stores submitted form data.
The var table should be used for non-persistent data storage only. Note that self.timer is
automatically incremented by the framework but should be externally reset.

Additional properties included by the framework. (defaults)

	on_registration(self, pos, sender),
	on_construct = function(self),
	on_receive_fields = function(self, fields, sender),
	animation = {
		stand_START = 0,
		stand_END = 79,
		sit_START = 81,
		sit_END = 160,
		lay_START = 162,
		lay_END = 166,
		walk_START = 168,
		walk_END = 187,
		mine_START = 189,
		mine_END = 198,
		walk_mine_START = 200,
		walk_mine_END = 219,
	},
	animation_speed = 30,
	armor_groups = {immortal=1},
	inventory_image = "npcf_info_inv.png",
	show_nametag = true,
	nametag_color = "white",
	metadata = {},
	var = {},
	timer = 0,

### npcf:set_animation(luaentity, state)

Sets the entity animation state.

	on_activate = function(self, staticdata, dtime_s)
		npcf:set_animation(self, NPCF_ANIM_STAND)
	end,

### npcf:get_index()

Returns a table of all registered NPCs. (loaded or unloaded)

	{[npc_name] = owner_name, ... }

### npcf:get_luaentity(npc_name)

Returns a minetest ObjectRef of the npc entity.

### npcf:get_face_direction(v1, v2)

Helper routine used internally and by some of the example NPCs.
Returns a yaw value in radians for position v1 facing position v2.

### npcf:get_walk_velocity(speed, y, yaw)

Returns a velocity vector for the given speed, y velocity and yaw.

### npcf:show_formspec(player_name, npc_name, formspec)

Shows a formspec, similar to minetest.show_formspec() but with the npc_name included.
Submitted data can then be captured in the NPC's own 'on_receive_fields' callback.

Note that Form text fields, dropdown, list and checkbox selections are automatically 
stored in the NPC's metadata table. Image/Button clicks, however, are not.
