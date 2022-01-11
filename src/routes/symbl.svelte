<script context="module">
	export async function load({fetch}) {
		const appId = '516136434e7343797475694a58526c7736553537475a764d736b375555664532';
		const appSecret =
			'3971794d6c5f575f6d35693957304551594a4145794a5036764478685655346b314d66686e324c313438636e48533659725142734239486e58476c414e357675';

		const res = await fetch('https://api.symbl.ai/oauth2/token:generate', {
			method: 'POST',

			headers: {
				'Content-Type': 'application/json'
			},

			body: JSON.stringify({
				type:'application',
				appId, appSecret
			})		
		});

		if (res.ok) {
			const resjson = await res.json();
			return {
				props : {
					accessToken : resjson.accessToken
				}
			}
		}
		return {
			props : {
				accessToken : null
			}
		}
	}
</script>

<script>
	export let accessToken;
	let result;
	let ws = null;
	const btnOnclick = async () => {
		if(accessToken == null) {
			console.error('accessToken not available');
			return;
		}
		console.log(accessToken);
		// Refer to the Authentication section for how to generate the accessToken: https://docs.symbl.ai/docs/developer-tools/authentication
		const uniqueMeetingId = btoa('joana@moonbeam.ai');
		const symblEndpoint = `wss://api.symbl.ai/v1/realtime/insights/${uniqueMeetingId}?access_token=${accessToken}`;
		ws = new WebSocket(symblEndpoint);
		// Fired when a message is received from the WebSocket server
		ws.onmessage = (event) => {
			// You can find the conversationId in event.message.data.conversationId;
			const data = JSON.parse(event.data);
			if (data.type === 'message' && data.message.hasOwnProperty('data')) {
				console.log('conversationId', data.message.data.conversationId);
			}
			if (data.type === 'message_response') {
				for (let message of data.messages) {
					console.log('Transcript (more accurate): ', message.payload.content);
					result = message.payload.content;
				}
			}
			if (data.type === 'topic_response') {
				for (let topic of data.topics) {
					console.log('Topic detected: ', topic.phrases);
				}
			}
			if (data.type === 'insight_response') {
				for (let insight of data.insights) {
					console.log('Insight detected: ', insight.payload.content);
				}
			}
			if (data.type === 'message' && data.message.hasOwnProperty('punctuated')) {
				console.log('Live transcript (less accurate): ', data.message.punctuated.transcript);
				result = data.message.punctuated.transcript;
			}
			console.log(`Response type: ${data.type}. Object: `, data);
		};
		// Fired when the WebSocket closes unexpectedly due to an error or lost connetion
		ws.onerror = (err) => {
			console.error(err);
		};
		// Fired when the WebSocket connection has been closed
		ws.onclose = (event) => {
			console.info('Connection to websocket closed');
		};
		// Fired when the connection succeeds.
		ws.onopen = (event) => {
			ws.send(
				JSON.stringify({
					type: 'start_request',
					meetingTitle: 'Websockets How-to', // Conversation name
					insightTypes: ['question', 'action_item'], // Will enable insight generation
					config: {
						confidenceThreshold: 0.5,
						languageCode: 'pt-por',
						speechRecognition: {
							encoding: 'LINEAR16',
							sampleRateHertz: 44100
						}
					},
					speaker: {
						userId: 'example@symbl.ai',
						name: 'Example Sample'
					}
				})
			);
		};
		const stream = await navigator.mediaDevices.getUserMedia({ audio: true, video: false });
		/**
		 * The callback function which fires after a user gives the browser permission to use
		 * the computer's microphone. Starts a recording session which sends the audio stream to
		 * the WebSocket endpoint for processing.
		 */
		const handleSuccess = (stream) => {
			const AudioContext = window.AudioContext;
			const context = new AudioContext();
			const source = context.createMediaStreamSource(stream);
			const processor = context.createScriptProcessor(1024, 1, 1);
			const gainNode = context.createGain();
			source.connect(gainNode);
			gainNode.connect(processor);
			processor.connect(context.destination);
			processor.onaudioprocess = (e) => {
				// convert to 16-bit payload
				const inputData = e.inputBuffer.getChannelData(0) || new Float32Array(this.bufferSize);
				const targetBuffer = new Int16Array(inputData.length);
				for (let index = inputData.length; index > 0; index--) {
					targetBuffer[index] = 32767 * Math.min(1, inputData[index]);
				}
				// Send audio stream to websocket.
				if (ws.readyState === WebSocket.OPEN) {
					ws.send(targetBuffer.buffer);
				}
			};
		};
		handleSuccess(stream);
	};
	function closeWebSocket() {
		if (ws != null) {
			ws.close();
		}
	}
</script>


<h1>Welcome to Symbl Testing</h1>
<!-- <button on:click={connect}>Start Symbl</button> -->
<button on:click={btnOnclick}>Start Symbl Recording</button>
<p>{result}</p>


