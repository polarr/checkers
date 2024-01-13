<script lang="ts">
    import { browser } from '$app/environment'; 
    import type { Socket } from 'socket.io-client';
    import { onMount } from "svelte";

    export let socket: Socket;
    
    // Enhanced Canvas API
    class EnhancedCanvas {
        canvas: HTMLCanvasElement;
        ctx: CanvasRenderingContext2D;
        canvasWidth: number;
        canvasHeight: number;

        constructor(canvas: HTMLCanvasElement, ctx: CanvasRenderingContext2D){
            this.canvas = canvas;
            this.ctx = ctx;
            this.canvasWidth = canvas.width;
            this.canvasHeight = canvas.height;
        }

        getWidth(){
            return this.canvasWidth;
        }

        getHeight(){
            return this.canvasHeight;
        }

        setHiPPICanvas(w: number, h: number) {
            this.canvasWidth = w;
            this.canvasHeight = h;

            if (browser){
                let ratio = window.devicePixelRatio;
                this.canvas.width = w * ratio;
                this.canvas.height = h * ratio;
                this.canvas.style.width = w + "px";
                this.canvas.style.height = h + "px";
                this.ctx.scale(ratio, ratio);
            }
        }

        fill([r = 0, g = r, b = r], a = 255) {
            this.ctx.fillStyle = `rgb(${r}, ${g}, ${b}, ${a})`;
        }

        noStroke() {
            this.ctx.strokeStyle = "rgb(0, 0, 0, 0)";
        }

        stroke(r = 0, g = r, b = r, a = 255) {
            this.ctx.strokeStyle = `rgb(${r}, ${g}, ${b}, ${a})`;
        }

        rect(x: number, y: number, w: number, h: number) {
            this.ctx.fillRect(x, y, w, h);
            this.ctx.strokeRect(x, y, w, h);
        }

        background([r = 255, g = r, b = r]) {
            this.ctx.clearRect(0, 0, this.canvasWidth, this.canvasHeight);
            this.fill([r, g, b]);
            this.stroke(255);
            this.rect(0, 0, this.canvasWidth, this.canvasHeight);
        }

        circle(x: number, y: number, r: number) {
            this.ctx.beginPath();
            this.ctx.arc(x, y, r, 0, 2 * Math.PI);
            this.ctx.fill();
            this.ctx.stroke();
            this.ctx.closePath();
        }

        ellipse(x: number, y: number, w: number, h: number) {
            this.ctx.ellipse(x, y, w / 2, h / 2, 0, 0, 2 * Math.PI);
            this.ctx.fill();
            this.ctx.stroke();
        }

        triangle(x1: number, y1: number, x2: number, y2: number, x3: number, y3: number) {
            this.ctx.beginPath();
            this.ctx.moveTo(x1, y1);
            this.ctx.lineTo(x2, y2);
            this.ctx.lineTo(x3, y3);
            this.ctx.fill();
            this.ctx.stroke();
            this.ctx.closePath();
        }

        quad(x1: number, y1: number, x2: number, y2: number, x3: number, y3: number, x4: number, y4: number) {
            this.ctx.beginPath();
            this.ctx.moveTo(x1, y1);
            this.ctx.lineTo(x2, y2);
            this.ctx.lineTo(x3, y3);
            this.ctx.lineTo(x4, y4);
            this.ctx.fill();
            this.ctx.stroke();
            this.ctx.closePath();
        }

        textSize(n: number) {
            this.ctx.font = `${n}px sans-serif`;
        }

        text(t: number, x: number, y: number) {
            this.ctx.fillText(t, x, y);
        }

        translate(x: number, y: number) {
            this.ctx.translate(x, y);
        }

        resetMatrix() {
            this.ctx.setTransform(1, 0, 0, 1, 0, 0);
        }
    }

    class Game {
        board: number[][];
        canvas: EnhancedCanvas;
        playWhite: boolean;

        constructor(canvas: EnhancedCanvas, playWhite: boolean){
            // 0 is empty, 1 is white, 2 is white king, 3 is red, 4 is red king
            this.board = [
                [0, 1, 0, 1, 0, 1, 0, 1],
                [1, 0, 1, 0, 1, 0, 1, 0],
                [0, 1, 0, 1, 0, 1, 0, 1],
                [0, 0, 0, 0, 0, 0, 0, 0],
                [0, 0, 0, 0, 0, 0, 0, 0],
                [3, 0, 3, 0, 3, 0, 3, 0],
                [0, 3, 0, 3, 0, 3, 0, 3],
                [3, 0, 3, 0, 3, 0, 3, 0]
            ];

            this.canvas = canvas;
            this.playWhite = playWhite;
        }
        
        whitePlayer = 1;
        redPlayer = 3;

        /**
         * Render the board within the canvas
        */
        draw(){
            this.canvas.background([255]);
            this.canvas.noStroke();
            const width = this.canvas.getWidth()/8, height = this.canvas.getHeight()/8;

            let bd = this.board;
            if (this.playWhite){
                bd = this.rotateBoard(bd);
            }

            // loops over each board cell
            for (let i = 0; i < bd.length; i++){
                for (let j = 0; j < bd[i].length; j++){
                    // choose the board cell color on parity
                    let color: [number, number, number] = [50, 50, 50];
                    if ((i + j) % 2 == 0){
                        color = [200, 200, 200];
                    }
                    this.canvas.fill(color);
                    // draw the cell rectangle
                    this.canvas.rect(j * width, i * height, width, height);

                    // see if there is a piece on the cell
                    let piece = bd[i][j];
                    if (piece == 1 || piece == 2){
                        this.canvas.fill([255, 255, 255]);
                        // draw the corresponding piece as a circle
                        this.canvas.circle(j * width + width/2, i * height + height/2, 3 * width/8);
                    } else if (piece == 3 || piece == 4){
                        this.canvas.fill([200, 0, 0]);
                        this.canvas.circle(j * width + width/2, i * height + height/2, 3 * width/8);
                    }

                    if (piece == 2 || piece == 4){
                        this.canvas.fill([], 0);
                        this.canvas.stroke(0);
                        this.canvas.circle(j * width + width/2, i * height + height/2, width/4);
                    }
                }
            }
        }

        /**
         * Rotate the board 180 degrees
         * @param originalArray
         * @returns rotated 2d array of this.board
        */
        rotateBoard(originalArray: number[][]){
            // Get the number of rows and columns in the original array
            const rows = originalArray.length;
            const cols = originalArray[0].length;

            // Create a new array with the same dimensions
            const rotatedArray = new Array(rows);

            for (let i = 0; i < rows; i++) {
                rotatedArray[i] = new Array(cols);
            }

            // Populate the rotated array by reversing rows and columns
            for (let i = 0; i < rows; i++) {
                for (let j = 0; j < cols; j++) {
                    rotatedArray[i][j] = originalArray[rows - 1 - i][cols - 1 - j];
                }
            }

            return rotatedArray;
        }
        
        /**
         * Checks if player can take any pieces
         * @returns true or false
        */
        canTake(){
            // loop throgh board
            // check if x +- 1, y - 1 contains opponent piece
            // and if x +- 2, y - 2 is 0
            // if true, return true
            
            let opponent = 0
            if (this.playWhite) {
                opponent = 3
            } else {
                opponent = 1
            }
            for (let i = 0; i < this.board.length; i++) {
                for (let j = 0; j < this.board[0].length; j++) {
                    if (this.board[i + 1][j - 1] || this.board[i - 1][j - 1] == opponent) {
                        if (this.board[i + 2][j - 2] || this.board[i - 2][j - 2] == 0) {
                            return true
                        }
                    }
                }
            }
            return false
        }

        /**
         * return array of valid positions where it can move
         * @param player's piece
         * @return array of all possible spots to move to
        */
        validMove([x, y]){
            let possMoves: number[][] = [];
            // case 1a: move diagonally forward, check if spot exists on board, add move to array
            if (x > 0 && x < this.board.length) {
                if (y > 0 && y < this.board[0].length) {
                    if (this.board[x - 1][y - 1] === 0) {
                        // add left diagonal to array
                        possMoves.push([x - 1, y - 1]);
                    }
                }
                if (y >= 0 && y < this.board[0].length - 2) {
                    if (this.board[x - 1][y + 1] === 0) {
                        // add right diagonal to array
                        possMoves.push([x - 1, y + 1]);
                    }
                }
            }
            // case 1b: piece is king so can move diagonally backward, check if space is on board, add move to array
            if (this.board[x][y] === 2 || this.board[x][y] === 4) {
                if (x >= 0 && x < this.board.length - 2){
                    if (y > 0 && y < this.board[0].length){
                        if (this.board[x + 1][y - 1] === 0) {
                            // add left back diagonal to array
                            possMoves.push([x + 1, y - 1]);
                        }
                    }
                    if (y >= 0 && y < this.board[0].length - 2) {
                        if (this.board[x + 1][y + 1] === 0) {
                            // add right back diagonal to array
                            possMoves.push([x + 1, y + 1]);
                        }
                    }
                }
            }
            return possMoves;
        }

        /**
         * return array of valid positions where it can move after eating
         * @param player's piece
         * @return array of all possible spots to move to after eating
        */
        validEat([x,y]){
            let possEats: number[][] = []; 
            // case 2a: Remove your opponent’s checkers from the board if opponent’s checker is diagonal and there is an empty space to go
            if (this.canTake()) { // if canTake returns true, must move piece and remove opponent's piece
                // if it's white eating red
                if (this.board[x][y] === 1 || this.board[x][y] === 2) {
                    if (this.board[x - 1][y - 1] === 3 || this.board[x - 1][y - 1] === 4 || this.board[x - 1][y + 1] === 3 || this.board[x - 1][y + 1] === 4){
                        if (x > 1 && x < this.board.length) {
                            if (y > 1 && y < this.board[0].length) {
                                if (this.board[x - 2][y - 2] === 0) {
                                    // add left diagonal to array
                                    possEats.push([x - 2, y - 2]);
                                }
                            }
                            if (y >= 0 && y < this.board[0].length - 2) {
                                if (this.board[x - 2][y + 2] === 0) {
                                    // add right diagonal to array
                                    possEats.push([x - 2, y + 2]);
                                }
                            }
                        }
                    }
                }
                // if it's red eating white
                if (this.board[x][y] === 3 || this.board[x][y] === 4) {
                    if (this.board[x - 1][y - 1] === 1 || this.board[x - 1][y - 1] === 2){
                        if (x > 1 && x < this.board.length) {
                            if (y > 1 && y < this.board[0].length) {
                                if (this.board[x - 2][y - 2] === 0) {
                                    // add left diagonal to array
                                    possEats.push([x - 2, y - 2]);
                                }
                            }
                            if (y >= 0 && y < this.board[0].length - 2) {
                                if (this.board[x - 2][y + 2] === 0) {
                                    // add right diagonal to array
                                    possEats.push([x - 2, y + 2]);
                                }
                            }
                        }
                    }
                }
                // case 2b: king piece has backwards diagonal kill option
                // if it's white king eating red
                if (this.board[x][y] === 2) {
                    if (this.board[x + 1][y - 1] === 3 || this.board[x + 1][y - 1] === 4 || this.board[x + 1][y +1 ] === 3 || this.board[x + 1][y + 1] === 4){
                        if (x >= 0 && x < this.board.length - 3){
                            if (y > 1 && y < this.board[0].length){
                                if (this.board[x + 2][y - 2] === 0) {
                                    // add left back diagonal to array
                                    possEats.push([x + 2, y - 2]);
                                }
                            }
                            if (y >= 0 && y < this.board[0].length - 3) {
                                if (this.board[x + 2][y +2 ] === 0) {
                                    // add right back diagonal to array
                                    possEats.push([x + 2, y + 2]);
                                }
                            }
                        }
                    }
                }
                // if it's red king eating white
                if (this.board[x][y] === 4) {
                    if (this.board[x + 1][y - 1] === 1 || this.board[x + 1][y - 1] === 2 || this.board[x + 1][y + 1] === 1 || this.board[x + 1][y + 1] === 2){
                        if (x >= 0 && x < this.board.length - 3){
                            if (y > 1 && y < this.board[0].length){
                                if (this.board[x + 2][y - 2] === 0) {
                                    // add left back diagonal to array
                                    possEats.push([x + 2, y - 2]);
                                }
                            }
                            if (y >= 0 && y < this.board[0].length - 3) {
                                if (this.board[x + 2][y + 2] === 0) {
                                    // add right back diagonal to array
                                    possEats.push([x + 2, y + 2]);
                                }
                            }
                        }
                    }
                }
            }
            return possEats;
        }

        /**
         * Move the piece selected to spot selected
         * 
         * If there is an option to eat, the player must eat, if more eating is possible it must make the move(s)
         * Turn ends if piece is turned into a king
         * 
         * @param player's selected piece
         * @param player's selected spot to move to
         */ 
        move([x1, y1], [x2, y2]) {
            // if piece eats opponent's piece, check if another move can be made, then move
            if (this.canTake()) {
                for (let i = 0; i < this.validEat.length; i++) {
                    if (this.validEat[i][0] === x2 && this.validEat[i][1] === y2) {
                        let piece = this.board[x1][y1];
                        this.board[x1][y1] = 0;
                        if (x1 - x2 > 0) {
                            if (y1 - y2 > 0) {
                                this.board[x1 - 1][y1 - 1] = 0;
                            }
                            else {
                                this.board[x1 - 1][y1 + 1] = 0;
                            }
                        }
                        if (x1 - x2 < 0) {
                            if (y1 - y2 > 0) {
                                this.board[x1 + 1][y1 - 1] = 0;
                            }
                            else {
                                this.board[x1 + 1][y1 + 1] = 0;
                            }
                        } 
                        this.board[x2][y2] = piece;
                        // if on furthest row switch to king
                        if (x2 === 0) {
                            this.makeKing([x2, y2]);
                            return;
                        }
                        // see if another jump is possible then move using recursion
                        let possJumps = this.validEat([x2, y2]);
                        if (possJumps.length > 0) {
                            for (let i = 0; i < possJumps.length; i++) {
                                this.move([x2, y2], this.validEat[i]);
                            }
                        }
                    }
                }
            }
            /// check if move is valid, move piece if valid
            else {
                for (let i = 0; i < this.validMove.length; i++) {
                    if (this.validMove[i][0] === x2 && this.validMove[i][1] === y2) {
                        let piece = this.board[x1][y1];
                        this.board[x1][y1] = 0;
                        this.board[x2][y2] = piece;
                        // if on furthest row switch to king
                        if (x2 === 0) {
                            this.makeKing([x2, y2]);
                            return;
                        }
                    }
                    else {
                        console.log("Move not valid. Please select another.");
                    }
                }
            }
        }
        
        /**
         * Changes regular piece to king piece
         * @param player's piece to switch
         */ 
        makeKing([x, y]) {
            if (this.board[x][y] == 1 || this.board[x][y] == 3) {
                this.board[x][y] += 1
            }
        }

        /**
         * Checks if someone has won the game and then ends the game
         */ 
        findWinner() {
            let winner;
            for (let row = 0; row < this.board.length; row++) {
                for (let col = 0; col < this.board[0].length; col++) {
                    // no white pieces left
                    if (this.board[row][col] !== 1 && this.board[row][col] !== 2) {
                        // END GAME, red has won
                        winner = this.redPlayer;
                        this.gameOver(winner);
                    }
                    // no red pieces left
                    else if (this.board[row][col] !== 3 && this.board[row][col] !== 4) {
                        // END GAME, white has won
                        winner = this.whitePlayer;
                        this.gameOver(winner);
                    }
                }
            }
        }
        gameOver(isWinner) {
            if(isWinner) {
                console.log("Congratulations! You won!");
            }
            else {
                console.log("Game Over. You lost.");
            }
        }
    }

    let game: Game;
    let roomCode: string;
    
    socket.on("start-game", ({code, time, playWhite})=> {
        if (browser){
            console.log(playWhite);

            roomCode = code;

            let canvas = document.getElementById("board") as HTMLCanvasElement;
            let ctx = canvas.getContext("2d") as CanvasRenderingContext2D;

            let enhancedCanvas = new EnhancedCanvas(canvas, ctx);

            let canvasWidth = Math.min(canvas.clientWidth, canvas.clientHeight), canvasHeight = Math.min(canvas.clientWidth, canvas.clientHeight);
            enhancedCanvas.setHiPPICanvas(canvasWidth, canvasHeight);

            game = new Game(enhancedCanvas, playWhite);
            game.draw();
        }
    });
</script>

<article>
    <canvas style="width: 100%; height: clamp(128px, 60%, 1000px);" class="board" id="board"> 
        Your browser does not support HTML5. Please use a modern browser like Firefox, Chrome or Edge.
    </canvas>
    <aside class = "info-panel">
        <div class = "time-panel">
            30:00
        </div>
        <section class = "move-panel">
            <div style = "font-weight: bold;">Moves</div>
            <article>
                fkdlajflsa
            </article>
        </section>
        <div class = "time-panel">
            30:00
        </div>
    </aside>
</article>

<style lang="scss">
    article {
        height: 100%;
        display: flex;
        justify-content: center;
        align-items: center;

        canvas {
            box-shadow: 0 0 100px #ccc;
        }
    }

    .info-panel {
        display: flex;
        flex-direction: column;
        height: 60%;
        width: clamp(100px, 20%, 400px);

        margin-left: 1rem;

        .time-panel {
            padding: 1rem;
            border: 1px solid black;
        }

        .move-panel {
            padding: 1rem;
            height: 100%;
            border: 1px solid black;
        }
    }
</style>
