<script>
    import DraggableEquation from './components/equation/DraggableEquation.svelte';
    import { onMount } from 'svelte'
    
    let player = 0;
    let screens = [null, null, null, null]
    $: screenRects = [{width: 200, height: 100}, {width: 1000, height: 1000}, {width: 1000, height: 1000}, {width: 1000, height: 1000}]
    $: offsetX = player % 2 === 0 ? "100vw" : "0";
    $: offsetY = Math.floor(player / 2) === 0 ? "0" : "100vh";

    onMount(() => {
        screenRects = screens.map(screen => screen.getBoundingClientRect());
        parent.postMessage({command: "updateRect", rect: screenRects[player], player: player}, window.location.origin);
    });
</script>

<div class="root">
    <div class="screens" style={`top:${offsetY}; left:${offsetX};`}>
        <DraggableEquation screenRects={screenRects} player={0} start={{x: -500, y: 200}} />
        <div bind:this={screens[0]} style={player === 0 ? "width: 100vw; height: 100vh" : `width: ${screenRects[0].width}px; height: ${screenRects[0].height}px`} class="player-screen one">
            <div class="border top three" 
                style={`width: ${screenRects[2].width - 100}px; justify-self: end; border-bottom-left-radius: 100px`} >3</div>
            <div class="border top-right four">4</div>
            <div class="border right two" 
                style={`height: ${screenRects[1].height - 100}px; align-self: start; border-bottom-left-radius: 100px`}>2</div>
        </div>
        <div bind:this={screens[1]} style={player === 1 ? "width: 100vw; height: 100vh" : `width: ${screenRects[1].width}px; height: ${screenRects[1].height}px`} class="player-screen two">
            <div class="border left one">1</div>
            <div class="border top four">4</div>
            <div class="border top-left three">3</div>
        </div>
        <div bind:this={screens[2]} style={player === 2 ? "width: 100vw; height: 100vh" : `width: ${screenRects[2].width}px; height: ${screenRects[2].height}px`} class="player-screen three">
            <div class="border bottom one">1</div>
            <div class="border bottom-right two">2</div>
            <div class="border right four">4</div>
        </div>
        <div bind:this={screens[3]} style={player === 3 ? "width: 100vw; height: 100vh" : `width: ${screenRects[3].width}px; height: ${screenRects[3].height}px`} class="player-screen four">
            <div class="border bottom two">2</div>
            <div class="border bottom-left one">1</div>
            <div class="border left three">3</div>
        </div>
    </div>
</div>

<style>
    .root {
        overflow: hidden;
        height: 100%;
        width: 100%;
        color: #fff;
    }
    .player {
        z-index: 100;
        position: fixed;
    }
    .screens {
        width: 0;
        height: 0;
        position: relative;
    }
    .player-screen {
        text-align: center;
        font-size: 30px;
        position: absolute;
        display: grid;
        grid-template-columns: 100px auto 100px;
        grid-template-rows: 100px auto 100px;
        grid-template-areas: 
            "top-left top top-right"
            "left . right"
            "bottom-left bottom bottom-right";
    }
    .player-screen.one {
        /* background: #f33870; */
        right: 100%;
    }
    .player-screen.two {
        /* background: #eebd55; */
    }
    .player-screen.three {
        /* background: #55caee; */
        bottom: 100%;
        right: 100%;
    }
    .player-screen.four {
        /* background: #e155ee; */
        bottom: 100%;
    }
    .border.one {
        background: #f33870;
    }
    .border.two {
        background: #ffcd62;
    }
    .border.three {
        background: #55caee;
    }
    .border.four {
        background: #cf61da;
    }
    .border.top {
        grid-area: top;
    }
    .border.bottom {
        grid-area: bottom;
    }
    .border.left {
        grid-area: left;
    }
    .border.right {
        grid-area: right;
    }
    .border.top-right {
        grid-area: top-right;
    }
    .border.top-left {
        grid-area: top-left;
    }
    .border.bottom-right {
        grid-area: bottom-right;
    }
    .border.bottom-left {
        grid-area: bottom-left;
    }
</style>