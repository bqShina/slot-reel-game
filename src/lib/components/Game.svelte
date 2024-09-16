<script lang="ts">
import {
    Application,
    Container,
    Texture,
    Sprite,
    Graphics,
    Text,
    TextStyle,
    Loader,

} from 'pixi.js';
import { onMount } from 'svelte';
import { gsap } from 'gsap';

interface Reel {
    container: Container,
    symbols: Sprite[],
    position: number,
    previousPosition: number 
}

let app: Application;
let playText: Text;
const REEL_WIDTH = 160;
const SYMBOL_SIZE = 80;
const gap = 30;
const PLAY = "Spin the wheels!";
const STOP = "Stop the wheels!";


onMount(() => {
    // Initialize the application
    app = new Application({
        width: 500,
        height: 500,
        backgroundColor: 0xFEE9D7
    });
    // Append applicaton to document
    const container = document.getElementById('pixi-container');
    if (container) {
        container.appendChild(app.view);
    }

    const loader = new Loader();
    loader
        .add('cherry', 'assets/cherry.png')
        .add('watermelon', 'assets/watermelon.png')
        .add('orange', 'assets/orange.png')
        .add('purpleFruit', 'assets/purple-fruit.png')
        .add('diamond', 'assets/diamond.png');
    // Create slot elements

    loader.load(() => {
        const slotTextures: Texture[] = [
            loader.resources['cherry'].texture,
            loader.resources['watermelon'].texture,
            loader.resources['orange'].texture,
            loader.resources['purpleFruit'].texture,
            loader.resources['diamond'].texture
        ];
        const elementsCount = slotTextures.length;

        // Build the reels
        const reels: Reel[] = [];
        const reelContainer = new Container();

        for (let i = 0; i < 3; i++) {
            const rc = new Container();

            rc.x = i * REEL_WIDTH;
            reelContainer.addChild(rc);

            const reel: Reel = {
                container: rc,
                symbols: [],
                position: 0,
                previousPosition: 0           
            }

            // Build elements
            for (let j = 0; j < elementsCount; j++) {
                const symbol = new Sprite(slotTextures[Math.floor(Math.random() * elementsCount)]);

                symbol.y = j * SYMBOL_SIZE;
                symbol.scale.x = symbol.scale.y = Math.min(SYMBOL_SIZE / symbol.width, SYMBOL_SIZE / symbol.height);
                symbol.x = gap + Math.round((SYMBOL_SIZE - symbol.width) / 2);
                reel.symbols.push(symbol);
                rc.addChild(symbol);
            }
            reels.push(reel);
        }

        app.stage.addChild(reelContainer);

        // Build title and bottom button
        let running = false;
        const margin = (app.screen.height - SYMBOL_SIZE * 3) / 2;

        reelContainer.y = margin;
        reelContainer.x = Math.round(app.screen.width - REEL_WIDTH * 3);

        const top = new Graphics();

        top.beginFill(0xFFAF87);  
        top.drawRect(0, 0, app.screen.width, margin);  
        top.endFill();  

        const bottom = new Graphics();
        bottom.beginFill(0xFFAF87);  
        bottom.drawRect(0, SYMBOL_SIZE * 3 + margin, app.screen.width, margin);  
        bottom.endFill();
        
        // add Title
        const titleText = new Text('Play 3x3: Hold The Spin Slot', {fill: 0x34222E});
        titleText.x = Math.round((app.screen.width - titleText.width) / 2);
        titleText.y = Math.round((margin - titleText.height) / 2);

        top.addChild(titleText);
        // add play button
        const buttonStyle = new TextStyle({
            fontFamily: 'Arial',
            fontSize: 24,
            fontWeight: 'bold',
            fill: ['#2A363B'], 
            align: 'center',
        });

        playText = new Text(PLAY, buttonStyle);
        playText.x = Math.round((bottom.width - playText.width) / 2);
        playText.y = app.screen.height - margin + Math.round((margin - playText.height) / 2);
        bottom.addChild(playText);

        app.stage.addChild(top);
        app.stage.addChild(bottom);

        // set click event
        bottom.interactive = true;
        bottom.buttonMode = true;
        bottom.cursor = 'pointer';
        bottom.on('pointerdown', () => {
            running ? stopPlay(): startPlay();
        });

        function startPlay() {
            
            if (running) return;
            running = true;

            playText.text = STOP;
            gsap.killTweensOf(reels);
            

            for (let i = 0; i < reels.length; i++) {
                const r = reels[i];
                const extra = Math.floor(Math.random() * 3);
                const target = r.position + 10 + i * 5 + extra;
                const time = 2500 + i * 600 + extra * 600;
                
                gsap.to(r, {
                    duration: time / 1000,
                    position: target,
                    ease: 'elastic"',
                    onUpdate: () => {
                        updateReel(r);
                    },
                    onComplete: () => {
                        if (i === reels.length - 1) {
                            running = false; 
                            playText.text = PLAY;
                            checkForWins(reels);
                        }
                    }
                })
            }
            
            
        }
        function updateReel(reel: Reel) {
            for (let j = 0; j < reel.symbols.length; j++) {
                const s = reel.symbols[j];
                const prevy = s.y;

                s.y = ((reel.position + j) % reel.symbols.length) * SYMBOL_SIZE - SYMBOL_SIZE;
                if (s.y < 0 && prevy > SYMBOL_SIZE) {
                    s.texture = slotTextures[Math.floor(Math.random() * slotTextures.length)];
                    s.scale.x = s.scale.y = Math.min(SYMBOL_SIZE / s.texture.width, SYMBOL_SIZE / s.texture.height);
                    s.x = Math.round((SYMBOL_SIZE - s.width) / 2) + gap;
                }
            }
        }

        function stopPlay() {
            if (!running) return;
            playText.text = PLAY;

            gsap.killTweensOf(reels);
            let completedReels = 0;
            for(const reel of reels) {
                gsap.to(reel, {
                    duration: 0.5,
                    position: Math.round(reel.position),
                    ease: 'elastic"',
                    onUpdate: () => {
                        updateReel(reel);
                    },
                    onComplete: () => {
                        completedReels++;
                        if (completedReels === reels.length) {
                            running = false;
                            
                            checkForWins(reels); 
                
                            
                        }
                                    
                    }                
                })
            } 
                
        }
        function captureVisibleSymbols(reels: Reel[]): Sprite[][] {
            const visibleSymbols: Sprite[][] = [[], [], []];

            // Iterate over each reel and capture visible symbols based on their y position
            reels.forEach((reel, reelIndex) => {
                const reelChildren = reel.container.children as Sprite[];

                reelChildren.forEach((sprite, spriteIndex) => {
                    if (sprite.y >= 0 && sprite.y < SYMBOL_SIZE * 3) {
                        // Calculate the row based on the y position
                        const row = Math.floor(sprite.y / SYMBOL_SIZE);
                        visibleSymbols[row][reelIndex] = sprite;
                    }
                });
            });

            return visibleSymbols;
        }
    function checkForWins(reels: Reel[]) {
    const symbolMatrix = captureVisibleSymbols(reels);

    const winSymbols: Sprite[][] = [];

    const checkLine = (line: Sprite[]) => {
        const symbols = line.map(sprite => sprite?.texture?.textureCacheIds[0]);

        const uniqueSymbols = new Set(symbols);
        if (uniqueSymbols.size === 1) {
            winSymbols.push(line);
        }
    };

    // Check rows
    for (const row of symbolMatrix) {
        checkLine(row);
    }

    // Check diagonals
    checkLine([symbolMatrix[0][0], symbolMatrix[1][1], symbolMatrix[2][2]]);
    checkLine([symbolMatrix[0][2], symbolMatrix[1][1], symbolMatrix[2][0]]);

    handleWin(winSymbols);
    }


        function handleWin(wins: Sprite[][]) {
            for (const symbols of wins) {
                for (const symbol of symbols) {
                    // change the size of the symbol
                    gsap.to(symbol.scale, {  
                        duration: 0.35, 
                        x: 0.30,  
                        y: 0.30,
                        ease: 'power2.inOut',
                        repeat: 1, 
                        yoyo: true
                    });
                }
            }
        }
    })
    
    

})


</script>

<div id="pixi-container" class="pixi-container"></div>


<style>
 .pixi-container {
    border: 5px solid #FFAF87;
    border-radius: 8px;
    overflow: hidden;
    height: 500px;
 }
</style>

