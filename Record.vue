<template>
  <div>
      
    <div class="popup" v-if="showPopup">
        <div class="close" @click="showPopup=false"> X</div>
        <div class="song" v-for="(item,i) in song" :key="i" @click="selectAudio(item)"> {{item.name}}</div>
    </div>

    <audio id="audioPlayer" controls style="display:none;" mediagroup="a11y_vid">
        <source id="audioSource" src="https://money91kxtvk2qmhk.cdn.e2enetworks.net/7014c6ab-4f8f-4b6a-9f97-a56b9c1df9d3.mpga" type="audio/mpeg">
    </audio>

    <video id="myVideo" mediagroup="a11y_vid" class="video-js vjs-default-skin" playsinline :width="width" :height="height"></video>

    <button class="addmusic" @click="showMusic" v-if="showRecord">Add Music</button>
    <button class="record" @click="startRecord" v-if="showRecord">Record</button>
    <button class="record" @click="stopRecord" v-if="showStop">Stop</button>
    <button class="record" @click="playRecord" v-if="showPlay">Play</button>
    <button class="next" v-if="showNext" @click="post()">Post</button>
    <button class="back" v-if="showBack" @click="backRecord()">Back</button>
    
    <loader91 v-if="isLoading" />
  </div>
    
</template>

<script>
    /* eslint-disable */
    import 'video.js/dist/video-js.css'
    import 'videojs-record/dist/css/videojs.record.css'
    import videojs from 'video.js'
    import 'webrtc-adapter'
    import RecordRTC from 'recordrtc'


    // the following imports are only needed when you're recording
    // audio-only using the videojs-wavesurfer plugin
    /*
    import WaveSurfer from 'wavesurfer.js';
    import MicrophonePlugin from 'wavesurfer.js/dist/plugin/wavesurfer.microphone.js';
    WaveSurfer.microphone = MicrophonePlugin;

    // register videojs-wavesurfer plugin
    import videojs_wavesurfer_css from 'videojs-wavesurfer/dist/css/videojs.wavesurfer.css';
    import Wavesurfer from 'videojs-wavesurfer/dist/videojs.wavesurfer.js';
    */

    import Record from 'videojs-record/dist/videojs.record.js'

    export default {
        data() {
            return {
                song:[
                    {
                        url:"https://money91kxtvk2qmhk.cdn.e2enetworks.net/c04bbf42-cad5-4606-bd62-0fea1e8abcf3.mpga",
                        name:'Lehanga Jass Manak (Official Song)'
                    },
                    {
                        url:"https://money91kxtvk2qmhk.cdn.e2enetworks.net/cbf41607-f5b9-46b3-857d-4e7c50347ecf.mpga",
                        name:'Badshah - DJ Waley Babu '
                    },
                    {
                        url:"https://money91kxtvk2qmhk.cdn.e2enetworks.net/7014c6ab-4f8f-4b6a-9f97-a56b9c1df9d3.mpga",
                        name:'Genda Phool song from delhi 6'
                    },
                    {
                        url:"https://money91kxtvk2qmhk.cdn.e2enetworks.net/c2b37ba6-aa9e-4af3-abb6-044d60bae375.mpga",
                        name:'लॉलीपॉप लागेलू - Pawan Singh'
                    }

                ],
                xauthtoken : this.$cookies.get("X-Auth-Token"),
                jwtToken : this.$cookies.get("JWT-TOKEN"),
                cityName : this.$cookies.get("cityName"),
                xauthid : this.$cookies.get("X-Auth-Id"),
                showPopup:false,
                showStop:false,
                showRecord:true,
                showPlay:false,
                showBack:false,
                selectedAudio:null,
                width: window.innerWidth,
                height: window.innerHeight,
                
                player: '',
                audioPlayer: null,
                dataAvailable:false,
                showNext:false,
                isLoading: false,
                options: {
                    controls: false,
                    autoplay: false,
                    loop: false,
                    fluid: false,
                    responsive: true,
                    // aspectRatio: '9:16',
                    // fill: true,
                    width: window.innerWidth,
                    height: window.innerHeight,
                    bigPlayButton: false,
                    controlBar: {
                        volumePanel: false
                    },
                    plugins: {
                        /*
                        // this section is only needed when recording audio-only
                        wavesurfer: {
                            backend: 'WebAudio',
                            waveColor: '#36393b',
                            progressColor: 'black',
                            debug: true,
                            cursorWidth: 1,
                            displayMilliseconds: true,
                            hideScrollbar: true,
                            plugins: [
                                // enable microphone plugin
                                WaveSurfer.microphone.create({
                                    bufferSize: 4096,
                                    numberOfInputChannels: 1,
                                    numberOfOutputChannels: 1,
                                    constraints: {
                                        video: false,
                                        audio: true
                                    }
                                })
                            ]
                        },
                        */
                        // configure videojs-record plugin
                        record: {
                            image: false,
                            audio: true,
                            video: true,                            
                            maxLength: 30,
                            debug: true,
                            video: {
                                // video media constraints: set resolution of camera
                                width: window.innerWidth,
                                height: window.innerHeight
                            },

                            frameWidth:window.innerWidth,
                            frameHeight:window.innerHeight,
                            // screen:true,
                            // autoMuteDevice:true,
 
                        }
                    }
                }
            };
        },

        computed: {
            music: {
                get() {
                    return this.$store.state.music;
                },
                set(newValue) {
                    return this.$store.dispatch("setMusic", newValue);
                }
            },
            user: {
                get() {
                    return this.$store.state.user;
                },
                set(newValue) {
                    return this.$store.dispatch("setUser", newValue);
                }
            }
        },

        mounted() {
            // if(Object.keys(this.user).length ==0) this.$router.go(-1)

            var _this = this
            _this.audioPlayer = document.getElementById('audioPlayer')
            
            /* eslint-disable no-console */
            _this.player = videojs('#myVideo', _this.options, () => {
                // print version information at startup
                var msg = 'Using video.js ' + videojs.VERSION +
                    ' with videojs-record ' + videojs.getPluginVersion('record') +
                    ' and recordrtc ' + RecordRTC.version;
                videojs.log(msg);
                _this.player.record().getDevice();
            });

            // _this.player.ready()

            // device is ready
            _this.player.on('deviceReady', () => {
                console.log('device is ready!');
                // _this.player.record().start()

            });

            // user clicked the record button and started recording
            _this.player.on('startRecord', () => {
                console.log('started recording!');
                if(this.selectedAudio!=null) {
                    _this.audioPlayer.play()
                    console.log('played!');

                }
                // _this.dataAvailable = false;
                // _this.showNext = false;
                // Create a track object.
                // Get the track we created in an earlier example.
            });

            // user completed recording and stream is available
            _this.player.on('finishRecord', () => {
                // the blob object contains the recorded data that
                // can be downloaded by the user, stored on server etc.
                console.log('finished recording: ', _this.player.recordedData);
                _this.dataAvailable=true,
                console.log(_this.player)
                _this.audioPlayer.pause()
                _this.selectedAudio = null

                // var canvas = document.getElementById('canvas');
                // var video = document.getElementById('video');
                // canvas.getContext('2d').drawImage(video, 0, 0, video.videoWidth, video.videoHeight);
            });

            // error handling
            _this.player.on('error', (element, error) => {
                console.warn(error);
            });

            _this.player.on('deviceError', () => {
                console.error('device error:', _this.player.deviceErrorCode);
            });

            _this.player.on('play', () => {
                if(_this.dataAvailable) {
                    _this.showNext = true
                    _this.showBack = true
                }
                console.log('play!');
                // Create a track object.
                // Get the track we created in an earlier example.
            });
        },

        methods: {
            selectAudio(item) {                
                this.selectedAudio = item.url
                this.showPopup = false
                console.log(this.selectedAudio)
                var source = document.getElementById('audioSource');
                source.src = this.selectedAudio
                this.audioPlayer.load();
                // this.audioPlayer.play();
            },

            showMusic() {
                this.showPopup = true
                console.log('showMusic')
            },

            backRecord() {
                this.showBack = false
                this.showRecord = true
                this.showStop = false                
                this.showPlay = false
                this.showNext = false

                this.dataAvailable = false
                this.player.pause()
                this.audioPlayer.pause()
                this.selectedAudio = null
                
            },

            startRecord() {
                this.showStop = true
                this.showRecord = false
                this.showPlay = false
                this.player.record().start();
            },
            stopRecord() {
                this.showStop = false
                this.showRecord = false
                this.showPlay = true
                this.player.record().stop();
            },
            playRecord() {
                this.showStop = false
                this.showRecord = false
                this.showPlay = true
                this.player.play();
            },

            async post() {
                this.isLoading = true

                var file = await this.axios.put('https://test4.services-money91.com/upload_file/binary?fileType=webm', this.player.recordedData, {
                    headers: {
                        "Content-Type": "application/octet-stream",
                        "X-Auth-Id": this.xauthid,
                        "JWT-TOKEN": this.jwtToken
                    }
                })

                console.log('file',file.data,this.user)
                if (file.data.Success) {
                    let res = await this.axios.post(
                    `${window.serverUrl}api/uploadvideo`,
                    {
                        userId: this.user.id,
                        xauthtoken:this.xauthtoken,
                        name:null,
                        poster: null,
                        url: file.data.Data.CdnUrl

                    });

                    if (res !== null && res.data.status) {
                        this.isLoading = false;
                        
                    } else {
                        this.$toast.bottom(res.data.message);
                    }

                    this.$router.go('-1')
                    
                } else {
                    this.isLoading = false
                    console.log("not uploaded!");
                }
            }
        },

        beforeDestroy() { 
            if (this.player) {
                this.player.dispose();
            }
        }
    }
</script>

<style scoped> 
.popup {
    position: absolute;
    z-index: 9;
    top: 10%;
    background: #fff;
    width: 100%;
    padding: 15px;
}
.song{
    padding: 10px 0;
}
.close {
    text-align: center;
}
.record {
    position: absolute;
    z-index: 9;
    bottom: 10%;
    left: 40%;
    border-radius: 50%;
    height: 60px;
    width: 60px;
}
.back, .addmusic {
    position: absolute;
    z-index: 9;
    bottom: 10%;
    left: 10%;
    border-radius: 50%;
    height: 60px;
    width: 60px;
}
.next {
    position: absolute;
    z-index: 9;
    bottom: 10%;
    right: 10%;
    border-radius: 50%;
    height: 60px;
    width: 60px;
}
.video-js .vjs-tech {
    object-fit: cover;
}


</style>