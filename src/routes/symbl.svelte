<script context="module">
	export async function load({ fetch }) {
		const appId = '6b41733850736b4e4552525a587a4b4d32454658336b5433526d523044493462';
		const appSecret =
			'46494639614569694f45635f6865784b596e5f757363336e5179736f556c6d4f4554664a44714d414531314b6b2d7749684e33414633614a366a536a714d7249';

		const res = await fetch('https://api.symbl.ai/oauth2/token:generate', {
			method: 'POST',

			headers: {
				'Content-Type': 'application/json'
			},

			body: JSON.stringify({
				type: 'application',
				appId,
				appSecret
			})
		});

		if (res.ok) {
			const resjson = await res.json();
			return {
				props: {
					accessToken: resjson.accessToken
				}
			};
		}
		return {
			props: {
				accessToken: null
			}
		};
	}
</script>

<script>
	export let accessToken;
	let result;
	let ws = null;
	const btnOnclick = async () => {
		if (accessToken == null) {
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
			console.warn(JSON.stringify(data),null,2);
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

			if (data.type === 'tracker_response') {
				console.table(data.trackers);
				/*for (let tracker of data.trackers) {
					
					console.log('tracker detected: ', insight.payload.content);
				}*/
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
					insightTypes: ['question', 'action_item','ideas'], // Will enable insight generation
					trackers: [
						{
							name: 'Promotion Mention',
							vocabulary: [
								'We have a special promotion going on if you book this before',
								'I can offer you a discount of 10 20 percent you being a new customer for us',
								'monthly special',
								'We have a sale right now'
							]
						},
						{
							name: 'Meeting Mention',
							vocabulary: [
								'When we meet',
								'have booked a room',
								'designers will also join us',
								'agenda of the meeting will be'
							]
						}
					],
					config: {
						confidenceThreshold: 0.5,
						languageCode: 'pt-por',
						speechRecognition: {
							encoding: 'LINEAR16',
							sampleRateHertz: 44100
						}
					},
					speaker: {
						userId: 'afan201353@gmail.com',
						name: 'Saym'
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
<button on:click={closeWebSocket}>Stop Symbl</button>
<p>{result}</p>
