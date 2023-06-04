<script>
    import { onMount } from "svelte";
    import ReconnectingWebSocket from 'reconnecting-websocket';
    let socket;
    let users = [];
    let isConnected = false;
    let messages = "";
    onMount(()=>{
        socket = new ReconnectingWebSocket("ws://127.0.0.1:8080/ws")
    socket.onopen = () =>{
        console.log("CONNECTED!!!")
        isConnected = true;
    }
    socket.onclose = () =>{
        console.log("CONNECTION CLOSED!!")
        isConnected = false;
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
            case "broadcast":
                messages += data.message + "<br>";
            default:
                break;
        }
    }
    })
    let message = "";
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

    const sendMessage = () => {
        if(message == "") {
            alert("Enter a Message");
            return;
        }
        if (username === ""){
            alert("Enter a Username");
            return;
        }
        let jsonData = {};
        jsonData.action = "broadcast";
        jsonData.username = username;
        jsonData.message = message;
        socket.send(JSON.stringify(jsonData));
        message = "";
    }
    const Keydown = (e) =>{
             if (e.keyCode == 13 && e.shiftKey) {
            console.log("Shift key");
            
        } else if((e.keyCode == 13)){
            if (!socket) {
                console.log("No Connection")
            }
            e.preventDefault();
            message.replaceAll("\n", "<br>")
            sendMessage();
        }
    }

</script>
<svelte:window on:beforeunload={beforeUnload}/>

<div class="win-items">
    <div class="form-group">
        <form action="">
          <label for="text">Username:</label>
        <input type="text" bind:value={username} on:change={enter_username}>
        </form>
    
    
        <form on:submit|preventDefault={sendMessage}>
            <label for="text">Message:</label>
        <textarea id="send-message" bind:value={message} on:keydown={Keydown}></textarea>
        <button id="send-message">Send Message</button>
        </form>
        {#if isConnected}
        <div class="status connected">Connected</div>
        {:else}
        <div class="status not-connected">Disconncted</div>
        {/if}
        <div class="output">
            <pre>
                {@html messages}
            </pre>
        
    </div>
    </div>
    
    
    <div class="users">
        <h3>Users Online:</h3>
        <ul>
            {#each users as user}
            <li>{user}</li>
            {/each}
        </ul>
    </div>
</div>
<style>
    form {
        display: flex;
        flex-direction: column;
    }

    .output {
        border: 1px solid black;
        padding: 1rem;
    }
    .win-items{
        display: flex;
    }
    .form-group{
        display: flex;
        flex-direction: column;
        margin: 2rem;
        width: 100%;
        flex: 3;
    }
    .users{
        width: 100%;
        flex:1;
    }
    .status{
        text-align: center;
    }
    .connected{
        background-color: aquamarine;
    }
    .not-connected{
        background-color: orangered;
    }
    pre{
        white-space: pre-line;
    }
</style>