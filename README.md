# Rooftop Drift

A browser drifting game. Race one rival up a parking garage, or race a friend.
Built to run on iPhone Safari and on desktop Chrome and Safari.

**Play:** https://parthysarkar.github.io/Rooftop-Drift/

## Playing with a friend

One player presses **Create room** and gets a 6-digit code. The other presses
**Join room** and types it, or opens the invite link, which carries the code.
Two players per room. Invites are shared over WhatsApp, Share or a copied link.

Multiplayer needs the game on a normal web address like the GitHub Pages link
above. Opened from a preview that blocks peer-to-peer connections, the game
says so and stays single-player.

## Running it locally

Any static web server from this folder, for example:

    python3 -m http.server 8765

then open http://localhost:8765/. Opening `index.html` straight from the disk
does not work, because the car models load as separate files.

## What is in here

    index.html     the whole game: three.js scene, physics, tracks, rooms
    cars/<name>.js one car model each, a GLB file wrapped as base64
    CLAUDE.md      project notes: decisions, what is done, what is left

## Car models

From Sketchfab, simplified and restyled for this game:
[Ddiaz Design](https://sketchfab.com/ddiaz-design) (CC BY-NC-SA 4.0),
[vecarz](https://sketchfab.com/heynic) (Skyline R34, CC BY 4.0) and
[autoNgraphic](https://sketchfab.com/autoNgraphic) (911 RWB, CC BY 4.0).
The CC BY-NC-SA licence means this game stays non-commercial.
