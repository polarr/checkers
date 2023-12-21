<script lang="ts">
    import { browser } from '$app/environment'; 
    import { onMount } from "svelte";
    let canvas: HTMLCanvasElement;
    let ctx: CanvasRenderingContext2D;
    let canvasWidth, canvasHeight;
    

    function setHiPPICanvas(w: number, h: number) {
        if (browser){
            let ratio = window.devicePixelRatio;
            canvas.width = w * ratio;
            canvas.height = h * ratio;
            canvas.style.width = w + "px";
            canvas.style.height = h + "px";
            ctx.scale(ratio, ratio);
        }
    }

    // Canvas API Wrapper
    function fill([r = 0, g = r, b = r], a = 255) {
        ctx.fillStyle = `rgb(${r}, ${g}, ${b}, ${a})`;
    }

    function noStroke() {
        ctx.strokeStyle = "rgb(0, 0, 0, 0)";
    }

    function stroke(r = 0, g = r, b = r, a = 255) {
        ctx.strokeStyle = `rgb(${r}, ${g}, ${b}, ${a})`;
    }

    function rect(x: number, y: number, w: number, h: number) {
        ctx.fillRect(x, y, w, h);
        ctx.strokeRect(x, y, w, h);
    }

    function background([r = 255, g = r, b = r]) {
        ctx.clearRect(0, 0, canvasWidth, canvasHeight);
        fill([r, g, b]);
        stroke(255);
        rect(0, 0, canvasWidth, canvasHeight);
    }

    function circle(x: number, y: number, r: number) {
        ctx.beginPath();
        ctx.arc(x, y, r, 0, 2 * Math.PI);
        ctx.fill();
        ctx.stroke();
        ctx.closePath();
    }

    function ellipse(x, y, w, h) {
        ctx.ellipse(x, y, w / 2, h / 2, 0, 0, 2 * Math.PI);
        ctx.fill();
        ctx.stroke();
    }

    function triangle(x1, y1, x2, y2, x3, y3) {
        ctx.beginPath();
        ctx.moveTo(x1, y1);
        ctx.lineTo(x2, y2);
        ctx.lineTo(x3, y3);
        ctx.fill();
        ctx.stroke();
        ctx.closePath();
    }

    function quad(x1, y1, x2, y2, x3, y3, x4, y4) {
        ctx.beginPath();
        ctx.moveTo(x1, y1);
        ctx.lineTo(x2, y2);
        ctx.lineTo(x3, y3);
        ctx.lineTo(x4, y4);
        ctx.fill();
        ctx.stroke();
        ctx.closePath();
    }

    function textSize(n) {
        ctx.font = `${n}px sans-serif`;
    }

    function text(t, x, y) {
        ctx.fillText(t, x, y);
    }

    function translate(x, y) {
        ctx.translate(x, y);
    }

    function resetMatrix() {
        ctx.setTransform(1, 0, 0, 1, 0, 0);
    }

    class Game {
        board: number[][];
        ctx: CanvasRenderingContext2D;

        constructor(ctx){
            // 0 is empty, 1 is white, 2 is white king, 3 is red, 4, is red king
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

            this.ctx = ctx;
        }
        
        draw(){
            background([255]);
            noStroke();
            const width = canvasWidth/8, height = canvasHeight/8;

            // loops over each board cell
            for (let i = 0; i < this.board.length; i++){
                for (let j = 0; j < this.board[i].length; j++){
                    // choose the board cell color on parity
                    let color: [number, number, number] = [50, 50, 50];
                    if ((i + j) % 2 == 0){
                        color = [200, 200, 200];
                    }
                    fill(color);
                    // draw the cell rectangle
                    rect(j * width, i * height, width, height);

                    // see if there is a piece on the cell
                    let piece = this.board[i][j];
                    if (piece == 1 || piece == 2){
                        fill([255, 255, 255]);
                        // draw the corresponding piece as a circle
                        circle(j * width + width/2, i * height + height/2, 3 * width/8);
                    } else if (piece == 3 || piece == 4){
                        fill([200, 0, 0]);
                        circle(j * width + width/2, i * height + height/2, 3 * width/8);
                    }

                    if (piece == 2 || piece == 4){
                        fill([], 0);
                        stroke(0);
                        circle(j * width + width/2, i * height + height/2, width/4);
                    }
                }
            }
        }

        // return rotated 2d array of this.board
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

        // return array of valid positions where it can move
        validMove([x1, y1]){
            var possMoves = [[],[]]; 
            // case 1: move diagonally on dark squares.
            
            if (this.board[x1-1][y1-1] === 0) {
                //add to array
            }
            // case 2: Remove your opponent’s checkers from the board by jumping them if your checker is diagonal to your opponent’s and there is an empty dark space to hop to.
            
            
            return possMoves;
        }

        // attempts to move the piece at first coordinate to second
        move([x1, y1], [x2, y2]){
            //check if move is valid, move piece if valid, get rid of any eaten piece
            for (let i = 0; i < this.validMove.length; i++) {
                if (this.validMove[i][0] === x2 && this.validMove[i][1] === y2) {
                    let piece = this.board[x1][y1];
                    this.board[x1][y1] = 0;
                    this.board[x2][y2] = piece;
                    //if on furthest row switch to king
                    if (x2 === 0) {
                        this.makeKing([x2, y2]);
                    }
                }
            }
            //if piece eats opponent's piece, check if another move can be made, then move
            if (true) {
                
            }
        }
        
        //changes regular piece to king piece
        makeKing ([x, y]) {
            //white to white king
            if (this.board[x][y] = 1) {
                this.board[x][y] = 2;
            }
            //red to red king
            if (this.board[x][y] = 3) {
                this.board[x][y] = 4;
            }
        }
        
    }

    let game: Game;

    onMount(()=> {
        canvas = document.getElementById("board") as HTMLCanvasElement;
        ctx = canvas.getContext("2d") as CanvasRenderingContext2D;

        canvasWidth = Math.min(canvas.clientWidth, canvas.clientHeight), canvasHeight = Math.min(canvas.clientWidth, canvas.clientHeight);
        setHiPPICanvas(canvasWidth, canvasHeight);

        game = new Game(ctx);
        game.draw();
        window.requestAnimationFrame(()=> {
            
        });
    });

</script>

<div class = "wrapper">
    <nav class = "navbar">
        <span style = "margin-right: auto; font-weight: bold;">
            Online Checkers
        </span>
        <span>
            Log In
        </span>
        <span>
            Sign Up
        </span>
    </nav>
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
</div>

<style lang="scss">
    .wrapper {
        width: 100vw;
        height: 100vh;

        display: flex;
        flex-direction: column;
        align-items: stretch;

        font-family:'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    }

    .navbar {
        background-color: #eee;
        padding: 1rem;

        display: flex;
        justify-content: right;
        
        > span + span {
            padding-left: 1rem;
        }
    }

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
