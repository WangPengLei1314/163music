<template>
    <div class="player-box">
      <!-- <audio :src="songUrl" muted autoplay></audio> -->
      <audio :src="songUrl" controls></audio>
    </div>
</template>
<script>
import { mapGetters } from 'vuex';
import eventbus from '../javascript/eventbus';

export default {
    data() {
        return {
            audio: undefined,
        };
    },
    computed: {
        ...mapGetters(['isPlay', 'songUrl', 'currentTime']),
    },
    watch: {
        songUrl(nval) {
            // url updated
            if (nval) {
                /**
                 * @see https://developers.google.com/web/updates/2017/09/autoplay-policy-changes#webaudio
                 */
                // reset song currentTime
                this.$store.commit('updateCurrentTime', 0);
                // this.play();
            }
        },
        isPlay(nval) {
            // flag updated
            if (nval) {
                this.play();
            } else {
                this.pause();
            }
        },
        currentTime(nval, oval) {
            if (Math.abs(nval - oval) > 1) {
                this.audio.currentTime = nval;
            }
        },
    },
    methods: {
        play() {
            return this.audio.play().then(() => {}).catch((e) => {
                console.error(e.message);
            });
        },
        pause() {
            return this.audio.pause();
        },
    },
    mounted() {
        const vm = this;
        this.audio = this.$el.querySelector('audio');

        /**
         * @todo
         */
        this.audio.oncanplay = function oncanplay() {
            // listen audio is ready
            vm.$store.commit('updateDuration', parseInt(this.duration, 10));
            if (vm.isPlay) {
                vm.play();
            } else {
                vm.pause();
            }
        };

        this.audio.addEventListener('timeupdate', (event) => {
            vm.$store.commit('updateCurrentTime', parseInt(event.currentTarget.currentTime, 10));
        }, false);

        this.audio.addEventListener('ended', () => {
            // 触发结束事件
            eventbus.$emit('songEnd');
        });
    },
};
</script>
<style lang="scss" scoped>
.player-box {
    display: none;
}
</style>
