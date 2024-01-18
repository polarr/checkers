<script lang="ts">
    import { browser } from '$app/environment'; 
    import type { Socket } from 'socket.io-client';
    import { onMount } from "svelte";

    export let socket: Socket;
    
    let gameMessage = "";
    let playerWhite = false;
    let myTimer: number;
    let myTimerString: string = "";
    let myTimestamp: number;
    let oppTimer: number;
    let lastTimer: number;
    let oppTimerString: string = "";
    let oppTimestamp: number;
    let myTimeout;
    let oppTimeout;

    let parseTime = (ms: number)=> {
        ms = Math.max(ms, 0);
        return Math.floor(ms / 60000).toLocaleString('en-US', {
            minimumIntegerDigits: 2,
            useGrouping: false
        }) + ":" + Math.floor((ms % 60000)/1000).toLocaleString('en-US', {
            minimumIntegerDigits: 2,
            useGrouping: false
        });
    }

    let updateMyTimer = (interval: number)=> {
        myTimeout = setTimeout(()=> {
            let diff = Date.now() - myTimestamp;
            myTimestamp = Date.now();
            myTimer -= diff;
            if (myTimer < 0){
                clearTimeout(myTimeout);
                return;
            }
            myTimerString = parseTime(myTimer);
            updateMyTimer(interval);
        }, interval);
    };

    let updateOppTimer = (interval: number)=> {
        oppTimeout = setTimeout(()=> {
            let diff = Date.now() - oppTimestamp;
            oppTimestamp = Date.now();
            oppTimer -= diff;
            if (oppTimer < 0){
                clearTimeout(oppTimeout);
                return;
            }
            oppTimerString = parseTime(oppTimer);
            updateOppTimer(interval);
        }, interval);
    };

    /**
     * This class enhances the canvas API and 2D rendering context
     * 
     * @param canvas The HTML Canvas element to render to
     * @param ctx The Canvas element's 2D rendering context API
     * @param canvasWidth The width of the canvas in pixels
     * @param canvasHeight The height of the canvas in pixels
     */
    class EnhancedCanvas {
        canvas: HTMLCanvasElement;
        ctx: CanvasRenderingContext2D;
        canvasWidth: number;
        canvasHeight: number;

        /**
         * Constructor - populates the properties of EnhancedCanvas
         * @param canvas The HTML Canvas element to render to
         * @param ctx The Canvas element's 2D rendering context API
         * 
         * The width and height are set directly to the HTML canvas element's width and height
         */
        constructor(canvas: HTMLCanvasElement, ctx: CanvasRenderingContext2D){
            this.canvas = canvas;
            this.ctx = ctx;
            this.canvasWidth = canvas.width;
            this.canvasHeight = canvas.height;
        }

        /**
         * Getter function for the width
         * @returns width of the canvas
         */
        getWidth(){
            return this.canvasWidth;
        }

        /**
         * Getter function for the width
         * @returns height of the canvas
         */
        getHeight(){
            return this.canvasHeight;
        }

        /**
         * Getter function for the left edge's position
         * @returns X-coordinate of the canvas' left side from the viewport boundary
         */
        getX(){
            return this.canvas.getBoundingClientRect().x;
        }

        /**
         * Getter function for the top edge's position
         * @returns Y-coordinate of the canvas' top side from the viewport boundary
         */
        getY(){
            return this.canvas.getBoundingClientRect().y;
        }

        /**
         * Setter function for the canvas' dimensions, supporting high pixel densities (like retina display on Mac)
         * @param w Width to the set the canvas to
         * @param h Height to set the canvas to
         */
        setHiPPICanvas(w: number, h: number) {
            // Sets the HTML element dimensions to the given values
            this.canvasWidth = w;
            this.canvasHeight = h;

            // Sets the Canvas rendering dimensions, adjusting for pixel density
            if (browser){
                let ratio = window.devicePixelRatio;
                this.canvas.width = w * ratio;
                this.canvas.height = h * ratio;
                this.canvas.style.width = w + "px";
                this.canvas.style.height = h + "px";
                this.ctx.scale(ratio, ratio);
            }
        }

        /**
         * Sets the fill color for the rendering API
         * @param color (destructured into r, g, b) The color to fill shapes with, in RGB
         * @param a The transparency of the color to fill shapes with
         */
        fill([r = 0, g = r, b = r], a = 255) {
            this.ctx.fillStyle = `rgb(${r}, ${g}, ${b}, ${a})`;
        }

        /**
         * Removes the storke for the rendering API
         */
        noStroke() {
            this.ctx.strokeStyle = "rgb(0, 0, 0, 0)";
        }

        /**
         * Sets the stroke for the rendering API
         * @param r Red value of the stroke
         * @param g Green value of the stroke
         * @param b Blue value of the stroke
         * @param a Transparency value of the stroke
         */
        stroke(r = 0, g = r, b = r, a = 255) {
            this.ctx.strokeStyle = `rgb(${r}, ${g}, ${b}, ${a})`;
        }

        /**
         * Draws a color and stroke filled rectangle at the given location
         * @param x X-coordinate of the top left corner
         * @param y Y-coordinate of the top left corner
         * @param w Width of the rectangle
         * @param h Height of the rectangle
         */
        rect(x: number, y: number, w: number, h: number) {
            this.ctx.fillRect(x, y, w, h);
            this.ctx.strokeRect(x, y, w, h);
        }

        /**
         * Sets the background of the entire Canvas
         * @param color Color in RGB to set the background to
         */
        background([r = 255, g = r, b = r]) {
            this.ctx.clearRect(0, 0, this.canvasWidth, this.canvasHeight);
            this.fill([r, g, b]);
            this.stroke(255);
            this.rect(0, 0, this.canvasWidth, this.canvasHeight);
        }

        /**
         * Draws a color and stroke filled circle at the given location
         * @param x X-coordinate of the center
         * @param y Y-coordinate of the center
         * @param r Radius of the circle
         */
        circle(x: number, y: number, r: number) {
            this.ctx.beginPath();
            this.ctx.arc(x, y, r, 0, 2 * Math.PI);
            this.ctx.fill();
            this.ctx.stroke();
            this.ctx.closePath();
        }
    }

    class Checkers {
        board: number[][];
        constructor(board?: number[][]){
            this.board = board ?? [
                [0, 1, 0, 1, 0, 1, 0, 1],
                [1, 0, 1, 0, 1, 0, 1, 0],
                [0, 1, 0, 1, 0, 1, 0, 1],
                [0, 0, 0, 0, 0, 0, 0, 0],
                [0, 0, 0, 0, 0, 0, 0, 0],
                [3, 0, 3, 0, 3, 0, 3, 0],
                [0, 3, 0, 3, 0, 3, 0, 3],
                [3, 0, 3, 0, 3, 0, 3, 0]
            ];
        }

        /**
         * Transforms the coordinate into its 180 degree rotation
         * @param coordinate Given coordinate
         * @returns Transformed coordinate if the board was rotated 180 degrees
         */
        rotateCoord([x, y]){
            return [7 - x, 7 - y];
        }

        getBoard(){
            return this.board;
        }

        setBoard(board: number[][]){
            this.board = board;
        }

        /**
         * Rotate the given board 180 degrees
         * @param originalBoard (optional) this.board by default, or given board
         * @returns Rotated deep cloned 2d array of originalBoard
        */
        rotateBoard(originalArray: number[][] = this.board){
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
         * Check if the given coordinate is valid and contains a Piece of the given color
         * @param isWhite Is the color white?
         * @param coordinate Given coordinate
         * @returns true if coordinate is valid and contains a piece of given color, false otherwise
         */
        validPiece(isWhite: boolean, [x, y]){
            if (x < 0 || x > 7 || y < 0 || y > 7){
                return false;
            }

            let piece = this.board[y][x];
            if (isWhite){
                return piece == 1 || piece == 2;
            }

            return piece == 3 || piece == 4;
        }

        /**
         * Check if given coordinate contains nothing
         * @param coordinate Given coordinate
         * @returns true if coordinate is valid and contains empty space, otherwise false
         */
        isEmpty([x, y]){
            if (x < 0 || x > 7 || y < 0 || y > 7){
                return false;
            }

            return this.board[y][x] == 0;
        }

        /**
         * Check if the given coordinate contains a King Piece
         * @param coordinate Given coordinate
         * @returns true if it contains a King, false otherwise
         */
        isKing([x, y]){
            if (x < 0 || x > 7 || y < 0 || y > 7){
                return false;
            }

            return this.board[y][x] == 2 || this.board[y][x] == 4;
        }

        /**
         * Checks if a given piece at a coordinate can take any pieces, and returns those taking coordinates
         * @param coordinate
         * @returns Potentially empty array of possible taking coordinates of a piece
        */
        pieceCanTake([x, y]){
            let takes: number[][] = [];

            // check if valid piece
            if (this.isEmpty([x, y])){
                return takes;
            }

            let isWhite = (this.board[y][x] == 1 || this.board[y][x] == 2);

            // which direction is up? if white then +1 to row and if red then -1 to row
            let up = isWhite ? 1:-1;

            if (this.validPiece(!isWhite, [x - 1, y + up]) && this.isEmpty([x - 2, y + 2 * up])){
                // a diagonally adjacent opponent piece exists and can be taken
                takes.push([x - 2, y + 2 * up]);
            }
            if (this.validPiece(!isWhite, [x + 1, y + up]) && this.isEmpty([x + 2, y + 2 * up])){
                // a diagonally adjacent opponent piece exists and can be taken
                takes.push([x + 2, y + 2 * up]);
            }
            if (this.isKing([x, y])){
                // can move "down"
                if (this.validPiece(!isWhite, [x - 1, y - up]) && this.isEmpty([x - 2, y - 2 * up])){
                    // a diagonally adjacent opponent piece exists and can be taken
                    takes.push([x - 2, y - 2 * up]);
                }
                if (this.validPiece(!isWhite, [x + 1, y - up]) && this.isEmpty([x + 2, y - 2 * up])){
                    // a diagonally adjacent opponent piece exists and can be taken
                    takes.push([x + 2, y - 2 * up]);
                }
            }

            return takes;
        }

        /**
         * Checks if player can take any pieces
         * @returns true or false
        */
        canTake(isWhite: boolean){
            // loop throgh board
            // check if row - 1, col +- 1 exists and contains opponent piece
            // and if row - 2, col +- 2 exists and is 0
            // if true, then can take that piece return true
            for (let row = 0; row < this.board.length; row++) {
                for (let col = 0; col < this.board[0].length; col++) {
                    if (this.validPiece(isWhite, [col, row])) {
                        // check if given piece can take
                        if (this.pieceCanTake([col, row]).length > 0){
                            return true;
                        }
                    }
                }
            }

            // cannot take any piece
            return false;
        }

        /**
         * Finds the possible coordinates a piece can move to
         * @param coordinate of given piece
         * @return array of all possible coordinates to move to
        */
        pieceCanMove([x, y]){
            let possMoves: number[][] = [];

            // check if valid piece
            if (this.isEmpty([x, y])){
                return possMoves;
            }

            let isWhite = [1, 2].includes(this.board[y][x]);

            // which direction is up? if white then +1 to row and if red then -1 to row
            let up = isWhite ? 1:-1;

            // case 1a: move diagonally up, check if spot exists on board, add move to array
            if (this.isEmpty([x - 1, y + up])){
                possMoves.push([x - 1, y + up]);
            }
            if (this.isEmpty([x + 1, y + up])){
                possMoves.push([x + 1, y + up]);
            }

            // case 1b: piece is king so can move diagonally backward, check if space is on board, add move to array
            if (this.isKing([x, y])) {
                if (this.isEmpty([x - 1, y - up])){
                    possMoves.push([x - 1, y - up]);
                }
                if (this.isEmpty([x + 1, y - up])){
                    possMoves.push([x + 1, y - up]);
                }
            }

            return possMoves;
        }

        /**
         * Finds the overall possible moves for a player and piece
         * @param isWhite If the player is playing white or not
         * @param coordinates of piece
         * @return array of possible moves for a player and piece
         */
        piecePossMoves(isWhite: boolean, [x, y]){
            if (!this.validPiece(isWhite, [x, y])){
                return [];
            }

            // can take
            if (this.canTake(isWhite)){
                console.log('cantake')
                // must take
                return this.pieceCanTake([x, y]);
            }
            // cannot take, only move
            return this.pieceCanMove([x, y]);
        }
    }

    /**
     * This class implements the UI logic and graphical rendering of the checkers game
     * It also implements synchronization with the server-side
     * @param checkers The Checkers object that stores the actual game states
     * @param canvas The EnhancedCanvas object that graphics will be rendered to
     * @param isWhite A boolean indicating if the player is playing white (or red, otherwise)
     * @param isTurn A boolean indicating if it is currently the player's turn
     * @param code Room code of the game lobby
     * @param clickedPiece Storing the coordinates of the currently selected piece (upon mouse interaction)
     */

    class Game {
        checkers: Checkers;
        canvas: EnhancedCanvas;
        isWhite: boolean;
        isTurn: boolean;
        code: string;
        clickedPiece: number[] = [];

        /**
         * Constructor
         * Initializes the properties of the class
         * @param canvas The EnhancedCanvas object that graphics will be rendered to
         * @param isWhite A boolean indicating if the player is playing white (or red, otherwise)
         * @param isTurn A boolean indicating if it is currently the player's turn
         * @param time The starting time control for both players
         */
        constructor(canvas: EnhancedCanvas, isWhite: boolean, code: string, time: number){
            // initializes the properties
            this.checkers = new Checkers();
            this.canvas = canvas;
            this.isWhite = isWhite;
            playerWhite = isWhite;
            this.isTurn = !isWhite;

            // sets the global variables tracking time
            myTimer = time;
            oppTimer = time;
            myTimestamp = Date.now();
            oppTimestamp = Date.now();
            myTimerString = parseTime(time);
            oppTimerString = parseTime(time);

            // sets the tip message indicating whose turn it is
            if (this.isTurn){
                gameMessage = "Your turn to move!";
                updateMyTimer(100);
            } else {
                gameMessage = "Opponent is thinking...";
                updateOppTimer(100);
            }
            this.code = code;
        }

        /**
         * Handle clicking a piece/cell on the canvas
         * @param e - MouseEvent of the click event
         */
        click(e: MouseEvent){
            if (!this.isTurn){
                return;
            }

            // Get the cell size of the board
            const width = this.canvas.getWidth()/8, height = this.canvas.getHeight()/8;

            // Calculate the 2D array coordinates of the clicked cell
            let coordX = Math.floor((e.clientX - this.canvas.getX()) / width);
            let coordY = Math.floor((e.clientY - this.canvas.getY()) / height);

            // Log this for development purposes
            console.log("Click", e.clientX, e.clientY, this.canvas.getX(), this.canvas.getY(), coordX, coordY);

            // Transform the clicked coordinate into the actual board coordinate by rotating if necessary (board is flipped on white)
            let [x, y] = this.isWhite ? this.checkers.rotateCoord([coordX, coordY]) : [coordX, coordY];

            // If no piece has been selected yet, assign this piece to be selected if it is a valid piece of the right color
            if (!this.clickedPiece?.length){
                if (this.checkers.validPiece(this.isWhite, [x, y])){
                    this.clickedPiece = [x, y];
                    this.draw([coordX, coordY]);
                }
                return;
            }

            // If this piece was already selected, deselect it
            if (this.clickedPiece[0] == x && this.clickedPiece[1] == y){
                this.clickedPiece = [];
                this.draw();
                return;
            }

            // Find the possible moves for the player at this point
            let possMoves = this.checkers.piecePossMoves(this.isWhite, this.clickedPiece as [number, number]);

            // If player is trying to move a piece somewhere, check if he is allowed to, and if so, send a request to the server to move
            if (possMoves.length && possMoves.some((el)=> {
                return el[0] == x && el[1] == y;
            })){
                console.log("send", [x, y]);
                socket.emit('move', {
                    code: this.code,
                    from: this.clickedPiece,
                    to: [x, y]
                });
            }
        }
        /**
         * Render the board within the canvas
         * @param (Optional) coordinate - Coordinate of clicked piece
        */
        draw(coordinate:number[] = []){
            let [coordX, coordY] = coordinate;

            // Clear the canvas
            this.canvas.background([255]);
            this.canvas.noStroke();

            // Get the cell dimensions
            const width = this.canvas.getWidth()/8, height = this.canvas.getHeight()/8;

            // Rotate the board if necessary (board is flipped on white)
            let bd = this.isWhite ? this.checkers.rotateBoard():this.checkers.getBoard();

            // If there is a clicked piece, check to see which cells that piece can move to
            let moveList: number[][] = [];

            if (coordinate.length){
                let [x, y] = this.isWhite ? this.checkers.rotateCoord([coordX, coordY]) : [coordX, coordY];

                if (this.checkers.canTake(this.isWhite)){
                    moveList = this.checkers.pieceCanTake([x, y]);
                } else {
                    moveList = this.checkers.pieceCanMove([x, y]);
                }

                console.log(moveList);
            }

            // loops over each board cell
            for (let i = 0; i < bd.length; i++){
                for (let j = 0; j < bd[i].length; j++){
                    // choose the board cell color on parity
                    let color: [number, number, number] = [50, 50, 50];
                    if ((i + j) % 2 == 0){
                        color = [200, 200, 200];
                    }
                    if (coordinate.length && coordX == j && coordY == i){
                        color = [100, 0, 0];
                    }
                    this.canvas.fill(color);
                    // draw the cell rectangle
                    this.canvas.rect(j * width, i * height, width, height);

                    // If the clicked piece can move to this cell, draw a small circle to indicate that
                    let coord = this.isWhite ? this.checkers.rotateCoord([j, i]) : [j, i];

                    if (coordinate.length && moveList.some((el)=> {
                        return el[0] == coord[0] && el[1] == coord[1];
                    })){
                        this.canvas.fill([100, 0, 0]);
                        this.canvas.circle(j * width + width/2, i * height + height/2, width/8);
                    }

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

                    // if the piece is a king, draw a ring inside the piece to indicate that
                    if (piece == 2 || piece == 4){
                        this.canvas.fill([], 0);
                        this.canvas.stroke(0);
                        this.canvas.circle(j * width + width/2, i * height + height/2, width/4);
                        this.canvas.noStroke();
                    }
                }
            }
        }

        /**
         * Set the state of the board and draw it
         * @param board Given board state
         */
        setBoard(board: number[][]){
            this.checkers.setBoard(board);
            this.draw();
        }

        /**
         * Update the player's turn according to the given turn
         * @param turn A boolean indicating if it is the player's turn
         */
        setTurn(turn: boolean){
            // Clear the clicked piece
            this.clickedPiece = [];

            // If the turn has changed, pause the current running timer and start the opposite one
            let newTurn = (turn != this.isWhite);
            if (this.isTurn != newTurn){
                myTimestamp = Date.now();
                oppTimestamp = Date.now();
                if (newTurn){
                    gameMessage = "Your turn to move!";
                    clearTimeout(oppTimeout);
                    updateMyTimer(100);
                } else {
                    gameMessage = "Opponent is thinking...";
                    clearTimeout(myTimeout);
                    updateOppTimer(100);
                }
            }

            this.isTurn = newTurn;
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

            game = new Game(enhancedCanvas, playWhite, code, time);
            game.draw();
        }
    });

    socket.on('update-game', ({board, turn})=> {
        game.setTurn(turn);
        game.setBoard(board);
    });

    let win = "";

    socket.on('game-over', ({winner})=> {
        clearTimeout(myTimeout);
        clearTimeout(oppTimeout);
        if (winner == socket.id){
            win = "You";
        } else {
            win = "Opponent";
        }
    });

    const handleClick = (e: MouseEvent)=> {
        console.log(e);
        if (browser){
            game.click(e);
        }
    }
</script>

<article>
    <canvas style="width: 100%; height: clamp(128px, 60%, 1000px);" class="board" id="board" on:click={handleClick}> 
        Your browser does not support HTML5. Please use a modern browser like Firefox, Chrome or Edge.
    </canvas>
    {#if game}
        <aside class = "info-panel">
            <div class = "time-panel">
                {playerWhite ? "RED":"WHITE"} {oppTimerString}
            </div>
            <section class = "move-panel">
                <article>
                    {#if win}
                        {"GGs, " + win + " won!"}
                    {:else}
                        {gameMessage}
                    {/if}
                </article>
            </section>
            <div class = "time-panel">
                {playerWhite ? "WHITE":"RED"} {myTimerString}
            </div>
        </aside>     
    {/if}
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
