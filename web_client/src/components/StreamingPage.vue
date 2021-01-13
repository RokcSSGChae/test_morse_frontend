
<template>
    <div>
        <html>
        <head>
        <title>Kurento Tutorial 3: Video Call 1 to N with WebRTC</title>
        </head>
        <body>
            <div class="container">
                <div class="row">
                    <div class="col-md-5">
                        <div class="row">
					<div class="col-md-12">
						<a id="presenter" @click="test()" class="btn btn-success"><span
							class="glyphicon glyphicon-play"></span> Presenter </a> 
					</div>
				</div>
            </div>
                <div class="col-md-7">
                    <div id="videoBig">
                            <video id="video" autoplay width="640px" height="480px" poster="../assets/logo.png"></video>
                        </div>
                    </div>
                </div>
            </div>
        </body>
        </html>
    </div>
</template>

<script
src="bower_components/kurento-utils/js/kurento-utils.js"></script>
<script>
import axios from "axios"
import kurentoUtils from "kurento-utils"
import Vue from "vue"

var webRtcPeer;
var ws;
var video;

function showSpinner() {
    video = document.getElementById('video');
	video.poster = '../assets/logo.png';
	video.style.background = 'center transparent url("../assets/spinner.gif") no-repeat';
}

function onOfferPresenter(error, offerSdp) {
    if (error)
        return console.error('Error generating the offer');
    console.log('Invoking SDP offer callback function ' + offerSdp);
    var message = {
        id : 'presenter',
        sdpOffer : offerSdp
    }
    console.log(message.sdpOffer)
    vm.sendToServer(message);
};

function onIceCandidate(candidate) {
    //console.log("Local candidate" + JSON.stringify(candidate));

    var message = {
        id : 'onIceCandidate',
        candidate : JSON.stringify(candidate)
    };
    vm.sendToServer(message);
};

var vm = new Vue({
    methods:{
        sendToServer(message){
            console.log("Sending message : " + JSON.stringify(message));
            if(message.id=='presenter'){
                axios({
                    method:"POST",
                    url:"https://localhost:8443/room/create",
                    data:{
                        id:message.id,
                        offerSdp:message.sdpOffer.toString()
                    },
                })
                .then((res)=>{
                    console.log('create response : ' + JSON.stringify(res.data));
                    if(res.data.message=='accepted'){
                        webRtcPeer.processAnswer(res.data.sdpAnswer,function(error) {
                            if(error){
                                return console.log('error : '+error);
                            }
                        })
                    }
                })
                .catch({
                    function (error) {
                        console.log(error+"error");
                    }
                });
            }else if(message.id=='onIceCandidate'){
                console.log(JSON.stringify(message.candidate))
                axios({
                    method:"POST",
                    url:"https://localhost:8443/room/icecandidate",
                    data:{
                        id:message.id,
                        candidate:message.candidate
                    },
                })
                .then((res)=>{
                    console.log(res.data);
                    webRtcPeer.addIceCandidate(JSON.parse(res.data.candidate));
                })
                .catch({
                    function (error) {
                        console.log(error+"error");
                    }
                });
            }
        }
    }
})

export default { 
    created(){
        console.log("try connection")
        ws=new WebSocket('wss://localhost:8443/call');
        ws.onopen = () =>{
            console.log("connect");
        };
        //this.video = document.getElementById('video');
        //this.disableStopButton();
    },
    data(){
        return{
            
        }
    },
    methods:{
            test(){
                this.presenter();
            },
            presenter() {
                showSpinner();

                var options = {
                    localVideo : video,
                    onicecandidate : onIceCandidate
                }
                webRtcPeer = new kurentoUtils.WebRtcPeer.WebRtcPeerSendonly(options,
                    function(error) {
                        if (error) {
                            return console.log(error);
                        }
                        webRtcPeer.generateOffer(onOfferPresenter)
                    });
            },
    }
}
</script>

<style>
    html {
        position: relative;
        min-height: 100%;
    }
    body {
        padding-top: 40px;
    }

    video, #console {
        display: block;
        font-size: 14px;
        line-height: 1.42857143;
        color: #555;
        background-color: #fff;
        background-image: none;
        border: 1px solid #ccc;
        border-radius: 4px;
        -webkit-box-shadow: inset 0 1px 1px rgba(0, 0, 0, .075);
        box-shadow: inset 0 1px 1px rgba(0, 0, 0, .075);
        -webkit-transition: border-color ease-in-out .15s, box-shadow
            ease-in-out .15s;
        transition: border-color ease-in-out .15s, box-shadow ease-in-out .15s;
    }

    #console {
        overflow-y: auto;
        width: 100%;
        height: 175px;
    }

    #videoContainer {
        position: absolute;
        float: left;
    }

    #videoBig {
        width: 640px;
        height: 480px;
        top: 0;
        left: 0;
        z-index: 1;
    }

    div#videoSmall {
        width: 240px;
        height: 180px;
        padding: 0px;
        position: absolute;
        top: 15px;
        left: 400px;
        cursor: pointer;
        z-index: 10;
        padding: 0px;
    }

    div.dragged {
        cursor: all-scroll !important;
        border-color: blue !important;
        z-index: 10 !important;
    }
</style>