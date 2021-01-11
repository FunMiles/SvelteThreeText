<script>
    import { onDestroy, onMount, getContext, setContext } from 'svelte';
    import { tweened } from 'svelte/motion';
    import { quadOut } from 'svelte/easing';
    import * as THREE from 'three';
    import { key } from './threeDViewTag';

    export let minZoom = 0.5;
    export let maxZoom = 4.0;

    let zoom = 1;

    let target = {
        rotation: new THREE.Vector3()
    };

    const rotInterp = (a, b) => ( (t) => new THREE.Vector3(t*b.x+(1-t)*a.x, t*b.y+(1-t)*a.y, t*b.z+(1-t)*a.z) );
    let rotation = tweened(new THREE.Vector3(), {duration: 3000, interpolate: rotInterp, easing: quadOut});

    $: { target.rotation.x = $rotation.x; target.rotation.y = $rotation.y; parentContext.render(); }

    const parentContext = getContext(key);
    // console.log('Ancestor context', parentContext);
    const { getScene, getDomElement }  = parentContext;

    let parentScene = getScene();
    let rScene = new THREE.Scene();
    parentScene.add(rScene);
    // Enclosing element at which event listeners can be attached.
    let domElement = getDomElement();

    const newContext = { ...parentContext};
    newContext.getScene = () => ({
        add: (obj) => {
            target = rScene;
            rScene.add(obj);
            parentContext.markDirty();
        }
    });

    setContext(key, newContext);

    let touchContext = {};
    
    let pointerContext = {};

    let touchDriven = false;

    function registerListeners() {
        domElement.addEventListener( 'contextmenu', onContextMenu, false );

        domElement.addEventListener( 'pointerdown', onPointerDown, false );
	    domElement.addEventListener( 'wheel', onMouseWheel, false );

	    domElement.addEventListener( 'touchstart', onTouchStart, false );

	    domElement.addEventListener( 'keydown', onKeyDown, false );
    }

    onMount( ()=>registerListeners() );

    function changeZoom(fz) {
        zoom *= fz;
        if(zoom < minZoom)
            zoom = minZoom;
        else if (zoom > maxZoom)
            zoom = maxZoom;
        target.scale.x = zoom;
        target.scale.y = zoom;
        target.scale.z = zoom;
    }

    function incrementRotation(dx, dy, duration = 0) {
        let rx = target.rotation.x + 2*dx/zoom;
        if (rx > Math.PI/2)
            rx = Math.PI/2;
        if (rx < -Math.PI/2)
            rx = -Math.PI/2;
        let ry = target.rotation.y + 2*dy/zoom;
        let finalRotation = new THREE.Vector3(rx, ry, 0);
        rotation.set(finalRotation, { duration });
    }

    function getPointerVec(event) {
        const rect = domElement.getBoundingClientRect();
        return { x: event.pageX - (rect.left + window.scrollX), 
                y: event.pageY - (rect.top + window.scrollY), 
                 w: rect.width, h: rect.height,
                time: event.timeStamp };
    }

    function onContextMenu(event) {

    }

    function onPointerDown(event) {
        // console.log('pointer down');
        pointerContext.last = pointerContext.start = getPointerVec(event);
        // console.log('last', pointerContext.last);
        domElement.addEventListener( 'pointermove', onPointerMove );
        window.addEventListener( 'pointerup', onPointerUp, false );
    }
    function onPointerUp(event) {
        domElement.removeEventListener('pointermove', onPointerMove);
        window.removeEventListener('pointerup', onPointerUp);

        if(pointerContext.last.speed) {
            const lst = pointerContext.last;
            // Tween the rotation.
            let vx = lst.speed.x/lst.w;
            let vy = lst.speed.y/lst.h;
            let sp = Math.sqrt(vx*vx + vy*vy);
            if (isFinite(sp)) {
                let duration = 500000*sp;
                incrementRotation(0.5*duration*vy, 0.5*duration*vx, duration);
            }
        }
    }

    function onPointerMove(event) {
        if (touchDriven)
            return;
        // console.log('pointerMove', event);
        let p = getPointerVec(event);
        event.preventDefault();
        let dx = p.x - pointerContext.last.x;
        let dy = p.y - pointerContext.last.y;
        p.speed = {
                x: dx / (p.time-pointerContext.last.time), 
                y: dy / (p.time-pointerContext.last.time)
            };
        // console.log('speed', p.speed);
        pointerContext.last = p;
        // Do rotations...
        incrementRotation(dy / p.h, dx / p.w);
        parentContext.render();
    }

    function onMouseWheel(event) {
        event.preventDefault();
        changeZoom(1-0.02*event.deltaY);
        parentContext.render();
    }

    function getTouchVec(event) {
        const rect = domElement.getBoundingClientRect();
        let touches = [ ... event.touches];

        return { x: touches.map(touch => touch.pageX - (rect.left + window.scrollX)), 
                y: touches.map(touch => touch.pageY - (rect.top + window.scrollY)), 
                 w: rect.width, h: rect.height, time: event.timeStamp };
    }

    function onTouchStart(event) {
        event.preventDefault();
        touchDriven = true;
	    window.addEventListener( 'touchend', onTouchEnd, false );
	    domElement.addEventListener( 'touchmove', onTouchMove, false );
        touchContext.last = getTouchVec(event);
    }

    function onTouchEnd(event) {
        // event.preventDefault();
        touchDriven = false;
        domElement.removeEventListener('touchmove', onTouchMove);
        window.removeEventListener('touchend', onTouchEnd);
        if(touchContext.last.speed) {
            const lst = touchContext.last;
            // Tween the rotation.
            let vx = lst.speed.x/lst.w;
            let vy = lst.speed.y/lst.h;
            let sp = Math.sqrt(vx*vx + vy*vy);
            if (isFinite(sp)) {
                let duration = 500000*sp;
                incrementRotation(0.5*duration*vy, 0.5*duration*vx, duration);
            }
        }
    }

    function onTouchMove(event) {
        event.preventDefault();
        let p = getTouchVec(event);
        if(p.x.length == 1) {
            let dx = p.x[0] - touchContext.last.x[0];
            let dy = p.y[0] - touchContext.last.y[0];
            // Do rotations...
            incrementRotation(dy / p.h, dx / p.w);
            p.speed = {
                x: dx / (p.time-touchContext.last.time), 
                y: dy/ (p.time-touchContext.last.time)
            };
        } else if(p.x.length == 2 && touchContext.last.x.length == 2) {
            let dx0 = touchContext.last.x[1] - touchContext.last.x[0];
            let dy0 = touchContext.last.y[1] - touchContext.last.y[0];
            let dx = p.x[1] - p.x[0];
            let dy = p.y[1] - p.y[0];
            let r = Math.sqrt((dx*dx+dy*dy)/(dx0*dx0+dy0*dy0));
            changeZoom(r);
        }
        touchContext.last = p;

        parentContext.render();
    }

    function onKeyDown(event) {

    }

    function getOffset(el) {
        const rect = el.getBoundingClientRect();
        return {
          left: rect.left + window.scrollX,
          top: rect.top + window.scrollY
        };
  }
</script>
<slot></slot>