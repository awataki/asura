<template>
  <v-app>
    <v-app-bar app>
      <v-spacer />
      <v-text-field
        v-model="url"
        append-outer-icon="mdi-plus"
        label="Paste Youtube/Twitch URL here"
        hide-details
        @click:append-outer="addURL()"
      />
      <v-spacer />
    </v-app-bar>
    <v-main>
      <v-snackbar v-model="snackbar" :timeout="1500" top="true">
        {{ errMsg }}
        <template v-slot:action="{ attrs }">
          <v-btn dark text v-bind="attrs" @click="snackbar = false">
            Close
          </v-btn>
        </template>
      </v-snackbar>
      <v-container fluid fill-height>
        <v-row style="height: 100%">
          <v-col
            v-for="i in urls"
            :key="i"
            ref="videos"
            :cols="
              isMobile ? 12 : Math.round(12 / Math.ceil(Math.sqrt(urls.length)))
            "
          />
        </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
import Vue from 'vue';

export default Vue.extend({
  data() {
    return {
      url: '',
      urls: [],
      snackbar: false
    };
  },
  computed: {
    isMobile() {
      switch (this.$vuetify.breakpoint.name) {
        case 'xs':
          return true;
        case 'sm':
          return true;
        case 'md':
          return true;
        default:
          return false;
      }
    }
  },
  methods: {
    async addURL() {
      try {
        const url = new URL(this.url);

        // if invalid url throw exception.
        // eslint-disable-next-line no-throw-literal
        if (
          !(
            (url.host === 'www.youtube.com' && url.searchParams.has('v')) ||
            url.host === 'www.twitch.tv'
          )
        )
          // eslint-disable-next-line no-throw-literal
          throw 'invalid url';

        await this.urls.push({ src: url });
        this.url = '';
        const elm = this.$refs.videos[this.$refs.videos.length - 1];

        switch (url.host) {
          case 'www.youtube.com':
            // eslint-disable-next-line no-case-declarations
            const child = document.createElement('iframe');
            child.setAttribute('autoplay', 'true');
            child.setAttribute('frameborder', 0);
            child.setAttribute('allowfullscreen', true);
            child.setAttribute('scrolling', 'no');
            child.setAttribute('height', 378);
            child.setAttribute('width', 620);
            child.setAttribute(
              'src',
              'https://youtube.com/embed/' + url.searchParams.get('v')
            );
            elm.appendChild(child);
            break;
          case 'www.twitch.tv':
            elm.setAttribute('id', this.urls.length);
            // eslint-disable-next-line no-undef,no-new
            new Twitch.Player(this.urls.length.toString(), {
              channel: url.pathname.substr(1)
            });
            break;
        }
      } catch (e) {
        this.snackbar = true;
        this.errMsg = '正しいURLを入力してください';
      }
    }
  },
  head() {
    return {
      script: [{ src: 'https://player.twitch.tv/js/embed/v1.js' }]
    };
  }
});
</script>

<style>
.col {
  padding: 0;
}
iframe {
  width: 100%;
  height: 100%;
}
</style>
