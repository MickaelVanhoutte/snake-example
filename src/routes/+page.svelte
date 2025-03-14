<script lang="ts">
    import {onMount} from 'svelte';

    interface Position {
        x: number;
        y: number;
    }

    interface SnakeI {
        direction: 'N' | 'S' | 'E' | 'W',
        tail: Position[]
    }

    const grid: string[][] = [];
    let finished = false;
    let intervalId = 0;
    let apple: Position | undefined;

    let snake: SnakeI = {direction: 'N', tail: []};

    function initGrid() {
        for (let i = 0; i < 20; i++) {
            grid.push([]);
            for (let j = 0; j < 20; j++) {
                grid[i].push('');
            }
        }
        const snakeX = generateRandom(0, 19);
        const snakeY = generateRandom(0, 19);
        snake = {
            direction: 'N',
            tail: [
                {
                    x: snakeX,
                    y: snakeY
                },
            ]
        }
    }

    function addApple() {
        const x = generateRandom(0, 19);
        const y = generateRandom(0, 19);
        if ((snake.x === x && snake.y === y) || (apple?.x === x && apple?.y === y)) {
            addApple();
            return;
        }
        apple = {
            x: x,
            y: y
        }
    }

    function generateRandom(min: number, max: number) {
        return Math.floor(Math.random() * (max - min + 1) + min);
    }

    initGrid();

    function startGameLoop(): number {
        return setInterval(() => {
            console.log('tick', snake.direction);

            updateTailPositions();
            switch (snake.direction) {
                case "N":
                    if (snake.tail[0].y === -1) {
                        alert('Game Over');
                        finished = true;
                    }
                    break;
                case "S":
                    if (snake.tail[0].y === 20) {
                        alert('Game Over');
                        finished = true;
                    }
                    break;

                case "E":
                    if (snake.tail[0].x === 20) {
                        alert('Game Over');
                        finished = true;
                    }
                    break;
                case "W":
                    if (snake.tail[0].x === -1) {
                        alert('Game Over');
                        finished = true;
                    }
                    break;
            }
            if (snake.tail.slice(1, snake.tail?.length - 1).some(elem => elem.x === snake.tail[0].x && elem.y === snake.tail[0].y)) {
                alert("bouffeur de queue !");
                finished = true;
            }

            if (apple?.x === snake.tail[0].x && apple?.y === snake.tail[0].y) {
                growSnakeTail();
                apple = undefined;
                addApple();
            }
        }, 200);
    }

    function updateTailPositions() {

        //---x
        for (let i = snake.tail?.length -1; i > 0; i--) {

            snake.tail[i] = {
                x: snake.tail[i - 1].x,
                y: snake.tail[i - 1].y
            }
        }
        snake.tail[0].x = snake.direction === 'E' ? snake.tail[0].x + 1 : snake.direction === 'W' ? snake.tail[0].x - 1 : snake.tail[0].x;
        snake.tail[0].y = snake.direction === 'S' ?  snake.tail[0].y +1 : snake.direction === 'N' ?  snake.tail[0].y - 1 :  snake.tail[0].y;
        console.log(snake.tail);
    }

    function growSnakeTail() {
       /*
        const direct = N|S|E|W
        en fonction des 2 derniers de la tail
        deduire le x et y selon le dernier element de la tail et la direction
       */
        snake.tail.push({
            x: 0,
            y: 0
        });
    }


    $: if (finished) {
        window.clearInterval(intervalId);
    }

    onMount(() => {

        window.addEventListener('keydown', (e) => {
            switch (e.key) {
                case 'ArrowUp':
                    snake.direction = 'N';
                    break;
                case 'ArrowDown':
                    snake.direction = 'S';
                    break;
                case 'ArrowLeft':
                    snake.direction = 'W';
                    break;
                case 'ArrowRight':
                    snake.direction = 'E';
                    break;
            }
        });

        intervalId = startGameLoop();
        addApple();
    })
</script>

<div class="wrapper">
    {#each grid as col, j}
        <div class="flex">
            {#each col as row, i}
                <div class="w-8 h-8 border border-gray-300 bg-white"
                     class:bgred={apple?.x === i && apple?.y === j}
                     class:tail={snake.tail.some(elem => elem.x === i && elem.y === j)}
                >

                </div>
            {/each}
        </div>
    {/each}
</div>

<style>
    @import "tailwindcss";

    .wrapper {
        height: 50dvh;
        width: 50dvw;

    }

    .bgblack {
        background-color: black;
    }

    .bgred {
        background-color: red;
    }

    .tail {
        background-color: black;
        border-radius: 8px;
    }
</style>