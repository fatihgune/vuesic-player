<template>
    <div id="vuesic-player">
        <div class="vp-player">
            <!-- the Next Up List -->
            <transition name="slide-fade">
                <div class="vp-next-up" :class="showNextUpList ? 'show' : 'hide'" v-show="showNextUpList">
                    <div class="vp-wrap dark:bg-gray-800 bg-white dark:text-white text-gray-900">
                        <div class="vp-next-up-header justify-between">
                            <h4 class="font-bold">Next Up</h4>
                            <div>
                                <button class="vp-next-up-clear-button vp-selected" @click="clearNextUp">Clear</button>
                                <button class="ml-5 mr-3" @click="closeNextUp">X</button>
                            </div>
                        </div>

                        <div class="vp-next-up-content">
                            <div class="vp-song-wrap z-50" v-for="(song, index) in  nextUpList.songs"
                                 :key="index"
                                 :class="{'vp-selected' : isSelected(song.id, index)}"
                                 @mouseover="itemHoveredInNextUpList = index"
                                 @mouseleave="itemHoveredInNextUpList = null">
                                <div class="vp-song-art"
                                     :style="{'background-image': 'url(' + song.cover_art_url + ')'}"
                                     @click="play(song)">
                                    <button v-if="displayPlayPauseButtonForSongsInNextUpList(song.id, index)"
                                            class="text-gray-900">
                                        <component class="vp-button-icon"
                                                   :style="isSelectedSongPlaying(song.id) ? '' : 'padding-left: 2px'"
                                                   :is="isSelectedSongPlaying(song.id) ? icons.PauseIcon : icons.PlayIcon"/>
                                    </button>
                                </div>
                                <div class="vp-song-details" @click="play(song)">
                                    <div class="vp-song-title">
                                        <button>{{ song.title }}</button>
                                    </div>
                                    <div class="vp-song-author">
                                        <button class="dark:text-white text-gray-400"><small>{{ song.artist }}</small>
                                        </button>
                                    </div>
                                </div>
                                <button class="vp-close-button z-10"
                                        v-if="displayXButtonForSongsInNextUpList(song.id, index)"
                                        @mouseover="keepSelectedState(song.id, index)"
                                        @click="removeFromNextUpList(song.id)">x
                                </button>
                                <button class="vp-song-like-button-in-next-up" @click="likeSongToggle(song.id)">
                                    <component :is="song.isLiked ? icons.HeartIconSolid : icons.HeartIcon"/>
                                </button>
                            </div>
                        </div>
                    </div>
                </div>
            </transition>
            <!-- end of Next Up List -->
            <!-- the music player code starts here -->
            <div class="vp-player-bar dark:bg-gray-700 bg-white grid grid-cols-3 dark:text-white text-gray-900"
                 id="vp-player">
                <div class="vp-player-left flex place-items-center">
                    <div class="vp-album-art">
                        <img :src="currentSong.cover_art_url" class="vp-img"/>
                    </div>
                    <div class="vp-title-and-artist">
                        <div class="vp-title">
                            {{ currentSong.title }}
                        </div>
                        <div class="vp-artist font-thin">
                            <div class="vp-artist-inner-1" @mouseover="artistNameSlideHovered"
                                 @mouseleave="artistNameSlideLeft">
                                <div class="vp-artist-inner-2">
                                    <div class="vp-artist-inner-3" @mouseenter="artistNameSlide">
                                        <div class="vp-artist-inner-4">
                                            <span class="vp-artist-inner-span">
                                                 {{ currentSong.artist }}
                                            </span>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                    <button class="vp-song-like-button-left" @click="likeSongToggle(currentSong.id)">
                        <component :is="currentSong.isLiked ? icons.HeartIconSolid : icons.HeartIcon"/>
                    </button>
                </div>
                <div class="vp-player-center vp-main-controls flex flex-col">
                    <div class="vp-seek align-text-center">
                        <div class="vp-controls-wrapper flex justify-center">
                            <div class="vp-controls dark:text-white text-gray-900">
                                <div class="vp-skip-backward cursor-pointer">
                                    <component :is="icons.ChevronDoubleLeftIcon" aria-hidden="true"
                                               @click="skip('backward')"/>
                                </div>

                                <div class="vp-play cursor-pointer">
                                    <component :is="icons.PlayIcon" aria-hidden="true" v-if="!isPlaying"
                                               @click="playCurrentSong"/>
                                    <component :is="icons.PauseIcon" aria-hidden="true" v-else @click="pause"/>
                                </div>

                                <div class="vp-skip-forward cursor-pointer">
                                    <component :is="icons.ChevronDoubleRightIcon" aria-hidden="true"
                                               @click="skip('forward')"/>
                                </div>

                            </div>
                        </div>

                        <div class="vp-progress-container">
                            <div class="vp-time text-gray-400 m-0 mr-1"><small>{{ currentPlayedTime }}</small></div>
                            <div class="vp-progress p-0" id="vp-progress-wrap">
                                <input type="range" min="0" max="100" @click="seek" step="0.01" :value="progressValue"
                                       class="vp-bar dark:bg-gray-400 bg-white"/>
                            </div>
                            <div class="vp-time text-gray-400 m-0 ml-1"><small>{{ duration }}</small></div>
                        </div>

                    </div>
                </div>
                <div class="vp-player-right flex justify-end">
                    <div class="vp-extra-controls dark:text-white text-gray-900">
                        <div class="vp-next-up-icon cursor-pointer" @click="toggleShowNextUpList">
                            <component class="i" :is="icons.ListBulletIcon" aria-hidden="true"/>
                        </div>
                        <div class="vp-shuffle-icon cursor-pointer">
                            <component class="i" :class="isShuffled ? 'vp-rotate-down' : 'vp-rotate-up'"
                                       :is="icons.ArrowsUpDownIcon" @click="shuffleToggle"
                                       aria-hidden="true"/>
                        </div>

                        <div class="vp-repeat cursor-pointer">
                            <div class="vp-repeat-info" v-if="onRepeat">
                                {{ loop.value }}
                            </div>
                            <component class="i" :is="icons.ArrowPathRoundedSquareIcon" @click="repeat"
                                       aria-hidden="true"/>
                        </div>
                        <div class="vp-volume cursor-pointer">
                            <div class="vp-slider-container" v-show="volumeSliderShown"
                                 @mouseleave="hideShowVolumeSlider"
                                 @mouseover="showVolumeSlider">
                                <input type="range" min="0" max="100" value="100" orient="vertical"
                                       id="vp-volume-slider" aria-hidden="true">
                            </div>
                            <component :is="isMuted ? icons.SpeakerXMarkIcon : icons.SpeakerWaveIcon"
                                       class="vp-volume-slider-icon"
                                       @mouseleave="hideVolumeSlider" @mouseover="showVolumeSlider"
                                       aria-hidden="true" @click="mute"/>
                        </div>
                        <button class="vp-song-like-button-right" @click="likeSongToggle(currentSong.id)">
                            <component :is="currentSong.isLiked ? icons.HeartIconSolid : icons.HeartIcon"/>
                        </button>
                    </div>
                </div>


            </div>
        </div>
        <!-- the audio player code ends here -->
        <audio :loop="innerLoop" ref="audiofile" :src="defaultSong" preload style="display: none" controls></audio>
    </div>


</template>

<script>
import {
    ArrowPathRoundedSquareIcon,
    ArrowsUpDownIcon,
    ChevronDoubleLeftIcon,
    ChevronDoubleRightIcon,
    HeartIcon,
    ListBulletIcon,
    PauseIcon,
    PlayIcon,
    SpeakerWaveIcon,
    SpeakerXMarkIcon,
} from '@heroicons/vue/24/outline'

import {HeartIcon as HeartIconSolid} from '@heroicons/vue/24/solid'

export default {
    name: "MusicPlayer",
    data() {
        return {
            icons: {
                PlayIcon,
                PauseIcon,
                ListBulletIcon,
                ArrowPathRoundedSquareIcon,
                ChevronDoubleRightIcon,
                ChevronDoubleLeftIcon,
                SpeakerXMarkIcon,
                ArrowsUpDownIcon,
                SpeakerWaveIcon,
                HeartIcon,
                HeartIconSolid,
            },
            defaultSong:
                "https://www.soundhelix.com/examples/mp3/SoundHelix-Song-1.mp3",
            isPlaying: false,
            isLoaded: false,
            isCurrentlyPlaying: "",
            onRepeat: false,
            volumeSliderShown: false,
            isShuffled: true,
            isMuted: false,
            itemHoveredInNextUpList: null,
            loop: {
                state: false,
                value: 1
            },
            durationSeconds: 0,
            currentSeconds: 0,
            audioPlayer: undefined,
            previousVolume: 0,
            volume: 100,
            autoPlay: false,
            progressValue: 0,
            artistNameSlideLoopInitiated: false,
            artistNameSlideLoopPaused: false,
            songs: [
                {
                    id: 1,
                    title: "Wow",
                    artist: "Post Malone, Post Malone Post Malone, Post Malone",
                    album: "",
                    url:
                        "https://www.soundhelix.com/examples/mp3/SoundHelix-Song-2.mp3",
                    cover_art_url:
                        "https://upload.wikimedia.org/wikipedia/en/5/5e/Post_Malone_-_Wow.png"
                },
                {
                    id: 2,
                    title: "Gods Plan",
                    artist: "Drake",
                    album: "",
                    url:
                        "https://www.soundhelix.com/examples/mp3/SoundHelix-Song-3.mp3",
                    cover_art_url:
                        "https://i.ytimg.com/vi/FrsOnNxIrg8/maxresdefault.jpg"
                },
            ],

            nextUpList: {
                currentIndex: 0,
                songs: [
                    {
                        id: 1,
                        title: "Wowow",
                        artist: "Post Malone, Post Malone Post Malone, Post Malone",
                        album: "Some Album 1",
                        url: "https://www.soundhelix.com/examples/mp3/SoundHelix-Song-2.mp3",
                        cover_art_url: "https://upload.wikimedia.org/wikipedia/en/5/5e/Post_Malone_-_Wow.png",
                        isLiked: true,
                    },
                    {
                        id: 2,
                        title: "God's Plan",
                        artist: "Drake",
                        album: "Some Album 2",
                        url: "https://www.soundhelix.com/examples/mp3/SoundHelix-Song-3.mp3",
                        cover_art_url: "https://i.ytimg.com/vi/FrsOnNxIrg8/maxresdefault.jpg",
                        isLiked: false,
                    },
                    {
                        id: 3,
                        title: "Test 3",
                        artist: "Test 3",
                        album: "Some Album 3",
                        url:
                            "https://www.soundhelix.com/examples/mp3/SoundHelix-Song-13.mp3",
                        cover_art_url:
                            "https://vertex-academy.com/tutorials/wp-content/uploads/2016/10/Random-number-generation_Vertex-Academy.png",
                        isLiked: false,
                    },
                    {
                        id: 4,
                        title: "Test 4",
                        artist: "Test 4",
                        album: "Some Album 4",
                        url:
                            "https://www.soundhelix.com/examples/mp3/SoundHelix-Song-16.mp3",
                        cover_art_url:
                            "https://techsuplex.com/wp-content/uploads/2016/01/Random.jpg",
                        isLiked: false,
                    },
                ]
            },
            previousNextUpListIndex: 0,
            hasNext: false,
            originalSongArray: [],
            currentSong: {
                id: "",
                title: "",
                artist: "",
                album: "",
                url: "",
                cover_art_url: "",
                isLiked: true,
            },

            /** ui control variables*/
            showNextUpList: false
        }
    },

    created() {
        this.innerLoop = this.loop.state;
        window.addEventListener('keypress', this.onKeyPress);
    },
    mounted() {
        this.audioPlayer = this.$el.querySelectorAll("audio")[0];
        this.initPlayer();
    },
    beforeDestroy() {
        window.removeEventListener('keypress', this.onKeyPress);
    },
    methods: {
        /** UI control methods
         * these methods are used to control the UI */

        toggleShowNextUpList() {
            this.showNextUpList = !this.showNextUpList;
        },

        /** Music player methods
         * these methods are used to control the music player */

        initPlayer() {
            this.audioPlayer.src = this.nextUpList.songs[0].url;
            this.setCurrentSong(this.nextUpList.songs[0]);

            this.audioPlayer.addEventListener("timeupdate", this.updateTimer);
            this.audioPlayer.addEventListener("loadeddata", this.load);
            this.audioPlayer.addEventListener("pause", () => {
                this.isPlaying = false;
            });
            this.audioPlayer.addEventListener("play", () => {
                this.isPlaying = true;
            });

            this.audioPlayer.addEventListener("ended", this.playNextSongInNextUpList);

            this.volume = document.getElementById('vp-volume-slider');
            this.volume.addEventListener("change", this.volumeChange);
            this.volume.value = 100
            this.audioPlayer.volume = 1
        },

        play(song = {}) {
            if (typeof song === "object") {
                if (this.isLoaded) {
                    //check if song exists in next up list
                    if (this.currentSong.id === song.id && this.isPlaying) {
                        this.pause();
                    } else if (this.currentSong.id === song.id && !this.isPlaying) {
                        this.playCurrentSong();
                    } else if (this.currentSong.id !== song.id) {
                        if (!this.containsObjectWithSameId(song, this.nextUpList.songs)) {
                            this.addToNextUpList(song);
                        } else {
                            console.log("playMethod", "Song already in next up list");
                        }

                        this.setAudio(song.url);
                        this.setCurrentSong(song);
                        this.nextUpList.currentIndex = this.getObjectIndexFromArray(
                            song,
                            this.nextUpList.songs
                        );
                        this.previousNextUpListIndex = this.nextUpList.currentIndex;
                        this.audioPlayer.play();
                    }
                } else {
                    this.setAudio(song.url);
                    this.audioPlayer.play();
                }

                this.isPlaying = true;
            } else {
                throw new Error("Type Error : Song must be an object");
            }
        },

        playCurrentSong() {
            this.audioPlayer.play();
            this.isPlaying = true;
        },

        stop() {
            this.audioPlayer.currentTime = 0;
        },

        pause() {
            this.audioPlayer.pause();
            this.isPlaying = false;
        },

        repeat() {
            if (!this.loop.state && !this.onRepeat) {
                //firstclick
                this.loop.state = true;
                this.loop.value = 1;
                this.onRepeat = true;
            } else if (this.loop.state && this.onRepeat && this.loop.value === 1) {
                //second click
                this.loop.state = true;
                this.loop.value = "all";
                this.onRepeat = true;
            } else if (
                this.loop.state &&
                this.onRepeat &&
                this.loop.value === "all"
            ) {
                this.loop.state = false;
                this.loop.value = 1;
                this.onRepeat = false;
            }
        },

        skip(direction) {
            if (direction === "forward") {
                this.nextUpList.currentIndex += 1;
            } else if (direction === "backward") {
                this.nextUpList.currentIndex -= 1;
            }

            /** if the current Index of the next up list is greater or
             * equal to the length of the next up list songs (the index is out of range)
             * reset the index to 0. It could also mean that the next up list is at its end. */

            if (this.nextUpList.currentIndex >= this.nextUpList.songs.length) {
                this.nextUpList.currentIndex = 0;
            }

            if (this.nextUpList.currentIndex < 0) {
                this.nextUpList.currentIndex = this.nextUpList.songs.length - 1;
            }

            this.audioPlayer.src = this.nextUpList.songs[
                this.nextUpList.currentIndex
                ].url;
            this.setCurrentSong(this.nextUpList.songs[this.nextUpList.currentIndex]);

            //the code below checks if a song is playing so it can go ahead and auto play
            if (this.isPlaying) {
                this.audioPlayer.play();
            }
        },

        seek(e) {
            if (this.isLoaded) {
                let el = e.target.getBoundingClientRect();
                let seekPos = (e.clientX - el.left) / el.width;
                let seekPosPercentage = seekPos * 100;

                this.audioPlayer.currentTime = parseInt(
                    this.audioPlayer.duration * seekPos
                );

                this.progressValue = seekPosPercentage;
            } else {
                throw new Error("Song Not Loaded");
            }
        },

        mute() {
            if (this.volume.value !== '0') {
                this.previousVolume = this.volume.value;
            }

            let currentVolume = this.volume.value !== '0' ? this.volume.value : this.previousVolume;

            if (this.audioPlayer.muted) {
                this.volume.value = currentVolume;
                this.audioPlayer.muted = false;
                this.isMuted = false;
                return;
            }

            this.volume.value = 0;
            this.audioPlayer.muted = true;
            this.isMuted = true;
        },

        likeSongToggle(id) {
            const song = this.nextUpList.songs.find(song => song.id === id);
            song.isLiked = !song.isLiked
            // todo tie these together
            if (this.currentSong.id === id) {
                this.currentSong.isLiked = !this.currentSong.isLiked
            }
        },

        updateTimer() {
            this.currentSeconds = parseInt(this.audioPlayer.currentTime);
            this.progressValue =
                this.currentSeconds / parseInt(this.audioPlayer.duration) * 100 || 0;
        },

        isSongSelectedInNextUpList(id) {
            return this.currentSong.id === id && (this.isPlaying || this.currentSeconds >= 0)
        },

        isSelectedSongPlaying(id) {
            return this.isPlaying && this.currentSong.id === id
        },

        displayXButtonForSongsInNextUpList(id, index) {
            return this.itemHoveredInNextUpList === index && !this.isSongSelectedInNextUpList(id)
        },

        displayPlayPauseButtonForSongsInNextUpList(id, index) {
            return this.itemHoveredInNextUpList === index || this.isSongSelectedInNextUpList(id, index)
        },

        keepSelectedState(id, index) {
            if (this.displayXButtonForSongsInNextUpList(id, index)) {
                this.itemHoveredInNextUpList = index
            }
        },

        addAndPlayNext() {
            let selectedSong = {
                title: "Song Name 3",
                artist: "Artist Name",
                album: "Album Name",
                url: "./song2.mp3",
                cover_art_url: "/cover/art/url.jpg",
                isLiked: false,
                isPlaying: false,
            };

            //add the song to the next up list
            //get the index of the song that is currently being played in the player
            //insert the song at that position
            let indexOfCurrentSong = this.nextUpList.currentIndex;

            this.nextUpList.songs.splice(indexOfCurrentSong + 1, 0, selectedSong);
        },

        addToNextUpList(song) {
            this.nextUpList.songs.unshift(song);
        },

        isSelected(id, index) {
            return this.itemHoveredInNextUpList === index || (this.isSongSelectedInNextUpList(id))
        },

        playNextSongInNextUpList() {
            if (this.onRepeat && this.loop.value === 1) {
                this.audioPlayer.play();
            } else {
                if (this.nextUpList.songs.length > 1) {
                    if (this.random) {
                        //generate a random number
                        //set the current index of the next up list
                        this.nextUpList.currentIndex = this.generateRandomNumber(
                            0,
                            this.nextUpList.songs.length - 1,
                            this.previousNextUpListIndex
                        );

                        //set the src of the audio player
                        this.audioPlayer.src = this.nextUpList.songs[
                            this.nextUpList.currentIndex
                            ].url;
                        //set the current song
                        this.setCurrentSong(
                            this.nextUpList.songs[this.nextUpList.currentIndex]
                        );
                        //begin to play
                        this.audioPlayer.play();
                    } else {
                        /** If the current Index of the next up list is equal to the
                         * index of the last song played skip that song and add 1 */
                        if (this.nextUpList.currentIndex === this.previousNextUpListIndex) {
                            //increment the current index of the next up list by 1
                            this.nextUpList.currentIndex += 1;
                        }

                        /** If the current Index of the next up list is greater or equal
                         * to the length of the next up list songs (the index is out of range)
                         reset the index to 0. It could also mean that the next up list is at its end. */
                        if (this.nextUpList.currentIndex >= this.nextUpList.songs.length) {
                            if (this.onRepeat && this.loop.value === "all") {
                                //if repeat is on then replay from the top
                                this.nextUpList.currentIndex = 0;
                            } else {
                                return;
                            }
                        }

                        this.audioPlayer.src = this.nextUpList.songs[
                            this.nextUpList.currentIndex
                            ].url;
                        this.setCurrentSong(
                            this.nextUpList.songs[this.nextUpList.currentIndex]
                        );
                        this.audioPlayer.play();
                        this.nextUpList.currentIndex++;
                    }
                } else {
                }
            }
        },

        removeFromNextUpList(id) {
            let index = this.nextUpList.songs.findIndex(obj => obj.id === id);

            if (index !== -1) {
                this.nextUpList.songs.splice(index, 1);
            }
        },

        shuffleToggle() {
            //shuffle the next up list songs and rearrange
            this.nextUpList.songs = this.shuffleArray(this.nextUpList.songs);
            this.isShuffled = !this.isShuffled
            //reset the next up list index when changed and rest the previous next up list index
            this.nextUpList.currentIndex = this.getObjectIndexFromArray(
                this.currentSong,
                this.nextUpList.songs
            );
            this.previousNextUpListIndex = this.nextUpList.currentIndex;
        },

        // todo Probably should replace all this with a CSS only hover-slide..
        // The artist(s) names when they're longer than the div they're in
        // this is going to be moving back - forward - original position on hover.
        // Once activated the slide animation, next hovers will slow it down for user to click
        async artistNameSlide() {
            let span = document.querySelector('.vp-artist-inner-span');
            let artistNameSpanWidth = span.offsetWidth;
            // Max(ish) width of artist div in pixels
            if (artistNameSpanWidth > 172) {
                // keep it half to create a bounce effect
                const slideDistance = parseInt(artistNameSpanWidth / 2);
                const hoverDiv = document.querySelector('.vp-artist-inner-3');
                if (!this.artistNameSlideLoopInitiated) {
                    // To ensure loop is not called more than once when hovered again
                    this.artistNameSlideLoopInitiated = true;
                    // moving slowly to left with --trans-x
                    for (let i = 0; i < slideDistance; i++) {
                        await sleep(this.artistNameSlideLoopPaused ? 100 : 20)
                        hoverDiv.style.setProperty('--trans-x', "-" + i + "px");
                    }
                    // moving slowly back to righ, eventually to the orignal place
                    for (let i = -slideDistance; i <= 0; i++) {
                        await sleep(this.artistNameSlideLoopPaused ? 100 : 20)
                        hoverDiv.style.setProperty('--trans-x', +i + "px");
                    }
                    this.artistNameSlideLoopInitiated = false;
                }
            }
        },

        clearNextUp() {
            // todo check if currently a song is being played first.
            // todo currently mounted() will put the first song
            // todo from the passed array into the Next Up
            this.nextUpList.songs = this.nextUpList.songs.filter(obj => obj.id === this.currentSong.id);
        },

        /** Helper methods
         * these methods are usually used within other methods*/
        formatTime(secs) {
            let minutes = Math.floor(secs / 60) || 0;
            let seconds = Math.floor(secs - minutes * 60) || 0;
            return minutes + ":" + (seconds < 10 ? "0" : "") + seconds;
        },

        setAudio(song) {
            this.audioPlayer.src = song;
        },

        load() {
            if (this.audioPlayer.readyState >= 2) {
                this.isLoaded = true;
                this.durationSeconds = parseInt(this.audioPlayer.duration);
            } else {
                throw new Error("Failed to load sound file.");
            }
        },


        closeNextUp() {
            this.showNextUpList = false
        },


        containsObjectWithSameId(obj, list) {
            let i;
            for (i = 0; i < list.length; i++) {
                if (list[i].id === obj.id) {
                    return true;
                }
            }
        },

        getObjectIndexFromArray(obj, list) {
            //this function just returns the index of the item with the id
            let i;
            for (i = 0; i < list.length; i++) {
                if (list[i].id === obj.id) {
                    return i;
                }
            }
        },

        setCurrentSong(song) {
            this.currentSong.id = song.id;
            this.currentSong.title = song.title;
            this.currentSong.artist = song.artist;
            this.currentSong.album = song.album;
            this.currentSong.url = song.url;
            this.currentSong.cover_art_url = song.cover_art_url;

            this.previousNextUpListIndex = this.nextUpList.currentIndex;
        },

        generateRandomNumber(min, max, except) {
            let num = null;
            num = Math.floor(Math.random() * (max - min + 1)) + min;
            return num === except ? this.generateRandomNumber(min, max, except) : num;
        },

        shuffleArray(array) {
            let ctr = array.length;
            let temp;
            let index;

            // While there are elements in the array
            while (ctr > 0) {
                // Pick a random index
                index = Math.floor(Math.random() * ctr);
                // Decrease ctr by 1
                ctr--;
                // And swap the last element with it
                temp = array[ctr];
                array[ctr] = array[index];
                array[index] = temp;
            }
            return array;
        },

        artistNameSlideHovered() {
            this.artistNameSlideLoopPaused = true
        },
        artistNameSlideLeft() {
            this.artistNameSlideLoopPaused = false
        },

        volumeChange(e) {
            this.audioPlayer.volume = e.currentTarget.value / 100
        },
        hideVolumeSlider() {
            this.volumeSliderShown = false;
        },

        showVolumeSlider() {
            this.volumeSliderShown = true;
        },
        onKeyPress(e) {
            if (e.keyCode === 32) {
                if (this.isPlaying) {
                    this.pause();
                    return
                }
                this.playCurrentSong();
            }
        },
    },

    computed: {
        currentPlayedTime() {
            return this.formatTime(this.currentSeconds);
        },
        duration() {
            return this.formatTime(this.durationSeconds);
        },
        progressPercentage() {
            return parseInt(this.currentSeconds / this.durationSeconds * 100);
        },
        muted() {
            return this.volume / 100 === 0;
        }
    }
}
// Used in slowing down the for loop in the artist name slide
const sleep = (ms) => new Promise((res) => setTimeout(res, ms));
</script>

<style>
.vp-song .vp-wrapper {
    position: relative;
    width: 100%;
    height: auto;
}

.vp-song .vp-wrapper .vp-overlay-play {
    cursor: pointer;
    position: absolute;
    width: 40px;
    height: 40px;
    background-color: #fd6a02;
    bottom: 5px;
    right: 5px;
    border-radius: 50%;
}

.vp-song .vp-wrapper .vp-overlay-play i {
    font-size: 1.8em;
    line-height: 40px;
}

.vp-player {
    width: 100%;
    bottom: 0;
    position: fixed;
}

.vp-player-bar {
    min-height: 80px;
}

.vp-next-up {
    position: relative;
    z-index: 2;
}

.vp-next-up .show {
    overflow: auto;
    opacity: 0;
}

.vp-next-up .vp-wrap {
    transform: translate3d(0, 0, 0);
    opacity: 1;
    width: 480px;
    height: 600px;
    max-height: calc(100vh - 12rem);
    position: absolute;
    bottom: 10px;
    right: calc(1rem - 1px);
    border-radius: 5px;
    border: 1px solid rgba(0, 0, 0, .05);
    box-shadow: 0 1px 15px rgba(0, 0, 0, .1);
    background-clip: padding-box;
    pointer-events: auto;
    font-size: 14px;
    transition: opacity .2s, transform .2s cubic-bezier(.25, .8, .25, 1);
    display: flex;
    flex-direction: column;
}

.vp-next-up-header {
    padding: 1rem 1rem;
    border-bottom: 1px solid rgba(0, 0, 0, .05);
    display: flex;
    align-items: center;
}

.vp-next-up-clear-button {
    padding: 0.125rem 0.5rem;
    background-color: transparent;
    color: inherit;
    border-radius: 2px;
    border: 1px solid rgba(0, 0, 0, .05);
    font-size: 12px;
}

.vp-next-up-content {
    flex: 1;
    overflow-y: auto;
    -webkit-overflow-scrolling: touch;
}


.vp-song-wrap {
    display: -ms-flexbox;
    display: flex;
    padding: 0.5rem 1rem;
    cursor: pointer;
    position: relative;
    align-items: center;
}

.vp-song-art {
    border-radius: 2px;
    background-size: cover;
    background-position: 50% 50%;
    background-color: rgba(110, 120, 130, .1);
    flex: 0;
    width: 36px;
    height: 36px;
    min-width: 36px;
    cursor: pointer;
}

.vp-selected {
    background-color: rgba(150, 155, 160, .2);
    border-bottom-color: transparent;
}

.vp-song-art button {
    width: 20px;
    height: 20px;
    display: flex;
    margin: 8px;
    background: #fff !important;
    border: none;
    border-radius: 100%;
    padding: 0 !important;
    position: relative;
    vertical-align: middle;
    transition: box-shadow .4s cubic-bezier(.25, .8, .25, 1), transform .4s cubic-bezier(.25, .8, .25, 1);
    outline: 0 !important;
    justify-content: center;
    align-items: center;
}

.vp-song-art .vp-button-icon {
    height: 16px;
    width: 16px;
}

.vp-song-details {
    line-height: 1.4;
    padding: 0 1rem;
    flex: 1;
    width: 1%;
    cursor: pointer;
}

/** Vue transition CSS
    This is for the Next Up List */

.slide-fade-enter-active {
    transition: all 0.3s ease-out;
}

.slide-fade-leave-active {
    transition: all 0.3s cubic-bezier(1, 0.5, 0.8, 1);
}

.slide-fade-enter-from,
.slide-fade-leave-to {
    transform: translateX(20px);
    opacity: 0;
}

.vp-song-title {
    display: -webkit-box;
    max-height: none;
    -webkit-box-orient: vertical;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: normal;
    -webkit-line-clamp: 1;
    word-break: break-all;
    line-height: 19px;
}

.vp-close {
    display: none;
}

.vp-song-wrap .vp-close-button {
    background: 0 0;
    line-height: 1;
    font-weight: 400;
    font-size: 1rem;
    border: none;
    cursor: pointer;
    height: auto;
    padding: 0.5rem;
    margin-bottom: 3px;
}

.vp-song-like-button-in-next-up {
    height: auto;
    line-height: 0;
    display: inline-flex;
    align-items: center;
    padding: 0.5rem;
    min-width: 2rem;
    color: inherit;
    background-color: transparent;
    border: none;
    outline: 0;
}


.vp-song-like-button-left {
    padding: 10px;
    margin-left: 5px;
    min-width: 45px;
    border: none;
    outline: 0;
}

.vp-song-like-button-right {
    padding: 10px;
    margin-left: 5px;
    min-width: 45px;
    border: none;
    outline: 0;
    display: none;
}

.vp-next-up .vp-wrap::-webkit-scrollbar {
    width: 5px;
}

.vp-next-up .vp-wrap::-webkit-scrollbar-track {
    -webkit-box-shadow: inset 0 0 6px #919191;
    border-radius: 0;
}

.vp-next-up .vp-wrap::-webkit-scrollbar-thumb {
    border-radius: 0;
    -webkit-box-shadow: inset 0 0 6px rgba(238, 238, 238, 0.5);
}

.vp-next-up .vp-player {
    position: relative;
    z-index: 2;
    width: 100%;
    height: 80px;;
}

.vp-album-art {
    max-width: 48px;
    height: 48px;
    margin-left: 18px;
    margin-right: 18px;
}

.vp-album-art .vp-img {
    width: 100%;
    height: 100%;
    border-radius: 4px;
}

.vp-main-controls .i {
    cursor: pointer !important;
}

.vp-main-controls .vp-controls {
    width: 50%;
    height: 50px;
    display: flex;
    justify-content: space-around;
    align-items: center;
}

.vp-main-controls .vp-controls .i {
    font-size: 1.8em;
}

.vp-play {
    min-width: 40px;
    height: 28px;
}

.vp-skip-backward {
    min-width: 20px;
    height: 4px;
}

.vp-skip-forward {
    min-width: 20px;
    height: 4px;
}

.vp-play .i {
    line-height: 40px;
    text-align: center;
}

.vp-seek {
    width: 100%;
}

.vp-progress-container {
    position: relative;
    height: 20px;
    width: auto;
    display: flex;
    align-items: center;
}

.vp-progress-container .vp-progress {
    background-color: rgba(0, 0, 0, 0.05);
    height: 4px;
    width: 100%;
    margin: 0;
    padding: 0 2px;
    border-radius: 0;
    display: flex;
    align-items: center;
    box-shadow: 0 1px 1px rgba(0, 0, 0, .1);
}


.vp-progress-container .vp-progress .vp-bar {
    height: 4px;
    top: 1px;
    position: relative;
}

.vp-extra-controls {
    display: flex;
    flex-direction: row-reverse;
    align-items: center;
    margin-right: 18px;
    text-align: -webkit-center;
}

.vp-extra-controls .vp-like {
    padding-left: 8px;
    height: 25px;
}

.vp-like .i {
    font-size: 1em;
    line-height: 25px;
}

.vp-extra-controls .vp-repeat .vp-repeat-info {
    background-color: red;
    width: 12px;
    height: 12px;
    border-radius: 9px;
    position: absolute;
    font-size: 6px;
    line-height: 12px;
    text-align: center;
    right: -9px;
    top: -5px;
    color: #fff;
    text-transform: capitalize;
    letter-spacing: 1px;
}

.vp-rotate-down {
    -webkit-transition: -webkit-transform .8s ease-in-out;
    -ms-transition: -ms-transform .8s ease-in-out;
    transition: transform .8s ease-in-out;
    transform: rotate(90deg);
    -ms-transform: rotate(90deg);
}

.vp-rotate-up {
    -webkit-transition: -webkit-transform .4s ease-in-out;
    -ms-transition: -ms-transform .4s ease-in-out;
    transition: transform .4s ease-in-out;
    transform: rotate(180deg);
    -ms-transform: rotate(180deg);
}

.vp-title-and-artist {
    max-width: 160px;
}

.vp-artist {
    min-width: 0;
    width: 100%;
}

.vp-artist-inner-1 {
    margin-left: -6px;
    margin-right: -6px;
    -webkit-mask-image: linear-gradient(90deg, transparent 0, #000 6px, #000 calc(100% - 12px), transparent);
    overflow: hidden;
    position: relative;
}

.vp-artist-inner-2 {
    overflow: hidden;
    text-overflow: ellipsis;
}

.vp-artist-inner-3 {
    --trans-x: 0px;
    display: -webkit-box;
    display: -ms-flexbox;
    display: flex;
    padding-inline-end: 12px;
    padding-inline-start: 6px;
    -webkit-transform: translateX(var(--trans-x));
    transform: translateX(var(--trans-x));
    white-space: nowrap;
    width: -moz-fit-content;
    width: fit-content;
}

.vp-artist-inner-4 {
    box-sizing: border-box;
    -webkit-tap-highlight-color: transparent;
    margin-block: 0;
    font-size: 12px;;
    font-weight: 400;
    grid-column-start: badges;
    min-width: 0;
    width: 100%;
}

.vp-extra-controls .vp-next-up-icon {
    height: 25px;
    min-width: 25px;
    margin-left: 18px;
}

.vp-extra-controls .vp-volume {
    min-width: 25px;
    margin-bottom: 4px;
}

.vp-extra-controls .vp-shuffle-icon {
    min-width: 25px;
    margin-left: 18px;

}

.vp-extra-controls .vp-repeat {
    position: relative;
    min-width: 25px;
    margin-left: 18px;
}

.vp-height-enter-active {
    animation: bounce-in 0.5s;
}

.vp-height-leave-active {
    animation: bounce-in 0.5s reverse;
}

.vp-slider-container {
    bottom: 15px;
    margin-left: -8px;
    position: absolute;
    transform: rotate(-90deg);
    height: 40px;
    background: #1f2937;
    display: flex;
    border-radius: 10px;
    transform-origin: left top;
    align-items: center;
    justify-content: center;
}


input[type="range"].vp-bar {
    width: 100%;
}

input[type="range"] {
    -webkit-appearance: none !important;
    background: #ffffff;
    border: none;
    border-radius: 20px;
    outline: none;
    width: 75%;
    height: 3px;
}

input[type="range"]::-webkit-slider-thumb {
    -webkit-appearance: none !important;
    width: 12px;
    height: 12px;
    background: rgb(255, 255, 255);
    border: 2px solid #ffffff;
    border-radius: 50%;
    cursor: pointer;
}

input[type="range"]::-webkit-slider-thumb:hover {
    background: rgba(47, 37, 37, 0.87) !important;
}

.vp-volume-slider-icon {
    padding-top: 5px;
}

.vp-skip-backward {
    margin-right: 2rem;
}

.vp-skip-forward {
    margin-left: 2rem;
}

@media (max-width: 1023px) {
    .vp-song-like-button-right {
        display: block;
    }

    .vp-song-like-button-left {
        display: none;
    }

    .vp-volume {
        display: none;
    }

    .vp-artist {
        width: 80%;
    }
}

@media (max-width: 767px) {
    .vp-next-up .vp-wrap {
        width: auto;
        left: 1rem;
    }

    .vp-progress-container {
        position: absolute;
        left: 0;
        right: 0;
        top: -11px;
        bottom: auto;
        margin: 0 !important;
    }

    .vp-progress-container .vp-progress {
        padding: unset;
    }

    input[type="range"].vp-bar {
        border-radius: 0;
    }

    .vp-progress-container .vp-time {
        display: none;
    }

    .vp-song-like-button-left, .vp-song-like-button-right {
        min-width: 40px;
    }

    .vp-album-art {
        min-width: 32px;
        height: 32px;
    }

    .vp-title-and-artist {
        font-size: 12px;
    }

    .vp-extra-controls .vp-shuffle-icon {
        display: none;
    }

    .vp-extra-controls .vp-repeat {
        display: none;
    }

    .vp-main-controls {
        align-items: center;
        flex-direction: unset;
        margin-bottom: 11px;
    }

    .vp-artist {
        width: 70%;
    }

    @keyframes bounce-in {
        0% {
            max-height: 0;
        }
        100% {
            max-height: 75px;
        }
    }
}

@media (max-width: 565px) {
    .vp-skip-backward {
        margin-right: 1rem;
    }

    .vp-skip-forward {
        margin-left: 1rem;
    }

    .vp-artist {
        width: 40%;
    }
}
</style>
