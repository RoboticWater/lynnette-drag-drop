<script>
	import History from './components/History.svelte';
    import { history } from './stores/history.js';
    import { elasticOut } from 'svelte/easing';
    import { produce } from 'immer';

    let user = CTATConfiguration.getSingleParameterValues(['user_guid'])[0];
    
    let collaborators = [];
    let hasEquation = null;

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

    window.addEventListener("message", event => {
        if (event.data.command === "tutorready") {
            if (event.data.username !== user && !collaborators.some(c => c.name === event.data.username)) {
                collaborators = [...collaborators, {
                    name: event.data.from,
                    requesting: false,
                }];
                parent.postMessage({command: "addUser", username: user}, window.location.origin); //TODO this should be a check I can do with the tutor, not this feedback loop of readying
            } else {
                hasEquation = event.data.team_count === 1 ? -1 : 0; //TODO this should also be handled with states and SAIs
            }
        }
        if (event.data.command === "addUser" && event.data.from !== user) {
            collaborators = [...collaborators, {
                name: event.data.from,
                requesting: false,
            }];
        }
        if (event.data.command === "sendEquation" && event.data.from !== user && event.data.to === user) {
            hasEquation = -1;
        }
        if (event.data.command === "requestEquation" && event.data.from !== user) {
            handleIncomingRequest(event.data.from);
        }
    })

    function handleCollaboratorClick(event, index) {
        if (hasEquation !== -1)
            return;
        hasEquation = index;
        collaborators = produce(collaborators, draft => {
            draft[index].requesting = false;
        });
        parent.postMessage({command: "sendEquation", to: collaborators[index].name}, window.location.origin);
    }
    function handleRequestEquation(event) {
        if (hasEquation !== -1)
            parent.postMessage({command: "requestEquation"}, window.location.origin);
    }
    function handleIncomingRequest(collaboratorID) {
        collaborators = produce(collaborators, draft => {
            let collaborator = draft.find(c => c.name === collaboratorID);
            if (!collaborator)
                return;
            collaborator.requesting = true;
        });
        setTimeout(() => {
            collaborators = produce(collaborators, draft => {
                let collaborator = draft.find(c => c.name === collaboratorID);
                if (!collaborator)
                    return;
                collaborator.requesting = false;
            });
        }, 5000);
    }
</script>

<div class="root">
    <div class="collaborators">
        {#each collaborators as collaborator, i}
            <div class="collaborator"
                class:requesting={collaborator.requesting && hasEquation === -1}
                class:has-equation={hasEquation === i}
                in:grow="{{duration: 600}}" out:grow
                on:click={e => handleCollaboratorClick(e, i)}>
                <span>{collaborator.name}</span>
            </div>
        {/each}
    </div>
    {user}
    <div class="history">
        <div class="buttons">
			<button on:click={() => history.step(-1)}>Undo</button>
			<button on:click={() => history.step(1)}>Redo</button>
		</div>
		<History></History>
    </div>
    <div class="equation-area">
        <div class="equation" class:visible={hasEquation === -1}>
        </div>
    </div>
    <div class="request">
        <button on:click={handleRequestEquation}>Request Equation</button>
    </div>
</div>

<style>
    .root {
        display: grid;
        grid-template-rows: 150px auto 150px;
        grid-template-columns: 150px auto auto auto 150px;
        grid-template-areas: 
            "history collaborators collaborators collaborators ."
            "history equation equation equation ."
            "history . request . .";
        width: 100%;
        height: 100%;
    }
    .equation-area {
        grid-area: equation;
        padding: 20px;
    }
    .equation {
        background: #49f;
        width: 100%;
        height: 100%;
        opacity: 0;
        transition: transform 0.3s ease, opacity 0.3s ease-out;
        transform: scale(0.8);
    }
    .equation.visible {
        opacity: 1;
        transform: none;
    }
    .collaborators {
        display: flex;
        grid-area: collaborators;
        align-self: center;
        justify-self: center;
    }
    .collaborator {
        margin: 20px;
        width: 66px;
        height: 66px;
        border-radius: 100%;
        background: #55ee88;
        border: 0 #ffef5b solid;
        transition: transform 0.3s ease, background 0.3s ease, border 0.3s ease;
        color: #fff;
        display: flex;
        justify-content: center;
        align-items: center;
        font-size: 20px;
        cursor: pointer;
        user-select: none;
        box-sizing: border-box;
        -moz-box-sizing: border-box;
        -webkit-box-sizing: border-box;
    }
    .collaborator:hover {
        transform: scale(1.2);
        background: #96fab7;
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
    .request {
        grid-area: request;
        align-self: center;
        justify-self: center;
        font-size: 50px;
    }
    .history {
        grid-area: history;
        padding: 10px;
    }
</style>