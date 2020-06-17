<template>
    <div class="ebook-reader">
        <div id="reader"></div>
    </div>
</template>
<script>
import { mapGetters } from 'vuex'
import Epub from 'epubjs'
global.epub = Epub

export default {
    computed: {
        ...mapGetters(['fileName'])
    },
    methods: {
        initEpub () {
            const baseUrl = 'http://192.168.31.30:8081/epub/'
            const fileUrl = baseUrl + this.fileName + '.epub'
            this.book = new Epub(fileUrl)
            this.rendition = this.book.renderTo('reader', {
                width: innerWidth,
                height: innerHeight,
                methods: 'default'
            })
            this.rendition.display()
            this.rendition.on('touchstart', event => {
                this.touchStartX = event.changedTouches[0].clientX
                this.touchStartTime = event.timeStamp
            })
            this.rendition.on('touchend', event => {
                const offsetX = event.changedTouches[0].clientX - this.touchStartX
                const time = event.timeStamp - this.touchStartTime
                if (time < 500 && offsetX > 40) {
                    this.prePage()
                } else if (time < 500 && offsetX < -40) {
                    this.nextPage()
                } else {
                    this.toggleTitleAndMenu()
                }
                // event.preventDefault()  // css touch-action解决
                event.stopPropagation()
            })
        },
        prePage () {
            if (this.rendition) {
                this.rendition.prev()
            }
        },
        nextPage () {
            if (this.rendition) {
                this.rendition.next()
            }
        },
        toggleTitleAndMenu () {}
    },
    mounted () {
        const fileName = this.$route.params.fileName.split('|').join('/')
        this.$store.dispatch('setFileName', fileName).then(() => {
            this.initEpub()
        })
    }
}
</script>
<style scoped>
* { touch-action: none; }
</style>
