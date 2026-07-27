---
title:  "From Vector Art to Neat Arrangements of GameObjects"
date:   2026-06-20 12:25:30 -0700
categories: art games engineering
header:
  teaser: /assets/images/blog/1_etu_tile_pipeline/shapes_in_tile.jpg
---

So we're going to start the blog off by talking a bit about a feature I worked on for a previous game, Eat the Universe. ETU is sort of an "on-rails" runner arcade game, where the player character is automatically moving forwards along a track and trying to avoid certain objects and gather others.

<figure class="post-figure post-figure--phone">
  <img src="/assets/images/games/etu/gameplay2.jpg" alt="Eat the Universe gameplay" loading="lazy">
  {% assign etu = site.games | where: "title", "Eat The Universe" | first %}
  <figcaption>Eat the Universe, now available on <a href="{{ etu.app_store_url }}" target="_blank" rel="noopener noreferrer">iOS</a> and <a href="{{ etu.play_store_url }}" target="_blank" rel="noopener noreferrer">Android</a></figcaption>
</figure>

The track is composed of tiles that are dynamically placed during runtime as players move through the scene. A tile is a container for some arrangement of obstacles — for example one tile might have a set of asteroids in the shape of a circle, another is shaped as two interwoven triangles. As players move along the track, tiles are placed in front of them, and a given level has an ordered set of tiles that increase in complexity/difficulty/obstacle count. As the level progresses, tiles from the level contents are randomly selected for placement via a moving window.

<figure class="post-figure">
  <img src="/assets/images/blog/1_etu_tile_pipeline/shapes_in_tile.jpg" alt="Example tile with an arrangement of obstacles" loading="lazy">
  <figcaption>Example tile with an arrangement of obstacles</figcaption>
</figure>

<figure class="post-figure">
  <img src="/assets/images/blog/1_etu_tile_pipeline/track_example.jpg" alt="Example track with different tiles" loading="lazy">
  <figcaption>Example track with different tiles</figcaption>
</figure>

Of course, to get the most out of this infrastructure, tiles themselves need to be interesting enough to actually leverage this system. Having interesting arrangements of obstacles for players to navigate through, consume, avoid, etc. was an important part of the gameplay thesis.

At the beginning, these arrangements of obstacles existed as prefabs, and creating each obstacle set prefab meant you were manually creating and positioning some number of child objects into some interesting shape or pattern, and then saving that object as a prefab to be used later. Obviously this was kind of a pain to do at any sort of scale and for increasingly complex arrangements, and so I investigated some alternate solutions.

One neat thing I learned while doing so is that the Unity Inspector supports assigning mathematical expressions to the numerical properties of selected object sets. So you can select 5 game objects in the scene and type `L(-5, 5)` into their X position to space them linearly along the X axis between -5 and 5. This was fun to mess around with briefly and allowed for easy creation of some basic shape arrangements, but wasn't practical for increasingly complex targets. Additionally, we wanted a solution that was a bit more designer-facing and designer-friendly.

<figure class="post-figure">
  <img src="/assets/images/blog/1_etu_tile_pipeline/shapes_moons.jpg" alt="Moon-shaped obstacle arrangement" loading="lazy">
  <figcaption>How to enable the team to make lots of stuff like this quickly?</figcaption>
</figure>

Well, after some research and iteration, we ended up with a solution involving vector art and Unity's Vector Graphics package. The system I created ended up taking simple vector art assets, deconstructing the vector art into a series of points, and then sampling those points to end up with a set of positional data that captures a 'stippled tracing' of the input vector art.

<figure class="post-figure post-figure--small">
  <img src="/assets/images/blog/1_etu_tile_pipeline/vector_art_source.png" alt="Inbound vector art" loading="lazy">
  <figcaption>Inbound vector art</figcaption>
</figure>

<div class="post-figure-pair">
  <figure class="post-figure">
    <img src="/assets/images/blog/1_etu_tile_pipeline/vector_art_imported.jpg" alt="Spawning objects with default settings" loading="lazy">
    <figcaption>An example of spawning objects with some default settings</figcaption>
  </figure>
  <figure class="post-figure">
    <img src="/assets/images/blog/1_etu_tile_pipeline/vector_art_imported_adjusted.jpg" alt="Spawning objects with adjusted settings" loading="lazy">
    <figcaption>After adjusting some settings to allow for the amount of space we need between positions. Still not perfect, but much better!</figcaption>
  </figure>
</div>

This example imported version here is a bit too densely packed right now, but the script has lots of configurability in how it samples and builds the resultant positional data.

<figure class="post-figure">
  <img src="/assets/images/blog/1_etu_tile_pipeline/generator_inspector.png" alt="Generator inspector settings" loading="lazy">
</figure>

While working on this, we discovered some limitations that required consideration. Firstly, we needed to ensure that our resultant set of sample positions had a sufficient amount of space between each point to accommodate the objects of varying sizes that could spawn there.

<figure class="post-figure">
  <img src="/assets/images/blog/1_etu_tile_pipeline/generator_overloaded_shape.jpg" alt="Overloaded shape sampling" loading="lazy">
  <figcaption>More frequent sampling will trace the shape better, of course, but readability and the overall quality of visuals deteriorates</figcaption>
</figure>

The vector art needed some adjustments and requirements enforced. If the vector art had paths that were too close together, the sampling might skip points due to an enforced minimum distance between points. This would cause pieces of the line art to be missing and the quality of the stipple image to deteriorate quickly.

Some outputs required a bit of additional touchup — usually just shifting an object slightly or deleting points that were too close together. It's certainly not perfect, but it allowed the team to build hundreds of tiles dramatically faster than the previous workflow, which was exactly the goal!

<figure class="post-figure-pair post-figure-pair--shared-caption">
  <div class="post-figure-pair__images">
    <img src="/assets/images/blog/1_etu_tile_pipeline/shapes_top_down_1.jpg" alt="WIP shape arrangement 1" loading="lazy">
    <img src="/assets/images/blog/1_etu_tile_pipeline/shapes_top_down_2.jpg" alt="WIP shape arrangement 2" loading="lazy">
  </div>
  <figcaption>All sorts of random WIP shapes and arrangements.</figcaption>
</figure>

{% assign next_post = site.posts | where: "title", "From Tiles to Tracks: Runtime Track Generation" | first %}
[Next time]({{ next_post.url }}) I'll go over some special types of tiles that are built during runtime and how the track uses those tiles during gameplay.

<figure class="post-figure">
  <img src="/assets/images/blog/1_etu_tile_pipeline/solar_system_preview.jpg" alt="Solar system preview" loading="lazy">
  <figcaption>Solar Systems!</figcaption>
</figure>
