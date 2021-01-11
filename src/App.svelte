<script>
	import {Vector3} from 'three';
	import ThreeDView from './ThreeD/ThreeDView.svelte';
	import Text from './ThreeD/Text.svelte';
	import ObjectSpinZoom from './ThreeD/ObjectSpinZoom.svelte';
	export let name;

	let objects = [
		{text: `Hello ${name}`, position: new Vector3(0,0,0) }
	];
	function click(e) {
		const position = new Vector3(Math.random()-0.5, Math.random()-0.5, Math.random()-0.5).normalize();
		objects.push( {text: name, position});
		objects = objects;
	}
</script>

<main>
	<div class='view3d'>
		<ThreeDView >
			<ObjectSpinZoom>
				{#each objects as object}
				<Text text={object.text}  position={object.position} fontSize={0.2}/>
				{/each}
			</ObjectSpinZoom>
		</ThreeDView>
	</div>
	<button type="button" on:click={click}>Click Me!</button>
</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	.view3d {
		width: 80%;
		height: 600px;
		margin: 16px;
		display: flex;
		background-color: green;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>