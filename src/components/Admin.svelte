<script lang='typescript'>
    import NewVideo from './NewVideo.svelte'
    import NewUser from './NewUser.svelte'
    import { fade } from 'svelte/transition'

    // Props
    export let videoUrl: string
    export let onMountTime: number
    export let watchTime: number
    export let curTime: number
    export let playState: string
    export let paramNum = Array.from((new URL(document.location.href)).searchParams).length
    export let userAgent: string

    // state
    let currentPanel = 1
    enum Page  {
        Admin = 1,
        Video,
        User,
    }
    
</script>

<style>
    .dev {
		display: flex;
		flex-direction: column;
		align-items: flex-start;
		justify-content: center;
		width: 50vw;
		font-size: 1.2rem;
		background-color: lightblue;
		padding-left:1rem;
	}
    .dev>div {
        width: 50vw;
        display: flex;
        align-items: center;
        justify-content: space-evenly;
    }
	.dev>input {
		width: 95%;
		height: 2em;
		font-size: 1.2rem;
	}
    button {
        margin: 1em 1em;
        font-size: 1.2rem;
    }
</style>



{#if currentPanel == Page.Video}
    <NewVideo/>
    <button on:click="{() => currentPanel = Page.Admin}">Stats</button>   
{:else if currentPanel == Page.User}
    <NewUser/>
    <button on:click="{() => currentPanel = Page.Admin}">Stats</button>   
{:else}
    <div class="dev" in:fade={{duration: 500}}>
        <p>Put a youtube video link to change the video here:</p>
        <input type="text" bind:value={videoUrl}>
        <p>Your session time: {onMountTime}</p>
        <p>Current Watched time: {watchTime}</p>
        <p>Videos Current time: {curTime}</p>
        <p>Player State is: {playState}</p>
        <p>Full URL: {window.location.href}</p>
        <p>Host Name: {window.location.hostname}</p>
        <p>Path Name: {window.location.pathname}</p>
        <p>Parameters: {paramNum}</p>
        <p>{userAgent}</p> 
        <div>
            <!-- <button on:click="{() => currentPanel = Page.Video}">Add Video</button> -->
            <button on:click="{() => currentPanel = Page.User}">Add Users & Videos</button>
        </div>
    </div>

{/if}
