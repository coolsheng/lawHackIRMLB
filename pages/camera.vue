<template>
    <div id="app">
        <div><video ref="video" id="video" width="640" height="480" autoplay></video></div>
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
        navigator.mediaDevices.getUserMedia({ video: true }).then(stream => {
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
        let labels = await this.$axios.$post("http://localhost:4000/label", {
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
        },

    async textDetection(){
        let image = this.capture();
        let text = await this.$axios.$get("https://us-east1-lawhack-215604.cloudfunctions.net/ImageFunction",{
            image: {
                content: decodeURIComponent(image)
            }
        })
        console.log(text);
        let str = '<ul>' + 'Result: '
        text.forEach(element => {
            str += '<li>' + element.description + '</li>';
            console.log(element.description)
        });
        str += '</ul>'
        document.getElementById("result").innerHTML = str;
        },        
    }

         }
    
</script>

<style>
    #app {
        text-align: center;
        color: #2c3e50;
        margin-top: 60px;
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