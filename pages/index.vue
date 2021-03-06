<template>
  <div class="wrapper">
    <div class="container">
      <div class="text-xs-center">
        <div class="section-title">>_ speakers.getMostPopular()</div>
        <v-chip v-for='speaker in speakers' :key="speaker.id" @click="filterBySpeaker(speaker)" @input="removeSpeaker(speaker)" :close="currentSpeakers && currentSpeakers.name === speaker.name">
          <v-avatar>
            <img :src="speaker.image" :alt="speaker.name">
          </v-avatar>
          {{speaker.name}} ({{speaker.videosCount}})
        </v-chip>
      </div>
    </div>
    <div class='container container-videos'>
      <div style="margin-bottom: 40px" :key="video._id" v-for="video in videos">
        <nuxt-link :to="'/video/'+ video._id">
          <v-card>
            <iframe
              width=""
              height=""
              :src="video.url"
              frameborder="0"
              allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture"
              allowfullscreen></iframe>
              <v-card-title>
              <div>
                <v-list-tile-title>{{ video.title }}</v-list-tile-title>
                <div class="speaker-conf">
                  <v-chip @click="filterBySpeaker(video.speaker[0])" @input="removeSpeaker(video.speaker[0])" :close="currentSpeakers && currentSpeakers.name === video.speaker[0].name">
                    <v-avatar>
                      <img :src="video.speaker[0].image" alt="trevor">
                    </v-avatar>
                    {{video.speaker[0].name}}
                  </v-chip>
                  <span>at <strong>{{ video.conference }}</strong></span>
                </div>
                <span class="grey--text">{{moment(video.date).fromNow()}}</span><br>
                <br>
                <span>{{video.description}}</span>
                <br><br>
                <div class="tags">
                   <v-chip :key="video._id + tag" v-for="tag in video.tags">#{{ tag }}</v-chip>
                </div>
              </div>
            </v-card-title>
          </v-card>
        </nuxt-link>
      </div>
      <div class="loading" @click="loadMore" v-if="!thatsAll">
        <v-btn color="info">_ videos.load() <span v-if="loading">...</span></v-btn>
      </div>

      <div class="thatsAll  " v-if="thatsAll">
        😞 That's all for now
      </div>
    </div>
  </div>
</template>

<script>
import moment from 'moment'
import axios from 'axios'
export default {
  fetch ({ store }) {
    let speakersCall = axios.get('https://jstalks-d774.restdb.io/rest/speakers?h={"$orderby": {"videosCount": -1}}&max=6', {
      headers: {
        'x-apikey': '5c0319c8b83385326c1389f6'
      }
    })
    let VideosCall = axios.get('https://jstalks-d774.restdb.io/rest/data?h={"$orderby": {"date": -1}}&max=4', {
      headers: {
        'x-apikey': '5c0319c8b83385326c1389f6'
      }
    })
    return Promise.all([speakersCall, VideosCall]).then(([speakers, videos]) => {
      store.commit('setSpeakers', speakers.data)
      store.commit('setVideos', videos.data)
    })
  },
  data() {
    return {
      currentSpeakers: undefined,
      page: 1,
      loading: false,
      thatsAll: false
    }
  },
  computed: {
    speakers () {
      return this.$store.state.speakers
    },
    videos () {
      return this.$store.state.videos
    }
  },
  methods: {
    moment,
    filterBySpeaker (speaker) {
      this.loading = true
      this.thatsAll = false
      this.$store.commit('setVideos', [])
      this.currentSpeakers = (speaker)
      axios.get(`https://jstalks-d774.restdb.io/rest/data?h={"$orderby": {"date": -1}}&q={"speaker": { "name": "${speaker.name}" }}` , {
        headers: {
          'x-apikey': '5c0319c8b83385326c1389f6'
        }
      })
        .then((res) => {
          if (res.data.length === 0) this.thatsAll = true
          this.$store.commit('setVideos', res.data)
          this.loading = false
      })
    },
    removeSpeaker (speaker) {
      this.currentSpeakers = undefined
      this.loading = true
      this.thatsAll = false
      this.$store.commit('setVideos', [])
      this.page = 1
      let videosCall = axios.get('https://jstalks-d774.restdb.io/rest/data?h={"$orderby": {"date": -1}}&max=4', {
        headers: {
          'x-apikey': '5c0319c8b83385326c1389f6'
        }
      })
      videosCall.then((videos) => {
        this.$store.commit('setVideos', videos.data)
        this.loading = false
      })
    },
    loadMore () {
      if (this.thatsAll) return;
      this.page = this.page + 1
      this.loading = true
      axios.get('https://jstalks-d774.restdb.io/rest/data?h={"$orderby": {"date": -1}}&skip=' + (this.page - 1) * 4 + '&max=' + this.page * 2 , {
        headers: {
          'x-apikey': '5c0319c8b83385326c1389f6'
        }
      })
        .then((res) => {
          if (res.data.length === 0) this.thatsAll = true
          this.$store.commit('addVideos', res.data)
          this.loading = false
      })
    },
    handleScroll (e) {
      if (this.currentSpeakers) return;
      let bottomOfWindow = document.documentElement.scrollTop + window.innerHeight === document.documentElement.offsetHeight;

      if (bottomOfWindow && !this.loading && !this.thatsAll) {
        this.loadMore()
      }
    }
  },
  mounted () {
    window.addEventListener('scroll', this.handleScroll);
  },
  unmounted () {
    window.removeEventListener('scroll', this.handleScroll);
  }
}
</script>
