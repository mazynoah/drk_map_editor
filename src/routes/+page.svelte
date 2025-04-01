<script lang="ts">
	let width = 20;
	let height = 20;
	let selectedTileType = 'NONE';
	let playerNum = 1;

	const tileOptions = ['ALTAR', 'APPRENTICE', 'EGG', 'NONE'];

	const tileCharacters: { [key: string]: string } = {
		ALTAR: '%',
		APPRENTICE: '@',
		EGG: 'O',
		NONE: ''
	};

	let board = Array.from({ length: height }, () => Array(width).fill('NONE'));

	$: board = Array.from({ length: height }, (_, y) =>
		Array.from({ length: width }, (_, x) => board[y]?.[x] || 'NONE')
	);

	function queryEntityData(x: number, y: number): any {
		if (selectedTileType == 'EGG') {
			const fields = ['name', 'hatch_time', 'health', 'strength', 'attack_range', 'regen'];

			const egg: { position: { x: number; y: number }; [key: string]: any } = {
				position: { x: x + 1, y: y + 1 }
			};

			for (const field of fields) {
				let res = window.prompt(`Enter the value for ${field}`);
				if (res === null) {
					return false;
				}

				egg[field] = res;
			}

			console.log('egg:', egg);
			return egg;
		} else if (selectedTileType == 'APPRENTICE') {
			const fields = ['name', 'health', 'regen'];

			const apprentice: { [key: string]: any } = {
				position: { x: x + 1, y: y + 1 },
				player: playerNum
			};

			for (const field of fields) {
				let res = window.prompt(`Enter the value for ${field}`);
				if (res === null) {
					return false;
				}

				apprentice[field] = res;
			}

			console.log('apprentice:', apprentice);
			return apprentice;
		} else if (selectedTileType == 'ALTAR') {
			const altar = {
				position: { x: x + 1, y: y + 1 },
				player: playerNum
			};

			return altar;
		}

		return null;
	}

	function updateCell(x: number, y: number) {
		let res = queryEntityData(x, y);

		if (res != null) board[y][x] = { tiletype: selectedTileType, pl: playerNum, data: res };
	}

	function getColor(tiletype: string, playern: number) {
		const x: { [key: string]: any } = {
			ALTAR_1: 'background-color: red;',
			ALTAR_2: 'background-color: blue;',
			APPRENTICE_1: 'background-color: yellow;',
			APPRENTICE_2: 'background-color: purple;',
			EGG_1: 'background-color: green;',
			EGG_2: 'background-color: green;'
		};

		return x[tiletype + '_' + playern];
	}

	function exportBoard() {
		const mapSize = `${board.length} ${board[0].length}`;
		const altars: any = [];
		const apprentices: any = [];
		const eggs: any = [];

		board.forEach((row) => {
			row.forEach((cell) => {
				if (typeof cell === 'object') {
					const { tiletype, data } = cell;
					if (tiletype === 'ALTAR') {
						altars.push(`${data.player} ${data.position.y} ${data.position.x}`);
					} else if (tiletype === 'APPRENTICE') {
						const name = data.name.replace(/\s+/g, '_');
						apprentices.push(
							`${data.player} ${name} ${data.position.y} ${data.position.x} ${data.health} ${data.regen}`
						);
					} else if (tiletype === 'EGG') {
						const name = data.name.replace(/\s+/g, '_');
						eggs.push(
							`${name} ${data.position.y} ${data.position.x} ${data.hatch_time} ${data.health} ${data.strength} ${data.attack_range} ${data.regen}`
						);
					}
				}
			});
		});

		let output = `map:\n${mapSize}\naltars:\n${altars.join('\n')}\napprentices:\n${apprentices.join('\n')}\neggs:\n${eggs.join('\n')}`;

		const blob = new Blob([output], { type: 'text/plain' });
		const url = URL.createObjectURL(blob);

		const link = document.createElement('a');
		link.href = url;
		link.download = 'map.drk';
		link.click();

		URL.revokeObjectURL(url);
	}

	function clear() {
		board = Array.from({ length: height }, () => Array(width).fill('NONE'));
	}
</script>

<main>
	<h1>Welcome to drk editor</h1>
	<p>
		<b>How to:</b>
		<br />Select a player and a tile
		<br />Click anywhere on the board to place a tile
		<br />Right click a tile to delete it
	</p>

	<div class="input-container">
		<div class="input-group">
			<label for="width">Map width</label>
			<input type="range" name="width" id="width" min="20" max="60" step="1" bind:value={width} />
			<span>{width}</span>
		</div>

		<div class="input-group">
			<label for="height">Map height</label>
			<input
				type="range"
				name="height"
				id="height"
				min="20"
				max="40"
				step="1"
				bind:value={height}
			/>
			<span>{height}</span>
		</div>

		<div class="input-group">
			<label for="playern">Player</label>
			<select name="playern" id="playern" bind:value={playerNum}>
				<option value={1}>Player 1</option>
				<option value={2}>Player 2</option>
			</select>
		</div>
		<div class="input-group">
			<label for="pickedTile">Tile picker</label>
			<select name="pickedTile" id="pickedTile" bind:value={selectedTileType}>
				{#each tileOptions as tile}
					<option value={tile}>{tile}</option>
				{/each}
			</select>
		</div>
		<div class="input-group">
			<input type="button" value="Export" onclick={exportBoard} />
			<input type="button" value="Clear" onclick={clear} />
		</div>
	</div>

	<div
		class="board"
		style="grid-template-columns: repeat({width}, 1fr); grid-template-rows: repeat({height}, 1fr);"
	>
		{#each board as row, y}
			{#each row as cell, x}
				<!-- svelte-ignore a11y_no_static_element_interactions -->
				<!-- svelte-ignore a11y_click_events_have_key_events -->
				<div
					class="tile"
					style={getColor(cell.tiletype, cell.pl)}
					oncontextmenu={(e) => {
						e.preventDefault();
						board[y][x] = { tiletype: 'NONE', pl: playerNum };
					}}
					onclick={() => updateCell(x, y)}
				>
					{tileCharacters[cell.tiletype]}
				</div>
			{/each}
		{/each}
	</div>
	<footer>Noah Mazy</footer>
</main>

<style>
	main {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		gap: 2rem;
	}

	.input-container {
		display: flex;
		flex-direction: column;
		gap: 1rem;
	}

	.input-group {
		display: flex;
		align-items: center;
		gap: 0.5rem;
	}

	.board {
		display: grid;
		gap: 2px;
		width: 100%;
		max-width: 1000px;
		border: 1px solid #ccc;
	}

	.tile {
		width: 100%;
		aspect-ratio: 1 / 1;
		background-color: #f0f0f0;
		border: 1px solid #ddd;
		cursor: pointer;
		transition: background-color 0.2s;

		display: flex;
		justify-content: center;
		align-items: center;
	}

	.tile:hover {
		background-color: #e0e0e0;
	}
</style>
