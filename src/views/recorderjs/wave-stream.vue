<template>
  <div class="wave-stream">
    <svg preserveAspectRatio="none" id="visualizer" version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
    <defs>
        <mask ref="mask" id="mask">
            <g id="maskGroup">
          </g>
        </mask>
        <linearGradient id="gradient" x1="0%" y1="0%" x2="0%" y2="100%">
            <stop offset="0%" style="stop-color:#ff0a0astop-opacity:1" />
            <stop offset="20%" style="stop-color:#f1ff0astop-opacity:1" />
            <stop offset="90%" style="stop-color:#d923b9stop-opacity:1" />
            <stop offset="100%" style="stop-color:#050d61stop-opacity:1" />
        </linearGradient>
    </defs>
    <rect x="0" y="0" width="100%" height="100%" fill="url(#gradient)" mask="url(#mask)"></rect>
</svg>
  </div>
</template>

<script>
export default {
  props: {
    stream: {
      required: true
    }
  },
  watch: {
    stream(val) {
      this.drawWave(val)
    }
  },
  methods: {
    drawWave(stream) {
      // Audio stops listening in FF without // window.persistAudioStream = stream
      // window.persistAudioStream = stream

      let audioContent = new AudioContext()
      let audioStream = audioContent.createMediaStreamSource(stream)
      let analyser = audioContent.createAnalyser()
      audioStream.connect(analyser)
      analyser.fftSize = 1024

      let frequencyArray = new Uint8Array(analyser.frequencyBinCount)
      // visualizer.setAttribute('viewBox', '0 0 255 255')

      // Through the frequencyArray has a length longer than 255, there seems to be no
      // significant data after this point. Not worth visualizing.
      const mask = this.$refs.mask
      for (let i = 0; i < 255; i++) {
        let path = document.createElementNS('http://www.w3.org/2000/svg', 'path')
        path.setAttribute('stroke-dasharray', '4,1')
        mask.appendChild(path)
      }
      let doDraw = () => {
        requestAnimationFrame(doDraw)
        analyser.getByteFrequencyData(frequencyArray)
        let adjustedLength
        let paths = []
        for (let i = 0; i < 255; i++) {
          adjustedLength = Math.floor(frequencyArray[i]) - (Math.floor(frequencyArray[i]) % 5)
          paths[i].setAttribute('d', 'M ' + (i) + ',255 l 0,-' + adjustedLength)
        }
      }
      doDraw()
    }
  }
}
</script>

<style lang="less">
.wave-stream {
  svg{
    display: block;
    width: 100%;
    height: 100%;
    padding: 0;
    margin: 0;
    position:absolute;

  }
  path{
    stroke-linecap: square;
    stroke: white;
    stroke-width: 0.5px;
  }
}
</style>
