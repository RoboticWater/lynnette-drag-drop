<script>
    import { elasticOut } from 'svelte/easing';

    export let requesting;
    export let name;
    export let hasEquation;
    export let canGiveEquation;
    export let onClick;

    function grow(node, { duration }) {
		return {
			duration,
			css: t => {
				const eased = elasticOut(t);
				return `
                    transform: scale(${eased});
                    opacity: ${eased}`
			}
		};
	}
</script>


<div class="collaborator"
    class:requesting={requesting && canGiveEquation}
    class:has-equation={hasEquation}
    in:grow="{{duration: 600}}" out:grow
    on:click={onClick}>
    <span>{name}</span>
</div>

<style>
    .collaborator {
        margin: 20px;
        padding: 20px;
        border-radius: 5px;
        background: #333;
        border: 0 #ffef5b solid;
        transition: transform 0.3s ease, background 0.3s ease, border 0.3s ease;
        color: #fff;
        display: flex;
        justify-content: center;
        align-items: center;
        font-size: 30px;
        cursor: pointer;
        user-select: none;
        box-sizing: border-box;
        -moz-box-sizing: border-box;
        -webkit-box-sizing: border-box;
    }
    .collaborator:hover {
        transform: scale(1.2);
        background: #48adff;
    }
    .collaborator.requesting {
        transform: scale(1.4);
        border-width: 5px;
    }
    .collaborator.requesting:hover {
        transform: scale(1.6);
    }
    .collaborator:active {
        transform: scale(0.9);
    }
    .collaborator.requesting:active {
        transform: scale(1.2);
    }
    .collaborator.has-equation {
        border: 5px #48adff solid;
    }
</style>