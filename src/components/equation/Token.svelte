<script>
	import { spring } from 'svelte/motion';
	import { onMount } from 'svelte';
    import { draggable } from '../dragdrop/draggable.js';
    import Flaggable from '../Flaggable.svelte'
    import { draftEquation, dropData, dragData } from '../../stores/equation.js';
	import { history } from '../../stores/history.js';
    $: value = (token.constant !== null && !(token.variable !== null && token.constant === 1) ? token.constant: '') + (token.variable ? token.variable : '')

    export let error;
    export let hint;

    export let token;
    export let path;

    const audioFiles = {
        dragStart: {file: './audio/pop.wav', volume: 0.45},
        dropRecieve: {file: './audio/click.wav', volume: 0.4},
        dropError: {file: './audio/error.mp3', volume: 0.4},
    }
    var audioSource;
    onMount(() => {
		audioSource = new Audio('./audio/pop.wav');
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

	function handleDragStart(e) {
        audioSource.src = audioFiles.dragStart.file;
        audioSource.volume = audioFiles.dragStart.volume;
        audioSource.play();
        if (token.constant !== null) {
            coords.stiffness = coords.damping = 1;
            dragging = true;
            dragData.set(token, path);
        } else {
            e.stopPropagation();
        }
	}

	function handleDragMove(event) {
		coords.update($coords => ({
			x: event.detail.x,
			y: event.detail.y
		}));
    }
    
    function handleDropSend(event) {
        if ($history.current === $draftEquation) {
            handleDragEnd(event);
            return;
        }
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
        dropData.reset();
    }

    function handleMouseEnter(event) {
        hovering = true;
    }

    function handleMouseExit(event) {
        hovering = false;
        dragover = false;
    }

    function handleDragEnter(event) {
        event.stopPropagation();
        dragover = true;
        dropData.set(token, path, $dragData);
    }

    function handleDragExit(event) {
        event.stopPropagation();
        dragover = false;
        hovering = false;
    }

    function handleDropReceive(event) {
        event.stopPropagation();        
        let success = $history.current !== $draftEquation;
        audioSource.src = success ? audioFiles.dropRecieve.file : audioFiles.dropError.file;
        audioSource.volume = success ? audioFiles.dropRecieve.volume : audioFiles.dropError.volume;
        audioSource.play();
        draftEquation.apply();
    }

    function parseInput(v) {
        let constant = v.match(/[0-9]+/);
        let variable = v.match(/[A-Za-z]+/);
        return {constant: constant !== null ? constant[0] : variable !== null ? 1 : null, variable: variable !== null ? variable[0] : null};
    }

    function updateToken(e) {
        draftEquation.updateToken(token, e.target.value);
    }
</script>

<Flaggable error={error} hint={hint} size={150}>
    <div class="Token"
        class:dragging={dragging}
        class:hovering={hovering && !dropAnim}
        class:dragover={dragover}
        class:onTop={dragging || (Math.abs($coords.x) + Math.abs($coords.y) > 0.1)}
        class:editable={token.variable  === null && token.constant === null}
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
            {#if token.constant === null}
                <input type=text size={1} value={value} on:change={updateToken}>
            {:else}
                {token.value()}
            {/if}
        </div>
        <div class="mover"
            class:fade={dropAnim}
            style="transform:translate({$coords.x}px,{$coords.y}px)">
            <div class="content">
                {#if token.constant !== null}
                    {token.value()}
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
		cursor: pointer;
        -webkit-user-select: none;
        -moz-user-select: none;
        -ms-user-select: none;
        user-select: none;
        -webkit-text-stroke-width: 1px;
        -webkit-text-stroke-color: #fff0;
    }
	.Token.dragging {
        -webkit-text-stroke-color: #fff;
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
        box-sizing: border-box;
    }
    .Token.hovering .content {
        transform: scale(1.2);
    }
    .Token.dragging .content {
        transform: scale(1.3);
    }
    .Token.dragging .mover .content {
        opacity: 0.3;
    }
    .Token.dragover .content {
        /* color: var(--drag-highlight-color); */
        background: var(--drag-highlight-color);
        transform: scale(1.2);
    }
    :root {
        --size: 50px;
    }
    .content {
        min-width: var(--size);
        padding: 5px;
        cursor: pointer;
        text-align: center;
        vertical-align: middle;
        border-radius: 2px;
        font-size: 28px;
    }
    .content input {
        width: 100%;
        height: 100%;
        margin: 0;
        text-align: center; 
        color: #fff;
        border: none;
        padding: 0;
        background: none;
    }
    .mover .content {
        color: #333;
		transition: color 0.25s ease, opacity 0.25s ease, transform 0.25s ease;
        -webkit-text-stroke-width: 0;
        border-radius: 3px;
        padding: 5px;
        background: #fff;
    }
    .editable input {
        border: 3px dashed #fff;
    }
    .editable .mover .content {
        display: none;
    }
    .fade .content {
        transform: scale(0) !important;
        opacity: 0 !important;
    }
</style>