<script>
	import SvelteMarkdown from 'svelte-markdown';
	let question = '';
	let messages = [];
	let currentMessage = ''; // Temporary storage for accumulating message parts

	async function sendMessage() {
		if (question.trim() === '') return;
		messages = [...messages, `You: ${question}`]; // Optionally show the question in the chat
		const responseStream = await fetch(
			`http://ai.bibatalov.ru:8000/chat?question=${encodeURIComponent(question)}`,
			{
				method: 'GET'
			}
		);

		const reader = responseStream.body.getReader();
		readStream(reader);
		question = ''; // Reset question input after sending
	}

	async function readStream(reader) {
		const { done, value } = await reader.read();
		if (done) {
			console.log('Stream completed');
			if (currentMessage.trim() !== '') {
				messages = [...messages, currentMessage]; // Add the complete message to the history
				currentMessage = ''; // Reset current message
			}
			return;
		}
		const decoder = new TextDecoder();
		const messagePart = decoder.decode(value);
		console.log(value);
		console.log(messagePart);
		currentMessage += messagePart; // Accumulate message parts
		readStream(reader); // Recursive call to continue reading
	}
</script>

<div class="input-box">
	<input
		bind:value={question}
		on:keypress={(e) => e.key === 'Enter' && sendMessage()}
		placeholder="Ask a question"
	/>
	<button on:click={sendMessage}>Send</button>
</div>

<div class="chat-container">
	<div class="messages">
		{#each messages as message, index}
			<SvelteMarkdown source={message} />
		{/each}
		{#if currentMessage.trim() !== ''}
			<SvelteMarkdown source={currentMessage}></SvelteMarkdown>
		{/if}
	</div>
</div>

<style>
	code {
		width: 80%;
	}
	.messages {
		display: flex;
		flex-direction: column;
		width: 100%;
	}
	input {
		width: 600px;
		padding: 12px;
	}
	.input-box {
		position: absolute;
		bottom: 0;
		padding: 16px;
	}
	.chat-container {
		width: 80%;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
	}
</style>
