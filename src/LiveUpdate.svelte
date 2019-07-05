<script>
    import DraggableEquation from './components/equation/DraggableEquation.svelte';
    import { onMount, afterUpdate } from 'svelte'

    let user = CTATConfiguration.getSingleParameterValues(['user_guid']);
    
    let player = 0;
    let screens = [null, null, null, null]
    let screenRects = [null, null, null, null]
    let offsetX = player % 2 === 0 ? "100vw" : "0";
    let offsetY = ~~(player / 2) === 0 ? "0" : "100vh";

    let updateRect = -1;

    onMount(() => {
        screenRects[0] = {x: 0, y: 0, top: 0, left: 0, width: parent.innerWidth, height: parent.innerHeight};
    });

    window.addEventListener("message", event => {
        if (event.data.command === "tutorready" && event.data.from !== user) {
            console.log(event);
            player = event.data.team_count - 1;
            // updateRect = player;
            screenRects[player] = {x: 0, y: 0, top: 0, left: 0, width: parent.innerWidth, height: parent.innerHeight};
            parent.postMessage({command: "playerJoin", rect: {width: parent.innerWidth, height: parent.innerHeight}, player: player}, window.location.origin);
        }
        if (event.data.command === "playerJoin" && event.data.player !== player) { // TODO this would probably be better as a direct post message, so there wouldn't be a cascade of updates when a new user joins
            console.log(event);
            screenRects[event.data.player] = event.data.rect;
            updateRect = event.data.player;
            parent.postMessage({command: "updateRect", rect: {width: parent.innerWidth, height: parent.innerHeight}, player: player}, window.location.origin);
        }
        if (event.data.command === "updateRect" && event.data.player !== player) {
            console.log(event);
            screenRects[event.data.player] = event.data.rect;
            updateRect = event.data.player;
        }
    })

    var resizeTimer;
    window.addEventListener("resize", (e) => {
        clearTimeout(resizeTimer);
        resizeTimer = setTimeout(() => {
            screenRects[player] = {x: 0, y: 0, top: 0, left: 0, width: parent.innerWidth, height: parent.innerHeight};
            parent.postMessage({command: "updateRect", rect: {width: parent.innerWidth, height: parent.innerHeight}, player: player}, window.location.origin);
        }, 250);
    });

    afterUpdate(() => {
        if (updateRect >= 0) {
            screenRects[updateRect] = screens[updateRect].getBoundingClientRect();
            updateRect = -1;
        }
    });
</script>

<div class="root">
    <div class="screens" style={`top:${offsetY}; left:${offsetX};`}>
        <DraggableEquation screenRects={screenRects} player={player} start={{x: -500, y: 200}} />
        <div bind:this={screens[0]}
            class="player-screen one"
            style={player === 0 ? "width: 100vw; height: 100vh" : 
                `width: ${screenRects[0] ? screenRects[0].width : 0}px;
                height: ${screenRects[0] ? screenRects[0].height : 0}px`} >
            {#if player === 0}
                <div class="content">1</div>
                <div class="border top three"
                    class:visible={screenRects[2]}
                    style={
                        `width: ${screenRects[2] ? screenRects[2].width - 100 : 0}px;
                        justify-self: end;
                        border-bottom-left-radius: 100px`} >3</div>
                <div class="border right two" 
                    class:visible={screenRects[1]}
                    style={
                        `height: ${screenRects[1] ? screenRects[1].height - 100 : 0}px;
                        align-self: start;
                        border-bottom-left-radius: 100px`}>2</div>
                <div class="border top-right four"
                    class:visible={screenRects[3]}>4</div>
            {/if}
        </div>
        <div bind:this={screens[1]} 
            class="player-screen two"
            style={player === 1 ? "width: 100vw; height: 100vh" : 
                `width: ${screenRects[1] ? screenRects[1].width : 0}px;
                height: ${screenRects[1] ? screenRects[1].height : 0}px`}>
            {#if player === 1}
                <div class="content">2</div>
                <div class="border left one"
                    class:visible={screenRects[0]}
                    style={
                        `height: ${screenRects[0] ? screenRects[0].height - 100 : 0}px;
                        align-self: start;
                        border-bottom-right-radius: 100px`}>1</div>
                <div class="border top four"
                    class:visible={screenRects[3]}
                    style={
                        `width: ${screenRects[3] ? screenRects[3].width - 100 : 0}px;
                        justify-self: end;
                        border-bottom-right-radius: 100px`}>4</div>
                <div class="border top-left three"
                    class:visible={screenRects[2]}>3</div>
            {/if}
        </div>
        <div bind:this={screens[2]}
            class="player-screen three"
            style={player === 2 ? "width: 100vw; height: 100vh" :
                `width: ${screenRects[2] ? screenRects[2].height : 0}px;
                height: ${screenRects[2] ? screenRects[2].height : 0}px`}>
            {#if player === 2}
                <div class="content">4</div>
                <div class="border bottom one">1</div>
                <div class="border bottom-right two">2</div>
                <div class="border right four">4</div>
            {/if}
        </div>
        <div bind:this={screens[3]}
            class="player-screen four"
            style={player === 3 ? "width: 100vw; height: 100vh" :
                `width: ${screenRects[3] ? screenRects[3].height : 0}px;
                height: ${screenRects[3] ? screenRects[3].height : 0}px`}>
            {#if player === 3}
                <div class="content">3</div>
                <div class="border bottom two">2</div>
                <div class="border bottom-left one">1</div>
                <div class="border left three">3</div>
            {/if}
        </div>
    </div>
</div>

<style>
    .root {
        overflow: hidden;
        height: 100%;
        width: 100%;
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
            "left content right"
            "bottom-left bottom bottom-right";

    }
    .player-screen .content {
        grid-area: content;
        font-size: 200px;
        overflow: hidden;
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
    .border {
        opacity: 0;
        color: #0003;
        font-size: 90px;
    }
    .border.visible {
        opacity: 1;
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