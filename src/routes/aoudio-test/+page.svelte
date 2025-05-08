<script lang="ts">
    import { onMount } from "svelte";

    let canvasDOM:HTMLCanvasElement
    const VisulizerStyle = {
        Bar:"bar",
        off:"off",
        Other:"other"

    };
    let currentVisulizerStyle:string = "bar"

    onMount(()=>{ 
        
        const audioContext = new AudioContext();
        const audioElement = document.getElementById("mainAudio");
        const canvas = document.getElementById("maincanvas");
        const canvasContext = canvas.getContext("2d");
        const analyser = audioContext.createAnalyser();
        analyser.fftSize = 256
        const bufferLength = analyser.frequencyBinCount;
        
        let aoudioSorce = audioContext.createMediaElementSource(audioElement);
        aoudioSorce.connect(analyser)
        analyser.connect(audioContext.destination);
        const frequencydata = new Uint8Array(bufferLength);
        analyser.smoothingTimeConstant = 0.65;
        
        function draw(){
            const drawVisual = requestAnimationFrame(()=>draw());
            
            
            let canvas_width = 1800
            let canvas_height = 150

            canvasContext.clearRect(0, 0, canvas_width, canvas_height);
            switch (currentVisulizerStyle){
                case "off": break; 
                case "bar": style1(canvas_width,canvas_height); break;
                
            }
        }
        function style1(width,height){
            analyser.getByteFrequencyData(frequencydata);
            const barWidth = (width / bufferLength) * 2.5;
            let barHeight;
            let x = 0;
            canvasContext.fillStyle = `rgb(255 250 250)`; 
            canvasContext.clearRect(0, 0, height, width);
            for (let i = 0; i < bufferLength; i++) {
                barHeight = 1 + (frequencydata[i] / 2);
                canvasContext.fillStyle = `rgb(${barHeight + 100} ${Math.floor(x/2)}  ${i*8})`; 
                canvasContext.fillRect(x, height - barHeight / 2, barWidth, barHeight);

                x += barWidth + 1;
            }
        }
       draw();

    })

    function clickybutton(){
        
    }
    
</script>

<main>
    <div id=controll-pannel >
        <button id=button1 on:click={()=>{clickybutton}} >button 1</button>
        <audio src="aoudio\wise-mystical-magical-wizard.mp3" id="mainAudio" controls loop></audio>
    </div>
    <div id=visulizer-div >
        <canvas id=maincanvas width="1800" height="150" bind:this={canvasDOM}><p>visulizer</p></canvas>
    </div>
</main>

<style>
    button{
        color: aqua;
        background-color: bisque;
    }
    canvas{
        width: 100%;
        height: 100%;
        border-width: 2px;   
        border-color: brown;
    }
    #visulizer-div{
        width: 100vw;
        height: 80vh;
        border-width: 1px;   
    }
    #controll-pannel{
        display: flex;
    }
    
</style>