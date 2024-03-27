<script>
    export let submittedAnswer, chosenQuestions, booleanArray, handleInput;

    
</script>

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

<style>
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

    .shadow {
        box-shadow: rgba(0, 0, 0, 0.24) 0px 3px 8px;
    }
</style>