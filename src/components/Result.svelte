<script>
	import { createEventDispatcher, onMount } from 'svelte';
	import ControlButton from './ControlButton.svelte';

	// props
	export let quizData = [];
	export let totalScore = 0;

	let result = {
		title: '',
		image: '',
		message: ''
	};
	let score = totalScore ?? 0;

	const emit = createEventDispatcher();

	const displayResults = () => {
		const currentResult = quizData.find((el) => score >= el.minpoints && score <= el.maxpoints);

		result = {
			title: currentResult.title,
			image: `/assets/${currentResult.img}`,
			message: currentResult.message
		};
	};
	const resetQuiz = () => {
		emit('resetQuiz');
	};

	onMount(() => {
		displayResults();
	});
</script>

<div class="result-title">{result.title}</div>
<img src={result.image} alt="" />
<p class="message">{result.message}</p>
<div class="score-message">Your score is {score} %</div>
<ControlButton>
	<button on:click={resetQuiz} class="tool-btn start">Play Again</button>
</ControlButton>
