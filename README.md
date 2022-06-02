# FarmSimulator

Developed by G3P with Unreal Engine 5, using assets from the Unreal Marketplace.

Styled using the [Gamemakin UE5 Style Guide](https://github.com/Allar/ue5-style-guide), with some naming convention
changes like `EM` for Enumeration and `ST` for Structure.

----

## AI System

### Features

Every NPC in the game is controlled by their own behavior tree and tries to find, reserve, navigate to, and work on their own tasks from a list of filterable avaliable tasks around them. It is not efficient for finding non-random farm tasks, but it is a trial of making AI systems. Each NPC belong to a house and each house have a limit of max number of attachable NPCs. There is also a limit of max number of houses.

NPCs handle:
- [ ] Building of game objects
- [x] Harvesting of mature crops
- [ ] Picking fruit from trees
- [ ] Watering of crops

### Technical Details

- The task management system in-game uses Unreal Engine 5's newest beta testing feature - Smart Objects
	- A Smart Object is spawned by Game Objects that offer tasks like matured farm tiles, and can be filtered and discovered by AI agents
	- The avaliable agents scan around them and locate a random but closer object, and tries to reserve the slot. A slot can be claimed by multiple agents but only one reservation will be valid
	- If reservation successed, the AI agent will navigate to the slot and get information from the Smart Object about how to operate it ("behavior")
	- After the AI agent completed the behavior, or during the behavior when a notify completes the behavior early, additional code is triggered to make changes to the world as result of the "behavior" of the AI agent
	- After a successful task search and completion, the AI agent searches again in 2 seconds to allow animations to complete etc. If the task searching or completion failed, the AI agent wait for 5 seconds to start another round of searching, to reduce average performance impact of the search

## Grid System

![Grid System](https://user-images.githubusercontent.com/73323107/169708290-ea0854b4-0a4f-4de9-99c1-45a1cafc1b2f.gif)

### Features

- Place down objects in grid cells
- Object will occupy cell preventing another placement
- Temporary: Select objects/crops to place with 1-9 keys
- Show placement preview in placement mode
	- Hints if placement is valid/invalid
	- When planting crops, preview shows coresponding crop products instead of planted crops
	- Placement verification different for different Game Object types
	- Placement preview accuratly show validity and position
- Hover mouse over a placed object to highlight it
	- WIP: showing corresponding stats
- Click a placed object to select (different highlight) it, click again to unselect
	- Press backspace to delete selected objects
	- WIP: showing available actions

### Technical Details

- Line trace for placement target each tick if in placement mode
	- No actions if no change in calculated grid position
- Grid Manager contains array of ~~generated grid cell actors~~ Game Objects which index is the grid coordinates, Grid Count per row/column, and Cell Width
- Grid Manager also acts like a function library for functions like getting closest Grid Cell position or converting grid coordinates to world coordinates
- ~~Each Grid Cell keeps:~~
	- ~~Whether it is occupied~~
	- ~~Linked Game Object type enum~~
	- ~~Linked Game Object reference~~
	- ~~Neighboring Grid Cell references~~
	- ~~Grid Cell X and Y coordinates~~
- Each Game Object keeps:
	- ~~Linked Grid Cell reference~~
	- Game Object type enum
	- Sizes in each direction
	- Grid Coordinates
- ~~Clickable component for Game Objects, if added, manage mouse hover and click~~ (migrated to Game Object Base)
- ~~In Placement component for Game Objects, added to object placement previews, validifies object placement based on selected object/crop, changes mesh materials to hint valid/invalid placements~~ (migrated to Player Controller)

## Crops Sub-system

![Crop System](https://user-images.githubusercontent.com/73323107/169707754-7cc74dfd-b204-42e7-b150-3c3cc7305e08.gif)

### Features

- Place down crops on farm tile objects
- Can only plant on empty farm tiles
- Dictinct types of crops has:
	- Customizable number of stages
	- Different mature time requirement
	- Auto-calculated required time for each stage
	- Randomized growth rates
- Individual plants generated with random position, scale, and rotation to reduce repetitiveness
- Plants moves with wind
- Show planting preview of each plants

### Technical Details

- Crop Manager keeps array of currently occupied farm tiles
	- Increments groth progress randomly each second, notify the farm tiles to update crop meshes as groth process increases
- Crop details stored in customizable data table, adding a crop variant is less than 1 minutes of work
- Used animated material based on render position and time to calculate transforms for vertex positions
- Very easy-to-expand datatable for introducing new types of plants or altering plant properties
	- Included fields are:
		- crop name
		- number of growth stages
		- meshes for each growth stage
		- maximum "matureness" `(Crop Manager increment similar amount for each crop types, but each crop type have different max matureness causing them to mature quicker/slower and have shorter/longer stages if they have the same stage count)`

## Save Game System

![Save System](https://user-images.githubusercontent.com/73323107/169708155-dd83fa79-e4da-4f59-8cde-9839fc22b3c0.gif)

### Features

- Save current game
	- Grid properties
	- Basic Game Objects
	- Planted Crops and Crop growth
	- NPC locations and linked building coordinates
	- Player transformations
- Automatically load saved game
- Deleating saved game
- WIP: Save games to different slots
- Restart level with saved game

### Technical Details

- Data being saved:
	- Grid width and Cell width for Grid population
	- Array of Placed Game Objects
		- Game Object Class
		- Linked Grid Cell's X and Y coordinates
	- Array of placed crops
		- Crop Type Enum
		- Current Matureness
		- Linked Grid Cell's X and Y coordinates
	- Player transformation - not including zoom level etc., yet
- When repopulating world:
	- Spawn Grid Manager
	- Re-create Grid and neighbor connections based on stored values
	- Spawn Crop Manager
	- Re-spawn new Game Objects with Grid Cell's transformation
	- Re-plant crops and grow to recorded matureness, manager registration and stages calculation done by same function
	- Re-apply player transforms
	- Start Crop Manager growth
- Each crop plant's random transformation will be regenerated randomly

## Camera Controller

### Features

- Move camera with WASD
- Orbit and Pitch camera with QE, RF, and mouse right drag
- Zoom in with Left Shift, zoom out with Space
- Dynamic move speed based on zoom level
