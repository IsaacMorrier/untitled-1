<script>
	import throttle from 'just-throttle';
	import { tweened } from 'svelte/motion';
	import { elasticOut } from 'svelte/easing';
	
	export let value, name, key, note, sounds, playSound;
	
	let cx, cy, opacity = 1;
	
	const x = tweened(0, {
					duration: 1200,
					easing: elasticOut
				}),
				y = tweened(0, {
					duration: 1200,
					easing: elasticOut
				});
	
	function simulateClick() {
		
		opacity = 0.9;
		
		// SWING
		
		if (event.offsetY) {
			cx = event.target.offsetWidth * .5;
			cy = event.target.offsetHeight * .4;
			$x = (event.offsetY / cy) * (event.offsetX - cx) / cx;
			$y = (event.offsetY - cy) / cy;
		} else {
			$x = Math.random() - .5;
			$y = 2;
		}
		
		setTimeout(function(){ $x = 0; $y = 0; opacity = 1;}, 200);
	}
	
	function handleMouseenter() {
		playSound(sounds[note]);
		simulateClick();
	}
	
	function handleClick() {
		playSound(sounds[note]);
		simulateClick();
	}
	
	export function handleKey() {
		playSound(sounds[note]);
		simulateClick();
	}
	
</script>

<div on:mouseenter={throttle(handleMouseenter, 200, true)}
		 on:click={handleClick}>
	<button style="background-color: {value};
								 transform: rotateX({$y * -7}deg) rotateY({$x * 7}deg) rotateZ({$x * 3}deg);
								 opacity: {opacity};"
		 aria-label={`A ` + name.toLowerCase() + ` rectangle, playable with the ` + key + ` key.`}></button>
</div>

<style>
	div {
		height: 5rem;
		margin: .5rem;
		perspective: 60rem;
	}
	
	button {
		height: 100%;
		width: 100%;
		margin: 0;
		
		border: none;
		border-radius: 2px;
		box-shadow: 0 3px 11px 0px rgba(84,84,87,0.20);
		-webkit-appearance: none;
		
		transform-origin: 50% 12.5%;
	}
	button::-moz-focus-inner {
		border: 0;
		padding: 0;
	}
	
	@media screen and (min-width: 600px) {
		div {
			flex: 1 0 auto;
			height: auto;
			margin: 1%;
		}
		button {
			padding: 130% 0 0;
		}
	}
	
</style>