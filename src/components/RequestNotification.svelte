<script>
    import { cubicInOut } from 'svelte/easing';

    export let name;
    export let onClick;

    function slide(node, { duration }) {
		return {
			duration,
			css: t => {
				const eased = cubicInOut(t);
				return `
                    transform: translateX(${(1 - eased) * 100}%);
                    opacity: ${eased}`
			}
		};
	}
</script>



<div class="request" 
    in:slide="{{duration: 600}}" out:slide>
    <div class="content">{name} is requesting the equation <button class="send" on:click={onClick}>Send</button></div>
</div>

<style>
    .content {
        background: #fff;
        box-shadow: #2223 3px 3px 7px; 
        position: absolute;
        white-space: nowrap;
        right: 0;
        padding: 10px 25px 2px 10px;
        border-top-left-radius: 5px;
        border-bottom-left-radius: 5px;
        z-index: 100;
        pointer-events:all;
    }
    .send {
        margin-left: 10px;
    }
    .request {
        position: relative;
        height: 80px;
    }
</style>