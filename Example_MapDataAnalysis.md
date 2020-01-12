# Example: Parsing Ragnarok Online map data

This is a short example to show how to interpret the data that is included in a typical set of map files (GAT, RSW, GND). Only a few tiles will be interpreted to showcase the general approach that can be used to programmatically parse the various data files. I'm using the Lutie field map, xmas_fild01, but the approach is identical for any other map found in the client files.

Note: Correctly interpreting the blockType of a given tile, which is stored in the map's GAT file, requires information about the water level of the map, which is however stored in the RSW file instead. For illustrative purposes, this example will proceed one step at a time, so some backtracking is required to make sense of the blockType after we have parsed the RSW file.

For more info about the technical details, please see the respective file type's specification that can also be found in this repository.

## Tools

* A hex editor of your choice: I used [HxD](https://mh-nexus.de/en/hxd/)
* Basic heightmap generator: [Very basic version I created](https://github.com/SacredDuckwhale/RagnarokTools/)

## GAT: Altitude data and map dimensions

First we shall determine the dimensions (size) of the given map, and the general layout in terms of walkable tiles. This is by far the easiest part, although there are a select few things to keep in mind later on.

Opening a GAT file reveals the binary data, here represented as hexadecimal numbers and regular strings:

![Binary data contained in a GAT file](https://i.imgur.com/ULz9RNF.png)

At the beginning we recognize the GAT header (green border), then the map dimensions given in number of tiles (red), and then the individual tile data entries (blue). Since there are a LOT of tiles in each map, but the format is always identical, I've only drawn a border around the first set here.

The header is a fixed string ``GRAT\001\002`` and exactly as it should be, so the RO Client will assume the file contains valid altitude data. The last two bytes, ``01 02``, are probably a version number of sorts, but at least for GAT files there only appears to be a single version from what I've seen. For other file formats we need to be a bit more careful, as some come in different versions and may contain additional data fields in the more recent ones, but we shall worry about that when we get there.

The map dimensions are identical, which is common (though by no means required). This doesn't mean the walkable map area (as experienced ingame) is square, just that the underlying structure is. As we will later see, the walkable area of this map is in fact very small and not at all square, with much of the map being scenery to make it seem less like the glorified corridor leading to Lutie and the Toy Factory that it is.

Since the numbers are stored in little-endian, we need to reverse the byte order before interpreting the numbers as binary/hexadecimal data. Therefore, the bytes ``18 01 00 00 `` are reversed to ``00 00 01 18`` and can then be interpreted as 32-bit unsigned integers. This is fairly straight-forward (see [converting hexadecimal numbers to the decimal system](https://teachcomputerscience.com/converting-hexadecimal-to-decimal/)) and the result is the decimal number ``280``.

The interpretation is that the map has a width and a height (since both values are equal) of 280 tiles. You'll generally notice if the result is wrong, since all maps are in the ballpark of 200-400 tiles.

Somewhat more complex is determining the properties of a given tile. Let's take the first one (blue in the screenshot), which is the tile at the bottom left corner, i.e., at coordinates 0,0. We can already see that all four corners are at the same altitude, which means there is no slope. This is expected, considering a tile at the very corner of the map will likely be a filler/border of some sort and not part of the actual terrain.

The format here is a bit tricky, since the numbers are stored as single-precision floating points (see [this calculator](https://www.h-schmidt.net/FloatConverter/IEEE754.html)). Because they're also little endian, we have to reverse the byte order once more, and the bytes ``00 00 20 42`` become ``42 20 00 00 ``. This is the decimal value ``40.0``, meaning that the tile is at an altitude (height) of 40.

It should be noted that the interpretation of this value is entirely arbitrary, since no unit is given. However, as we will see in a moment, this value does correspond to what is called a height vector, i.e., the distance from an arrow of length 40 to the origin (ground level, zero) in direction of what will we would call the "sky" ingame, in an arbitrary unit.

By looking at the average altitude for a given tile we can therefore determine the height of the map's terrain without worrying about potential slopes. We could interpret each tile's corner as a separate tile, and connect them to form a block (tile), which more closely resembles the internal representation as a mesh (graph connected by lines). This gives us a much more fine-grained representation of the terrain, but it would ultimately not change the outcome very much and help us understand the format better.

Instead I have generated the heightmap using a simple average calculation, which is more than sufficient:

![https://i.imgur.com/jHQSHtF.png](https://i.imgur.com/jHQSHtF.png)

It's a bit blurry because the terrain is quite elated, but we can clearly see how it represents the hills and mountains that frame the scenery, while opening up at ground level so that players can travel north.

Now only the block type is left, and since we haven't determined the map's water height yet (see below) we will simply note that the bytes ``00 00 00 01`` (again, reversed) will trivially convert to the decimal number ``1``. Apparently, the tile is a basic block that is either walkable (if underwater) or not walkable (if ground level). The latter will most likely be true for a tile at the very edge of the map, but we'll see about that when we get there!

Ignoring the whole underwater/ground problem, it is possible to visualise the walkable tiles (excluding water) in a similar fashion:

![https://i.imgur.com/G1ieer8.png](https://i.imgur.com/G1ieer8.png)

The green area is a regular walkable block, and the black area represents ground tiles that are not walkable. While there are seven possible terrain types which make up the three categories (ground, cliff, water), this map only uses two of them. All this is still the same info extracted from the GAT file, which hopefully should be well-understood by this time.

For a better view on the actual geometry and especially the water, we'll have to take a look at the remaining file types. I've uploaded a few more terrain maps here, and height maps here, but that's really all there is to the GAT file type.

## RSW

TODO (Work in Progress)

## GND

TODO (Work in Progress)
