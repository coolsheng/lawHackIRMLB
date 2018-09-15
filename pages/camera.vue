<template>
    <div id="app">
        <div>
            <video ref="video" id="video" width="auto" height="480" autoplay></video>
</div>
        <div>
        <at-button v-on:click="textDetection()" >Text Detection</at-button>
        <at-button style="margin-left: 10px;" type="primary" v-on:click="labelDetection()" >Label Detection</at-button>
        </div>
        <canvas ref="canvas" id="canvas" width="640" height="480"></canvas>
<!-- <input type="file" accept="image/*"> -->
    <!-- <img src="" alt=""> -->
<div id="result" class="steps-content" style="margin-top: 16px; border: 1px solid #e9e9e9; border-radius: 6px;background-color: #fafafa; min-height: 200px; text-align: center; padding-top:10">
</div>


    </div>
</template>

<script>

    export default {
        name: 'app',
        data() {
            return {
                video: {},
                canvas: {},
                captures: []
            }
        },
        mounted() { 
    this.video = this.$refs.video;
    if(navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
        navigator.mediaDevices.getUserMedia({ video:  {facingMode: {exact: "environment"}} }).then(stream => {
            this.video.src = window.URL.createObjectURL(stream);
            this.video.play();
        });
    }
        },
        methods: {
    capture() {
        this.canvas = this.$refs.canvas;
        var context = this.canvas.getContext("2d").drawImage(this.video, 0, 0, 640, 480);
        // this.captures.push(canvas.toDataURL("image/png"));
        // this.video= false;
        // console.log(this.canvas.toDataURL("image/png"));
        
        // return this.canvas.toDataURL();
        return this.canvas.toDataURL().substring(22)
    },
    async labelDetection(){
        let image = this.capture();
        // console.log(image)
        let labels = await this.$axios.$post("https://us-central1-lawhack-215604.cloudfunctions.net/function-2", {
            image: {
                content: decodeURIComponent(image)
            }
        })
        let str = '<ul>' + 'Result: '
        labels.forEach(element => {
            str += '<li>' + element.description + '- ' + element.score.toFixed(3) + '</li>';
            console.log(element)
        });
        str += '</ul>'
        document.getElementById("result").innerHTML = str;
        console.log(labels);
        },

    async textDetection(){
        let image = this.capture();
        let text = await this.$axios.$post("https://us-central1-lawhack-215604.cloudfunctions.net/function-1",{
            image: {
                content: decodeURIComponent(image)
            }
        })
        console.log("wow",text);
        // let summary = '<ul>' + 'Summary(key clause): '
        let summary = "";
                            
        text.forEach(element => {
                    if(element.description === "prioritise"|| element.description === "subsidiary"){
                        summary +=  '<li>' + "The Directors must not prioritise the interest of any company of which the Company is a wholly-owned subsidiary over the interests of the Company. " + "</li>"
                    }
                    else if(element.description === "exercise" || element.description === "powers"){
                        summary += '<li>' + "the Directors must not exercise any powers of the Company in the month of January in 2020." + "</li>"
                    }
                    else if(element.description === "acquire"||element.description === "indirectly"){
                        summary += '<li>' + "the Company must not acquire or hold any interest in any company engaged, whether directly or indirectly, in any activity set out in Schedule 2." + "</li>"
                    }
        //             else{
        // summary += '<li>' + element.description + '</li>';
        // summary += '</ul>'             
        //             }   
            console.log(element.description)
        });


        document.getElementById("result").innerHTML = summary;
        },        
    }

         }
    
</script>

<style>
    #app {
        text-align: center;
        color: #2c3e50;
        margin-top: 30px;
    }
    #video {
        background-color: #000000;
    }
    #canvas {
        display: none;
    }
    /* li {
        display: inline;
        padding: 5px;
    } */
</style>