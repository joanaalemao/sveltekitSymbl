


<script>

	let result;
	let ws = null;
	const btnOnclick = async () => {
		const accessToken =
			'eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6IlFVUTRNemhDUVVWQk1rTkJNemszUTBNMlFVVTRRekkyUmpWQ056VTJRelUxUTBVeE5EZzFNUSJ9.eyJodHRwczovL3BsYXRmb3JtLnN5bWJsLmFpL3VzZXJJZCI6IjY2MjkyNDYzMjQ3MDMyMzIiLCJpc3MiOiJodHRwczovL2RpcmVjdC1wbGF0Zm9ybS5hdXRoMC5jb20vIiwic3ViIjoiUWE2Q05zQ3l0dWlKWFJsdzZVNTdHWnZNc2s3VVVmRTJAY2xpZW50cyIsImF1ZCI6Imh0dHBzOi8vcGxhdGZvcm0ucmFtbWVyLmFpIiwiaWF0IjoxNjQxNDcxMDc1LCJleHAiOjE2NDE1NTc0NzUsImF6cCI6IlFhNkNOc0N5dHVpSlhSbHc2VTU3R1p2TXNrN1VVZkUyIiwiZ3R5IjoiY2xpZW50LWNyZWRlbnRpYWxzIn0.xd4o9NvXzCdb3GWWMwtCTJBFm7C2XNBxd0YxFlFm3aoyAV1-lxNqVaJ1g3IcPzRP3r_A_BsYoixaVwJMFG5lsbOsF42M5INI0AW3sa22WEjsgXSqgP9uhbXopWTqRGgo_zZnBukvvBSUG4jWkEyfK6X5XTw4SB-_ZpLvQCbRJwQzgISFQwc0JhpRUqE9rheyyjJja-BxOPPpNrQCs5aPxEJXyQ3oxV7-qkbY3G2iS7-ICmGkc8NpMcyS-HXtxnIvqIP1qjanQCtC_2gyx33soeAokH17gocNO-2o9leJg2fISjERtM8WD-mowE9DIw2Ca_DRHpKUjLRl84TCxq3eHA';
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

<div class="logo">
    
<div class="text"> 
<h1>Welcome to Symbl Testing</h1>
<button on:click={btnOnclick}>Start Symbl Recording</button>
<p>{result}</p>

<button on:click={closeWebSocket}>Stop Symbl</button>
</div>
</div>

<style>
	h1 {
		display: flex;
		list-style: none;
		margin-left: 20px;
		padding: 0;
		border-left: 20px;
        border-top: 20px;
        font-family:Arial, Helvetica, sans-serif; 
        font-weight: lighter;
        color: goldenrod;
	}

	button {
		margin-right: 10px;
		margin-left: 20px;
		border-left: 20px;
        color: goldenrod;
        background-color: black;

	}

    div {

        border-left: 20px;

    }


</style>