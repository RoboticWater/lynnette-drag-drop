<script>
	import OperatorComponent from './components/equation/Operator.svelte';
	import PreviewEquation from './components/equation/PreviewEquation.svelte';
	import OperatorBox from './components/OperatorBox.svelte';
	import History from './components/History.svelte';
	import Alien from './components/Alien.svelte';
	import { draftEquation } from './stores/equation.js';
	import { history } from './stores/history.js';
	import { Operator } from './stores/classes';
	import { parseGrammar } from './stores/equation';
	import { messageManager } from './stores/messageManager';
	let operators = [new Operator('PLUS'), new Operator('MINUS'), new Operator('TIMES'), new Operator('DIVIDE')];
	window.test = false;
	$ : test = window.test;

	function undo() {
		history.step(-1);
		messageManager.reset();
	}

	function done() {
		CTATCommShell.commShell.processDone("Test")
	}

	let val = '';
	window.setEqn = newEqn => {
		history.reset();
		history.push(window.parse.algParse(newEqn));
	}
	let testing = false;
</script>

<div class="root">
	<div class="sidebar"></div>
	<div class="content">
		<div class="operators">
			<OperatorBox operators={operators}/>
		</div>
		<div class="equation-container" class:disable={$messageManager.error}>
			{#if $history.current}<PreviewEquation state={parseGrammar($history.current)} draft={parseGrammar($draftEquation)} error={$messageManager.side}/>{/if}
		</div>
	</div>
	<div class="alien-overlay">
		<div class="history">
			<div class="history-title">History</div>
			<div class="history-items">
				{#if $history.current}<History></History>{/if}
			</div> 
		</div>
		<div class="alien">
			<Alien state={$messageManager.success ? 'success' : $messageManager.error ? 'error' : 'default'}/>
		</div>
		{#if $messageManager.error || $messageManager.hint || $messageManager.success}
			<div class="message">
				{#if $messageManager.error}
					<div class="error">{$messageManager.error.message}</div>
				{/if}
				{#if $messageManager.hint}
					<div class="hint">{$messageManager.hint.message}</div>
				{/if}
				{#if $messageManager.success}
					<div class="success">{$messageManager.success.message}</div>
				{/if}
			</div>
		{/if}
		<div class="buttons">
			<button on:click={undo} class="button undo" class:active={$messageManager.error}>Undo</button>
			<!-- <button on:click={undo} class="button undo" class:active={true}>Undo</button> -->
			<div class="bottom">
				<button class="button button-done" on:click={done}>Done</button>
				<!-- <button class="button button-hint">Hint</button> -->
			</div>
		</div>
	</div>
	{#if testing}<div class="testing">
		<button on:click={() => {if ($messageManager.error) messageManager.reset(); else messageManager.setError('Error');}}>Toggle Error</button>
		<button on:click={() => {if ($messageManager.success) messageManager.reset(); else messageManager.setSuccess('Success!');}}>Toggle Success</button>
		<button on:click={undo}>Undo</button>
		<input type="text" bind:value={val}>
		<button on:click={() => window.setEqn(val)}>Set Eqn</button>
	</div>{/if}
	<input type="checkbox" bind:checked={testing} style={"position: fixed; bottom: 0px; right: 0px; z-index: 100; margin: 0; opacity:0;"}>
</div>

<style>
	:root {
		/* --drag-highlight-color: #ffe364; */
		--drag-highlight-color: #b964ff;
	}
	.testing {
		position: fixed;
		bottom: 30px;
		left: 50%;
		transform: translateX(-50%);
	}
	.root {
		position: relative;
		height: 100vh;
		background: center / cover no-repeat url("./images/lynnette-sapce-bg.png")
	}
	.message {
  		justify-self: start;
  		align-self: start;
		position: relative;
		top: 30px;
		left: -30px;
		grid-area: message;
		min-height: 60px;
		background: #fff;
		color: #333;
		border-radius: 4px;
		padding: 30px 15px 15px 15px;
		font-size: 30px;
	}
	.message:after {
		content: '';
		position: absolute;
		left: -40px;
		top: 35px;
		width: 0;
		height: 0;
		border-style: solid;
		border-width: 0 40px 40px 0;
		border-color: transparent #fff transparent transparent;
	}
	.sidebar {
		width: 250px;
		position: fixed;
		top: 0;
		left: 0;
		bottom: 0;
		background: center / cover no-repeat url("./images/lynnette-side-bar.png");
	}
	.alien-overlay {
		pointer-events: none;
		position: fixed;
		top: 0;
		left: 0;
		right: 0;
		bottom: 0;
		display: grid;
		grid-template-areas: "history . buttons" "alien message buttons";
		grid-template-columns: 200px 1fr;
		grid-template-rows: 1fr 200px;
	}
	.history {
		grid-area: history;
		color: #333;
		padding: 10px 55px 0px 10px;
	}
	.buttons {
		pointer-events: none;
		grid-area: buttons;
		/* position: absolute;
		z-index: 2;
		right: 0;
		top: 0;
		bottom: 0;
		padding: 20px; */
	}
	.alien-overlay div > * {
		pointer-events: all;
	}
	.alien {
		grid-area: alien;
		justify-self: end;
		position: relative;
		display: inline-block;
	}
	.history-items {
		border-top: 2px #333 solid;
	}
	.history-title {
		text-align: center;
	}
	.content {
		padding: 40px 0;
		grid-area: content;
		position: relative;
		/* background: #0f0; */
	}
	.equation-container {
		display: flex;
		justify-content: center;
	}
	.equation-container.disable {
		pointer-events: none;
	}
	.operators {
		display: flex;
		justify-content: center;
		padding: 5px;
		color: #fff;
	}

	.equals > div {
		height: 20px;
	}
	

	@keyframes ripple {
		0% {
			opacity: 1;
			width: 100%; 
			height: 100%;
		}
		100% {
			opacity: 0;
			width: 150%; 
			height: 150%;
		}
	}
	.button-done {
		background: #dbf471;
		color: #353f615b;
	}
	.button-hint {
		background: #ffe364;
		color: #7c5f295b;
	}
	.button {
		border: none;
		width: 100px;
		height: 100px;
		border-radius: 50%;
		margin: 20px;
		font-size: 30px;
		cursor: pointer;
	}
	.buttons .bottom {
		position: absolute;
		display: flex;
		bottom: 20px;
		right: 40px;
	}
	.undo {
		opacity: 0;
		pointer-events: none;
		font-size: 40px;
		transition: opacity 0.3s ease, box-shadow 0.3s cubic-bezier(.27,.27,.08,.96);
		position: relative;
		cursor: pointer;
		width: 150px;
		height: 150px;
		border-radius: 50%;
		background: #f1384d;
		border: none;
		color: #361b2477;
		/* border: 10px #eed836 solid; */
	}
	.undo:after {
		opacity: 0;
		content: '';
		width: 100%; 
		height: 100%;
		top: 50%;
		left: 50%;
		transform: translate(-50%, -50%);
		position: absolute;
		border: 3px solid #f1384d;
		box-sizing: border-box;
		border-radius: 50%;
		z-index: 0;
		-webkit-animation: ripple 1.5s infinite;
		-moz-animation:    ripple 1.5s infinite;
		-o-animation:      ripple 1.5s infinite;
		animation:         ripple 1.5s infinite; 
	}
	.undo.active {
		opacity: 1;
		pointer-events: all;
	}
	/* .testing {
		opacity: 0;
		transition: 0.5s opacity ease;
	}
	.testing:hover {
		opacity: 1;
	} */
</style>