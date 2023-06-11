<script>
import { onMount } from "svelte";
import ReconnectingWebSocket from 'reconnecting-websocket';
let socket;
let users = [];
let are_typing = [];
let isConnected = false;
let isTyping = false;
let messages = [];
let output;
let started = false;
let message = "";
    let username = "";
    const autoscroll = () =>{
        let visibleHeight = output.offsetHeight;
        let totalHeight = output.scrollHeight;
        let scrollOffset = output.scrollTop + visibleHeight;
        if (totalHeight <= scrollOffset + 100) {
        output.scrollTop = output.scrollHeight;
    }
}
    onMount(()=>{
        socket = new ReconnectingWebSocket("ws://127.0.0.1:8080/ws");
        started = true;
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
               let messagesplit = data.message.split(":");
                messages = [...messages,{"sender":messagesplit[0], "message":messagesplit[1]}];
                autoscroll();
            case "is_typing":
                are_typing = data.typing_users
            default:
                break;
        }
    }
    })

    const enter_username = () =>{
        console.log("updating: " + username);
        let jsonData = {};
        jsonData.action = "username";
        jsonData.username = username;
        socket.send(JSON.stringify(jsonData));
    }
    const beforeUnload = () =>{
        console.log("Leaving");
        let jsonData = {};
        jsonData.action = "left";
        socket.send(JSON.stringify(jsonData));
    }

    const sendMessage = () => {
        if(message == "") {
            console.log(message);
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
//     let typingTimer;
//     let doneTypingInterval = 2000;
//     const OnTyping = () =>{
//         if (message != "") {
//             clearTimeout(typingTimer);
//             typingTimer = setTimeout(stoppedTyping, doneTypingInterval);
//         if (isTyping) {
//             console.log("do nothing");
//         }else{
//             typing();  
//             isTyping = true;
//         }

// }
//     }  

//     const typing = () =>{
//         console.log("typing")
//         let jsonData = {};
//         jsonData.action = "is_typing";
//         jsonData.username = username;
//         socket.send(JSON.stringify(jsonData));
//     }
//     const stoppedTyping = () =>{
//         let jsonData = {};
//         jsonData.action = "stopped_typing";
//         jsonData.username = username;
//         socket.send(JSON.stringify(jsonData));
//         isTyping = false;
//     }
    const Keydown = (e) =>{
        // OnTyping();
             if (e.keyCode == 13 && e.shiftKey) {
            console.log("Shift key");
            
        } else if((e.keyCode == 13)){
            if (!socket) {
                alert("No Connection");
            }
            e.preventDefault();
            sendMessage();
        }
    }
//     $: if (message == "" && started) {
//         stoppedTyping();
//     }



</script>
<svelte:window on:beforeunload={beforeUnload}/>

<div class="win-items">
    <div class="form-group">
        <form action="">
          <label for="text">Username:</label>
        <input type="text" bind:value={username} on:change={enter_username}>
        </form>
    
    
        
        {#if isConnected}
        <div class="status connected">Connected</div>
        {:else}
        <div class="status not-connected">Disconncted</div>
        {/if}
        <div class="output" bind:this={output}>
            <pre>
                {#each messages as msg}
                <strong class:userstyle={username === msg.sender}>{msg.sender}</strong>:<br>{msg.message}
                {/each}
            </pre>
        
    </div>
<!-- <p>{#each are_typing as typing, n}{typing} {#if n < are_typing.length -2},{:else if n < are_typing.length - 1}and{/if} {/each}are typing...</p> -->
    <form on:submit|preventDefault={sendMessage}>
        <label for="text">Message:</label>
    <textarea id="send-message" on:keydown={Keydown} bind:value={message} ></textarea>
    <button id="send-message">Send Message</button>
    </form>
    </div>
    
    
    <div class="users">
        <h3>Users Online:</h3>
        <ul>
            {#each users as user}
            <li class:userstyle={username === user}>{user} {#if username === user}(You){/if}</li>
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
        height: 20rem;
        overflow-y: auto;
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
        border: 1px solid black;
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
    textarea{
        resize: none;
        height: 2rem;
    }
    .userstyle{
        color:Red;
    }
</style>