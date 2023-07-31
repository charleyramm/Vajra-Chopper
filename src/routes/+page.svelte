<script>
	// USAGE
	// # Enter a word list and press ENTER
	// # Hold space to record a word
	// # Press D to download the recorded word and move to the next word

	// ISSUES


	// TODOS
	// # Enable waveform rendering

	import { onMount } from 'svelte'; // Callback a function when the component is ready

	let wordsMatrix = [];
	let keyDown = false;
	let recording;
    let mediaRecorder;

	function csv(text){
		let rows = text.split('\n');
		for (let i = 0; i<rows.length; i++) {
			rows[i] = rows[i].split('	');
		}
		return rows;
	}


    function record(){


		const handleSuccess = function (stream) {
			const options = { mimeType: 'audio/webm' };
			const recordedChunks = [];
			mediaRecorder = new MediaRecorder(stream, options);

			mediaRecorder.addEventListener('dataavailable', function (e) {
				if (e.data.size > 0) recordedChunks.push(e.data);
			});

			mediaRecorder.addEventListener('stop', function () {
				recording = URL.createObjectURL(new Blob(recordedChunks));

   				const player = new Audio(recording);
				player.addEventListener(
					'canplaythrough',
					function () {
						console.log('canplaythrough');
						player.play();
					},
					{ once: true }
				);
			});


			mediaRecorder.start();
		};

		navigator.mediaDevices.getUserMedia({ audio: true, video: false }).then(handleSuccess);
    }

	function importWords() {
		console.log('importWords');
		const el = document.querySelector('textarea');
		const text = el.value;
		console.log(text);

		wordsMatrix = csv(text);

		return;
	}

	function onSpaceDown(e) {
		console.log('onSpaceDown');
		// Start recording. Browser will request permission to use your microphone.
        setTimeout(record, 100); // wait 100ms so we miss the keystroke sound.
	}

	function onSpaceUp(e) {
		console.log('onSpaceUp');
		console.log(wordsMatrix);
		// Stop recording
        mediaRecorder.stop();
	}

	function saveAs(uri, filename) {
		console.log('saveAs');
		var link = document.createElement('a');
		if (typeof link.download === 'string') {
			document.body.appendChild(link);
			link.download = filename;
			link.href = uri;
			link.click();
			document.body.removeChild(link);
		} else {
			location.replace(uri);
		}
	}

	function fileName(){
		let filenameRow = 0; // skip the headers in the first row
		let filenameColumn = wordsMatrix[filenameRow].length-1; // last column has the filename
		return wordsMatrix[filenameRow][filenameColumn]
	}

	function onKeyD() {
		console.log('onKeyD');
		if (!recording || !wordsMatrix?.[0]) return;

		saveAs(recording, fileName() + '.webm');
		wordsMatrix.shift();
		wordsMatrix = wordsMatrix; // svelte hooks assignment, it dont check inside objects

		if (wordsMatrix.length <= 1) { // the last word, delete wordsMatrix and start again.
			wordsMatrix = null;
		}
	}

	function onEnter() {
		console.log('onEnter');
		importWords();
	}

	function onKeyDown(e) {
		if (e.code === 'Space') {
			e.preventDefault();
		}
		if (keyDown) {
			return;

		console.log('onKeyDown');
		} else {
			keyDown = true;
		}
		if (e.code === 'Space') {
			onSpaceDown(e);
		}
		if (e.code === 'Enter') {
			onEnter();
		}
		if (e.code === 'KeyD') {
			onKeyD();
		}
	}

	function onKeyUp(e) {
		console.log('onKeyUp');
		if (!keyDown) {
			return;
		} else {
			keyDown = false;
		}
		if (e.code === 'Space') {
			onSpaceUp(e);
		}
	}

	onMount(function () {
		console.log('onMount');
		document.addEventListener('keydown', onKeyDown);
		document.addEventListener('keyup', onKeyUp);
	});
</script>

<h1>Vajra Chopper</h1>
<div>
	<ul>
		<li>ENTER: Tee up words tsv from the text box.<br>
		- Any number of columns are allowed<br>
		- The final column will be assigned as the filename</li>
		<li>SPACE: Hold to record</li>
		<li>D KEY: Download recorded word</li>
	</ul>
</div>

{#if !wordsMatrix || !wordsMatrix.length}
	<textarea>its	ɪts	ɪts	o		A1	determiner		672	its	672its
my	maɪ	maɪ	o		A1	determiner		673	my	673my
of	ɒv	ɒv	o	BELONG	A1	preposition		674	of	674of
on	ɒn	ɒn	o	DAY/DATE	A1	preposition		675	on	675on
it	ɪt	ɪt	o	THING	A1	pronoun		676	it	676it
it	ɪt	ɪt	o	SUBJECT/OBJECT	A1	pronoun		677	it	677it
me	miː	miː	o		A1	pronoun		678	me	678me
please	pliːz	pliːz	o	POLITE REQUEST	A1	exclamation	communication	679	please	679please
other	ˈʌðə	ˈʌð.ə	Oo	ADDITIONAL	A1	determiner	describing things	680	other	680other
our	ˈaʊə	aʊə	Oo		A1	determiner		681	our	681our
past	pɑːst	pɑːst	o	TIME	A1	preposition		682	past	682past
thanks	θæŋks	θæŋks	o	THANK YOU	A1	exclamation	communication	683	thanks	683thanks
thank you	ˈθæŋk ˌjuː	ˈθæŋk ˌjuː	Oo	GRATEFUL	A1	exclamation	communication	684	thank_you	684thank_you
some	sʌm	sʌm	o	UNKNOWN AMOUNT	A1	determiner	describing things	685	some	685some
some	sʌm	sʌm	o	UNKNOWN AMOUNT	A1	pronoun	describing things	686	some	686some
some	sʌm	sʌm	o	NOT ALL	A1	pronoun	describing things	687	some	687some
something	ˈsʌmθɪŋ	ˈsʌm.θɪŋ	Oo	THING	A1	pronoun	describing things	688	something	688something
that	ðæt	ðæt	o	CAN BE SEEN	A1	determiner		689	that	689that
that	ðæt	ðæt	o	ALREADY MENTIONED	A1	determiner		690	that	690that
the	ðə	ðə	o	PARTICULAR	A1	determiner		691	the	691the
the	ðə	ðə	o	ONLY ONE	A1	determiner		692	the	692the
the	ðə	ðə	o	PLACE	A1	determiner		693	the	693the
their	ðeə	ðeə	o	BELONGING TO GROUP	A1	determiner		694	their	694their
these	ðiːz	ðiːz	o		A1	determiner		695	these	695these</textarea>
{:else}
	<ul>
		<h1>Recording: {fileName()}.webm</h1>
		<table>
		{#each wordsMatrix as row}
			<tr>
				{#each row as col}
					<td>{col}</td>
				{/each}
			</tr>
		{/each}
		</table>
	</ul>
{/if}

<style>
	table tr:nth-child(1){
		font-weight: bold;
	}
</style>