<template>
  <div class="next-tracks">
    <h1>{{ $t('next.nowPlaying') }}</h1>
    <TrackList
      :tracks="[currentTrack]"
      type="playlist"
      dbclick-track-func="none"
    />
    <div v-if="playNextTracks.length > 0">
      <h1>
        插队播放
        <button @click="player.clearPlayNextList()">清除队列</button>
      </h1>
      <TrackList
        :tracks="playNextTracks"
        type="playlist"
        :highlight-playing-track="false"
        :show-position="false"
        :enabled="false"
        dbclick-track-func="playTrackOnListByID"
        item-key="id+index"
        :extra-context-menu-item="['removeTrackFromQueue']"
      />
    </div>
    <h1>{{ $t('next.nextUp') }}</h1>
    <TrackList
      v-if="filteredTracks.length > 0"
      :tracks="filteredTracks"
      type="playlist"
      :highlight-playing-track="false"
      dbclick-track-func="playTrackOnListByID"
    />
  </div>
</template>

<script>
import { mapState, mapActions } from 'vuex';
import { getTrackDetail } from '@/api/track';
import TrackList from '@/components/TrackList.vue';

export default {
  name: 'Next',
  components: {
    TrackList,
  },
  data() {
    return {
      tracks: [],
    };
  },
  computed: {
    ...mapState(['player', 'localMusic']),
    currentTrack() {
      return this.player.currentTrack;
    },
    playerShuffle() {
      return this.player.shuffle;
    },
    filteredTracks() {
      let trackIDs = this.player.list.slice(
        this.player.current + 1,
        this.player.current + 100
      );
      return this.tracks.filter(t => trackIDs.includes(t.id));
    },
    playNextList() {
      return this.player.playNextList;
    },
    playNextTracks() {
      return this.playNextList.map(tid => {
        return this.tracks?.find(t => t.id === tid);
      });
    },
  },
  watch: {
    currentTrack() {
      this.loadTracks();
    },
    playerShuffle() {
      this.loadTracks();
    },
    playNextList() {
      this.loadTracks();
    },
  },
  mounted() {
    this.$parent.$refs.main.style.paddingBottom = '0';
    this.loadTracks();
  },
  beforeDestroy() {
    this.$parent.$refs.main.style.paddingBottom = '96px';
  },
  methods: {
    ...mapActions(['playTrackOnListByID']),
    async loadTracks() {
      // 获取播放列表当前歌曲后100首歌
      let trackIDs = this.player.list.slice(
        this.player.current + 1,
        this.player.current + 100
      );

      // 将playNextList的歌曲加进trackIDs
      trackIDs.push(...this.playNextList);

      // // 获取已经加载了的歌曲
      let loadedTrackIDs = this.tracks.map(t => t.id);

      const localTracks = this.localMusic.tracks.filter(t =>
        trackIDs.includes(t.id)
      );
      let newTracks = localTracks.filter(t => !loadedTrackIDs.includes(t.id));

      const newTrackIDs = trackIDs.filter(
        t => !localTracks.map(s => s.id).includes(t)
      );

      if (newTrackIDs.length > 0) {
        await getTrackDetail(newTrackIDs.join(',')).then(data => {
          newTracks.push(
            ...data.songs.filter(t => !loadedTrackIDs.includes(t.id))
          );
        });
      }
      newTracks = newTracks
        .filter(t => trackIDs.includes(t.id))
        .sort((a, b) => trackIDs.indexOf(a.id) - trackIDs.indexOf(b.id));
      this.tracks.push(...newTracks);
    },
    scrollTo(top) {
      this.$parent.$refs.main.scrollTo({ top, behavior: 'smooth' });
    },
  },
};
</script>

<style lang="scss" scoped>
h1 {
  margin-top: 36px;
  margin-bottom: 18px;
  cursor: default;
  color: var(--color-text);
  display: flex;
  justify-content: space-between;
  button {
    color: var(--color-text);
    border-radius: 8px;
    padding: 0 14px;
    display: flex;
    justify-content: center;
    align-items: center;
    transition: 0.2s;
    opacity: 0.68;
    font-weight: 500;
    &:hover {
      opacity: 1;
      background: var(--color-secondary-bg);
    }
    &:active {
      opacity: 1;
      transform: scale(0.92);
    }
  }
}
</style>
