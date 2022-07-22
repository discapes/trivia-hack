<script>
	import { onMount } from "svelte";
	import { decodeHTML } from "entities";

	const delay = (s) =>
		new Promise((resolve) => setTimeout(resolve, s * 1000));

	onMount(async () => {
		await newQuestion();
	});

	let answers = [];
	let question = "Loading...";
	let showResult = false;
	let streak = +localStorage.getItem("streak") || 0;
	$: localStorage.setItem("streak", streak);
	
	function submitAnswer(ans) {
		ans.picked = true;
		showResult = true;
		if (ans.correct) streak++;
		else streak = 0;
		return answers;
	}
	async function newQuestion(delayS) {
		answers = [];
		showResult = false;
		question = "Question loading...";
		await delay(delayS);
		let difficulty = Math.random() > 0.5 ? "easy" : "medium";
		let res;
		do {
			res = (
				await fetch(
					`https://opentdb.com/api.php?amount=1&type=multiple&difficulty=${difficulty}`
				).then((res) => res.json())
			).results[0];
		} while (res.category.toLowerCase().includes("entertainment"));
		answers = [
			...res.incorrect_answers.map((name) => ({ name })),
			{ name: res.correct_answer, correct: true },
		];
		shuffleArray(answers);
		question = decodeHTML(res.question);
	}

	function shuffleArray(array) {
		for (let i = array.length - 1; i > 0; i--) {
			const j = Math.floor(Math.random() * (i + 1));
			const temp = array[i];
			array[i] = array[j];
			array[j] = temp;
		}
	}
</script>

<main>
	<h1>{question}</h1>
	<div class="answers">
		{#each answers as answer}
			<button
				disabled={showResult}
				class:correct={answer.correct && showResult}
				class:incorrect={!answer.correct && showResult && answer.picked}
				on:click={() => (answers = submitAnswer(answer))}
			>
				{answer.name}
			</button>
		{/each}
	</div>
	<h2>🔥 {streak}</h2>
	<button
		on:click={() => newQuestion()}
		style:visibility={showResult ? "visible" : "hidden"}>Next</button
	>
</main>

<style>
	button {
		background: white;
		border-width: 0;
		border-radius: 5px;
	}
	button:disabled {
		color: #666666;
	}
	button:hover:enabled,
	button:active:enabled {
		background: #f6f6f6;
	}
	* {
		box-sizing: border-box;
	}
	h2 {
		margin: 5px 0 20px 0;
	}
	h1 {
		margin-block-start: 0.67em;
		margin-block-end: 0.67em;
		text-align: center;
		max-width: 800px;
	}
	:global(body, html) {
		margin: 0;
		padding: 0;
	}
	main {
		background: lightblue;
		padding: 40px;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		height: 100vh;
		overflow-y: auto;
		overflow-x: hidden;
	}
	.answers {
		display: flex;
		flex-direction: column;
		align-items: stretch;
		width: 100%;
		margin: 10px 50px;
		max-width: 500px;
	}
	.correct {
		background-color: lightgreen;
		color: black !important;
	}
	.incorrect {
		background-color: lightpink;
	}
</style>
