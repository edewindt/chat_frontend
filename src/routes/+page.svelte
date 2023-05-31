<script>
    const socket = new WebSocket("ws://127.0.0.1:8080/ws")
    socket.onopen = () =>{
        console.log("CONNECTED!!!")
    }
    socket.onclose = () =>{
        console.log("CONNECTION CLOSED!!")
    }
    socket.onerror = () =>{
        console.log("YOU SCREWED UP!!")
    }
    socket.onmessage = msg =>{
        console.log(msg);
    }
</script>


<div class="form-group">
    <label for="text">Username:</label>
    <input type="text">
    <label for="text">Message:</label>
    <input type="text">
</div>
<div class="output"></div>

<h1>Welcome to SvelteKit</h1>
<p>Visit <a href="https://kit.svelte.dev">kit.svelte.dev</a> to read the documentation</p>
