# Design

The AI automatically generates a layout for the room and builds the structures
for the current RCL. A `scout` creep or observer explores external harvest rooms.
New rooms are acquired if the current number doesn't fit the GCL. Fallen
rooms will be survived and basically defended. The AI sends some waves of
auto attacks. Minerals are fetched from the extractor and transported to the
terminal. Reactions are basically implemented. Depending on a threshold minerals
are sold on the market.

## [Base building](BaseBuilding.md)

### Logic

The number of structures are checked and if applicable new constructionSites
are places. Links are triggered to transfer energy to link near the storage.
Towers attack incoming creeps or heal my creeps. If no spawn is available
`nextroomer` from other rooms are called, to build up the room.

The basic creep is the `harvester` which can make sure, that enough energy
will be available to build the rest of the creeps. For this we check if
a `harvester` is within the room, otherwise spawn it. For the rest a priority
queue is used.


## Role

 - `upgrader` get energy from the storage, puts it into the controller.
 - `filler` get energy from a link and transfers it to the tower or storage.
 - `sourcer` get energy from source.
   - Controlled room: Transfers the energy to the link.
   - External room: Builds container, fills container, calls `carry` to get
   the energy.
 - `reserver` reservs an external controller and calls `sourcer`.
 - `carry` gets energy from the target container and fills structures and
 storage on the way back. If there is a creep in front the energy is transfered.
 - `scout` Breadth-first search based room exploring.
 - `harvester` moves on the harvester path, and transfers energy to free structures
   on the path. On low energy in storage, the `harvester` falls back to the
   start up phase without relying on anything (storage, links, other creeps).
 - `nextroomer` moves to target room and builds up that room.
 - `repairer` build walls and ramparts.


## Routing

The routing from `start` to `end` is first done on room level:

 - Find `Game.map.findRoute(start, end)` plus the start room added as first
   entry in the array. This is stored together with the `routePos` in memory
   of the creep.
 - Inside a single room:
   - First room (`routePos == 0`): Own rooms have a layout set with a path to
     each exit pre-calculated. The first part of the path name is `Start` and
     the second the room to move to.
   - Last room (`routePos == route.length -1`): the `targetId` is stored in the
     memory of the creep. So the first part of the path name is the previous
     room and the second part is the `targetId`.
   - Rooms on the path: The previous room is the first part of the path name,
     the next room is the second part of the path name.
   The path is cached in the memory of the room with a `created` attributes
   to allow invalidation.
