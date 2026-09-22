# CHEESE-

Underground syndicate mob play-to-earn game demo on Solana.

## Combined demo

This repository now contains a self-contained browser demo inspired by the art direction in [`Mobcheese/mobster-mouse-assets`](https://github.com/Mobcheese/mobster-mouse-assets). The demo works without a build step and includes:

- A top-down neon city game board
- A playable mobster mouse controlled with WASD or arrow keys
- Collectible cheese tokens and a score HUD
- District switching
- A wallet-connect demo button (UI only; no real transaction is performed)
- An asset manifest documenting the source art repository

Open `index.html` directly in a browser, or run a local server:

```bash
python3 -m http.server 8080
```

Then visit <http://localhost:8080>.

> The assets repository is private, so the demo uses lightweight CSS/SVG visuals locally rather than unauthenticated image URLs. The source asset paths are recorded in `assets/asset-manifest.json` for replacing the demo visuals when the repositories are deployed together.
