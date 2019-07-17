<script>
	import History from './components/History.svelte';
	import Waiting from './components/WaitingScreen.svelte';
	import Collaborator from './components/Collaborator.svelte';
	import Notification from './components/RequestNotification.svelte';
    import { history } from './stores/history.js';
    import { produce } from 'immer';

    // let user = "test";
    // let collaborators = "test,reallylongname,test2".split(',').map(user => ({name: user, requesting: false}));
    let user = CTATConfiguration.get('user_guid');
    let collaborators = CTATConfiguration.get('collaborators').split(',').map(user => ({name: user, requesting: false}));
    let index = collaborators.findIndex(c => c.name === user)
    let team_count = 3;
    let hasEquation = 0;
    

    window.addEventListener("message", event => {
        switch (event.data.command) {
            case "tutorready":
                team_count = event.data.team_count;
                break;
            case "sendEquation":
                hasEquation = collaborators.findIndex(c => event.data.to === c.name);
                break;
            case "requestEquation":
                if (event.data.from !== user) {
                    handleIncomingRequest(event.data.from);
                }
                break;
            default:
                break;
        }
    })

    function handleCollaboratorClick(event, i) {
        if (hasEquation !== index)
            return;
        collaborators = produce(collaborators, draft => {
            draft[i].requesting = false;
        });
        parent.postMessage({command: "sendEquation", to: collaborators[i].name}, window.location.origin);
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
    {#if team_count < collaborators.length} 
        <Waiting/>
    {/if}
    <div class="collaborators">
        {#each collaborators as collaborator, i}
            {#if collaborator.name !== user}
                <Collaborator {...collaborator}
                    onClick={e => handleCollaboratorClick(e, i)}
                    hasEquation={hasEquation === i}
                    canGiveEquation={hasEquation === index}/>
            {/if}
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
        <div class="equation" class:visible={hasEquation === index}>
        </div>
    </div>
    <div class="request-button">
        <button on:click={handleRequestEquation}>Request Equation</button>
    </div>
    <div class="requests">
        {#each collaborators as request, i}
            {#if request.requesting}
                <Notification name={request.name} onClick={e => handleCollaboratorClick(e, i)} />
            {/if}
        {/each}
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
    .request-button {
        grid-area: request;
        align-self: center;
        justify-self: center;
        font-size: 50px;
    }
    .history {
        grid-area: history;
        padding: 10px;
    }
    .requests {
        position: fixed;
        overflow: hidden;
        top: 10%;
        left: 0;
        right: 0;
        bottom: 0;
        pointer-events:none;    
    }
</style>