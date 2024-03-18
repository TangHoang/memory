<script>
    import { emoji } from "./emoji.js"

    const State = {
        start: "start",
        playing: "playing",
        paused: "paused",
        won: "won",
        lost: "lost",
    }

    let state = State.start;
    let size = 20;
    let grid = createGrid();
    let maxMatches = grid.length / 2;
    let selected = [];
    let matches = [];
    let timerId = null;
    let time = 20;

    function startGameTimer() {
        function countdown() {
            state !== State.paused && (time -= 1)
            console.log(time);
        }

        timerId = setInterval(countdown, 1000);
    }

    function createGrid() {
        let cards = [];
        let maxSize = size / 2;
        
        while (cards.length < maxSize) {
            const randomIndex = Math.floor(Math.random() * emoji.length)
			cards.push(emoji[randomIndex])
        }

        return shuffle([...cards, ...cards]);
    }

    function shuffle(array) {
        return array.sort(() => Math.random() - 0.5);
    }

    function selectCard(cardIndex) {
        // Index, da emoji ein array ist -- siehe matchCards()
        selected = selected.concat(cardIndex);
    }

    function matchCards() {
        const [first, second] = selected;

        if( grid[first] === grid[second]) {
            matches = matches.concat(grid[first]);
        }
        setTimeout(()=> selected = [], 800);
    }

    function gameWon() {
        state = State.won;
        resetGame();
    }

    function gameLost() {
        state = State.lost;
        resetGame();
    }

    function resetGame() {
        timerId && clearInterval(timerId);
        grid = createGrid();
        maxMatches = grid.length / 2;
        selected = [];
        matches = [];
        timerId = null;
        time = 20;
    }

    $: if (state === State.playing) {
        if(timerId === null) startGameTimer();
    }

    $: selected.length === 2 && matchCards();
    $: maxMatches === matches.length && gameWon();
    $: time === 0 && gameLost();

    $: console.log(state, selected, matches);

</script>

{#if state === "start"}
    <h1>Memory</h1>
    <button on:click={() => state = State.playing}>Play</button>
{/if}

{#if state === "playing"}
    <h1 class="timer" class:pulse={time < 6}> {time}</h1>
    <div class="matches">
        {#each matches as card}
            <div>{card}</div>
        {/each}
    </div>
    <div class="cards" >
        {#each grid as card, cardIndex}
            {@const isSelected = selected.includes(cardIndex)}
            {@const isSelectedOrMatch = selected.includes(cardIndex) || matches.includes(card)}
            {@const match = matches.includes(card)}
            <button 
                class="card" 
                class:selected={isSelected} 
                class:flip={isSelectedOrMatch}
                disabled={isSelectedOrMatch} 
                on:click={() => selectCard(cardIndex)}
            >
                <div class:match class="back">{card}</div>
            </button>
        {/each}
    </div>
{/if}

{#if state === State.lost}
    <h1>Verloren!</h1>
    <button on:click={() => state = State.playing}>Play Again</button>
{/if}

{#if state === State.won}
    <h1>Gewonnen!</h1>
    <button on:click={() => state = State.playing}>Play Again</button>
{/if}

<style>
    .cards {
        display: grid;
        grid-template-columns: repeat(5, 1fr);
        gap: 0.5rem
    }

    .card {
        height: 150px;
        width: 150px;
        font-size: 4.5rem;
        background-color: var(--bg-2);
        transition: rotate 0.3s ease-out;
        transform-style: preserve-3d;

        &.flip {
            rotate: y 180deg;
            pointer-events: none;
        }

        & .back {
            position: absolute;
            inset: 0;
            display: grid;
            place-content: center;
            backface-visibility: hidden;
            rotate: y 180deg;
        }

        &.selected {
        border: 4px solid var(--border);
        }
        & .match {
            transition: opacity 0.2s ease-out;
            opacity: 0.4;
        }
    }
    .matches {
        display: flex;
        gap: 1rem;
        margin-block: 2rem;
        font-size: 3rem;
    }
    
    .timer {
        transition: color 0.3s ease;
    }

    .pulse {
        color: var(--pulse);
        animation: pulse 1s infinite ease;
    }

    @keyframes pulse {
        to {
            scale: 1.3;
        }
    }

</style>

