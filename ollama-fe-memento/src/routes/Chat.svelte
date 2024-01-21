<script>
    import { onMount } from 'svelte';
    
    let content = '';
    let messages = [];
    
    const fetchMessages = async () => {
     const response = await fetch('http://localhost:11434/api/chat', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          model: 'llama2',
          messages: [{ role: 'user', content: content }]
        })
     });
    
     if (response.ok) {
        const reader = response.body.getReader();
    
        while (true) {
          const { done, value } = await reader.read();
    
          if (done) break;
    
          const chunk = new TextDecoder("utf-8").decode(value);
          const message = JSON.parse(chunk);
    
          message.message.content = message.message.content.join(' ');
          messages = [...messages, message];
        }
     } else {
        console.error('Error:', response.status, response.statusText);
     }
    };
    
    onMount(fetchMessages);
    
    function scrollToBottom() {
     if (typeof window !== 'undefined') {
        const messagesDiv = document.getElementById('messages');
        messagesDiv.scrollTop = messagesDiv.scrollHeight;
     }
    }
    
    // Call scrollToBottom every time messages updates
    $: scrollToBottom();
    
    function handleKeyDown(event) {
     if (event.key === 'Enter') {
        fetchMessages();
     }
    }
    </script>
    
    <style>
    #messages {
     height: 80vh;
     width: 100%;
     border: 1px solid #ddd;
     padding: 1em;
     overflow-y: auto;
    }
    
    .message {
     margin-bottom: 1em;
    }
    
    .message.user {
     align-self: flex-end;
    }
    
    .message.assistant {
     align-self: flex-start;
    }
    </style>
    
    <div id="messages">
     {#each messages as message (message.created_at)}
        <div class="message {message.message.role}">
          <p>{message.message.content}</p>
        </div>
     {/each}
    </div>
    
    <input bind:value={content} placeholder="Enter your text here..." on:keydown={handleKeyDown} />
    <button on:click={fetchMessages}>Submit</button>
    