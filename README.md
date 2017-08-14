# libblueskies
## Overview
Hopefully a tool to turn 3DS Flipnote Studio 3D flips into something a regular PC can display.
Why's it called libblueskies? Well, my initial motivation for making this tool was to deserialise all of [mellowforests'](http://mellowforests.deviantart.com/) (previously known as 'Blueskies') flipnotes. And it's better than... "deflippifier".

*libblueskies* is in early development at the moment. Consequently, it is
far from viable to employ it for practical use. Bugs are everywhere, it's
slow (maybe), and badly documented (for now).
### Depencencies:
- Lua 5.3 or higher.
- sox, more likely.

### Goals:
- Implement src/indices.lua properly.
- Finish the binding function.
- Parse/store the other headers.
- Obviously, we need to decipher whatever kind of video format .kwz files utilise to display content to the screen.
- Write a sox pseudo-api.
## What has been implemented
Currently, a new flipnote object can be generated by calling `flipnote:new()` and supplying it a file handle. This returns a flipnote
object with every individual header found in `flip_obj.raw_header` (e.g. `KSN`- sound data, can be found in `flip_obj.raw_header`).
Individual fields have also been parsed appropriately, and can be accessed as ordinary lua values:
```lua
-- flipnote:new() calls :close() on the given handle.
local flip_obj = flipnote:new(io.open("file.kwz","r"));
local KFH=flip_obj.header.KFH;

print(KFH.creator_id); -- => 0x122599F5A060D29A5500
```
An iterator may be generated by **calling** the header:
```lua
for key,value in KFH() do
	-- Whatever...
end
```
For a list of which value corresponds to which field, see `src/indices.lua`.
## What is currently being implemented
The `KMI` header contains frame data separated into 28 bytes for each frame- a new handler shall be written that allows one to
navigate frame data by `KMI[frame_number].field` and such.
The `KSN` and `KFH` headers are the only two headers that currently have metatables assigned to them- the rest needs be tested still.

`__newindex` metamethods need still be implemented that allow individuals to add/alter data found in the various headers
(including specialised hooks that modify checksums, header length, etc.) This will be done in the near future.

## proof_of_concept.lua
`proof_of_concept.lua` accepts a path to a flipnote (handed on the command line), rips the `KSN` data (background music, sound effects, etc.) and outputs these into the files `bgm_data.pcm` and `se{1..4}_data.pcm`. It also prints a serialised lua table to
`stdout` that describes the contents of the `KFH` header.

The PCM data can be wav-ified by using `sox`: `sox -N -t ima -v 1 -r 16500 INPUT_FILE OUTPUT_FILE.wav`
(setting `-v` to a real number to use as the volume.) In the current implementation, sox assumes this is `4-bit IMA-ADPCM` at
`16.5 kilohertz`. This does still sound rather choppy and off- this needs to be fixed.

Note that for the output for the UTF-16 fields on mellowforests' flipnotes, `☆zrms™☆`
is outputted (the UTF-16 is little-endian). Whereas the `Nintendo 3DS`
displays the `PRIVATE USE  ()` sign as `CIRCLED LATIN CAPITAL LETTER A
(Ⓐ)`. The UTF-16 hex code in the files remains `0xe000`. Either this is an
odd quirk, or I'm a giant idiot...
## Licensing and attribution.
The entirety of this repository is registered under the GNU GPL v3.0+ (see `LICENSE` for details), **EXCEPT FOR:**

### flipnote-collective
the [Flipnote Collective](http://github.com/Flipnote-Collective/) have done an absolutely amazing job at documenting the various aspects of Flipnote Hatena 3Ds'
workings. They have written extensive documentation on the subject [here](http://github.com/Flipnote-Collective/flipnote-studio-3d-docs).
Currently, `src/indices.lua` semi-directly rips some of the lookup tables and information they have on display. The lookup
tables in this file are registered under a [**Creative Commons Attribution-ShareAlike 4.0 International License**](https://creativecommons.org/licenses/by-sa/4.0/).
This is noted in the aforementioned file as well.

### mellowforests' samples
The flipnotes used for samples/analysing in `sample/` belong to [mellowforests on deviantart](http://mellowforests.deviantart.com/)
and [youtube](https://www.youtube.com/user/Blueskiez14) ("Z", Zara, etc.). The animations (but not the music or sound) contained within belong to them (even if nobody can actually see them
appropriately, yet.) When redistributing these for any reason whatsoever- attribute the
__animations__ (contents of the entire file, sans anything included in the `KSN` header to [mellowforests](http://mellowforests.deviantart.com/).
You shall not claim these for yourself.