<script>
    import { emoji } from "../data/emoji.js"
    import { questions} from "../data/mathQuestions.js"
    import Header from "../components/Header.svelte"
    import Cards from "../components/Cards.svelte";
    import Questions from "../components/Questions.svelte";

    const State = {
        start: "start",
        playingMemory: "playingMemory",
        playingMath: "playingMath",
        paused: "paused",
        won: "won",
        lost: "lost",
    }

    let state = State.start;
    let size = 36;
    let grid = createGrid();
    let chosenQuestions = chooseQuestions();
    let points = 0;
    let submittedAnswer = null;
    let booleanArray = [] // used to keep track of correct or false answers
    let maxMatches = grid.length / 2;
    let selected = [];
    let matches = [];
    let timerId = null;
    let time = 120;
    let memoryTime = 0;

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
        return emoji.sort(() => Math.random() - 0.5).slice(0, Math.min(emoji.length, size));
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
    <h1>Memory</h1>
    <button on:click={() => state = State.playingMemory}>Play</button>
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
    <button on:click={() => state = State.playingMemory}>Play Again</button>
{/if}

{#if state === State.won}
    <h1>Gewonnen!</h1>
    <button on:click={() => state = State.playingMemory}>Play Again</button>
{/if}

<style>
</style>

