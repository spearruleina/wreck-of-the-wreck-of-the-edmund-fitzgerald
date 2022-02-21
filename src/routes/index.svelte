<script>
	const snippetLength = 500;
	let player, isPlaying, destroyMode, requestConfirmation;
	import { onMount } from 'svelte';

	onMount(() => {
		player.addEventListener('play', () => {
			isPlaying = true;
		});
		player.addEventListener('playing', () => {
			isPlaying = true;
		});
		player.addEventListener('pause', () => {
			isPlaying = false;
			destroyMode = false;
		});

		isPlaying = !player.paused;

		setInterval(() => {
			if (destroyMode) {
				const snippetCount = (player.duration * 1000) / snippetLength;
				const curSnippet = Math.floor(Math.random() * snippetCount);
				const jumpPoint = (curSnippet * snippetLength) / 1000;
				player.currentTime = jumpPoint;
				// player.play();
			}
		}, snippetLength);
	});

	function confirmDestruction(evt) {
		requestConfirmation = true;
	}

	function destroyArtistry() {
		destroyMode = true;
		requestConfirmation = false;
	}
</script>

<table>
	<tr
		><td>
			<img src="/edmond-fitzgerald.jpg" style="width:200px; margin-right:30px;" alt="Edmund Fitz" />
		</td>
		<td>
			<h1>The Wreck of The Wreck of The Wreck of the Edmund Fitzgerald</h1>

			<p>
				Exceptional storytelling! That fateful night when the boat broke in half and sunk. The
				larger boats on the U.P. waters could more easily fall victim to the broken wave patterns
				that often accompanied storms on Superior. They call it the bath tub effect, where the waves
				have no clear pattern. Smaller boats just get tossed, but the larger boats get high
				centered, double dropped or hit mid ship by a second wave before coming down. Gordon does
				honor to the loss of this crew and its boat with this folksy ballad.
			</p>
		</td>
	</tr>
</table>

<div>
	<audio bind:this={player} controls autoplay>
		<source src="/wreck.ogg" type="audio/ogg" />
	</audio>
</div>

{#if isPlaying}
	{#if destroyMode}
		<div class="dialog">
			Destroying The Wreck of the Edmund Fitzgerald

			<div class="dialogButtons">
				<button on:click={() => (destroyMode = false)}>Please stop</button>
			</div>
		</div>
	{:else}
		<button on:click={confirmDestruction}>Destroy artistry</button>
		{#if requestConfirmation}
			<div class="dialog">
				Are you SURE you want to be held personally responsible for making your own unconscionable
				mockery of heartfelt balladry?
				<div class="dialogButtons">
					<button on:click={() => (requestConfirmation = false)}>No, cancel</button>
					<button on:click={destroyArtistry}>Yes, destroy</button>
				</div>
			</div>
		{/if}
	{/if}
{/if}

<style>
	.dialog {
		width: 400px;
		background-color: #ddf;
		border: 1px solid #666;
		padding: 10px;
		margin: 10px;
	}
	.dialogButtons {
		margin-top: 5px;
		text-align: right;
	}
</style>
