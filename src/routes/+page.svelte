<script>
    const socket = new WebSocket("ws://127.0.0.1:8080/ws")
    socket.onopen = () =>{
        console.log("CONNECTED!!!")
    }
</script>

<h1>Welcome to SvelteKit</h1>
<p>Visit <a href="https://kit.svelte.dev">kit.svelte.dev</a> to read the documentation</p>
