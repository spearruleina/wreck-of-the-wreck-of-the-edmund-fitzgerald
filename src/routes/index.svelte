<script lang='ts'>
	import { onMount } from 'svelte'; 
	import { Howl, Howler } from 'howler';
	const snippetLength = 6500;
	let player, isPlaying, destroyMode, requestConfirmation;
	let howler;
	let isnip = 37;
	var icouplet = 0;
	let snippets = [
		[ 0 ],
		[ 16100, "The legend lives on from the Chippewa on down" ],
		[ 20150, "Of the big lake they call Gitche Gumee (*Gichigami, in updated Ojibwe)" ],
		[ 24200 ],
		[ 25600, "The lake, it is said, never gives up her dead" ],
		[ 29200, "When the skies of November turn gloomy" ],
		[ 33760 ],
		[ 35100, "With a load of iron ore twenty-six thousand tons more" ],
		[ 39370, "Than the Edmund Fitzgerald weighed empty" ],
		[ 43400 ],
		[ 44500, "That good ship and true was a bone to be chewed" ],
		[ 48630, "When the gales of November came early" ],
		[ 52900 ],
		[ 54900, "The ship was the pride of the American side" ],
		[ 58500, "Coming back from some mill in Wisconsin" ],
		[ 62300, "As the big freighters go, it was bigger than most" ],
		[ 66400, "With a crew and good captain well seasoned" ],
		[ 70200, "Concluding some terms with a couple of steel firms" ],
		[ 74100, "When they left fully loaded for Cleveland" ],
		[ 77900, "And later that night when the ship's bell rang" ],
		[ 81800, "Could it be the North Wind they'd been feelin'?" ],
		[ 85800 ],
		[ 96000, "The wind in the wires made a tattle-tale sound" ],
		[ 99780, "And a wave broke over the railing" ],
		[ 104100 ],
		[ 105600, "And every man knew, as the captain did too" ],
		[ 109400, "T'was the witch of November come stealin'" ],
		[ 113700 ],
		[ 115400, "The dawn came late and the breakfast had to wait" ],
		[ 118810, "When the Gales of November came slashin'" ],
		[ 123100, "When afternoon came it was freezin' rain" ],
		[ 126800, "In the face of a hurricane west wind" ],
		[ 131100 ],
		[ 146700, "When suppertime came, the old cook came on deck" ],
		[ 150180, "Sayin' \"Fellas, it's too rough to feed yas\"" ],
		[ 154800 ],
		[ 156300, "At seven PM, a main hatchway caved in" ],
		[ 159880, "He said \"Fellas, it's been good to know yas\"" ],
		[ 164380 ],
		[ 166100, "The captain wired in he had water comin' in" ],
		[ 169550, "And the good ship and crew was in peril" ],
		[ 173440, "And later that night when his lights went outta sight" ],
		[ 177350, "Came the wreck of the Edmund Fitzgerald" ],
		[ 181500 ],
		[ 200700, "Does anyone know where the love of Gord goes" ],
		[ 204700, "When the waves turn the minutes to hours?" ],
		[ 209000 ],
		[ 210400, "The searchers all say they'd have made Whitefish Bay" ],
		[ 214200, "If they'd put fifteen more miles behind her" ],
		[ 218600 ],
		[ 220100, "They might have split up or they might have capsized" ],
		[ 223900, "They may have broke deep and took water" ],
		[ 227900, "And all that remains is the faces and the names" ],
		[ 231630, "Of the wives and the sons and the daughters" ],
		[ 235350 ],
		[ 250900, "Lake Huron rolls, Superior sings" ],
		[ 255200, "In the rooms of her ice-water mansion" ],
		[ 258960, "Old Michigan steams like a young man's dreams" ],
		[ 262800, "The islands and bays are for sportsmen" ],
		[ 267020 ],
		[ 268580, "And farther below Lake Ontario" ],
		[ 272560, "Takes in what Lake Erie can send her" ],
		[ 276250, "And the iron boats go as the mariners all know" ],
		[ 280160, "With the gales of November remembered" ],
		[ 284000 ],
		[ 303900, "In a musty old hall in Detroit they prayed" ],
		[ 307390, "In the maritime sailors' cathedral" ],
		[ 311820 ],
		[ 313620, "The church bell chimed 'til it rang twenty-nine times" ],
		[ 317120, "For each man on the Edmund Fitzgerald" ],
		[ 321000 ],
		[ 323100, "The legend lives on from the Chippewa on down" ],
		[ 326660, "Of the big lake they called Gitche Gumee" ],
		[ 330800 ], 
		[ 332600, "Superior, they said, never gives up her dead" ],
		[ 336200, "When the gales of November come early" ], 
		[ 339990 ],
	].reduce((acc, elem, idx) => { 
		acc[idx] = { 
			idx: "snip" + idx, 
			start: elem[0], 
			lyric: elem[1],
			icouplet: (typeof elem[1] == 'undefined') ? undefined : icouplet++ / 2,
		}; 
		if (idx > 0) {
			acc[idx - 1]['duration'] = elem[0] - acc[idx - 1]['start'];
		}
		return acc;
	}, new Array);
	
	function shuffle(array) {
		for (let i = array.length - 1; i > 0; i--) {
			let j = Math.floor(Math.random() * (i + 1));
			[array[i], array[j]] = [array[j], array[i]];
		}
	}

	function splitem(ship) {
		let { even, odd } = ship.filter((e) => { 
			return typeof e.lyric !== 'undefined';
		}).reduce((acc, elem, idx) => {
			let slot = idx % 2 === 0 ? 'even' : 'odd';
			acc[slot].push(elem);
			return acc;
		}, { "even": [], "odd": [] });
		return [even, odd];
	}

	function foldem(orig, even, odd) {
		let icouplet = 0;
		return orig.reduce((acc, e, i) => { 
			acc.push(
				(typeof e.lyric == 'undefined') ? e
			  :	(e.icouplet % 1)                ? odd[Math.floor(e.icouplet)]
			  :                                   even[e.icouplet]
			)
			return acc;
		}, new Array);
	}

	function wreck(ship) {
		let [even, odd] = splitem(ship);
		shuffle(odd);
		shuffle(even);
		return foldem(ship, even, odd);
	}

	function addLyric(lyrics) {
		let p = document.createElement('p');
		lyrics.forEach((e) => { 
			p.append(
				document.createTextNode(e), 
				document.createElement('br')
			)
		});
		return p;
	}

	function lyricize(shipwreck) {
		let iacc = 0;
		return shipwreck.reduce((acc, e) => {
			if (typeof acc[iacc] == 'undefined') acc[iacc] = new Array;
			if (typeof e.lyric == 'undefined' && e.duration > 2000) iacc++;
			if (e.lyric) acc[iacc].push(e.lyric);
			return acc;
		}, new Array);
	}

	function writeLyrics(shipwreck) {
		const lyrics = document.querySelector('#lyrics'); 
		lyrics.innerHTML = '';
		lyricize(shipwreck).forEach((stanza) => { lyrics.appendChild(addLyric(stanza)) });
	}

	function wreckWreck() {
		let iwreck = 0;
		let shipwreck = wreck(snippets);
		howler._sprite = shipwreck.reduce((acc, item, idx) => { 
				acc[item.idx] = [ item.start, item.duration ];
				return acc;
			}, new Object
		);
		// Kluge the finale's duration
		let finale = shipwreck[shipwreck.length - 1];
		howler._sprite[finale.idx][1] = howler.duration() * 1000 - finale.start;
		// chain 'em together
		howler.on('end', () => { 
			console.log(shipwreck[iwreck + 1]);
			if (destroyMode && iwreck < shipwreck.length - 1) {
				howler.play(shipwreck[++iwreck].idx);
			}
		});
		writeLyrics(shipwreck);
		// Fire it up!
		howler.play(shipwreck[iwreck].idx); 
	}
	onMount(() => {
		howler = new Howl({ src: [ 'static/wreck.ogg' ] });
		writeLyrics(snippets);
		/*
		player.addEventListener('play', () => {
			isPlaying = true;
			player.currentTime = snippets[isnip]['start'] / 1000;
		});
		*/
	});

	function confirmDestruction(evt) {
		requestConfirmation = true;
	}

	function destroyArtistry() {
		destroyMode = true;
		requestConfirmation = false;
		wreckWreck();
	}
</script>

<table>
	<tr
		><td>
			<img src="/edmond-fitzgerald.jpg" style="width:200px; margin-right:30px;" alt="Edmund Fitz" />
		</td>
		<td>
			<h1>The Wreck of The Wreck of the Edmund Fitzgerald</h1>

			<p>
				Exceptional storytelling of a grave tragedy... <br/>
				That fateful night when the SS Edmund Fitzgerald broke in half and sank. 
				The larger boats on the U.P. waters could more easily fall victim to the broken wave patterns
				that often accompanied storms on Superior. They call it the bath tub effect, where the waves
				have no clear pattern. Smaller boats just get tossed, but the larger boats get high
				centered, double dropped or hit mid ship by a second wave before coming down. 

			</p>
			<p>
				Gordon does honor to the loss of this crew and its boat with this folksy ballad.
			</p>
			<p>
				(And now, YOU can dishonor the entire affair by Wrecking the Song itself, but you really shouldn't...<br/> 
				You don't really want to do this. Seriously, <strong>don't</strong> press this button. <br/>
				We cannot be held responsible for your Wreckage of the Song*) 
			</p>
			<small>*Truly, this is an unthinkably horrible tragedy where 29 people died in the most terrifying manner; 
				furthermore, i dearly and sincerely love this heartwrenching song.
				But i also have an Imp of the Perverse within me that couldn't resist the amusement of scrambling these repeating musical phrase couplets...
			</small>
		</td>
	</tr>
</table>

{#if destroyMode}
	<div class="dialog">
		Wrecking The Wreck of the Edmund Fitzgerald...

		<div class="dialogButtons">
			<button on:click={() => { 
				destroyMode = false; 
				howler.on('end', null);  // TODO: this doesn't work, and howlers just multiply exponentially!
				howler.stop(); 
				writeLyrics(snippets); // restore original lyrics
			}}>Please, please stop</button>
		</div>
	</div>
{:else}
	<button on:click={confirmDestruction}>Wreck the Wreck</button>
	{#if requestConfirmation}
		<div class="dialog">
			Are you SURE <strong>you</strong> want to be held personally responsible for making your own unconscionable
			mockery of heartfelt balladry around horrific circumstances?
			<div class="dialogButtons">
				<button on:click={() => (requestConfirmation = false)}>
					No, i repent, please cancel immediately;<br/> 
					this is not funny;<br/> 
					take me back to dry land
				</button>
				<button on:click={destroyArtistry}>Yes, Wreck It</button>
			</div>
		</div>
	{/if}
{/if}
<!-- 
<audio bind:this={player} controls autoplay>
	<source src="/wreck.ogg" type="audio/ogg" />
</audio> 
-->
<div id='lyrics'></div>


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
