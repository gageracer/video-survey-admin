<script lang="typescript">
    import { fade } from 'svelte/transition'
    import { salt, ntos, speDec } from '../stores/store'
    import Papa from 'papaparse'


    // From NewVideo
    let videoUrl: string
    let videoName: string
    let videoGroup: string
    let videoDate: string
    let videoId: string | null
    let downloadLink = false
    let downloadLinks : {}[]
    let generateLink = false
    let data : string

    //let file: File
    let groupData : {} = {}
    let videosData : {} = {}

    const videosText = "day"

    function onFileChange(event:Event) {
        
        event && Papa.parse(event.target.files[0],{
                complete: async function(results: object){
                    getGroupsVideos(await results.data)
            }
        })
        // const file = files[0]
        // let file = event.target!.files[0]

        // // Check if the file is an image.
        // if (file.type && file.type.indexOf('text/csv') === -1) {
        //     console.log('File is not an csv.', file.type, file);
        //     return;
        // }
        // let reader = new FileReader()
        // reader.readAsText(file)

        // reader.onload = (e) => {
        //     let fileAsText = e.target.result
        //     console.log(fileAsText)
        // }
    }

    const getGroupsVideos = (data:[]) => {
        let videosIndex = 0
        const regex = /(?<=\?v=).{11}/
        let tmp :string[] = []
        let tmpOjb: {}
        let date = ""
        groupData = {}
        videosData = {}
        //console.log(data)
        data.forEach( (element: [],index) => {
            if(index != 0){
                if(videosIndex == 0){
                    if(element[0] !== videosText){
                        groupData[element[2]] ?  groupData[element[2]] = {...groupData[element[2]],
                                [element[0]]:{
                                    groups: element.splice(3).toString(),
                                    visited: 0
                                }
                            }
                        
                        : groupData[element[2]] = {
                                [element[0]]:{
                                    groups: element.splice(3).toString(),
                                    visited: 0
                                }
                            }
                    }
                    else{
                        videosIndex = index
                    }
                }
                else{
                    tmp = element.splice(0,3)
                    let video = tmp[2] ? tmp[2].match(regex)!.toString() : null
                    if(date == ""){
                        tmpOjb = {
                        [tmp[0]]: {
                            [tmp[1]]: {
                                videoId: video,
                                visited: 0
                                }
                            }
                        }
                        date = tmp[0]
                        console.log("here1")
                    }
                    else if(date == tmp[0]){
                        tmpOjb[date][tmp[1]] = {
                                videoId: video,
                                visited: 0
                        }
                        console.log("here2")
                    }
                    else{
                        index != data.length ? videosData[date] =  tmpOjb[date] : null
                        date = tmp[0]
                        tmpOjb = {
                        [tmp[0]]: {
                            [tmp[1]]: {
                                videoId: video,
                                visited: 0
                                }
                            }
                        }
                        console.log("here3")
                    }
                    
                }
            }
        });
        console.log("group",groupData)
        console.log("videos",videosData)
        groupData && videosData ? generateLink = true : generateLink = false
    }

    // Checking the form inputs
    const checkUsers = (userId:string, userGroup: string[]) => {
        const userIdCheck = userId.match(/[0-9]/g)!.toString()  ? userId : null
        const userGroupCheck = userGroup.length >= 0 && userGroup.forEach( e => e.match(/[a-z]/))

        return userIdCheck && userGroupCheck 
    }

    // Sending the new video to db
    const submitUser =  async() => {
            try {
                const resVid = await fetch(`${speDec(import.meta.env.SNOWPACK_PUBLIC_USER_URL)}userId.json`,{
                    method: 'PATCH',
                    body: JSON.stringify(groupData),
                    headers: {
                        'Content-Type': 'application/json'
                    }
                }).then(res => {
                    if(!res.ok){
                        throw new Error('Failed!')
                    }
                    return res.json()
                }).then( () =>{
                    console.log("users succesful!")
                })
            } catch (error) {
                console.log(error)
            }

        return true
    }

    // Checking the form inputs
    const checkVideos = (videoUrl:string,videoDate:string ,videoGroup:string) => {
        const regex = /(?<=\?v=).{11}/
        videoId = videoUrl ? videoUrl.match(regex)!.toString() : null
        const videoDateCheck = videoDate ? videoDate : null
        const videoGroupCheck = videoGroup.length == 1
        return videoId && videoDateCheck && videoGroupCheck
    }

    // Sending the new video to db
    const submitVideo = async() => {
            try {
                const resVid = await fetch(`${speDec(import.meta.env.SNOWPACK_PUBLIC_VIDEO_URL)}videoLink.json`,{
                    method: 'PATCH',
                    body: JSON.stringify(videosData),
                    headers: {
                        'Content-Type': 'application/json'
                    }
                }).then(res => {
                    if(!res.ok){
                        throw new Error('Failed!')
                    }
                    return res.json()
                }).then( () =>{
                    console.log("videos succesful!")
                })
            } catch (error) {
                console.log(error)
            }

        return true
    }

    const generateLinks = () => {
        if(!(submitVideo() && submitUser())){ return false}
        const link = (d: string,i:number,v:string,r:string) => {return { [i] : `https://whatsappstudy2021.vercel.app/?d=${d}&i=${i}&v=${v}&r=${r}`} }
        downloadLinks = []
        for (const date in videosData) {
            const mydate = date
            for (const grp in videosData[date]) {
                let d = mydate.replaceAll('-','')
                let v = grp
                const date = parseInt(d?.slice(-4)!,10)
                const vidId = parseInt(v.charCodeAt(0).toString(4),10)
                
                for (const usr in groupData){
                    if(groupData[usr][mydate].groups.includes(grp)){
                        console.log(usr)
                        const id = parseInt(usr!,10)
                        const mid = date ^ id ^ vidId
                        const result = ntos($salt.map( x => (x ^ mid) % 512))
                        downloadLinks = downloadLinks ? [...downloadLinks, link(d,id,v,result)] : [link(d,id,v,result)]
                    }
                }
            }
            
        }
        
        console.log(downloadLinks.length)
        data = "text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(downloadLinks))
        downloadLink = true
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
        {#if downloadLink}
             <p>Here is the links for the users:</p>
             <a href="{`data:${data}`}" download="{`links_${videoDate}_${videoGroup}_${videoName}.json`}">download JSON</a>
        {/if}
        <!-- <p>All are mandatory. User ID:</p>
        <input type="text" placeholder="example: 1234" bind:value={userId} maxlength="8">
        <p>User Group:</p>
        <input type="text" placeholder="example: a" bind:value={userGroup} maxlength="1"> -->
        <p>Put the csv file that contains the daily user groups and daily videos here:</p>
        <input type="file" accept=".csv" on:change={onFileChange}>
        {#if generateLink}
            <button on:click={generateLinks}>Generate Links</button>
        {/if}
     </div>