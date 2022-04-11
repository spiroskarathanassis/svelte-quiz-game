<script>
	import { createEventDispatcher, onMount } from 'svelte';
	import ControlButton from './ControlButton.svelte';

	const QUESTION_TIMEOUT = 3000;

	// props
	export let quizData = [];

	// definitions
	let buttons;
	let isSettingNextQuestion = false;
	let totalQuestions = 0;
	let previousAnswerIndex = '';
	let activeAnswers = []; //all answers indexing by loop 1-4
	const points = {
		player: 0,
		totalGame: 0
	};

	// reactive
	let question = {
		currentIndex: 0,
		type: '',
		title: '',
		image: '',
		points: 0,
		possibleAnswers: []
	};
	let answer = {
		classNames: [],
		isCorrect: false
	};
	let message = {
		isEnabled: false,
		isAnswerCorrect: false,
		text: ''
	};

	// computed
	$: isNextBtnDisabled = !activeAnswers.includes(true) || isSettingNextQuestion;

	const emit = createEventDispatcher();

	const displayQuestion = () => {
		const questionIndex = quizData[question.currentIndex];

		question = {
			...question,
			type: questionIndex.question_type,
			title: questionIndex.title,
			image: `/assets/${questionIndex.img}`,
			points: questionIndex.points,
			possibleAnswers:
				questionIndex.question_type === 'truefalse'
					? ['True', 'False']
					: questionIndex.possible_answers
		};
	};

	const setNextQuestion = async () => {
		isSettingNextQuestion = true;

		const isCorrect = compareAnswers();
		addPoints(isCorrect);
		printMessage(isCorrect);
		displayCorrectAnswers(isCorrect);

		const isFinished = question.currentIndex >= totalQuestions - 1;
		await stopTime(isFinished);

		isSettingNextQuestion = false;
		// Show points
		console.clear();
		console.table(points);
	};

	const addPoints = (isCorrect) => {
		if (isCorrect) {
			points.player += question.points;
		}
		points.totalGame += question.points;
	};

	const printMessage = (isCorrect) => {
		message = {
			...message,
			isEnabled: true,
			isAnswerCorrect: isCorrect
		};
	};
	const resetMessage = (isCorrect) => {
		message = {
			...message,
			isEnabled: false,
			isAnswerCorrect: false
		};
	};

	const uncheckAnswers = () => {
		resetMessage();
		previousAnswerIndex = '';
		activeAnswers = [];
		answer = { classNames: [], isCorrect: false };
	};

	const stopTime = (isFinished) => {
		return new Promise((resolve) => {
			const timer = setTimeout(() => {
				uncheckAnswers();

				if (!isFinished) {
					++question.currentIndex;
					displayQuestion();
				} else {
					prepareResults();
				}

				resolve();
				clearTimeout(timer);
			}, QUESTION_TIMEOUT);
		});
	};

	const toggleAnswers = (e) => {
		const isSelected = e.target.getAttribute('data-selected') !== 'true';

		// toggle answers on one choice answer
		if (question.type !== 'mutiplechoice-multiple') {
			if (isSelected === false) return;

			if (previousAnswerIndex) {
				activeAnswers[previousAnswerIndex] = !isSelected;
			}

			previousAnswerIndex = e.target.value;
		}

		activeAnswers[e.target.value] = isSelected;
	};

	const compareAnswers = () => {
		if (question.type !== 'mutiplechoice-multiple') {
			const correctAnswer = quizData[question.currentIndex].correct_answer;
			const answerIndex = activeAnswers.findIndex((el) => el === true);

			if (question.type === 'truefalse') return !answerIndex === correctAnswer;

			return question.possibleAnswers[answerIndex].a_id === correctAnswer;
		}

		// case mutiplechoice-multiple below
		const correctAnswers = quizData[question.currentIndex].correct_answer;
		let sameAnswers = 0;
		const numberOfAnswered = activeAnswers.filter((el) => el === true).length;

		correctAnswers.forEach((ans) => {
			activeAnswers.map((el, index) => {
				if (el === true) {
					if (question.possibleAnswers[index].a_id === ans) sameAnswers++;
				}

				return el;
			});
		});

		return sameAnswers === correctAnswers.length && numberOfAnswered === correctAnswers.length;
	};

	const displayCorrectAnswers = (isCorrect) => {
		answer.isCorrect = isCorrect;
		let correctAnswers = [];

		if (question.type === 'mutiplechoice-multiple') {
			correctAnswers = quizData[question.currentIndex].correct_answer;
		} else {
			correctAnswers[0] = quizData[question.currentIndex].correct_answer;
		}

		Array.from(buttons.children).forEach((button, index) => {
			correctAnswers.forEach((ans) => {
				if (button.getAttribute('data-id') === String(ans)) {
					answer.classNames[index] = true;
				}
			});
		});
	};

	const prepareResults = () => {
		let results = null;
		let errorMsg = null;

		try {
			results = Math.floor((points.player * 100) / points.totalGame);
		} catch (e) {
			// console.log(e.message);
			errorMsg = 'Something went wrong with game points';
		}

		emit('printResults', { results, errorMsg });
	};

	onMount(() => {
		totalQuestions = quizData.length;
		displayQuestion();
	});
</script>

<div class="question-number">Question {question.currentIndex + 1} / {totalQuestions}</div>
<div>
	<span>{question.title}</span>
	<span hidden={question.type !== 'mutiplechoice-multiple'} title="mutiplechoice question">
		<strong>*</strong>
	</span>
</div>
<img hidden={!question.image} src={question.image} alt="" />
<p hidden={!message.isEnabled} class="message">
	{message.isAnswerCorrect ? 'Correct Answer' : 'Wrong Answer'}
</p>

<div class="container-answers" bind:this={buttons}>
	{#if question.possibleAnswers}
		{#each question.possibleAnswers as ans, index}
			<button
				value={index}
				data-id={question.type === 'truefalse' ? !index : ans.a_id}
				data-selected={activeAnswers[index] === true}
				on:click={toggleAnswers}
				class="answer-btn"
				class:correct={answer.classNames[index] && answer.isCorrect}
				class:wrong={answer.classNames[index] && !answer.isCorrect}
				class:active={activeAnswers[index]}
			>
				{question.type !== 'truefalse' ? ans.caption : ans}
			</button>
		{/each}
	{/if}
</div>

<ControlButton>
	<button on:click={setNextQuestion} disabled={isNextBtnDisabled} class="tool-btn">
		{question.currentIndex !== totalQuestions - 1 ? 'Next' : 'Finish'}
	</button>
</ControlButton>
