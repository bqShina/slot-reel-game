# Slot Reel Game

## Description

This project is a simple slot reel game built with PixiJS and SvelteKit. The game features a 3x3 slot reel where users can spin the wheels to see if they land a winning combination. It utilizes GSAP for animations and includes a responsive design.

## Features

- **Interactive Slot Reels**: Spin and stop the reels to see the outcome.
- **Winning Combinations**: Check for wins based on rows and diagonals.
- **Responsive Design**: Adjusts to fit within a 500x500px container.

## Installation

Ensure you have Node.js and npm installed. Then run:

```bash
npm install
```

## Run the Project

Start the development server:

```bash
npm run dev

# or start the server and open the app in a new browser tab
npm run dev -- --open
```

Open your browser and navigate to http://localhost:5173 to see the game in action.

## Building

To create a production version of your app:

```bash
npm run build
```

You can preview the production build with `npm run preview`.

## Usage

### Play the Game

- Click the "Spin the wheels!" button to start spinning the reels.
- Click the "Stop the wheels!" button to stop the reels.

### Winning

The game checks for winning combinations based on rows and diagonals. Winning symbols are animated to indicate a win.

## Project Structure

- **`/src`**: Contains the source code for the game.
  - **`/lib/components`**: Svelte components for this game.
  - **`/static/assets`**: Image assets used in the game.

## Code Explanation

- **PixiJS**: Used for rendering the game graphics and managing the slot reels.
- **GSAP**: Handles animations for spinning and stopping the reels.
- **SvelteKit**: Provides a framework for building the front-end application.

## Acknowledgements

- **PixiJS**: [pixijs.com](https://pixijs.com)
- **GSAP**: [greensock.com/gsap](https://greensock.com/gsap)
- **SvelteKit**: [svelte.dev](https://svelte.dev)
