<script>
    import { onDestroy, onMount, getContext } from 'svelte';
    import * as THREE from 'three';
    import { key } from './threeDViewTag';
    import {Text} from 'troika-three-text'; 

    export let text;
    export let position = new THREE.Vector3();
    export let quaternion = new THREE.Quaternion();
    export let fontSize = 0.02;
    export let color = 0xffffff;
    export let outlineWidth = 0;
    export let outlineColor = 0;
    const { getScene, markDirty } = getContext(key);

    const triggerRender = () => {
        markDirty && markDirty();
    };

    const myText = new Text();
    myText.addEventListener('synccomplete', triggerRender);
    myText.anchorX = 'center';
    myText.anchorY = 'middle';
    $: myText.text = text;
    $: { 
        myText.position.x = position.x; 
        myText.position.y = position.y; 
        myText.position.z = position.z;
        myText.quaternion.x = quaternion.x;
        myText.quaternion.y = quaternion.y;
        myText.quaternion.z = quaternion.z;
        myText.quaternion.w = quaternion.w;
    }
    $: myText.fontSize = fontSize;
    $: myText.color = color;
    $: myText.outlineWidth = outlineWidth;
    $: myText.outlineColor = outlineColor;
    $: { 
        myText.sync();
    }


    onMount( () => {
        console.log('Adding to scene');
        
        getScene().add(myText);
        markDirty && markDirty();
    });
    onDestroy( () => {
        if(myText.parent)
            myText.parent.remove(myText);
        myText.removeEventListener('synccomplete', triggerRender);
        myText.dispose();
    });
</script>