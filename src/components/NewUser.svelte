<script lang="typescript">
    import { fade } from 'svelte/transition'

    let userId: string
    let userGroup: string

    // Checking the form inputs
    const checkInputs = () => {
        const userIdCheck = userId.match(/[0-9]/g)!.toString()  ? userId : null
        const userGroupCheck = userGroup.length == 2

        return userIdCheck && userGroupCheck 
    }

    // Sending the new video to db
    const submitUser = async() => {
        if(!checkInputs()) return false

        try {
            const resVid = await fetch(`${(import.meta.env.SNOWPACK_PUBLIC_USER_URL)}userId/${userGroup[0]}/${userId}.json`,{
                method: 'PUT',
                body: JSON.stringify(
                    {   
                        videoId: videoId,
                        visited: 0,
                        group: userGroup,
                        name: videoName
                }),
                headers: {
                    'Content-Type': 'application/json'
                }
            }).then(res => {
                if(!res.ok){
                    throw new Error('Failed!')
                }
                return res.json()
            }).then( () =>{
                console.log("succesful!")
            })
        } catch (error) {
            console.log(error)
        }
        return true
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
		background-color: rgb(230, 224, 173);
		padding-left:1rem;
	}

	.dev>input {
		width: 95%;
		height: 1.5em;
		font-size: 1.2rem;
        margin-bottom: 1em;
	}
    
    .dev>button {
		width: auto;
		height: 1.5em;
		font-size: 1.2rem;
        margin-bottom: 1em;
	}

</style>

     <div class="dev" in:fade={{duration: 500}} >
        <p>All are mandatory. User ID:</p>
        <input type="text" placeholder="example: 1234" bind:value={userId} maxlength="8">
        <p>User Group:</p>
        <input type="text" placeholder="example: a1" bind:value={userGroup} maxlength="2">
        <button on:click={submitUser}>Submit</button>
     </div>