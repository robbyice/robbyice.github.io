---
title:  "From Tiles to Tracks: Runtime Track Generation"
date:   2026-07-27 12:00:00 -0700
categories: art games engineering
header:
  teaser: /assets/images/blog/2_etu_track_generation/angled_track_shot_2.jpg
---

{% assign prev_post = site.posts | where: "title", "From Vector Art to Neat Arrangements of GameObjects" | first %}
[Last time]({{ prev_post.url }}) we looked at how the tile content for Eat The Universe (ETU) is generated from vector art assets, allowing the team to quickly create hundreds of interesting arrangements and shapes of obstacles for tiles to be used in levels. Today we'll look at the systems that build a track from tiles and how we managed to get unique and replayable levels.

Reviewing the core concepts here:

- **Track:** The track the player character moves along as the game progresses
- **Tile:** A tile is a container for some arrangement of obstacles. Tiles are placed on the track as the player moves along the track.

<div class="post-figure-pair">
  <figure class="post-figure">
    <img src="/assets/images/blog/1_etu_tile_pipeline/shapes_in_tile.jpg" alt="Example tile with an arrangement of obstacles" loading="lazy">
    <figcaption>Example tile with an arrangement of obstacles</figcaption>
  </figure>
  <figure class="post-figure">
    <img src="/assets/images/blog/1_etu_tile_pipeline/track_example.jpg" alt="Example track with different tiles" loading="lazy">
    <figcaption>Example track with different tiles</figcaption>
  </figure>
</div>

<figure class="post-figure">
  <img src="/assets/images/blog/1_etu_tile_pipeline/shapes_top_down_2.jpg" alt="Generated tile arrangements" loading="lazy">
  <figcaption>Here's an example of some of the tiles we ended up generating.</figcaption>
</figure>

After generating the tiles, we organized them into themes, each of which was assigned to a Galaxy. So each Galaxy is effectively just a themed set of levels with a particular environment — for example, one galaxy is all spirals, curves, circles, etc; another is square and rectangle shapes, so on and so forth.

This set of themed tiles is sorted by number of object positions (points) they contain, which is a rough approximation of "difficulty," and then distributed amongst the levels in the themed galaxy. The idea being if you're moving through levels in "Blockbreaker Belt" (squares, rectangular shapes), you'd on average see simpler square obstacle arrangements at early levels and more complex ones at a higher level.

<figure class="post-figure post-figure--phone">
  <img src="/assets/images/blog/2_etu_track_generation/galaxies_panel.jpg" alt="Galaxies panel" loading="lazy">
  <figcaption>Galaxy and level selection panel</figcaption>
</figure>

We knew we wanted each level to have at least a couple different tile arrangements, but not every theme had a large number of unique arrangements — so there is some repeat and overlap for tiles between levels. Basically, tiles within a theme are assigned to levels using a moving window on the theme's tile list (sorted by difficulty). All of this is done automatically by our scripts — the sorting of tiles into themes, building the levels and level content, and organizing everything into galaxy and level ScriptableObject assets.

<figure class="post-figure">
  <img src="/assets/images/blog/2_etu_track_generation/level_tile_content_1.png" alt="Level tile content data" loading="lazy">
  <figcaption>An early level has very simple tiles - basic lines and caret signs</figcaption>
</figure>

<figure class="post-figure">
  <img src="/assets/images/blog/2_etu_track_generation/level_tile_content_2.png" alt="Level tile content data" loading="lazy">
  <figcaption>A later level with denser arrangements - zig zags, lightning bolts, etc.</figcaption>
</figure>

So at this point we've got a level and maybe 3-5 tiles for building the track. Just randomly sampling the tiles we have will do a decent enough job of randomizing the tracks and making them more interesting, but we wanted to take it a bit further. So we introduced a few special types of tiles that are built and work differently from the stuff we've seen so far from the tile pipeline. These tiles are more dynamic — they are generated during gameplay rather than an asset built ahead of time.

## Compound Tiles

Any time the track fetches a tile to be placed, there is a chance that tile is a special compound tile. Compound tiles are runtime-generated tiles that, as you might expect, contain some number of sub-tiles. Any tiles within the compound tile are placed flush with each other, rather than following the usual spacing mechanisms. The goal for compound tiles is twofold — firstly to allow for more intense moments in gameplay with denser areas of objects and obstacles, and secondly to increase the overall variety and potential difficulty of tiles placed on the track.

<figure class="post-figure">
  <img src="/assets/images/blog/2_etu_track_generation/compound_tile_2.jpg" alt="Compound tile example" loading="lazy">
</figure>

The fact that the tiles are placed flush together in particular lets us do some cool stuff with "seamless" starting tiles. For example, we can make one tile the first piece of a zigzag, and if set up correctly to be "seamless," we can use compound tiles to create a zigzag arrangement of obstacles of a dynamic and configurable length. Pretty neat!

<figure class="post-figure">
  <img src="/assets/images/blog/2_etu_track_generation/compound_tile_1.jpg" alt="Zigzag compound tile" loading="lazy">
  <figcaption>This compound tile is using two different "zigzag" pieces so it's not quite seamless, but you get the idea…</figcaption>
</figure>

## Solar Systems

<figure class="post-figure">
  <img src="/assets/images/blog/2_etu_track_generation/angled_track_shot_1.jpg" alt="Solar system tile on the track" loading="lazy">
</figure>

One other special type of tile that can be selected for placement on the track is a Solar System tile. These are what you'd expect, tiles with a central star and orbiting concentric rings of planets and asteroids. Similar to compound tiles, these are dynamically built during gameplay rather than being prefab assets built through the tile building pipeline.

It's also worth noting that these tiles serve a distinct and important gameplay purpose — they are the only tiles that spawn planet and star objects, which players will try to consume once they've reached a certain size in order to further their growth and eventually get temporary special powers from eating stars.

The arrangement of the Solar System — i.e. how many objects, what types of objects, how many rings, the radii of the rings, etc — is not random. To ensure that the visual presentation of solar systems looks good, Solar Systems are built from a configuration ScriptableObject asset that provides all needed parameters. So there's a bit less dynamism here; we basically have Solar Systems that are prescribed to certain game states, and we just serve a system that is configured to look good in the given state. More explicitly, we serve a more voluminous and overall denser system the larger the player character is. So when the player is at their smallest size and can't even eat planets yet, Solar Systems are much smaller and will have few or no planets orbiting them.

<figure class="post-figure">
  <img src="/assets/images/blog/2_etu_track_generation/solar_system_config.jpg" alt="Solar system configuration" loading="lazy">
</figure>

There's *some* configurability for when and how often Solar Systems appear, but the systems and their content are relatively fixed and simple.

Solar Systems are one of the most important and probably interesting parts of the gameplay, but overall the systems behind them stayed pretty simple and were never iterated on much. I think improving Solar Systems to make them even more dynamic and interesting would provide some relatively cost-effective value to the game. Maybe one day!

<figure class="post-figure post-figure--phone">
  <img src="/assets/images/blog/2_etu_track_generation/solar_system_end.jpg" alt="Solar system in action" loading="lazy">
  <figcaption>The player maneuvers around enemy objects through the system to eat planets and hopefully the star!</figcaption>
</figure>

So, just to review, here's all the systems and tools we've covered that help enable Eat the Universe to have dynamic procedural tracks:

1. **Vector Art To Tile Pipeline**
   1. Scan resources for vector art, for each art asset take vector art path data, tessellate and sample positional data, persist positional data as prefab GameObject named for the shape in the art
2. **Level Content Generation**
   1. Given our hundreds of generated tile GameObjects, take all the tiles, organize them into themes by keyword identifiers, build galaxy and level data assets, distribute sorted tile content across all levels
3. **Runtime Track Generation**
   1. Given a level with some number of tiles, randomly polls for tiles to place on the track. Sometimes polling will return a Solar System or Compound tile, which is then built dynamically either from level content (Compound) or additional data assets (Solar System)

Of course, there are a few more systems on top of this, but I'll leave it here for now.
