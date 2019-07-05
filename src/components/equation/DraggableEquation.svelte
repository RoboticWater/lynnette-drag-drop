<script>
    import {draggableEqn} from '../dragdrop/draggableEqn.js'
    
    export let screenRects;
    export let player;
    export let start;


    let dragging = false;
    let curPlayer;

	let coords = start || {x: 0, y: 0}
	let vel = {x: 0, y: 0}
	let last_time = window.performance.now();
    let frame;
    let cornerFlags = [0,0,0,0]
    
    let prevList = []
    
    let draggableRef;
    let dRect;

    window.addEventListener("message", event => {
        console.log(1);
        
        if (event.origin === window.location.origin)
            return;
        if (event.data.command === "dragEqn") {
            coords = event.data.coords;
            curPlayer = event.data.player;
            if (event.data.vel)
                vel = event.data.vel;
            
        }
    })

    function pointInRect(point, rect) {
        return (point.x <= rect.x + rect.width && point.x >= rect.x) && (point.y <= rect.y + rect.height && point.y >= rect.y)
    }

	function lerp(value1, value2, amount) {
        amount = amount < 0 ? 0 : amount;
        amount = amount > 1 ? 1 : amount;
        return value1 + (value2 - value1) * amount;
    }

    function avg() {
		let ax = 0;
        let ay = 0;        
		for (let i = 1; i < prevList.length; i++) {
			ax += prevList[i].x - prevList[i - 1].x;
			ay += prevList[i].y - prevList[i - 1].y;
		}
		return {x: ax / prevList.length, y: ay / prevList.length}
    }
    
    function checkCorners() {
        cornerFlags = [-1, -1, -1, -1];
        dRect = draggableRef.getBoundingClientRect();
        let corners = [
            {x: dRect.x + dRect.width / 2, y: dRect.y},
            {x: dRect.x + dRect.width, y: dRect.y + dRect.height / 2},
            {x: dRect.x + dRect.width / 2, y: dRect.y + dRect.height},
            {x: dRect.x, y: dRect.y + dRect.height / 2}];
        corners.forEach((corner, i) => {
            screenRects.forEach((screen, j) => {
                if (pointInRect(corner, screen)) {
                    cornerFlags[i] = j;
                }
            })
        })
    }

    (function update() {
		frame = requestAnimationFrame(update);

		const time = window.performance.now();
		const delta = time - last_time;
        if (dragging) {
            if (frame % 10 === 0) {
                let prev = prevList[prevList.length - 1];
                if (cornerFlags.some(f => f > -1 && f !== player) && prev && (Math.floor(prev.x) !== Math.floor(coords.x) || Math.floor(prev.y) !== Math.floor(coords.y))) {
                    parent.postMessage({command: "dragEqn", coords: coords, player: player}, window.location.origin);
                }
            }
            if (frame % 2 === 0) {
                prevList.push(Object.assign({}, coords));
                if (prevList.length > 3)
                    prevList.shift();
                if (frame % 4 === 0) {
                    checkCorners();
                }
            }
        } else {
            if (vel.x + vel.y !== 0) {
                coords.x += vel.x * delta * 0.15;
                coords.y += vel.y * delta * 0.15;
                vel.x = lerp(vel.x, 0, 0.01 * delta);
                vel.y = lerp(vel.y, 0, 0.01 * delta);
                if (Math.abs(vel.x) < 0.25) {
                    vel.x = 0;
                }
                if (Math.abs(vel.y) < 0.25) {
                    vel.y = 0;
                }
                
                if (frame % 5 === 0) {
                    checkCorners()
                }
                // if (frame % 10 === 0) {
                //     parent.postMessage({command: "dragEqn", coords: coords, vel: vel}, window.location.origin)
                // }
            }
            
            let out = [cornerFlags[0] < 0, cornerFlags[1] < 0, cornerFlags[2] < 0, cornerFlags[3] < 0];
            if (out[0]) {
                vel.y += 0.3;
            }
            if (out[1]) {
                vel.x -= 0.3;
            }
            if (out[2]) {
                vel.y -= 0.3;
            }
            if (out[3]) {
                vel.x += 0.3;
            }
            if (out.every(c => c) && dRect) {
                let x = dRect.x - (screenRects[curPlayer].x + screenRects[curPlayer].width / 2);
                let y = dRect.y - (screenRects[curPlayer].y + screenRects[curPlayer].height / 2);
                let d = Math.sqrt(x * x + y * y);
                vel.x -= x / d * 0.3;
                vel.y -= y / d * 0.3;
            }
        }

		last_time = time;
    }());
    
    function handleDragEnd(e) {
        vel = avg();
        dragging = false;
        parent.postMessage({command: "dragEqn", coords: coords, vel: vel, player: player}, window.location.origin);
    }
</script>

<div class="draggable"
    bind:this={draggableRef}
    style="transform: translate({coords.x}px,{coords.y}px)"
    use:draggableEqn
    on:dragstart={e=> {
        prevList = [];
        dragging = true;
        curPlayer = player;
    }}
    on:dragmove={e => { coords.x = e.detail.x; coords.y = e.detail.y; vel = {x: 0, y: 0}; }}
    on:dragend={handleDragEnd}>
    <div class="corner one">{cornerFlags[0]}</div>
    <div class="corner two">{cornerFlags[1]}</div>
    <div class="corner three">{cornerFlags[2]}</div>
    <div class="corner four">{cornerFlags[3]}</div>
</div>


<style>
    .draggable {
        width: 300px;
        height: 300px;
        background: #fff;
        
        display: grid;
        grid-template-areas: 
            ". one ."
            "four . two"
            ". three .";

        user-select: none;
        touch-action: none;
        position: absolute;
        z-index: 5000;
    }
    .corner {
        background: #3be96f;
        line-height: 100px;
    }
    .one {
        grid-area: one;
    }
    .two {
        grid-area: two;
    }
    .three {
        grid-area: three;
    }
    .four {
        grid-area: four;
    }
</style>