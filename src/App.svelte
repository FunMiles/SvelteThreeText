<script>
	import {Vector3} from 'three';
	import ThreeDView from './ThreeD/ThreeDView.svelte';
	import Text from './ThreeD/Text.svelte';
	import ObjectSpinZoom from './ThreeD/ObjectSpinZoom.svelte';
	export let name;

	let outlineColor = '#ff0000';
	let bodyColor = '#0000ff';

	let position = new Vector3(Math.random()-0.5, Math.random()-0.5, Math.random()-0.5).normalize();

	$: console.log('Outline color:', outlineColor);
	let objects = [
		{text: `Hello ${name}`, position: new Vector3(0,0,0), bodyColor, outlineColor }
	];
	function click(e) {
		objects.push( {text: name, position, bodyColor, outlineColor});
		objects = objects;
		position = new Vector3(Math.random()-0.5, Math.random()-0.5, Math.random()-0.5).normalize();
	}
</script>

<main>
	<div class='view3d'>
		<ThreeDView >
			<ObjectSpinZoom>
				{#each objects as object}
				<Text text={object.text}  
				color={object.bodyColor}
				outlineColor={object.outlineColor}
				fontSize={0.2}
				outlineWidth={0.01}
				position={object.position} 
				/>
				{/each}
			</ObjectSpinZoom>
		</ThreeDView>
	</div>
	<label for="text">Text:</label>
	<input type="text" id="text" bind:value={name}>
	<label for='x'>x</label>  
	<input type='number' id='x' bind:value={position.x}>
	<label for='y'>y</label>  
	<input type='number' id='x' bind:value={position.y}>
	<label for='z'>z</label>  
	<input type='number' id='x' bind:value={position.z}>
	<p>Choose the text colors:</p>
	<div class='colorchoices'>
		<label for="body">Text body</label>
    	<input type="color" id="body" name="body" bind:value={bodyColor}>
		<label for="outline">Outline</label>
   		<input type="color" id="outline" name="outline" bind:value={outlineColor}>
		<button type="button" on:click={click}>Add Text</button>
	</div>
</main>

<style>
	main {
		text-align: left;
		padding: 0.1em;
		margin: 0 auto;
	}

	#body {
		height: 36px;
		margin: 4px;
		padding: 0;
	}
	#outline {
		height: 36px;
		margin: 4px;
		padding: 0;
	}
	button {
		padding: 8px;
		margin: 8px
	}
	.view3d {
		width: 80%;
		height: 600px;
		margin: 16px;
		display: flex;
		background-color: green;
	}
	.colorchoices {
		display: flex;
		flex-direction: horizontal;
		align-items: center;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>