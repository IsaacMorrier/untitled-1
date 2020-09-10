<script>
	import { onMount } from 'svelte';
	import Panel from './Panel.svelte';
	
	let spectrum = [
				{	name: 'Red',	value: '#ff0000',	key: 'a',	note: 'Db3'},
				{	name: 'Dark Red',	value: '#d4002b',	key: 's',	note: 'Eb3'},
				{	name: 'Dark Magenta',	value: '#aa0055',	key: 'd',	note: 'F3'},
				{	name: 'Purple',	value: '#800080',	key: 'f',	note: 'G3'},
				{	name: 'Violet',	value: '#5500aa',	key: 'j',	note: 'A3'},
				{	name: 'Indigo',	value: '#2b00d4',	key: 'k',	note: 'B3'},
				{	name: 'Blue',	value: '#0000ff',	key: 'l',	note: 'Db4'}
			],
			sounds = {},
			panels = [],
			playSound,
			canvas,
			favicon = document.querySelector('link[rel*="icon"]');
	
	// FAVICON

	onMount(() => {
		const ctx = canvas.getContext('2d');
		const c1 = hexToRGB("#ff0000");
		const c2 = hexToRGB("#0000ff");
		let frame;
		
		function hexToRGB(hex) {
			let r = 0, g = 0, b = 0;
			r = parseInt(hex[1] + hex[2], 16);
			g = parseInt(hex[3] + hex[4], 16);
			b = parseInt(hex[5] + hex[6], 16);
			return [r, g, b];
		}
		
		function colorNow(place, c1, c2) {
			let cn = ((place * c1) + (1 - place) * c2);
			return cn;
		}
		
		(function loop() {
			frame = requestAnimationFrame(loop);
			const t = window.performance.now();
			const p = Math.abs(((t % 20000) / 10000) - 1).toFixed(2);
			const r = colorNow(p, c1[0], c2[0]);
			const g = colorNow(p, c1[1], c2[1]);
			const b = colorNow(p, c1[2], c2[2]);
			ctx.fillStyle = "rgb(" + r + ", " + g + ", " + b + ")";
			ctx.fillRect(0, 0, canvas.width, canvas.height);
			
			favicon.href = canvas.toDataURL('image/png');
		}());
		
		return () => {
			cancelAnimationFrame(frame);
		};
		

	});
	
	// AUDIO
	
	const AudioContext = window.AudioContext || window.webkitAudioContext;
	if (!window.AudioContext) {
		
		for (const spectr of spectrum) {
			sounds[spectr.note] = new Audio("sounds/"+spectr.note+".m4a");
			sounds[spectr.note].load();
    }
		
		playSound = function(t) {
			t.cloneNode(true).play();
		}
		
	} else {
		
		for (const spectr of spectrum) {
			
			var ctx = new AudioContext(),
					gainNode = ctx.createGain();
			gainNode.connect(ctx.destination);

			function createBuffer() {
				ctx.decodeAudioData(this.response, function(b) {
					sounds[spectr.note] = b;
					}, function(e){console.warn(e)});
			}

			var file = "sounds/"+spectr.note+".m4a",
			xhr = new XMLHttpRequest();
			xhr.onload = createBuffer;
			xhr.open('GET', file, true);
			xhr.responseType = 'arraybuffer';
			xhr.send();
    }
		
		playSound = function(buf){
			var source = ctx.createBufferSource();
			source.buffer = buf;
			source.connect(gainNode);
			source.onended = function() {
				if (this.stop) this.stop();
				if (this.disconnect) this.disconnect();
			}
			source.start(0);
		}
	}
	
	// KEY HANDLER
	
	function handleKeydown(event) {

		for (const spectr of spectrum) {
			if (event.key === spectr.key) {
				panels[spectrum.indexOf(spectr)].handleKey();
			}
    }
		
    return false;
	}
	
</script>

<svelte:window on:keydown={handleKeydown}/>

<main>
	{#each spectrum as spectr, i}
		<Panel {...spectr} {sounds} {playSound} bind:this={panels[i]}/> 
	{/each}
</main>
<footer>
	<p>{#each spectrum as spectr, i}<kbd style="color:{spectr.value}">{spectr.key}</kbd>{/each}</p>
</footer>
<canvas
	bind:this={canvas}
	width={32}
	height={32}
	hidden></canvas>

<style>
	main, footer {
		padding: 1em;
	}
	footer {
		font-family: sans-serif;
		font-size: 0.875rem;
		text-align: center;
		color: #888;
	}
	p {
		line-height: 1.5;
	}
	kbd {
		background: #eee;
		padding: .2rem .4rem;
		margin-left: .2rem;
		border-radius: 2px;
	}
	@media screen and (min-width: 600px) {
		main {
			display: flex;
			flex-direction: row;
		}
	}
</style>