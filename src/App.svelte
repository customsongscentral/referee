<script>
	const wsUrl = import.meta.env.VITE_WS_URL;
	const iconUrl = 'https://static-cdn.jtvnw.net/jtv_user_pictures/4640a0e5-399d-42e5-89ca-1e004966a89f-profile_image-70x70.png';
	const emptyEmbed = {
		name: '\u200B',
		value: '\u200B',
		inline: true
	};

	let phase = 'Elite League Season 2 - Group stage';
	let winCondition = 4;
	const localStorageSongs = localStorage.getItem('songs');
	let songs1Raw = localStorageSongs ? JSON.parse(localStorageSongs) : [
		{ "cat": "Solo", "artist": "Mike Orlando", "name": "Full Speed X", "charter": "Chemfinal", "source": "AH2" },
		{ "cat": "Solo", "artist": "Alkan", "name": "Comme Le Vent (110%)", "charter": "Paturages", "source": "Personal" },
		{ "cat": "Solo", "artist": "LeaF & sak", "name": "Heterochromia Iridis", "charter": "Mintorment", "source": "2021-02" },
		{ "sep": true },
		{ "cat": "Strum", "artist": "Keep of Kalessin", "name": "Novae Ruptis", "charter": "Branspeed", "source": "AH1" },
		{ "cat": "Strum", "artist": "The Black Dahlia Murder", "name": "Miasma", "charter": "GHX", "source": "GHX" },
		{ "cat": "Strum", "artist": "Camellia", "name": "Bangin' Burst (110%)", "charter": "Rek3dge", "source": "Fuse Box" },
		{ "sep": true },
		{ "cat": "Hybrid", "artist": "Angel Vivaldi", "name": "Acid Reign (125%)", "charter": "Dillski", "source": "AH2" },
		{ "cat": "Hybrid", "artist": "Johnny Hiland", "name": "Barnyard Breakdown (110%)", "charter": "jdurandTV", "source": "CTH3" },
		{ "cat": "Hybrid", "artist": "Michael Paouris", "name": "Django's Rainy Heart (for Django)", "charter": "Haggis", "source": "Exclusive" },
		{ "cat": "Hybrid", "artist": "Mike Dawes", "name": "One (Metallica Cover) [CSC:CS edit] (110%)", "charter": "3-UP", "source": "2021-06" },
		{ "sep": true },
		{ "cat": "Tiebreaker (solo)", "artist": "Vitalij Kuprij", "name": "Destination", "charter": "Peddy & OrangeHat", "source": "MH2" },
		{ "cat": "Tiebreaker (strum)", "artist": "Allegaeon", "name": "Apoptosis (130%)", "charter": "Miscellany", "source": "AH2" },
		{ "cat": "Tiebreaker (hybrid)", "artist": "Dragonforce", "name": "Body Breakdown", "charter": "Chemfinal & Peddy", "source": "DFD v2" }
	];
	let songs1 = songs1Raw.filter(s => !s.sep);

	let scoreDiscordUrls = localStorage.getItem('scoreDiscordUrls');
	let pickbanDiscordUrls = localStorage.getItem('pickbanDiscordUrls');
	let player1 = localStorage.getItem('player1') || 'Player 1';
	let player2 = localStorage.getItem('player2') || 'Player 2';
	let picker = localStorage.getItem('picker') || 1;
	let winner = localStorage.getItem('winner') || 0;
	let imageUrl = '';
	let screenshots = [];

	let storageMatchScore = localStorage.getItem('matchScore')
	let matchScore = storageMatchScore ? JSON.parse(storageMatchScore) : [, 0, 0]

	let storageBans = localStorage.getItem('bans');
	let storagePicks = localStorage.getItem('picks');
	let bans = storageBans ? JSON.parse(storageBans) : [];
	let picks = storagePicks ? JSON.parse(storagePicks) : [];
	let tiebreaker = localStorage.getItem('tiebreaker');
	let showChangePool = false;

	const pick = song => {
		picks = [...picks, { picker, song }];
		
		// Alternate pickers
		// picker = picker == 1 ? 2 : 1;
		
		// Compute tiebreaker
		let solos = 0, strums = 0;
		picks.forEach(pick => {
			if (pick.song.cat.toLowerCase().includes('solo')) solos++;
			if (pick.song.cat.toLowerCase().includes('strum')) strums++;
		});
		if (solos > strums) tiebreaker = 'strum';
		else if (strums > solos) tiebreaker = 'solo';
		else tiebreaker = 'hybrid';
	}
	const ban = song => {
		bans = [...bans, { picker, song }];
		if (bans.length == 2)
			pickPhase = 'pick';

		picker = picker == 1 ? 2 : 1;
	}
	const undoPick = () => {
		picks = picks.slice(0, -1);
	}
	const undoBan = () => {
		bans = bans.slice(0, -1);
	}

	const setWin = p => {
		winner = p;
		matchScore = matchScore.map((x, i) => +(winner == i) + x);
		if (picks.length) picks[picks.length - 1].winner = winner;
		
		const [, s1, s2] = matchScore;
		const tbCondition = winCondition - 1;
		if (s1 == tbCondition && s1 == s2) picker = 0; // TB
		else if (s1 > tbCondition || s2 > tbCondition) picker = 3; // End of match
		else picker = p == 1 ? 2 : 1; // Loser always picks otherwise

		if (picker == 0) {
			alert('TB HYPE! Send the score update, then pick TB then send score update again');
		} else if (picker == 3) {
			alert('END OF MATCH! ' + (s1 > s2 ? player1 : player2) + ' wins!');
		}
	};

	const sendScoreUpdate = () => {
		let pickWin = winner == 3 ? "It's a tie! Next point counts double." : `**${[, player1, player2][winner]}** wins the pick!`;
		//const p = picker === 3 ? -1 : -2;
		const p = -1;
		pickWin += `\n[${picks.slice(p)[0].song.cat}] **${picks.slice(p)[0].song.artist} - ${picks.slice(p)[0].song.name}**\n`;

		let nextPickOrMatchEnd;
		if (picker == 3) { // Match end
			nextPickOrMatchEnd = `\n**${matchScore[1] > matchScore[2] ? player1 : player2}** wins the match!`;
		} else if (picks.length && !picks[picks.length - 1].winner) { // The pick has been done
			const pick = picks.slice(-1)[0];
			nextPickOrMatchEnd = `\nNext pick: [${pick.song.cat}] **${pick.song.artist} - ${pick.song.name}** picked by **${['Tiebreaker', player1, player2][pick.picker]}**`;
			pickWin = '';
		} else { // Just after the win, the pick has to be done
			nextPickOrMatchEnd = `Next pick:** ${['Tiebreaker', player1, player2][picker]}**`;
		}

		const embeds = [
			{
				title: `${winner == 1 ? '__' : ''}[     ${matchScore[1]}     ]${winner == 1 ? '__' : ''}     -     ${winner == 2 ? '__' : ''}[     ${matchScore[2]}     ]${winner == 2 ? '__' : ''}`,
				description: winner ? `${pickWin}${nextPickOrMatchEnd}` : `A new match has started!\n${nextPickOrMatchEnd}`,
				fields: [
					{
						name: `${player1}'s bans`,
						value: bans.map(({ picker, song }) => picker == 1 ? `[${song.cat}] ${song.artist} - ${song.name}` : '').filter(Boolean).join('\n') || 'No bans',
						inline: true
					},
					{
						name: `${player2}'s bans`,
						value: bans.map(({ picker, song }) => picker == 2 ? `[${song.cat}] ${song.artist} - ${song.name}` : '').filter(Boolean).join('\n') || 'No bans',
						inline: true
					},
					emptyEmbed,
					{
						name: `${player1}'s picks`,
						value: picks.map(({ winner, picker, song }) => picker == 1 ? `[${song.cat}] ${song.artist} - ${song.name}${winner ? ` (won by ${[,player1,player2][winner]})` : ''}` : '\u200B').join('\n') || '\u200B',
						inline: true
					},
					{
						name: `${player2}'s picks`,
						value: picks.map(({ winner, picker, song }) => picker == 2 ? `[${song.cat}] ${song.artist} - ${song.name}${winner ? ` (won by ${[,player1,player2][winner]})` : ''}` : '\u200B').join('\n') || '\u200B',
						inline: true
					},
					emptyEmbed
				].filter(Boolean),
				author: {
					name: `${phase}:     ${player1}     vs.     ${player2}`,
					"icon_url": iconUrl
				},
				footer: {
					"text": "Custom Songs Central " + phase,
					"icon_url": iconUrl
				},
				...(imageUrl && { image: { url: imageUrl } }),
			}
		];

		const body = new FormData();
		body.append('payload_json', JSON.stringify({
			embeds,
			username: "CSC:CS",
			avatar_url: iconUrl
		}));
		if (screenshots && screenshots.length) body.append('file', screenshots[0]);

		// Send Discord messages
		for (const url of scoreDiscordUrls.split('\n')) {
			if (url.startsWith('https://')) fetch(url, { method: 'POST', body });
		}
		// Send scene changes
		const ws = new WebSocket(wsUrl);
		ws.onopen = () => {
			ws.send(JSON.stringify({
				songs: songs1Raw,
				player1,
				player2,
				picker,
				winner,
				matchScore,
				bans,
				picks,
				tiebreaker,
				winCondition
			}));
			ws.close();
		};

		localStorage.setItem('scoreDiscordUrls', scoreDiscordUrls);
		localStorage.setItem('player1', player1);
		localStorage.setItem('player2', player2);
		localStorage.setItem('picker', picker);
		localStorage.setItem('winner', winner || 0);
		localStorage.setItem('matchScore', JSON.stringify(matchScore));
		localStorage.setItem('bans', JSON.stringify(bans));
		localStorage.setItem('picks', JSON.stringify(picks));
		localStorage.setItem('tiebreaker', tiebreaker);
		localStorage.setItem('phase', phase);
		localStorage.setItem('winCondition', winCondition);
		localStorage.setItem('songs', JSON.stringify(songs1Raw));
		screenshots = [];
		imageUrl = '';
		document.getElementById('upload').value = '';
		document.getElementById('clipboard').src = 'about:blank';
	}

	let pickPhase = bans.length >= 2 ? 'pick' : 'ban';
	const sendPickbanStatus = () => {
		const songs = songs1;
		const embeds = [
			{
				"title": `${[, player1, player2][picker]}'s turn to ${pickPhase}`,
				description: songs.map(s => {
					if (s.cat.includes('Tiebreaker')) return;
					let str = `[${s.cat}] ${s.artist} - ${s.name}`;
					if (bans.find(b => b.song.artist == s.artist && b.song.name == s.name)) {
						str = `~~${str}~~`;
					} else if (picks.find(p => p.song.artist == s.artist && p.song.name == s.name)) {
						str = `__${str}__`;
					} else {
						str = `--> **${str}**`;
					}
					return str;
				}).filter(Boolean).join('\n'),
				author: {
					name: `${phase}:     ${player1}     vs.     ${player2}`,
					"icon_url": iconUrl
				},
				"footer": {
					"text": "Crossed songs are banned, underlined songs were already picked, songs in bold are available"
				}
			}
		];
		
		const body = new FormData();
		body.append(
			"payload_json",
			JSON.stringify({
				embeds,
				username: "CSC:CS",
				avatar_url: iconUrl,
			})
		);
		// Send Discord messages
		for (const url of pickbanDiscordUrls.split('\n')) {
			if (url.startsWith('https://')) fetch(url, { method: 'POST', body });
		}
		// Send scene changes
		const ws = new WebSocket(wsUrl);
		ws.onopen = () => {
			ws.send(JSON.stringify({
				songs: songs1Raw,
				player1,
				player2,
				picker,
				winner,
				matchScore,
				bans,
				picks,
				tiebreaker,
				winCondition
			}));
			ws.close();
		};
	};

	let simpleMessage;
	const sendSimpleMessage = () => {
		const body = new FormData();
		body.append(
			"payload_json",
			JSON.stringify({
				content: simpleMessage,
				username: "CSC:CS",
				avatar_url: iconUrl,
			})
		);
		for (const url of scoreDiscordUrls.split('\n')) {
			if (url.startsWith('https://')) fetch(url, { method: 'POST', body });
		}
	};

	const reset = () => {
		localStorage.clear();
		localStorage.setItem('scoreDiscordUrls', scoreDiscordUrls);
		localStorage.setItem('pickbanDiscordUrls', pickbanDiscordUrls);
		localStorage.setItem('player1', player1);
		localStorage.setItem('player2', player2);
		localStorage.setItem('songs', JSON.stringify(songs1Raw));
		localStorage.setItem('phase', phase);
		localStorage.setItem('winCondition', winCondition);
		location.reload();
	};

	const restartServer = server => {
		const ws = new WebSocket(wsUrl);
		const pw = {
			andromeda: import.meta.env.VITE_PW_1,
			bumblefoot: import.meta.env.VITE_PW_2,
			cassiopea: import.meta.env.VITE_PW_3,
			dragonforce: import.meta.env.VITE_PW_4,
			equipoise: import.meta.env.VITE_PW_5
		}[server];
		ws.onopen = () => {
			ws.send(`please restart ${pw}`);
			ws.close();
		};
	};

	const updatePool = e => {
		songs1Raw = e.target.value.split('\n').map(line => {
			if (!line.trim()) return { sep: true };
			const [cat, full, charter,, source] = line.split('\t');
			const [artist, name] = full.split(' - ');
			return { cat, artist, name, charter, source };
		});
		songs1 = songs1Raw.filter(s => !s.sep);
	}

	let randomNumber = 0;
	const roll = () => (randomNumber = (1 + Math.random() * 100) >> 0);

	document.onpaste = event => {
		const items = (event.clipboardData || event.originalEvent.clipboardData).items;
		
		for (let index in items) {
			const item = items[index];
			if (item.kind === 'file') {
				const blob = item.getAsFile();
				screenshots = [blob];
				const reader = new FileReader();
				reader.onload = event => (document.getElementById('clipboard').src = event.target.result); // data url!
				reader.readAsDataURL(blob);
			}
		}
	}
</script>

<main>
	{#if showChangePool}
	<div class="pool-modal">
		<div class="modal-overlay" />
		<div class="modal-body">
			Copy/paste from the sheet directly<br />
			Leave an empty line for separation in the stream scene<br />
			<textarea class="modal-textarea" on:input={updatePool}></textarea>
			<br />
			<button on:click={() => (showChangePool = false)}>OK</button>
		</div>
	</div>
	{/if}
	<div class="form">
		<button style="background:#ff9999" on:click={reset}>Reset everything</button>
		&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
		&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
		<button on:click={() => (showChangePool = true)}>Change pool</button>
		&emsp;Phase <input bind:value={phase} style="width:25em" />
		<div class="players">
			<h3>Players</h3>
			<input placeholder="player 1" bind:value={player1} />
			<input placeholder="player 2" bind:value={player2} />
			Random number: <b>{randomNumber}</b>
			<button on:click={roll}>Roll</button>
			<h4>Who <b>{pickPhase}s</b>?</h4>
			<b>Phase</b>
			<button on:click={() => (pickPhase = 'ban')}>Ban</button>
			<button on:click={() => {
				pickPhase = 'pick';
				picker = picker == 1 ? 2 : 1;
			}}>Pick</button>
			<br />
			<button on:click={() => (picker = 1)}>{player1 || 'Player 1'}</button>
			<button on:click={() => (picker = 2)}>{player2 || 'Player 2'}</button>
			<button on:click={() => (picker = 0)}>Tiebreaker</button>
			<button on:click={() => (picker = 3)}>End</button>
			Current: <b>{['Tiebreaker', player1, player2, 'End'][picker]}</b>
		</div>
		<div class="songs">
			<h3>Songs</h3>
			<div class="songs-container">
				{#each songs1 as song, i}
					<div class={`song song--${song.cat} ${song.cat.toLowerCase().includes(tiebreaker) ? 'tb-pick' : ''}`}>
						<div class="song-label">
							[{song.cat}]<br />
							{song.artist}<br />
							{song.name}
						</div>
						{#if bans.find(b => b.song.artist == song.artist && b.song.name == song.name)}
							<b style="color:red">Banned</b>
						{:else if picks.find(b => b.song.artist == song.artist && b.song.name == song.name)}
							<b>Picked</b>
						{:else if picker && song.cat.includes('Tiebreaker')}
							<b>can't pick TB yet</b>
						{:else if !picker && !song.cat.includes('Tiebreaker')}
							<b style="color:#999">TB situation</b>
						{:else if pickPhase == 'pick'}
							<button on:click={() => pick(song)}>Pick</button>
						{:else}
							<button on:click={() => ban(song)}>Ban</button>
						{/if}
					</div>
				{/each}
			</div>
		</div>
	</div>
	<div class="preview">
		<textarea placeholder="webhook urls" bind:value={scoreDiscordUrls} wrap="off" style="font-size: 10px; width: 30em; height: 10em"></textarea>
		<button on:click={sendScoreUpdate}>Send score update</button>
		<textarea placeholder="emergency message" style="font-size: 10px; width: 30em; height: 10em" bind:value={simpleMessage}></textarea>
		<button on:click={sendSimpleMessage}>Send emergency message</button>
		<p>
			{phase}: <b>{player1}</b> vs. <b>{player2}</b><br />
			Score:
			<button on:click={() => (matchScore = [, matchScore[1]+1, matchScore[2]])}>+</button>
			<button on:click={() => (matchScore = [, matchScore[1]-1, matchScore[2]])}>-</button>
			<b>{matchScore[1]} - {matchScore[2]}</b>
			<button on:click={() => (matchScore = [, matchScore[1], matchScore[2]+1])}>+</button>
			<button on:click={() => (matchScore = [, matchScore[1], matchScore[2]-1])}>-</button>
			&emsp;
			pts to win <input type="number" bind:value={winCondition} />
		</p>
		
		<div>
			Screenshot <input placeholder="Screenshot" type="file" bind:files={screenshots} id="upload" /> or <input placeholder="url" bind:value={imageUrl} />
			or just paste from clipboard
			<br />
			<img id="clipboard" src="about:blank" alt="" />
		</div>
		<p class="scores">
			{#if picks.length && !picks[picks.length - 1].winner}
				<b>Winner</b>
				<button on:click={() => setWin(1)}>{player1 || 'Player 1'}</button>
				<button on:click={() => setWin(2)}>{player2 || 'Player 2'}</button>
				<button on:click={() => { winner = 3; }}>Tie</button>
			{/if}
			Last winner: <b>{['-', player1, player2, 'Tie'][winner]}</b>
		</p>

		<div>
			Bans
			<div class="picks">
				<div>
					{#each bans as { picker, song }}
						<div>{picker == 1 ? `[${song.cat}] ${song.artist} - ${song.name}` : '-'}</div>
					{/each}
					<button on:click={undoBan}>Undo</button>
				</div>
				<div>
					{#each bans as { picker, song }}
						<div>{picker == 2 ? `[${song.cat}] ${song.artist} - ${song.name}` : '-'}</div>
					{/each}
				</div>
			</div>
		</div>
		<div>
			Picks
			<div class="picks">
				<div>
					{#each picks as { picker, song, winner }}
						<div>{picker == 1 ? `[${song.cat}] ${song.artist} - ${song.name} (winner: ${[, player1, player2][winner] || 'N/A'})` : '-'}</div>
					{/each}
					<button on:click={undoPick}>Undo</button>
				</div>
				<div>
					{#each picks as { picker, song, winner }}
						<div>{picker == 2 ? `[${song.cat}] ${song.artist} - ${song.name} (winner: ${[, player1, player2][winner] || 'N/A'})` : '-'}</div>
					{/each}
				</div>
			</div>
		</div>
		<textarea placeholder="webhook urls" bind:value={pickbanDiscordUrls} wrap="off" style="font-size: 10px; width: 30em; height: 10em"></textarea>
		<button on:click={sendPickbanStatus}>Send ban/pick status</button>
		<div class="servers">
			(please only click once on those, you should get feedback in #server-log)<br />
			<button style="background:#ff9999" on:click={() => restartServer('andromeda')}>Restart Andromeda (10001)</button>
			<button style="background:#ff9999" on:click={() => restartServer('bumblefoot')}>Restart Bumblefoot (10002)</button>
			<button style="background:#ff9999" on:click={() => restartServer('cassiopea')}>Restart Cassiopea (10003)</button><br />
			<button style="background:#ff9999" on:click={() => restartServer('dragonforce')}>Restart Dragonforce (10004)</button>
			<button style="background:#ff9999" on:click={() => restartServer('equipoise')}>Restart Equipoise (10005)</button>
		</div>
	</div>
</main>

<style>
	main {
		display: flex;
		width: 100%;
		height: 100%;
	}

	.form, .preview {
		flex: 1;
		padding: 1em;
	}

	.form {
		border-right: 1px solid #aaa;
		overflow-y: scroll;
	}

	.songs-container {
		display: flex;
		flex-flow: row wrap;
	}

	.songs {
		margin-bottom: 4em;
	}

	.song {
		border: 1px solid #aaa;
		margin: .5em;
		text-align: center;
	}
	.song button {
		margin: 0 1em;
	}

	.song--Solo { background: #cfe2f3 }
	.song--Strum { background: #fff2cc }
	.song--Hybrid { background: #d9ead3 }
	.song--Gimmick { background: #ffaaff }
	.song--Boss { background: #d19494 }
	.song--Tiebreaker { background: #ea9999; opacity: .5; }
	.song--Tiebreaker.tb-pick { opacity: 1; }

	.song-label {
		width: 15em;
	}

	.picks {
		display: flex;
	}

	.picks > div {
		width: 30em;
	}

	#clipboard {
		width: 20em;
	}

	.servers {
		text-align: right;
		margin-top: 5em;
	}

	.pool-modal {
		position: fixed;
		top: 0;
		bottom: 0;
		left: 0;
		right: 0;
	}

	.modal-overlay {
		z-index: 1;
		position: absolute;
		height: 100%;
		width: 100%;
		background: #0009;
	}

	.modal-body {
		padding: 1em;
		z-index: 2;
		position: absolute;
		left: 50%;
		top: 10em;
		transform: translateX(-50%);
		width: 75%;
		height: 20em;
		background: white;
		border: #000e 2px solid;
	}

	.modal-textarea {
		width: 60rem;
		height: 15rem;
		font-size: 10px;
		font-family: 'Courier New', Courier, monospace;

	}
</style>