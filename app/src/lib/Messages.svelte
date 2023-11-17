<script lang="ts">
    import {onDestroy, onMount} from "svelte";
    import {currentUser, pb} from "./pocketbase";
    import type RecordModel from "pocketbase";
    import {Avatar} from "@skeletonlabs/skeleton";
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


<section class="card">
<div class="messages w-full h-full grid grid-cols-1">
    {#each messages as message (message.id)}
    <div class="grid grid-cols-[auto_1fr] gap-2">
        <Avatar src="https://api.dicebear.com/7.x/pixel-art/svg?seed=${message.expand?.author?.username}" width="w-12" />
        <div class="card p-4 variant-soft rounded-tl-none space-y-2">
            <header class="flex justify-between items-center">
                <p class="font-bold">@{message.expand?.author?.username}</p>
            </header>
            <p>{message.message}</p>
        </div>
    </div>
    {/each}

    <div class="input-group input-group-divider grid-cols-[auto_1fr_auto] rounded-container-token">
        <form on:submit|preventDefault ={sendMessage}>
        <input bind:value={newMessage} class="input" title="Input (text)" type="text" placeholder="input text" />
        <button class="variant-filled-primary">Send</button>
    </form>

    </div>

</div>
</section>		