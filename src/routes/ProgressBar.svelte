<script>
    export let value = 0; // Input value for the progress bar
</script>
  
<style>
    .progress-bar-container {
        width: 100%;
        background-color: #f0f0f0;
        border-radius: 8px;
        margin: 30px 0;
    }

    .progress-bar {
        height: 20px;
        background-color: #007bff;
        border-radius: 8px;
        transition: width 0.3s ease;

    }
</style>

<div class="progress-bar-container">
    <div class="progress-bar" style="width: {value}%;"></div>
</div>