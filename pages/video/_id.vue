<template>
<div>
  <v-card v-if="Object.keys(video).length">
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
          <v-chip>
            <v-avatar>
              <img :src="video.speaker[0].image" alt="trevor">
            </v-avatar>
            {{video.speaker[0].name}}
          </v-chip>
          <span>at <strong>{{ video.conference }}</strong></span>
        </div>
        <br>
        <span class="grey--text">{{moment(video.date).fromNow()}}</span><br>
        <span>{{video.description}}</span>
        <br><br>
        <div class="tags">
           <v-chip :key="tag" v-for="tag in video.tags">#{{ tag }}</v-chip>
        </div>
      </div>
    </v-card-title>
  </v-card>

  <div class="similaire container-videos">
    <div class="section-title">>_ video.getSimilaire()</div>
    <div class="similaire-holder">
      <nuxt-link :key="video._id" :to="'/video/'+ video._id" v-for="video in videos">
        <v-card>
          <v-card-title>
            <div>
              <span class="grey--text">{{moment(video.date).fromNow()}}</span><br>
              <div class="speaker-conf">
                <v-chip>
                  <v-avatar>
                    <img :src="video.speaker[0].image" alt="trevor">
                  </v-avatar>
                  {{video.speaker[0].name}}
                </v-chip>
                <span>at <strong>{{ video.conference }}</strong></span>
              </div>
              <br>
              <span>
                {{ video.title }}
                <span v-if="video.description.length > 100">...</span>
              </span>
              <br><br>
              <div class="tags">
                <v-chip :key="video._id + tag" v-for="tag in video.tags">#{{ tag }}</v-chip>
              </div>
            </div>
          </v-card-title>
        </v-card>
      </nuxt-link>
    </div>
  </div>
</div>
</template>

<script>

import moment from 'moment'
import axios from 'axios'
export default {
  head () {
    return {
      title: this.video.title || this.video.speaker.name + " at " + this.video.conference,
      meta: [
        { hid: 'description', name: 'description', content: this.video.description },
      ]
    }
  },
  fetch ({ store, params }) {
    let videoCall = axios.get('https://jstalks-d774.restdb.io/rest/data/' + params.id, {
      headers: {
        'x-apikey': '5c0319c8b83385326c1389f6'
      }
    })
    let VideosCall = axios.get('https://jstalks-d774.restdb.io/rest/data?max=6&h={"$orderby": {"date": -1}}', {
      headers: {
        'x-apikey': '5c0319c8b83385326c1389f6'
      }
    })

    return Promise.all([videoCall, VideosCall]).then(([video, videos]) => {
      store.commit('setVideo', video.data)
      store.commit('setVideos', videos.data)
    })
  },
  data() {
    return {
      video: this.$store.state.video,
      videos: this.$store.state.videos,
    }
  },
  computed: {
  },
  methods: {
    moment
  }
}
</script>


<style>
.similaire{
  margin-top: 40px;
}
.similaire-holder{
  display: flex;
  justify-content: flex-start;
  flex-wrap: wrap;

}

.similaire-holder > a{
  margin: 10px;
  width: calc(33% - 20px);
}

@media screen and (max-width: 720px) {
  .similaire-holder{

  }
  .similaire-holder > a{
    margin: 10px;
    width: 100%;
  }
}
</style>
