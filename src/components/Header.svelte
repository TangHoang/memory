<script>
    export let points = 0;
    export let time = 120;

    function time_converted(num) { 
        let minutes = Math.floor(num / 60);  
        let seconds = num % 60;
        if (seconds == 0) {
            return "0" + minutes + ":" + seconds + "0";  
        }
        return "0" + minutes + ":" + seconds;         
    }
</script>

<div class="header-container">
    <h1 class="score"> {points} </h1>
    <h1 class="timer" class:pulse={time < 6}> {time_converted(time)}</h1>
</div>

<style>
    .header-container {
        display: grid;
        grid-template-columns: 1fr 1fr 1fr;    
        margin: 30px 0px;

    }

    .timer {
        grid-column: 2/2;
        transition: color 0.3s ease;
    }

    .score {
        grid-column: 1/1;
        font-family: 'Courier New', Courier, monospace;
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