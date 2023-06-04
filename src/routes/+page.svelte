<script>
    import { onMount } from "svelte";
    let socket;
    let users = [];
    

    onMount(()=>{
        socket = new WebSocket("ws://127.0.0.1:8080/ws")
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
        // console.log(msg);
        // let j = JSON.parse(msg.data);
        // console.log(j);

        let data = JSON.parse(msg.data);
        console.log(data.action);
        switch (data.action) {
            case "list_users":
                users = [];
                if (data.connected_users.length > 0) {
                    users = data.connected_users;
                }
                break;
            default:
                break;
        }
    }
    })

    let username = "";
    const enter_username = () =>{
        console.log("updating: " + username);
        let jsonData = {};
        jsonData.action = "username";
        jsonData.username = username;
        socket.send(JSON.stringify(jsonData))
    }
    const beforeUnload = () =>{
        console.log("Leaving");
        let jsonData = {};
        jsonData.action = "left";
        socket.send(JSON.stringify(jsonData));
    }
</script>
<svelte:window on:beforeunload={beforeUnload}/>

<div class="form-group">
    <label for="text">Username:</label>
    <input type="text" bind:value={username} on:change={enter_username}>
    <label for="text">Message:</label>
    <input type="text">
</div>
<ul>
    {#each users as user}
    <li>{user}</li>
    {/each}
</ul>
<div class="output"></div>

<h1>Welcome to SvelteKit</h1>
<p>Visit <a href="https://kit.svelte.dev">kit.svelte.dev</a> to read the documentation</p>
