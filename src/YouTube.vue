<template>
  <div :style="responsive ? undefined : wrapperStyle">
    <div ref="youtube" :style="iframeStyle"></div>
  </div>
</template>

<script lang="ts">
import { defineComponent, PropType } from 'vue'
import getYouTubeID from 'get-youtube-id'

interface Window {
  onYouTubeIframeAPIReadyResolvers?: { (): void }[]
  onYouTubeIframeAPIReady?: { (): void }
}

export const PlayerState = {
  UNSTARTED: -1,
  ENDED: 0,
  PLAYING: 1,
  PAUSED: 2,
  BUFFERING: 3,
  CUED: 5,
}

export type SuggestedVideoQuality = YT.SuggestedVideoQuality
export type PlayerVars = YT.PlayerVars

const YouTube = defineComponent({
  name: 'YouTube',
  props: {
    src: {
      type: String as PropType<string>,
      required: true,
    },
    height: {
      type: [Number, String] as PropType<number | string>,
      default: 360,
    },
    width: {
      type: [Number, String] as PropType<number | string>,
      default: 640,
    },
    responsive: {
      type: Boolean as PropType<boolean>,
      default: false,
    },
    host: {
      type: String as PropType<string>,
      default: 'https://www.youtube.com',
    },
    vars: Object as PropType<PlayerVars>,
  },
  computed: {
    id(): string {
      return getYouTubeID(this.src) || this.src
    },
    wrapperStyle(): Record<string, string> {
      return {
        width: `${this.width}px`,
        height: `${this.height}px`,
        position: 'relative',
      }
    },
    iframeStyle(): Record<string, string> {
      if (this.responsive) {
        return {
          'aspect-ratio': '16 / 9',
          width: '100%',
        }
      }
      return this.wrapperStyle
    },
  },
  data() {
    // eslint-disable-next-line @typescript-eslint/no-empty-function
    let resolver = () => {
    }
    const promise = new Promise<void>((resolve) => {
      resolver = resolve
    })
    const data: {
      promise: Promise<void>
      resolver: () => void
      player: null | YT.Player
      initiated: boolean
      ready: boolean

    } = {
      promise,
      resolver,
      player: null,
      initiated: false,
      ready: false,
    }
    return data
  },
  mounted() {
    if (!(window as Window).onYouTubeIframeAPIReadyResolvers) {
      (window as Window).onYouTubeIframeAPIReadyResolvers = []
    }
    if (!(window as Window).onYouTubeIframeAPIReady) {
      (window as Window).onYouTubeIframeAPIReady = () => {
        // eslint-disable-next-line no-unused-expressions
        (window as Window).onYouTubeIframeAPIReadyResolvers?.forEach((resolver: Function) => {
          resolver()
        })
      }
    }
    this.promise.then(() => this.initPlayer())
    const id = 'youtube-iframe-js-api-script'
    let tag = document.getElementById(id) as HTMLScriptElement
    if (!tag) {
      // eslint-disable-next-line no-unused-expressions
      (window as Window).onYouTubeIframeAPIReadyResolvers?.push(this.resolver)
      tag = document.createElement('script')
      tag.id = id
      tag.src = 'https://www.youtube.com/iframe_api'
      const firstScriptTag = document.getElementsByTagName('script')[0]
      if (firstScriptTag && firstScriptTag.parentNode) {
        firstScriptTag.parentNode.insertBefore(tag, firstScriptTag)
      }
    } else {
      this.resolver()
    }
  },
  methods: {
    initPlayer(): void {
      this.initiated = true
      // eslint-disable-next-line no-undef
      this.player = new YT.Player(this.$refs.youtube as HTMLElement, {
        height: this.responsive ? 0 : this.height,
        width: this.responsive ? 0 : this.width,
        videoId: this.id,
        host: this.host,
        playerVars: this.vars,
        events: {
          onReady: (e) => {
            this.ready = true
            setTimeout(() => this.$emit('ready', e))
          },
          onStateChange: (e) => this.$emit('state-change', e),
          onPlaybackQualityChange: (e) => this.$emit('playback-quality-change', e),
          onPlaybackRateChange: (e) => this.$emit('playback-rate-change', e),
          onError: (e) => this.$emit('error', e),
          onApiChange: (e) => this.$emit('api-change', e),
        },
      })
    },
    /**
     * Queues a video by ID.
     *
     * @param videoId   Video ID.
     * @param startSeconds   Time from which the video should start playing.
     * @param suggestedQuality   Suggested video player quality.
     */
    cueVideoById(videoId: string, startSeconds?: number, suggestedQuality?: SuggestedVideoQuality): void {
      this.player?.cueVideoById(videoId, startSeconds, suggestedQuality)
    },

    /**
     * Loads and plays a video by ID.
     *
     * @param videoId   Video ID.
     * @param startSeconds   Time from which the video should start playing.
     * @param suggestedQuality   Suggested video player quality.
     */
    loadVideoById(videoId: string, startSeconds?: number, suggestedQuality?: SuggestedVideoQuality): void {
      this.player?.loadVideoById(videoId, startSeconds, suggestedQuality)
    },

    /**
     * Queues a video by media content URL.
     *
     * @param mediaContentUrl   Fully qualified player URL.
     * @param startSeconds   Time from which the video should start playing.
     * @param suggestedQuality   Suggested video player quality.
     */
    cueVideoByUrl(mediaContentUrl: string, startSeconds?: number, suggestedQuality?: SuggestedVideoQuality): void {
      this.player?.cueVideoByUrl(mediaContentUrl, startSeconds, suggestedQuality)
    },

    /**
     * Loads a video by media content URL.
     *
     * @param mediaContentUrl   Fully qualified player URL.
     * @param startSeconds   Time from which the video should start playing.
     * @param suggestedQuality   Suggested video player quality.
     */
    loadVideoByUrl(mediaContentUrl: string, startSeconds?: number, suggestedQuality?: SuggestedVideoQuality): void {
      this.player?.loadVideoByUrl(mediaContentUrl, startSeconds, suggestedQuality)
    },

    /**
     * Queues a playlist of videos.
     *
     * @param playlist   Video ID(s) to play.
     * @param index   Start index of the playlist, if not 0.
     * @param startSeconds   Time from which the video should start playing.
     * @param suggestedQuality   Suggested video player quality.
     */
    cuePlaylist(
      playlist: string | string[],
      index?: number,
      startSeconds?: number,
      suggestedQuality?: SuggestedVideoQuality,
    ): void {
      this.player?.cuePlaylist(playlist, index, startSeconds, suggestedQuality)
    },

    /**
     * Loads a playlist of videos.
     *
     * @param playlist   Video ID(s) to play.
     * @param index   Start index of the playlist, if not 0.
     * @param startSeconds   Time from which the video should start playing.
     * @param suggestedQuality   Suggested video player quality.
     */
    loadPlaylist(
      playlist: string | string[],
      index?: number,
      startSeconds?: number,
      suggestedQuality?: SuggestedVideoQuality,
    ): void {
      this.player?.loadPlaylist(playlist, index, startSeconds, suggestedQuality)
    },

    /**
     * Plays the currently cued/loaded video.
     */
    playVideo(): void {
      this.player?.playVideo()
    },

    /**
     * Pauses the currently playing video.
     */
    pauseVideo(): void {
      this.player?.pauseVideo()
    },

    /**
     * Stops and cancels loading of the current video.
     */
    stopVideo(): void {
      this.player?.stopVideo()
    },

    /**
     * Seeks to a specified time in the video.
     *
     * @param seconds   Time, in seconds from the beginning of the video.
     * @param allowSeekAhead   Whether the player is allowed to make a new request for unbuffered data.
     */
    seekTo(seconds: number, allowSeekAhead: boolean): void {
      this.player?.seekTo(seconds, allowSeekAhead)
    },

    /**
     * Loads and plays the next video in the playlist.
     */
    nextVideo(): void {
      this.player?.nextVideo()
    },

    /**
     * Loads and plays the previous video in the playlist.
     */
    previousVideo(): void {
      this.player?.previousVideo()
    },

    /**
     * Loads and plays the specified video in the playlist.
     *
     * @param index   Index of the video to play.
     */
    playVideoAt(index: number): void {
      this.player?.playVideoAt(index)
    },

    /**
     * Mutes the player.
     */
    mute(): void {
      this.player?.mute()
    },

    /**
     * Unmutes the player.
     */
    unMute(): void {
      this.player?.unMute()
    },

    /**
     * @returns Whether the player is muted.
     */
    isMuted(): boolean {
      return this.player ? this.player.isMuted() : false
    },

    /**
     * Sets the player volume.
     *
     * @param volume   An integer between 0 and 100.
     */
    setVolume(volume: number): void {
      this.player?.setVolume(volume)
    },

    /**
     * @returns The player's current volume, an integer between 0 and 100.
     */
    getVolume(): number {
      return this.player ? this.player.getVolume() : 0
    },

    /**
     * @returns Playback rate of the currently playing video.
     */
    getPlaybackRate(): number {
      return this.player ? this.player.getPlaybackRate() : 0
    },

    /**
     * Sets the suggested playback rate for the current video.
     *
     * @param suggestedRate   Suggested playback rate.
     */
    setPlaybackRate(suggestedRate: number): void {
      this.player?.setPlaybackRate(suggestedRate)
    },

    /**
     * @returns Available playback rates for the current video.
     */
    getAvailablePlaybackRates(): number[] {
      return this.player ? this.player.getAvailablePlaybackRates() : []
    },

    /**
     * Sets whether the player should continuously play a playlist.
     *
     * @param loopPlaylists   Whether to continuously loop playlists.
     */
    setLoop(loopPlaylists: boolean): void {
      this.player?.setLoop(loopPlaylists)
    },

    /**
     * Sets whether a playlist's videos should be shuffled.
     *
     * @param shufflePlaylist   Whether to shuffle playlist videos.
     */
    setShuffle(shufflePlaylist: boolean): void {
      this.player?.setShuffle(shufflePlaylist)
    },

    /**
     * @returns A number between 0 and 1 of how much the player has buffered.
     */
    getVideoLoadedFraction(): number {
      return this.player ? this.player.getVideoLoadedFraction() : 0
    },

    /**
     * @returns Current player state.
     */
    getPlayerState(): YT.PlayerState {
      // eslint-disable-next-line no-undef
      return this.player ? this.player.getPlayerState() : PlayerState.UNSTARTED
    },

    /**
     * @returns Elapsed time in seconds since the video started playing.
     */
    getCurrentTime(): number {
      return this.player ? this.player.getCurrentTime() : 0
    },

    /**
     * @returns Actual video quality of the current video.
     */
    getPlaybackQuality(): SuggestedVideoQuality {
      return this.player ? this.player.getPlaybackQuality() : 'default'
    },

    /**
     * Sets the suggested video quality for the current video.
     *
     * @param suggestedQuality   Suggested video quality for the current video.
     */
    setPlaybackQuality(suggestedQuality: SuggestedVideoQuality): void {
      this.player?.setPlaybackQuality(suggestedQuality)
    },

    /**
     * @returns Quality formats in which the current video is available.
     */
    getAvailableQualityLevels(): SuggestedVideoQuality[] {
      return this.player ? this.player.getAvailableQualityLevels() : []
    },

    /**
     * @returns Duration in seconds of the currently playing video.
     */
    getDuration(): number {
      return this.player ? this.player.getDuration() : 0
    },

    /**
     * @returns YouTube.com URL for the currently loaded/playing video.
     */
    getVideoUrl(): string {
      return this.player ? this.player.getVideoUrl() : ''
    },

    /**
     * @returns Embed code for the currently loaded/playing video.
     */
    getVideoEmbedCode(): string {
      return this.player ? this.player.getVideoEmbedCode() : ''
    },

    /**
     * @returns Video IDs in the playlist as they are currently ordered.
     */
    getPlaylist(): string[] {
      return this.player ? this.player.getPlaylist() : []
    },

    /**
     * @returns Index of the playlist video that is currently playing.
     */
    getPlaylistIndex(): number {
      return this.player ? this.player.getPlaylistIndex() : 0
    },
  },
  watch: {
    width() {
      this.player?.setSize(+this.width, +this.height)
    },
    height() {
      this.player?.setSize(+this.width, +this.height)
    },
    src() {
      if (this.initiated && this.player) {
        this.player.loadVideoById(this.id)
      }
    },
  },
  beforeUnmount() {
    this.player?.destroy()
  },
})

export type Methods = (typeof YouTube)['methods']

export default YouTube
</script>
