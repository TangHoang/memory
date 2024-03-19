<script>
    import { emoji } from "./emoji.js"
    import { questions} from "./mathQuestions.js"

    const State = {
        start: "start",
        playingMemory: "playingMemory",
        playingMath: "playingMath",
        paused: "paused",
        won: "won",
        lost: "lost",
    }

    let state = State.start;
    let size = 20;
    let grid = createGrid();
    let chosenQuestions = chooseQuestions();

    let submittedAnswer = null;
    let booleanArray = [] // used to keep track of correct or false answers

    let maxMatches = grid.length / 2;
    let selected = [];
    let matches = [];
    let timerId = null;
    let time = 120;
    let elapsedTime = 0;

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
            elapsedTime += 1;
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
        time = 120;
        elapsedTime = 0;
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

    function handleInput(id, correctAnswer) {
        let input = document.getElementById(`${id}`);
        if (input.value == correctAnswer) {
            booleanArray[id] = true;
        }
        return input.value;
    }

    $: if (state === State.playingMemory) {
        if(timerId === null) startGameTimer();
    }

    $: selected.length === 2 && matchCards();
    $: maxMatches === matches.length && gameWon();
    $: time === 0 && gameLost();
    $: booleanArray.length === 3 && playMemory();

    $: elapsedTime % 30 === 29 && playMath()

    $: console.log(state, selected, matches);

</script>

{#if state === State.start}
    <h1>Memory</h1>
    <button on:click={() => state = State.playingMemory}>Play</button>
{/if}

{#if state === State.playingMemory}
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
            {@const disabled = selected.includes(cardIndex) || matches.includes(card) || selected.length === 2}
            {@const match = matches.includes(card)}
            <button 
                class="card" 
                class:selected={isSelected} 
                class:flip={isSelectedOrMatch}
                disabled={disabled} 
                on:click={() => selectCard(cardIndex)}
            >
                <div class:match class="back">{card}</div>
            </button>
        {/each}
    </div>
{/if}

{#if state === State.lost}
    <h1>Verloren!</h1>
    <button on:click={() => state = State.playingMemory}>Play Again</button>
{/if}

{#if state === State.won}
    <h1>Gewonnen!</h1>
    <button on:click={() => state = State.playingMemory}>Play Again</button>
{/if}

{#if state === State.playingMath}
    <h1 class="timer" class:pulse={time < 6}> {time}</h1>
    <div class="questions">
        {#each chosenQuestions as question, questionId}
            <form on:submit|preventDefault={() => { 
                submittedAnswer = handleInput(questionId, question.answer)}} 
                class="question-block" 
                class:correct-answer = {booleanArray[questionId]}
            >
                    <div class="question"> {question.question}</div>
                    <input class="answer" id="{questionId}" type="numeric" autocomplete="off">
            </form>
        {/each}
    </div>
{/if}

<style>
    .cards {
        display: grid;
        grid-template-columns: repeat(5, 1fr);
        gap: 0.5rem
    }

    .question {
        height: 150px;
        width: 500px;
        display: grid;
        align-items: center;
        padding-left: 30px;
        padding-right: 30px;
    }

    .answer {
        font-size: 4.5rem;
        max-width: 200px;
        text-align: center;
        background-color: whitesmoke;
        border: none;
    }

    .correct-answer {
        border: 3px solid green;
    }

    .question-block {
        display: flex;
        flex-direction: row;
        background: var(--bg-2);
        overflow: hidden;
        width: auto;
        font-size: 4.5rem;
        margin-top: 10px;
        margin-bottom: 10px;
        border-radius: 15px;
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

