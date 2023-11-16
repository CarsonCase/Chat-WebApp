<script lang="ts">
    import Login from "../lib/login.svelte";
    import Messages from "../lib/Messages.svelte";
    import {currentUser, pb} from "../lib/pocketbase";

    function signOut(){
        pb.authStore.clear();
    }
</script>

<h1>ChatApp</h1>

{#if $currentUser}
<Messages/>
<button on:click={signOut}>Sign Out</button>
{:else}
<Login/>
{/if}