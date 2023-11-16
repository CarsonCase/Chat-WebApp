<script lang="ts">
    import {onDestroy, onMount} from "svelte";
    import {currentUser, pb} from "./pocketbase";
    import type RecordModel from "pocketbase";
    let messages: any = [];
    let unsubscribe: ()=> void;

    onMount(async() =>{
        const resultList = await pb.collection("messages").getList(1,50,{
            sort: 'created',
            expand: 'author',
        });
        messages = resultList.items;


        unsubscribe = await pb.collection("messages").subscribe("*", async({action, record})=>{
            if (action === "create") {
                const user = pb.collection("users").getOne(record.user);
                record.expand = {user};

                messages = [...messages, record];
            }
            if (action === "delete") {
                messages = messages.filter((m)=>m.id !== record.id);
            }
        })
    })

    onDestroy(()=>{
        unsubscribe();
    });

    let newMessage: string;
    async function sendMessage(){
        const data={
            message: newMessage,
            author: $currentUser.id
        }
        const createdMessage = await pb.collection("messages").create(data);
    }
</script>
<h3>Messages</h3>
<div class="messages">
    {#each messages as message (message.id)}
    <div class="msg">
        <img 
        src={`https://api.dicebear.com/7.x/pixel-art/svg?seed=${message.expand?.author}`} 
        alt="avatar"
        width="40px" 
        class="avatar"
        />
        <div>
            <small>
                Sent by @{message.expand?.author?.username}
            </small>
            <p class="msg-text">{message.message}</p>
        </div>
    </div>
    {/each}

    <form on:submit|preventDefault ={sendMessage}>
        <input type="text" bind:value={newMessage}>
        <button>Send Message</button>
    </form>
</div>