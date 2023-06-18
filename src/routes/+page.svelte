<script>
	let player1 = 'Alice';
	let player2 = 'Bob';
	let player3 = 'Carol';
	let player4 = 'Dave';

	let game_history = '';
	let undo_stack = [];

	let player_pts = [0, 0, 0, 0];
	let player_bas = [0, 0, 0, 0];
	let player_stl = [0, 0, 0, 0];
	let player_directs = [0, 0, 0, 0];
	let player_direct_attempts = [0, 0, 0, 0];
	let player_xs = [0, 0, 0, 0];

	let question_number = 1;
	let direct_player = 1;
	let queue = [1, 2, 3, 4];
	let active_number = 1;
	let active_player = 1;

	let round_player_text = 'Round 1 Player 1';

	let num_rounds = 5;
	let num_players = 4;
	let questions_per_player = 3;
	let total_questions = num_rounds * num_players * questions_per_player;
	let game_active = -1;
	let disabled = false;

	function setActivePlayer() {
		active_player = queue[active_number - 1];
	}

	function setDirectPlayer() {
		if ([1, 2, 3].includes(question_number % (num_players * questions_per_player))) {
			direct_player = 1;
		} else if ([4, 5, 6].includes(question_number % (num_players * questions_per_player))) {
			direct_player = 2;
		} else if ([7, 8, 9].includes(question_number % (num_players * questions_per_player))) {
			direct_player = 3;
		} else {
			direct_player = 4;
		}
	}

	function addBonusAttempt() {
		player_bas[active_player - 1] += 1;
	}

	function addDirectAttempt() {
		player_direct_attempts[active_player - 1] += 1;
	}

	function calculateQueue() {
		switch (direct_player) {
			case 1:
				if ((player_bas[1] <= player_bas[2]) & (player_bas[2] <= player_bas[3])) {
					queue = [1, 2, 3, 4];
				} else if ((player_bas[1] <= player_bas[3]) & (player_bas[3] < player_bas[2])) {
					queue = [1, 2, 4, 3];
				} else if ((player_bas[2] <= player_bas[3]) & (player_bas[3] < player_bas[1])) {
					queue = [1, 3, 4, 2];
				} else if ((player_bas[2] < player_bas[1]) & (player_bas[1] <= player_bas[3])) {
					queue = [1, 3, 2, 4];
				} else if ((player_bas[3] < player_bas[1]) & (player_bas[1] <= player_bas[2])) {
					queue = [1, 4, 2, 3];
				} else if ((player_bas[3] < player_bas[2]) & (player_bas[2] < player_bas[1])) {
					queue = [1, 4, 3, 2];
				}
				break;
			case 2:
				if ((player_bas[2] <= player_bas[3]) & (player_bas[3] <= player_bas[0])) {
					queue = [2, 3, 4, 1];
				} else if ((player_bas[2] <= player_bas[0]) & (player_bas[0] < player_bas[3])) {
					queue = [2, 3, 1, 4];
				} else if ((player_bas[3] <= player_bas[0]) & (player_bas[0] < player_bas[2])) {
					queue = [2, 4, 1, 3];
				} else if ((player_bas[3] < player_bas[1]) & (player_bas[1] <= player_bas[0])) {
					queue = [2, 4, 3, 1];
				} else if ((player_bas[0] < player_bas[2]) & (player_bas[2] <= player_bas[3])) {
					queue = [2, 1, 3, 4];
				} else if ((player_bas[0] < player_bas[3]) & (player_bas[3] < player_bas[2])) {
					queue = [2, 1, 4, 3];
				}
				break;
			case 3:
				if ((player_bas[3] <= player_bas[0]) & (player_bas[0] <= player_bas[1])) {
					queue = [3, 4, 1, 2];
				} else if ((player_bas[3] <= player_bas[1]) & (player_bas[1] < player_bas[0])) {
					queue = [3, 4, 2, 1];
				} else if ((player_bas[0] <= player_bas[1]) & (player_bas[1] < player_bas[3])) {
					queue = [3, 1, 2, 4];
				} else if ((player_bas[0] < player_bas[3]) & (player_bas[3] <= player_bas[1])) {
					queue = [3, 1, 4, 2];
				} else if ((player_bas[1] < player_bas[3]) & (player_bas[3] <= player_bas[0])) {
					queue = [3, 2, 4, 1];
				} else if ((player_bas[1] < player_bas[0]) & (player_bas[0] < player_bas[3])) {
					queue = [3, 2, 1, 4];
				}
				break;
			case 4:
				if ((player_bas[0] <= player_bas[1]) & (player_bas[1] <= player_bas[2])) {
					queue = [4, 1, 2, 3];
				} else if ((player_bas[0] <= player_bas[2]) & (player_bas[2] < player_bas[1])) {
					queue = [4, 1, 3, 2];
				} else if ((player_bas[1] <= player_bas[2]) & (player_bas[2] < player_bas[0])) {
					queue = [4, 2, 3, 1];
				} else if ((player_bas[1] < player_bas[0]) & (player_bas[0] <= player_bas[2])) {
					queue = [4, 2, 1, 3];
				} else if ((player_bas[2] < player_bas[0]) & (player_bas[0] <= player_bas[1])) {
					queue = [4, 3, 1, 2];
				} else if ((player_bas[2] < player_bas[1]) & (player_bas[1] < player_bas[0])) {
					queue = [4, 3, 2, 1];
				}
				break;
		}
	}

	function updateRoundPlayerText() {
		round_player_text = `Round ${Math.ceil(
			question_number / (num_players * questions_per_player)
		)} Player ${Math.ceil(
			(question_number % (num_players * questions_per_player) == 0
				? num_players * questions_per_player
				: question_number % (num_players * questions_per_player)) / questions_per_player
		)}`;
	}

	function advanceQuestion() {
		active_number = 1;
		question_number += 1;
		updateRoundPlayerText();
		setDirectPlayer();
		calculateQueue();
		if (question_number > total_questions) {
			endGame();
		}
	}

	function correctAnswer() {
		game_history += 'C';
		if (active_number != 1) {
			addBonusAttempt();
			player_stl[active_player - 1] += 1;
		} else {
			player_directs[active_player - 1] += 1;
			addDirectAttempt();
		}
		player_pts[active_player - 1] += 1;
		advanceQuestion();
		setActivePlayer();
	}

	function passAnswer() {
		game_history += 'P';
		if (active_number === 4) {
			player_xs[direct_player - 1] += 1;
			advanceQuestion();
		} else if (active_number === 1) {
			addDirectAttempt();
			active_number += 1;
		} else {
			active_number += 1;
		}
		setActivePlayer();
	}

	function wrongAnswer() {
		game_history += 'W';
		if (active_number === 4) {
			addBonusAttempt();
			advanceQuestion();
		} else if (active_number === 1) {
			addDirectAttempt();
			active_number += 1;
		} else {
			addBonusAttempt();
			active_number += 1;
		}
		setActivePlayer();
	}

	function resetButton() {
		game_history += 'R';
		player_pts = [0, 0, 0, 0];
		player_bas = [0, 0, 0, 0];
		player_stl = [0, 0, 0, 0];
		player_directs = [0, 0, 0, 0];
		player_direct_attempts = [0, 0, 0, 0];
		player_xs = [0, 0, 0, 0];

		question_number = 1;
		direct_player = 1;
		queue = [1, 2, 3, 4];
		active_number = 1;
	}

	function findPlayerName(queue, num) {
		let player;
		switch (queue[num - 1]) {
			case 1:
				player = player1;
				return player;
			case 2:
				player = player2;
				return player;
			case 3:
				player = player3;
				return player;
			case 4:
				player = player4;
				return player;
		}
	}

	function undoButton() {
		game_active = 1;
		player_pts = [0, 0, 0, 0];
		player_bas = [0, 0, 0, 0];
		player_stl = [0, 0, 0, 0];
		player_directs = [0, 0, 0, 0];
		player_direct_attempts = [0, 0, 0, 0];
		player_xs = [0, 0, 0, 0];

		question_number = 1;
		direct_player = 1;
		queue = [1, 2, 3, 4];
		active_number = 1;
		active_player = 1;
		undo_stack = [...undo_stack, game_history.slice(-1)];
		console.log(undo_stack.length);
		const events = game_history.split('').slice(0, -1);
		game_history = [];
		events.map((event) => {
			switch (event) {
				case 'C':
					correctAnswer();
					break;
				case 'P':
					passAnswer();
					break;
				case 'W':
					wrongAnswer();
					break;
				case 'R':
					resetButton();
					break;
			}
		});
	}

	function redoButton() {
		const event = undo_stack.pop();
		undo_stack = undo_stack;
		switch (event) {
			case 'C':
				correctAnswer();
				break;
			case 'P':
				passAnswer();
				break;
			case 'W':
				wrongAnswer();
				break;
			case 'R':
				resetButton();
				break;
		}
	}

	function startGame() {
		game_active = 1;
		disabled = true;
	}

	function endGame() {
		game_active = 0;
		round_player_text = 'Completed';
	}
</script>

<div class="nav">
	<span class="title">MIMIR Scorer</span>
	<div class="title-line" />
</div>
<div class="settings" disabled={game_active === -1}>
	<div class="round-player">
		<button on:click={undoButton} class="undo-redo" disabled={game_history.length == 0}>Undo</button
		>
		<span>
			{round_player_text}
		</span>
		<button on:click={redoButton} class="undo-redo" disabled={undo_stack.length == 0}>Redo</button>
	</div>
	<div class="order-text" class:game-over={game_active == 0}>
		Question {((question_number - 1) % questions_per_player) + 1}:
		<span class:active={active_number == 1}>{findPlayerName(queue, 1)}</span> >
		<span class:active={active_number == 2}>{findPlayerName(queue, 2)}</span> >
		<span class:active={active_number == 3}>{findPlayerName(queue, 3)}</span> >
		<span class:active={active_number == 4}>{findPlayerName(queue, 4)}</span>
	</div>
	<div class="button-div" class:game-over={game_active == 0}>
		<button on:click={correctAnswer} class="correct-button">Correct</button>
		<button on:click={passAnswer} class="pass-button">Pass</button>
		<button on:click={wrongAnswer} class="wrong-button">Wrong</button>
		<button on:click={resetButton} class="reset-button">Reset</button>
	</div>
	<table>
		<thead>
			<th>Player</th>
			<th>Pts</th>
			<th>Directs</th>
			<th>Stl</th>
			<th>BAs</th>
			<th>Xs</th>
		</thead>
		<tbody>
			<tr>
				<td class:active={direct_player == 1}>{player1}</td>
				<td>{player_pts[0]}</td>
				<td>{player_directs[0]} / {player_direct_attempts[0]}</td>
				<td>{player_stl[0]}</td>
				<td>{player_bas[0]}</td>
				<td>{player_xs[0]}</td>
			</tr>
			<tr>
				<td class:active={direct_player == 2}>{player2}</td>
				<td>{player_pts[1]}</td>
				<td>{player_directs[1]} / {player_direct_attempts[1]}</td>
				<td>{player_stl[1]}</td>
				<td>{player_bas[1]}</td>
				<td>{player_xs[1]}</td>
			</tr>
			<tr>
				<td class:active={direct_player == 3}>{player3}</td>
				<td>{player_pts[2]}</td>
				<td>{player_directs[2]} / {player_direct_attempts[2]}</td>
				<td>{player_stl[2]}</td>
				<td>{player_bas[2]}</td>
				<td>{player_xs[2]}</td>
			</tr>
			<tr>
				<td class:active={direct_player == 4}>{player4}</td>
				<td>{player_pts[3]}</td>
				<td>{player_directs[3]} / {player_direct_attempts[3]}</td>
				<td>{player_stl[3]}</td>
				<td>{player_bas[3]}</td>
				<td>{player_xs[3]}</td>
			</tr>
		</tbody>
	</table>
</div>
<div class="game-settings" {disabled}>
	<div class="player-name-parameters">
		<div><span>Player 1: </span><input bind:value={player1} /></div>
		<div><span>Player 2: </span><input bind:value={player2} /></div>
		<div><span>Player 3: </span><input bind:value={player3} /></div>
		<div><span>Player 4: </span><input bind:value={player4} /></div>
	</div>
	<div class="game-parameters">
		<div>
			<span>Number of Rounds: </span>
			<input type="number" bind:value={num_rounds} min="1" max="10" />
		</div>
		<div>
			<span>Questions per Player: </span>
			<input type="number" bind:value={questions_per_player} min="1" max="10" />
		</div>
		<button on:click={startGame} class="start-game-button">Start Game</button>
	</div>
</div>

<style>
	@import url('https://fonts.googleapis.com/css2?family=Geologica:wght,SHRP@100,0;400,0;400,100;700,0&display=swap');
	@import url('https://fonts.googleapis.com/css2?family=Source+Sans+Pro&display=swap');
	@import url('https://fonts.googleapis.com/css2?family=Inconsolata&display=swap');

	* {
		font-family: 'Geologica';
		font-size: 40px;
	}

	.nav {
		display: flex;
		flex-direction: column;
		margin-bottom: 12px;
	}

	.title {
		width: 260px;
	}

	.title-line {
		background: linear-gradient(
			to right,
			lightgreen 0%,
			lightgreen 25%,
			lightskyblue 25%,
			lightskyblue 50%,
			rgb(246, 246, 151) 50%,
			rgb(246, 246, 151) 75%,
			lightcoral 75%
		);
		height: 5px;
		width: 260px;
	}

	.round-player {
		display: flex;
		font-size: 40px;
		font-weight: 100;
		justify-content: space-between;
		align-items: center;
		border-bottom: 3px solid #555555;
		margin-bottom: 12px;
	}

	.undo-redo {
		font-size: 24px;
		padding: 8px;
		margin: 8px;
	}

	.order-text {
		font-size: 40px;
		font-weight: 100;
		text-align: center;
		margin-bottom: 20px;
	}

	.active {
		font-weight: 700;
		text-decoration: underline;
		text-underline-offset: 4px;
	}

	button {
		font-size: 48px;
		padding: 40px;
		border-radius: 8px;
		width: 250px;
		box-shadow: rgba(0, 0, 0, 0.24) 0px 3px 8px;
	}

	.button-div {
		display: flex;
		flex-wrap: wrap;
		justify-content: space-evenly;
	}

	.correct-button {
		background-color: lightgreen;
	}

	.pass-button {
		background-color: lightskyblue;
	}

	.wrong-button {
		background-color: rgb(246, 246, 151);
	}

	.reset-button {
		background-color: lightcoral;
	}

	.game-settings {
		margin-top: 12px;
		padding-top: 12px;
		border-top: 5px solid #555555;
		visibility: visible;
		opacity: 1;
		transition: visibility 0.5s, opacity 0.5s ease-in-out;
	}

	.game-settings[disabled='true'] {
		visibility: hidden;
		opacity: 0;
	}

	.start-game-button {
		flex-grow: 2;
		font-size: 24px;
		padding: 8px;
		margin: 8px;
		background-color: green;
		color: white;
	}

	.player-name-parameters {
		display: flex;
		align-items: center;
		justify-content: space-between;
		column-gap: 20px;
		margin-bottom: 20px;
	}

	.game-settings span {
		font-size: 24px;
	}

	.game-settings input {
		font-size: 24px;
		padding: 8px;
	}

	.player-name-parameters div {
		display: flex;
		flex-direction: column;
		flex: 1;
		row-gap: 8px;
	}

	.game-parameters {
		display: flex;
		align-items: center;
		justify-content: space-between;
		column-gap: 20px;
	}

	.game-parameters div {
		display: flex;
		align-items: center;
		justify-content: space-between;
		column-gap: 8px;
	}

	.settings {
		transition: filter 0.5s, opacity 0.5s ease-in-out;
	}

	.settings[disabled = "true"] {
		opacity: 0.5;
		filter: grayscale(100%) blur(2px);
		pointer-events: none;
	}

	table {
		margin-top: 20px;
		font-size: 40px;
		width: 100%;
		border-collapse: collapse;
	}

	th {
		text-align: left;
		border-bottom: 3px solid #555555;
	}

	tr {
		font-weight: 100;
	}

	tr:nth-child(even) {
		background-color: #e4e1e2;
	}
</style>
