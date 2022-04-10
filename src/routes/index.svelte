<script>
	import { onMount } from 'svelte';
	import ControlButton from '../components/ControlButton.svelte';
	import Question from '../components/Question.svelte';
	import Result from '../components/Result.svelte';

	let errorMessage = '';
	let isQuizActive = false;
	let displayedComponent = 'question';
	let totalScore = 0;
	let quizData = null;

	const fetchQuizData = async (statement) => {
		const baseUrl =
			'https://apis-game-default-rtdb.europe-west1.firebasedatabase.app/apis-game-default-rtdb/';

		await fetch(baseUrl + statement + '.json')
			.then((res) => res.json())
			.then((res) => {
				quizData = res;
			})
			.catch((e) => (errorMessage = 'Cannot fetch quiz data from api'));
	};

	const setComponent = (cmp) => {
		isQuizActive = true;
		displayedComponent = cmp;
	};

	const prepareResults = async (event) => {
		const { results, errorMsg } = event.detail;

		if (errorMsg != null) {
			errorMessage = errorMsg;
			return;
		}

		await fetchQuizData('results');
		setComponent('result');
		totalScore = results;
	};

	const resetQuiz = async () => {
		await fetchQuizData('questions');
		setComponent('question');
	};

	onMount(async () => {
		await fetchQuizData('questions');
	});
</script>

{#if errorMessage.length > 0}
	<div style="color: red;">{{ errorMessage }}<b>!</b></div>
{:else}
	<div class="container">
		{#if !isQuizActive}
			<div>{quizData?.title ?? ''}</div>
			<ControlButton extraClass="start-ctl">
				<button on:click={() => setComponent('question')} class="tool-btn start">Start</button>
			</ControlButton>
		{:else if displayedComponent === 'question'}
			<Question quizData={quizData.questions} on:printResults={prepareResults} />
		{:else}
			<Result quizData={quizData.results} {totalScore} on:resetQuiz={resetQuiz} />
		{/if}
	</div>
{/if}

<style lang="scss" global>
	@import '../styles/app.scss';
</style>
