<script lang=ts>
    let h = 50;
    let w = 50;
    $: area = h * w;
    let gridSize = 300;
    let timeout = .1;
    let playing = false;
    let totalIterations = 0;
    let endGame = false;
    let homeostasis = false;
    let editMode = false;

    let previous: number[];
    let grid: number[] = [];

    for (let i = 0; i < w * h; i++) {
        grid.push(Math.round(Math.random()));
    }

    function getRow(index: number) {
        let rowStarts = [];
        let count = 0;
        for (let i = 0; i < h; i++) {
            rowStarts.push(count);
            count += 4;
        }

        let row = 1;
        for (let i = 0; i < rowStarts.length; i++) {
            if (index < row * w) {
                return row
            };
            row += 1;
        }
    }

    function getColumn(index: number) {
        return ((index + w) % w) + 1;
    }

    function getNeighborCount(index: number) {
        let row = getRow(index);
        let column = getColumn(index);

        let testNeighborsArray = [
            row && row > 1 && grid[index - h],
            row && row > 1 && column > 1 && grid[index - h - 1],
            row && row > 1 && column < w && grid[index - h + 1],
            row && row < h && grid[index + h],
            row && row < h && column > 1 && grid[index + h - 1],
            row && row < h && column < w && grid[index + h + 1],
            column > 1 && grid[index - 1],
            column < w && grid[index + 1]
        ]
        
        let total = testNeighborsArray.filter(el => el).reduce((a, b) => {
            if (typeof a !== 'number') a = 0;
            if (typeof b !== 'number') b = 0;
                return a + b;
            }, 0);
        return total;
    }

    function tick() {
        previous = grid;
        let newGeneration = grid.map( (cell, i) => {
            let neighbors = getNeighborCount(i);

            if (!cell) {
                if (neighbors === 3) return 1;
            }
            
            if (cell) {
                if (neighbors === 2 || neighbors === 3) return 1;
            }
            return 0;
        })

        let difference = 0;
        for (let i = 0; i < area; i++) {
            if (newGeneration[i] !== previous[i]) {
                difference++
            }
        }

        if (!difference) {
            endGame = true;
            playing = false;
            clearInterval(interval);
            let cellsAlive = grid.reduce( (a,b) => a + b, 0);
            if (cellsAlive) {
                homeostasis = true;
            }
            return;
        }
        grid = [...newGeneration];
        totalIterations++;
    }

    function startPlaying() {
        playing = !playing;
    }

    let interval: any;
    $: if (playing) {
        clearInterval(interval);
        if (!interval) {
            tick();
        };
        interval = setInterval(() => {
            tick();
        }, timeout * 1000);
    }

    $: if (!playing) {
        clearInterval(interval);
    }
</script>
<svelte:head>
    <title>The Game of Life</title>
    <meta name=description content="It's everyone's favorite zero player game - John Conway's Game of Life" />
    <meta property='og:url' content="https://life-jet.vercel.app/" />
    <meta property="og:title" content="The Game of Life" />
    <meta property="og:description" content="It's everyone's favorite zero player game - John Conway's Game of Life" />
    <meta property="og:image" content="https://life-jet.vercel.app/lifeCrop.png" />
</svelte:head>

<main>

    <h1 style='font-size: 1.8em;'><a style='color: white;' href='https://en.wikipedia.org/wiki/Conway%27s_Game_of_Life'>Conway's Game of Life</a></h1>
    <span style="width: {gridSize}px; text-align: right;">Generations: {totalIterations}</span>
    <div class=gridArea
        style='
            width: {gridSize}px;
            height: {gridSize}px;
        '
    >
    {#each grid as cell, i}
        <div class=cell
        on:click={() => {
            if (editMode) {
                cell = cell ? 0 : 1
            }
            }
        }
        style='
            background-color: {cell ? 'grey' : 'white'};
            color: gray;
            width: {gridSize / w}px;
            height: {gridSize / h}px;
            '></div>
    {/each}
    <!-- <div style='display: flex; justify-content: space-around; width: {gridSize}px;'>
        <button on:click={() => {
            h -= 1;
            w -= 1;
        }}>
            {'<'}
        </button>
        <span>Grid Size: {h}</span>
        <button on:click={() => {
            h += 1;
            w += 1;
        }}>
            {'>'}
        </button>
    </div> -->
    <div class='playControls' style="display: flex; justify-content: space-around; width: {gridSize}px;">
        <button disabled={playing || endGame} style="width: 5em;"on:click={() => {
            tick();
        }}>Step</button>
        <button disabled={endGame} style='width: 5em;'on:click={() => startPlaying()}>{playing ? "||" : "l>"}</button>
    </div>
    {#if endGame}
        <p style="text-align: center;">The end. {#if homeostasis}Congratulations! You have reached homeostasis!{:else}You survived for {totalIterations} generations{/if}</p>
        <button 
            class=replayButton
            style='width: {gridSize / 2}px; margin: 0 auto;'
            on:click={() => {
                window.location.reload()
            }}
        >
            Play Again
        </button>
    {/if}
</div>
<!-- <div style="display: flex; flex-direction: row-reverse; justify-content: space-between; width: {gridSize}px; margin-top: .5em;">
    <button disabled={playing} on:click={() => grid = grid.map( cell => 0)}>Clear</button>
    <button disabled={playing} on:click={() => {
        grid = grid.map( cell => {
            return Math.round(Math.random());
        })
    }}>Random</button>
    <label>Edit
        <input bind:checked={editMode} type=checkbox />
    </label>
</div> -->
    <!-- <input type=range min=.1 max=2 step=.1 bind:value={timeout} on:change={() => console.log('time changed', timeout)}> -->
    
</main>
<style>
    @import url(http://fonts.googleapis.com/css?family=Roboto:400,400italic,500,500italic,700,700italic,900,900italic,300italic,300,100italic,100);

:global(body) {
    background-color: #222;
    color: white;
    font-family: 'Roboto', sans-serif;
    }
    main {
        display: flex;
        flex-direction: column;
        align-items: center;
    }
    .gridArea {
        display: flex;
        flex-wrap: wrap;
        margin-bottom: 2em;
        padding-bottom: 0;
    }

    .cell {
        outline: 1px solid #ccc;
        display: grid;
        place-content: center;
    }

    .playControls {
        background-color: #333;
        padding: 1em 0;
        margin-top: 0;
        outline: 1px solid #ccc;
    }

    .playControls button {
        background-color: #222;
        color: white;
        padding: 1em;
        font-weight: bold;
        font-size: 1.2em;
    }

    .playControls button:disabled {
        opacity: .5;
    }

    .replayButton {
        background-color: #222;
        color: white;
        font-size: 1.1em;
        padding: .5em;
    }
</style>