<script>
    import { onDestroy, onMount, setContext } from 'svelte';
    import { key } from './threeDViewTag';
    import * as THREE from 'three';

    export let style;

    let offsetWidth;
    let offsetHeight;
    let el;

    const scene = new THREE.Scene();
    let camera = new THREE.PerspectiveCamera(25, window.innerWidth / window.innerHeight, 0.9, 9.2);
    camera.position.z = 5;
    const light = new THREE.AmbientLight( 0xe0e0e0 ); // soft white light
    scene.add( light );
    const dirLight = new THREE.DirectionalLight( 0xffffff, 1, 100 );
    dirLight.position.set( 0, 1, 6 ); //default; light shining from slightly above the camera
    // dirLight.castShadow = true; // default false
    scene.add( dirLight );

    /// The element surrounding the canvas.
    let domElement;
    /// The three.js renderer
    let renderer;
    let updater;

    $: if(updater) { updater(); updater = null;}

    export const render = ()=> {
        if(renderer && scene && camera)
            renderer.render(scene, camera);
    }

    const animate = () => {
        // requestAnimationFrame(animate);
        renderer.render(scene, camera);
    };

    const resize = (width, height) => {
        if(renderer)
            renderer.setSize(width, height)
        camera.aspect = width / height;
        camera.updateProjectionMatrix();
        if(renderer)
            renderer.render(scene, camera);
    };

    $: { 
        console.log('resize',offsetWidth, offsetHeight); 
        resize(offsetWidth, offsetHeight); 
    }

    setContext(key, {
        getScene: () => ({ 
            add(object) {
                scene.add(object);
                requestAnimationFrame(animate);
            }
        }),
        setCamera: (cam) => (camera = cam),
        render: () => { renderer.render(scene, camera); },
        getDomElement: () => domElement,
        markDirty: () => { updater = render; }
    });

    onMount(() => {
        renderer = new THREE.WebGLRenderer({ antialias: true, canvas: el });
        resize(domElement.offsetWidth, domElement.offsetHeight);
        renderer.render(scene, camera);
    });

    onDestroy(()=> {
        renderer.dispose();
    });
</script>

<div {style} bind:offsetWidth bind:offsetHeight bind:this={domElement}>
    <canvas bind:this={el}>
        {#if renderer}
		<slot></slot>
	    {/if}
    </canvas>
</div>
<style>
    div {
        width: 100%;
        flex-grow: 1;
        flex-shrink: 1;
        overflow: hidden;
        background-color: red;
        margin: 0;
        border: 0;
        padding: 0;
        display: flex;
    }
    canvas {
        flex-grow: 1;
        flex-shrink: 1;
        margin: 0;
        border: 0;
        padding: 0;
    }
</style>