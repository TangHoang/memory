<script>
    import { emoji } from "../data/emoji.js"
    import { questions} from "../data/mathQuestions.js"
    import Header from "../components/Header.svelte"
    import Cards from "../components/Cards.svelte";
    import Questions from "../components/Questions.svelte";
  import anime from "animejs";

    const State = {
        start: "start",
        playingMemory: "playingMemory",
        playingMath: "playingMath",
        paused: "paused",
        won: "won",
        lost: "lost",
        leaderboard: "leaderboard",
        settings: "settings",
    }

    let booleanSelection = {
        smileys: false,
        fruits_vegetables: false,
        sports: false,
        animals: false,
    }

    let grid;
    let selection = "smileys";
    let state = State.start;
    let size = 36;
    let chosenQuestions = chooseQuestions();
    let points = 0;
    let submittedAnswer = null;
    let booleanArray = [] // used to keep track of correct or false answers
    let maxMatches;
    let selected = [];
    let matches = [];
    let timerId = null;
    let time = 120;
    let memoryTime = 0;
    let isClicked = false;

    function chooseQuestions() {
        let temp = []; 
        for(let i=1; i<4; i++) {
            temp.push(questions[Math.floor(Math.random() * questions.length)]);
        }
        return temp;
    }

    function startGameTimer() {
        function countdown() {
            state !== State.paused && (time -= 1)
            if (state === State.playingMath) {
                memoryTime += 0; // stop time, when playing math
            } else {
                memoryTime += 1; // increase when playing memory
            }
        }
        timerId = setInterval(countdown, 1000);
    }

    function createGrid() {
        // !!! grid layout anpassen an array größe !!! //
        return emoji[selection].sort(
            () => Math.random() - 0.5).slice(0, Math.min(Math.floor(emoji[selection].length / 2) * 2, size)); // making sure that the array length is even
    }

    function shuffle(array) {
        
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
        time = 120;
        memoryTime = 0;
        points = 0;
        booleanSelection.animals = booleanSelection.food = booleanSelection.smileys = booleanSelection.sports = false;
    }

    function playMath() {
        state = State.playingMath;
    }

    function playMemory() {
        setTimeout(() => {
            state = State.playingMemory;
            booleanArray = [];
            chosenQuestions = chooseQuestions();
        } , 1000);

    }

    function matchCards(selected) {
        const [first, second] = selected;
        if( grid[first] === grid[second]) {
            matches = matches.concat(grid[first]);
            points += 50;
        }
        selected.splice(0)
    }

    function handleInput(id, correctAnswer) {
        let input = document.getElementById(`${id}`);
        if (input.value == correctAnswer) {
            booleanArray[id] = true;
            points += 20;
        }
        return input.value;
    }

    function handleSelection(selected) {
        selection = selected;
        grid = createGrid();
        maxMatches = grid.length / 2; // defined here because grid needs to be created first
        booleanSelection = {
            smileys: false,
            fruits_vegetables: false,
            sports: false,
            animals: false,
        };
        booleanSelection[selected] = true;
    }

    $: maxMatches === matches.length && gameWon();
    $: time === 0 && gameLost();
    $: booleanArray.length === 3 && playMemory();
    $: memoryTime % 30 === 29 && playMath()
    $: console.log(state, selected, matches);
    $: if (state === State.playingMemory) {
        if(timerId === null) startGameTimer();
    }
</script>

{#if state === State.start}
    <h1>Speedolino</h1>
    <button class="click-button" on:click={() => state = State.settings}>Neues Spiel</button>
    <button class="click-button" on:click={() => state = State.leaderboard}>Rangliste</button>
{/if}

{#if state === State.settings}
    <div class="settings-container">
        <button on:click={() => handleSelection("smileys")} class:highlight={booleanSelection.smileys} class="deck-button">Smileys</button>
        <button on:click={() => handleSelection("animals")}  class:highlight={booleanSelection.animals} class="deck-button">Tiere</button>
        <button on:click={() => handleSelection("fruits_vegetables")} class:highlight={booleanSelection.fruits_vegetables} class="deck-button">Essen</button>
        <button on:click={() => handleSelection("sports")} class:highlight={booleanSelection.sports} class="deck-button">Sport</button>
        <button class="click-button play-button" on:click={() => state = State.playingMemory}>Spielen</button>
    </div>
{/if}

{#if state === State.playingMemory}
    <Header {points} {time}> </Header>
    <Cards {grid} {selected} {matches} matchCards= {matchCards}></Cards>
{/if}

{#if state === State.playingMath}
    <Header {points} {time}> </Header>
    <Questions {submittedAnswer} {booleanArray} {chosenQuestions} handleInput={handleInput}></Questions>
{/if}

{#if state === State.lost}
    <h1>Verloren!</h1>
    <button on:click={() => state = State.start}>Play Again</button>
{/if}

{#if state === State.won}
    <h1>Gewonnen!</h1>
    <button on:click={() => state = State.start}>Play Again</button>
{/if}

<style>

    .settings-container {
        display: grid;
        grid-template-columns: 1fr 1fr 1fr 1fr;
        grid-template-rows: 1fr 1fr;
        gap: 20px;
        place-content: center;
    }

    .play-button {
        grid-column: 2 / span 2;
        grid-row: 2/2;
    }

    .deck-button {
        font-size: 1.5rem;
        font-weight: 600;
        color: inherit;
        background: none;
        border-radius: 8px;
        text-transform: uppercase;
        cursor: pointer;
        transition: transform 0.2s ease;
    }

    .deck-button:hover {
        color: var(--border);
    }

    .highlight {
        color: var(--border);
        animation: scale 0.2s ease forwards;
    }

    @keyframes scale {
        0% {
            transform: scale(1);
        }

        100% {
            transform: scale(1.2);
        }
    }
</style>

