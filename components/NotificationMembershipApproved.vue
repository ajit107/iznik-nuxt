<template>
  <div class="clickme">
    <div class="media clickme">
      <div class="media-left">
        <div class="media-object">
          <v-icon name="info-circle" scale="2" />
        </div>
      </div>
      <div class="media-body">
        <span class="notification-title"><span>Your application was approved!</span>
          <span v-if="notification.url">
            <br>"<b class="notification__message">Visit the community <nuxt-link :to="notification.url">here</nuxt-link></b>"
          </span>
        </span>
        <div class="notification-meta">
          <abbr class="small">{{ notification.timestamp|timeago }}</abbr>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    notification: {
      type: Object,
      required: true
    }
  },
  methods: {}
}
</script>

<style scoped>
.media-object {
  width: 33px;
  height: 28px;
  padding-top: 5px;
}

.notification-title {
  max-width: 280px;
}
.notification__message {
  max-width: 200px;
  overflow-wrap: break-word;
}
</style>
