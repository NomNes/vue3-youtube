# vue3-youtube
Wrapper for [YouTube IFrame Player API](https://developers.google.com/youtube/iframe_api_reference)

## Installation
```bash
npm install vue3-youtube
# or
yarn add vue3-youtube
```

## Usage
Browser:
```html
<script src="dist/vue3-youtube.umd.min.js"></script>
```
Global registration:
```js
import Vue from 'vue'
import YouTube from 'vue3-youtube'

Vue.component('YouTube', YouTube)
```
or locally:
```vue
<template>
    <YouTube 
        src="https://www.youtube.com/watch?v=jNQXAC9IVRw" 
        @ready="onReady"
        ref="youtube" />
</template>

<script>
import { defineComponent } from 'vue'
import YouTube from 'vue3-youtube'

export default defineComponent({
    components: { YouTube },
    methods: {
        onReady() {
            this.$refs.youtube.playVideo()
        },
    },
})
</script>
```

### Props
| Name   | Type           | Default | Description                                                                              |
|--------|----------------|---------|------------------------------------------------------------------------------------------|
| src    | String         |         | Youtube video link or ID. Required.                                                      |
| width  | Number, String | 640     | Player width.                                                                            |
| height | Number, String | 360     | Player height.                                                                           |
| host   | String         |         | Points host to correct origin for CORS.                                                  |
| vars   | Object         |         | [Player parameters](https://developers.google.com/youtube/player_parameters#Parameters). |

### Events
- ready
- state-change
- playback-quality-change
- playback-rate-change
- error
- api-change

### Methods
[https://developers.google.com/youtube/iframe_api_reference#Functions](https://developers.google.com/youtube/iframe_api_reference#Functions)
