<script>
	import { spring } from 'svelte/motion';
	import { onMount } from 'svelte';
    import { draggable } from '../dragdrop/draggable.js';
    import Flaggable from '../Flaggable.svelte'
    import { draftEquation, dropData, dragData } from '../../stores/equation.js'
    import { history } from '../../stores/history.js'
    $: value = (token.constant && !(token.variable && token.constant === 1) ? token.constant: '') + (token.variable ? token.variable : '')

    export let error;
    export let hint;

    export let token;
    export let path;

    function parseInput(v) {
        let constant = v.match(/[0-9]+/);
        let variable = v.match(/[A-Za-z]+/);
        return {constant: constant !== null ? constant[0] : variable !== null ? 1 : null, variable: variable !== null ? variable[0] : null};
    }

    function updateToken(e) {
        let v = parseInput(e.target.value)
        let next = $history.current.updateToken(token, path, v.constant, v.variable)
        if ($history.current !== next)
            history.push(next);
    }

    const audioFiles = {
        dragStart: {file: 'pop.wav', volume: 0.25},
        dropRecieve: {file: 'click.wav', volume: 0.3},
    }
    var audioSource;
    onMount(() => {
		audioSource = new Audio('pop.wav');
	});

    let dragging = false;
    let hovering = false;
    let dragover = false;
    let dropAnim = false;

    let fadeAnimTime = 300;

	const coords = spring({ x: 0, y: 0 }, {
		stiffness: 0.05,
		damping: 0.3
	});

	function handleDragStart() {
		coords.stiffness = coords.damping = 1;
        dragging = true;
        audioSource.src = audioFiles.dragStart.file;
        audioSource.volume = audioFiles.dragStart.volume;
        audioSource.play();
	}

	function handleDragMove(event) {
		coords.update($coords => ({
			x: event.detail.x,
			y: event.detail.y
		}));
    }
    
    function handleDropSend(event) {
        dropAnim = true;
        hovering = false;
        setTimeout(() => {
            coords.set({ x: 0, y: 0 });
            dropAnim = false;
            dragging = false;
        }, fadeAnimTime)
    }

	function handleDragEnd(event) {
		coords.update($coords => ({
			x: event.detail.x,
			y: event.detail.y
		}));
        coords.stiffness = 0.05;
        coords.damping = 0.3;
        coords.set({ x: 0, y: 0 });
        dragging = false;
        hovering = false;
    }
    function handleMouseEnter(event) {
        hovering = true;
    }
    function handleMouseExit(event) {
        hovering = false;
        dragover = false;
    }
    function handleDragEnter(event) {
        dragover = true;
    }
    function handleDragExit(event) {
        dragover = false;
        hovering = false;
    }
    function handleDropReceive(event) {
        audioSource.src = audioFiles.dropRecieve.file;
        audioSource.volume = audioFiles.dropRecieve.volume;
        audioSource.play();
    }
</script>

<Flaggable error={error} hint={hint}>
    <div class="Token"
        class:dragging={dragging}
        class:hovering={hovering && !dropAnim}
        class:dragover={dragover}
        class:onTop={dragging || (Math.abs($coords.x) + Math.abs($coords.y) > 0.1)}
        use:draggable={{type: "token", accepts: ["operator", "token"]}}
        on:dragstart={handleDragStart}
        on:dragmove={handleDragMove}
        on:dragend={handleDragEnd}
        on:dragmouseenter={handleMouseEnter}
        on:dragmouseexit={handleMouseExit}
        on:dragenter={handleDragEnter}
        on:dragexit={handleDragExit}
        on:dropsend={handleDropSend}
        on:dropreceive={handleDropReceive}
        on:mouseup={() => {dragover = false;}}>
        <div class="content">
            {#if !token.constant}
                <input type=text value={value} on:change={updateToken}>
            {:else}
                {#if !(token.variable && token.constant === 1)}<span class="constant">{token.constant}</span>{/if}
                {#if token.variable}<span class="variable">{token.variable}</span>{/if}
            {/if}
        </div>
        <div class="mover"
            class:fade={dropAnim}
            style="transform:
            translate({$coords.x}px,{$coords.y}px)">
            <div class="content">
                {#if !token.constant}
                    <input type=text value={value} on:change={updateToken}>
                {:else}
                    {#if !(token.variable && token.constant === 1)}<span class="constant">{token.constant}</span>{/if}
                    {#if token.variable}<span class="variable">{token.variable}</span>{/if}
                {/if}
            </div>
        </div>
    </div>
</Flaggable>

<style>
	.Token {
        color: #fff0;
        touch-action: none;
		position: relative;
		transition: -webkit-text-stroke-color 0.2s ease;
		cursor: move;
        -webkit-user-select: none;
        -moz-user-select: none;
        -ms-user-select: none;
        user-select: none;
        -webkit-text-stroke-width: 1px;
        -webkit-text-stroke-color: #2220;
    }
	.Token.dragging {
        -webkit-text-stroke-color: #222;
	}
	.Token.onTop .mover {
        z-index: 10;
    }
	.mover {
        touch-action: none;
        position: absolute;
        pointer-events: none;
		top: 0;
		left: 0;
        width: 100%;
        height: 100%;
    }
    .content {
        touch-action: none;
        transition: transform 0.25s ease;
    }
    .Token.hovering .content {
        transform: scale(1.2);
    }
    .Token.dragging .content {
        transform: scale(1.3);
    }
    .Token.dragover .content {
        color: #33dcfe;
        transform: scale(1.2);
    }
    .fade .content {
        transform: scale(0);
        opacity: 0;
    }
    :root {
        --size: 40px;
    }
    .content {
        width: var(--size);
        height: var(--size);
        line-height: 35px;
        cursor: pointer;
        text-align: center;
        vertical-align: middle;
        border-radius: 2px;
        font-size: 30px;
    }
    .content span {
        pointer-events: none;
    }
    .content input {
        width: 100%;
        height: 100%;
        margin: 0;
        text-align: center; 
        color: #444;
        border: #444 solid 3px;
        padding: 0;
    }
    .mover .content {
        color: #444;
		transition: color 0.25s ease, opacity 0.25s ease, transform 0.25s ease;
        -webkit-text-stroke-width: 0;
    }
</style>