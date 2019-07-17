<script>
	import { fade } from 'svelte/transition';
</script>

<div class="waiting" transition:fade>
    <div class="text">
        <span>Waiting for team</span>
        <span class="balls">
            <div></div>
            <div></div>
            <div></div>
        </span>
    </div>
</div>

<style>
    .waiting {
        position: fixed;
        top: 0;
        left: 0;
        width: 100vw;
        height: 100vh;
        background: #fff;
        font-size: 5em;
    }
    .text {
        display: flex;
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
    }
    .balls {
        width: 1.2em;
        display: flex;
        flex-flow: row nowrap;
        align-items: center;
        justify-content: space-between;
        margin-left: 20px;
    }

    .balls div {
        width: 0.3em;
        height: 0.3em;
        border-radius: 50%;
        background-color: #333;
        transform: translateY(-100%);
        animation: wave 0.8s ease-in-out alternate infinite;
        
    }

    .balls div:nth-of-type(1) {
        animation-delay: -0.4s;
    }

    .balls div:nth-of-type(2) {
        animation-delay: -0.2s;
    }

    @keyframes wave {
        from {
            transform: translateY(-10%);
        }
        to {
            transform: translateY(80%);
        }
    }
</style>