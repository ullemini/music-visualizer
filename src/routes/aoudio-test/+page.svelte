<script lang="ts">
    import { onMount } from "svelte";
    import { Segment } from '@skeletonlabs/skeleton-svelte'; 
    import { FileUpload } from '@skeletonlabs/skeleton-svelte';
    import { Switch } from '@skeletonlabs/skeleton-svelte';
    
    // aoudio settup
    const audioContext = new AudioContext();
    let canvasDOM:HTMLCanvasElement;
    let audioElement:HTMLAudioElement;
    let analyser:AnalyserNode = audioContext.createAnalyser();
    analyser.fftSize = 2048;
    analyser.smoothingTimeConstant = 0.65;
    let bufferLength:number = analyser.frequencyBinCount;
    let canvasContext:CanvasRenderingContext2D ;
    let frequencydata:Uint8Array<ArrayBuffer> = new Uint8Array(bufferLength);
    //settings setttup
    const VisulizerStyle = {
        bar:"bar",
        wave:"wave",
        off:"off",
    };
    const fftSize = [
        "32", "64", "128", "256", "512", "1024", "2048", "4096", "8192", "16384","32768"
    ]
    let currentsong = $state("aoudio/I-LIKE-LOUD-THINGS.mp3")
    let currentVisulizerStyle:string = $state("bar")
    let currentfftSize:string = $state("2048")
    let showAllffsise = $state(false)
    let showfileupload = $state(true)
    let showSettings = $state(true)
    
    onMount(()=>{ 
        
        canvasContext = canvasDOM.getContext("2d")
        
        let aoudioSorce = audioContext.createMediaElementSource(audioElement);
        aoudioSorce.connect(analyser)
        analyser.connect(audioContext.destination);
        audioElement.src = currentsong
        function draw(){
            const drawVisual = requestAnimationFrame(()=>draw());
            
            
            let canvas_width = 1800
            let canvas_height = 150

            canvasContext.clearRect(0, 0, canvas_width, canvas_height);
            switch (currentVisulizerStyle){
                case VisulizerStyle.off: break; 
                case VisulizerStyle.wave:
                    visulizerStyleWave(canvas_width,canvas_height);
                    break;
                case VisulizerStyle.bar: 
                    visulizerStyleBar(canvas_width,canvas_height); 
                    break;
                
            }
        }
        
       draw();

    })
    let colur_style = ""
    function color_formulas(barHeight:number,i:number,x:number){
        switch (colur_style){
            case "1": return  `rgb(${barHeight + 100} ${i*10}  ${ Math.floor((x+2)/2)})`;
        }
        return `rgb(${barHeight + 100} ${i*6}  ${ Math.floor((x+2)/2)})`;
        
    }
    //visulisation styles >>> https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API/Visualizations_with_Web_Audio_API
    function visulizerStyleBar(width:number,height:number){
            analyser.getByteFrequencyData(frequencydata);
            const barWidth = (width / bufferLength) * 2.5;
            let barHeight;
            let x = 0;
            canvasContext.fillStyle = `rgb(255 255 255)`; 
            for (let i = 0; i < bufferLength; i++) {
                barHeight = 1 + (frequencydata[i] / 2);
                canvasContext.fillStyle = color_formulas(barHeight,i,x); 
                canvasContext.fillRect(x, height - barHeight / 2, barWidth, barHeight);

                x += barWidth + 1;
            }
    }
    function visulizerStyleWave(width:number,height:number){
        analyser.getByteTimeDomainData(frequencydata);
        canvasContext.fillStyle = "rgb(18 18 18)";
        canvasContext.fillRect(0, 0, width, height);
        canvasContext.lineWidth = 2;
        canvasContext.strokeStyle =  "rgb(250 250 250)";
        canvasContext.beginPath();
        const sliceWidth = width / bufferLength;
        let x = 0;
        for (let i = 0; i < bufferLength; i++) {
            const v = frequencydata[i] / 128.0;
            const y = v * (height / 2);
            if (i === 0) {
                canvasContext.moveTo(x, y);
                
            } else {
                canvasContext.lineTo(x, y)    
            }
            
            x += sliceWidth;
        }
        canvasContext.lineTo(width, height / 2);
        canvasContext.stroke();
    }
    //settings cange
    function changeffsise(zise:string) { 
        analyser.fftSize = parseInt(zise,10)
        bufferLength = analyser.frequencyBinCount 
        frequencydata = new Uint8Array(bufferLength);
        currentfftSize = zise
    }   
    //file upload haneler  >>>> https://stackoverflow.com/questions/33446206/how-to-load-a-file-into-a-html5-audio-tag
    function HadelFileUpload(files){
        var blob = window.URL || window.webkitURL;
        if (!blob) {
            console.log('Your browser does not support Blob URLs :('); 
        }
        console.log(files)
        currentsong = blob.createObjectURL(files[0])
        audioElement.src = currentsong
    }

</script>




<aside id="controll-pannel" >
    <div id="show-settings-switch" ><Switch name="show-settings" checked={showSettings} onCheckedChange={(e) => (showSettings = e.checked)}></Switch></div>

    {#if showSettings}
    <div id="style-settigs">
    <Segment value={currentVisulizerStyle} onValueChange={(e) => (currentVisulizerStyle = e.value)}>
        {#each Object.entries(VisulizerStyle) as [key,value]}
            <Segment.Item value={value}>{value}</Segment.Item>
        {/each}
    </Segment>
    </div>
    <Switch name="show-fileupload"  checked={showfileupload} onCheckedChange={(e)=>(showfileupload = e.checked)}></Switch>

    <div class="ff-settings">
        <div class=flex-column>   
        <Segment orientation="vertical" value={currentfftSize} onValueChange={(e) => (changeffsise(e.value))}>
            <button onclick={()=>{console.log(showAllffsise=!showAllffsise);showAllffsise!=showAllffsise}}>Cange ff-sise</button>
            {#each fftSize as sise}
                {#if showAllffsise || sise == currentfftSize}
                <Segment.Item value={sise}>
                    {sise}
                </Segment.Item>
                {/if}
            {/each}
        </Segment>
        
        </div>

    </div>
    {/if}
</aside>


<main id="visulizer-div" >
    <canvas id="maincanvas" width="1800" height="150" bind:this={canvasDOM}><p>Visulizer is displayed here</p></canvas>
</main>
<audio src={currentsong} id="mainAudio" controls={showSettings} loop bind:this={audioElement}></audio>

{#if showfileupload && showSettings}
<div id="file-upoad-div">
    <FileUpload 
    maxFiles={1}  
    allowDrop 
    name="audio upload" 
    accept="audio/*" 
    onFileAccept={(details)=>{HadelFileUpload(details.files)}} 
    onFileReject={console.error} 
    classes="w-full" 
    ></FileUpload>
</div>
{/if}

<style>
    
    canvas{
        width: 100%;
        height: 100%;
    }
    audio{
        width: 100%;
    }
    #visulizer-div{
        width: 100vw;
        height: 85vh;
       
    }
    #controll-pannel{
        display: flex;
        position: fixed;
        top: 50px;
        left: 10px;
        
    }
    #show-settings-switch{
        top:15px;
        left:15px;
        position: fixed;
    }
    #style-settigs{
        margin-right: 15px;
        
    }

    .ff-settings{
        z-index: 1;
        position: fixed;
        right: 0;
    }
    .flex-column{
        display: flex;
        flex-direction: column;
    }
    #file-upoad-div{
        z-index: 2;
        top: 25%;
        position: fixed;
        width: 100%;
        height: 25%;
    }
    

</style>