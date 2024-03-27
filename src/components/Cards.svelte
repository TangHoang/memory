<script>
    export let selected;
    export let matches, grid, matchCards
    let displaySelected = [];

    function selectCard(cardIndex) {
        // Index, da emoji ein array ist -- siehe matchCards()
        selected = selected.concat(cardIndex);
        displaySelected = [...selected]; // solution for asynchronous setTimeout in page.svelte causing flip animation to bug
        if (selected.length == 2) {
            setTimeout(() => {
            displaySelected = [];
            }, 1000);
        }
    }
    
    $: selected.length === 2 && matchCards(selected);

</script>

<div class="cards" >
    {#each grid as card, cardIndex}
        {@const isSelected = displaySelected.includes(cardIndex)}
        {@const isSelectedOrMatch = displaySelected.includes(cardIndex) || matches.includes(card)}
        {@const disabled = displaySelected.includes(cardIndex) || matches.includes(card) || displaySelected.length >= 2}
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