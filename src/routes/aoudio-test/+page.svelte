<script lang="ts">
    import { onMount } from "svelte";
    import { Segment } from '@skeletonlabs/skeleton-svelte'; 

    const audioContext = new AudioContext();
    let canvasDOM:HTMLCanvasElement;
    let audioElement:HTMLAudioElement;
    let analyser:AnalyserNode = audioContext.createAnalyser();
    let canvasContext:CanvasRenderingContext2D ;
    let frequencydata:Uint8Array<ArrayBuffer>
    const VisulizerStyle = {
        bar:"bar",
        off:"off",
        beta:"beta",
        other:"other"
    };
    const fftSize = {
        default:"256",
        dubble:"512"
    }
    let currentVisulizerStyle:string = $state("bar")
    let currentfftSize:string = $state(`${fftSize.default}`)
    onMount(()=>{ 
        
        canvasContext = canvasDOM.getContext("2d")
        analyser.fftSize = 256
        const bufferLength = analyser.frequencyBinCount;
        
        let aoudioSorce = audioContext.createMediaElementSource(audioElement);
        aoudioSorce.connect(analyser)
        analyser.connect(audioContext.destination);
        frequencydata = new Uint8Array(bufferLength);
        analyser.smoothingTimeConstant = 0.65;
        
        function draw(){
            const drawVisual = requestAnimationFrame(()=>draw());
            
            
            let canvas_width = 1800
            let canvas_height = 150

            canvasContext.clearRect(0, 0, canvas_width, canvas_height);
            switch (currentVisulizerStyle){
                case VisulizerStyle.off: break; 
                case VisulizerStyle.beta:
                    canvasContext.fillStyle = `rgb(255 255 255)`; 
                    canvasContext.fillRect(0, 0, canvas_width,canvas_height); 
                    break;
                case VisulizerStyle.bar: 
                    visulizerStyleBar(canvas_width,canvas_height,bufferLength,frequencydata); 
                    break;
                
            }
        }
        
       draw();

    })
    
    function visulizerStyleBar(width:number,height:number,bufferLength:number,frequencydata:Uint8Array<ArrayBuffer>){
            analyser.getByteFrequencyData(frequencydata);
            const barWidth = (width / bufferLength) * 2.5;
            let barHeight;
            let x = 0;
            canvasContext.fillStyle = `rgb(255 250 250)`; 
            for (let i = 0; i < bufferLength; i++) {
                barHeight = 1 + (frequencydata[i] / 2);
                canvasContext.fillStyle = `rgb(${barHeight + 100} ${Math.floor(x/2)}  ${i*8})`; 
                canvasContext.fillRect(x, height - barHeight / 2, barWidth, barHeight);

                x += barWidth + 1;
            }
        }
        function changeffsise(zise:string) {
            switch (zise){
                case "256": analyser.fftSize = 256; break;
                case "512": analyser.fftSize = 512; break;
            }
        }
</script>

<main >
    <div id="controll-pannel" >
        <Segment value={currentVisulizerStyle} onValueChange={(e) => (currentVisulizerStyle = e.value)}>
            <Segment.Item value="bar">Bar</Segment.Item>
            <Segment.Item value="beta">Beta</Segment.Item>
            <Segment.Item value="off">Off</Segment.Item>
        </Segment >

        <Segment value={currentfftSize} onValueChange={(e) => (changeffsise(e.value))}>
            {#each Object.entries(fftSize) as [name,sise]}
                <Segment.Item value={`${sise}`}>
                    {name} {sise}
                </Segment.Item>
            {/each}
            
        </Segment>
    </div>
    <div id="visulizer-div" >
        <canvas id="maincanvas" width="1800" height="150" bind:this={canvasDOM}><p>visulizer</p></canvas>
    </div>
    <audio src="aoudio\wise-mystical-magical-wizard.mp3" id="mainAudio" controls loop bind:this={audioElement}></audio>
</main>

<style>
    button{
        color: aqua;
        background-color: bisque;
    }
    canvas{
        width: 100%;
        height: 100%;
    }
    audio{
        width: 100vw;
    }
    #visulizer-div{
        width: 100vw;
        height: 80vh;
       
    }
    #controll-pannel{
        display: flex;
    }
    
</style>