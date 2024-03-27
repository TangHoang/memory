<script>
    export let points = 0;
    export let time = 120;
    let isScaled = false;
    // check for change in points
    // needed because animation would trigger, after loading new screen
    let previousPoints = points; 

    function time_converted(num) { 
        let minutes = Math.floor(num / 60);  
        let seconds = num % 60;
        if (seconds == 0) {
            return "0" + minutes + ":" + seconds + "0";  
        }
        return "0" + minutes + ":" + seconds;         
    }

    function pointsIncreased() {
        isScaled = true;
        previousPoints = points;
        setTimeout(() => {
            isScaled = false;
        }, 800);
    }

    $: points > previousPoints && pointsIncreased()

</script>

<div class="header-container">
    <div class="score-container border">
        <h1 class="score " class:scale={isScaled}> {points} </h1>
    </div>
    <div class="timer-container border">
        <h1 class="timer " class:pulse={time < 6}> {time_converted(time)}</h1>
    </div>
</div>

<style>
    .header-container {
        display: grid;
        grid-template-columns: 1fr 1fr 1fr 1fr; 
        margin: 30px 0px;
        height: 60px;
        width: 100%;
        gap: 20px;
    }

    .timer-container {
        display: flex;
        flex-direction: row;
        height: 100%;
        width: 100%;
        grid-column: 3/3;
        transition: color 0.3s ease;
        align-self: center;
        justify-self: center;
        align-items: center;
    }

    .timer {
        height: auto;
        width: 100%;
        padding: 5px 0px;
    }

    .score-container {
        display: flex;
        flex-direction: row;
        height: 100%;
        width: 100%;
        grid-column: 2/2;
        align-self: center;
        justify-self: center;
        align-items: center;
    }

    .score {
        height: auto;
        width: 100%;
        padding: 5px 0px;
    }

    .scale {
        animation: scale 0.5s ease-out;
    }

    h1 {
        font-size: 2rem;
        text-align: center;
        text-transform: capitalize;
    }

    .border {
        border-radius: 30px;
        border: 1px solid var(--border);
    }

    .pulse {
        color: var(--pulse);
        animation: pulse 1s infinite ease;
    }

    @keyframes scale {
        to {
            scale: 1.3;
        }
    }

    @keyframes pulse {
        to {
            scale: 1.3;
        }
    }
</style>