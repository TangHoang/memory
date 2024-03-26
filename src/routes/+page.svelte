<script>
    import { emoji } from "../data/emoji.js"
    import { questions} from "../data/mathQuestions.js"
    import Header from "../components/Header.svelte"

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
            points += 50;
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
        memoryTime = 0;
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
            points += 20;
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

    $: memoryTime % 30 === 29 && playMath()

    $: console.log(state, selected, matches);

</script>

{#if state === State.start}
    <h1>Memory</h1>
    <button on:click={() => state = State.playingMemory}>Play</button>
{/if}

{#if state === State.playingMemory}
    <Header {points} {time}> </Header>
    
    <div class="cards" >
        {#each grid as card, cardIndex}
            {@const isSelected = selected.includes(cardIndex)}
            {@const isSelectedOrMatch = selected.includes(cardIndex) || matches.includes(card)}
            {@const disabled = selected.includes(cardIndex) || matches.includes(card) || selected.length === 2}
            {@const match = matches.includes(card)}
            <button 
                class="card shadow" 
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
    <Header {points} {time}> </Header>
    <div class="questions">
        {#each chosenQuestions as question, questionId}
            <form on:submit|preventDefault={() => { 
                submittedAnswer = handleInput(questionId, question.answer)}} 
                class="question-block shadow" 
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
        grid-template-columns: repeat(6, 1fr);
        grid-template-rows: repeat(6, 1fr);
        gap: 0.5rem;
        margin-bottom: 80px;
        height: 80vh;
        width: 80vh;
    }

    .question {
        height: 130px;
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
        background-color: f0f0f0;
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
        font-size: 3.5rem;
        margin-top: 20px;
        margin-bottom: 20px;
        border-radius: 15px;
    }

    .card {
        height: auto;
        width: auto;
        font-size: 4rem;
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
    
    

    .shadow {
        box-shadow: rgba(0, 0, 0, 0.24) 0px 3px 8px;
    }

    

</style>

