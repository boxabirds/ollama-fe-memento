<script>
   import { onMount } from 'svelte';
 
   let content = '';
   let messages = [];
   let accumulatingMessage = '';
 
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
        if (done) {
         //  if (accumulatingMessage) {
         //    messages = [...messages, { message: { role: 'system', content: accumulatingMessage } }];
         //  }
          break;
        }

        const chunk = new TextDecoder("utf-8").decode(value);
        const message = JSON.parse(chunk);
        const content = message.message.content;

        if (content === '\n') {
          messages = [...messages, { message: { role: 'system', content: accumulatingMessage } }];
          accumulatingMessage = '';
        } else {
          accumulatingMessage += content;
        }
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
   {#each messages as message, index (index)}
     <div class="message {message.message.role}">
       <p>{message.message.content}</p>
     </div>
   {/each}
   {#if accumulatingMessage}
   <div class="message system">
     <p>{accumulatingMessage}</p> 
   </div>
   {/if}

 </div>
 
 <input bind:value={content} placeholder="Enter your text here..." on:keydown={handleKeyDown} />
 <button on:click={fetchMessages}>Submit</button>
 