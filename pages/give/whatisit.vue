<template>
  <div>
    <b-row class="m-0">
      <b-col cols="12" lg="6" class="p-0" offset-lg="3">
        <WizardProgress :active-stage="2" />
        <h1 class="text-center">
          Ok, now tell us about your item
        </h1>
        <ul v-for="(id, index) in ids" :key="'post-' + id" class="p-0 pt-1 list-unstyled">
          <li class="p-0">
            <b-card no-body>
              <b-card-body class="pt-0 pb-1">
                <PostMessage :id="id" type="Offer" />
              </b-card-body>
              <b-card-footer v-if="index === ids.length - 1" class="d-flex justify-content-between p-0 pt-1">
                <Postcode
                  :value="postcode ? postcode.name : null"
                  :find="false"
                  size="md"
                  class="d-inline"
                  @selected="postcodeSelect"
                />
                <ComposeGroup class="cg" />
              </b-card-footer>
            </b-card>
          </li>
        </ul>
        <div class="d-flex justify-content-end ml-1 mr-1">
          <b-btn v-if="ids.length > 1" variant="white" class="mr-1" @click="deleteItem">
            <v-icon name="trash-alt" />&nbsp;Delete last item
          </b-btn>
          <b-btn v-if="ids.length < 6" variant="primary" class="" @click="addItem">
            <v-icon name="plus" />&nbsp;Add another item
          </b-btn>
        </div>
        <transition name="fade">
          <b-row v-if="valid">
            <b-col cols="12" md="6" offset-md="3" class="text-center pt-2">
              <b-btn variant="success" size="lg" block :disabled="uploadingPhoto" @click="next">
                Next <v-icon name="angle-double-right" />
              </b-btn>
            </b-col>
          </b-row>
          <b-row v-else>
            <b-col cols="12" md="6" offset-md="3" class="text-center pt-2">
              <NoticeMessage variant="info">
                Please add the item name, and either a description or a photo.
              </NoticeMessage>
            </b-col>
          </b-row>
        </transition>
        <b-row>
          <b-col class="text-muted small pl-0 pt-1 text-center">
            We may show this post, but not your email address, to people who are not yet members of Freegle.
            This helps the community grow by showing people what's happening and encouraging them to join.
          </b-col>
        </b-row>
      </b-col>
      <b-col cols="0" md="3" />
    </b-row>
  </div>
</template>
<script>
import NoticeMessage from '../../components/NoticeMessage'
import loginOptional from '@/mixins/loginOptional.js'
import buildHead from '@/mixins/buildHead.js'

const PostMessage = () => import('~/components/PostMessage')
const Postcode = () => import('~/components/Postcode')
const ComposeGroup = () => import('~/components/ComposeGroup')
const WizardProgress = () => import('~/components/WizardProgress')

export default {
  components: {
    NoticeMessage,
    PostMessage,
    Postcode,
    ComposeGroup,
    WizardProgress
  },
  mixins: [loginOptional, buildHead],
  data: function() {
    return {}
  },
  computed: {
    uploadingPhoto() {
      return this.$store.getters['compose/getUploading']
    },
    postcode() {
      return this.$store.getters['compose/getPostcode']
    },
    ids() {
      const messages = Object.values(this.$store.getters['compose/getMessages'])

      let ids = []
      for (const message of messages) {
        if (message.id && message.type === 'Offer') {
          ids.push(message.id)
        }
      }

      if (ids.length === 0) {
        ids = [-1]
      }

      return ids
    },

    valid() {
      const messages = Object.values(this.$store.getters['compose/getMessages'])
      let valid = false

      if (messages && messages.length) {
        valid = true

        for (const message of messages) {
          if (this.ids.indexOf(message.id) !== -1 || !message.id) {
            const atts = Object.values(
              this.$store.getters['compose/getAttachments'](message.id)
            )

            // A message is valid if there is an item, and either a description or a photo.
            if (
              !message.item ||
              !message.item.trim() ||
              ((!message.description || !message.description.trim()) &&
                !atts.length)
            ) {
              valid = false
            }
          }
        }
      }

      return valid
    }
  },
  methods: {
    deleteItem() {
      this.$store.dispatch('compose/clearMessage', {
        id: this.ids[this.ids.length - 1]
      })
    },
    addItem() {
      // Find a new id.
      let nextId = -1
      for (const id of this.ids) {
        nextId = Math.min(id, nextId)
      }

      this.$store.dispatch('compose/setMessage', {
        id: --nextId,
        item: null,
        description: null,
        type: 'Offer'
      })
    },
    next() {
      const currentpc = this.$store.getters['compose/getPostcode']

      if (currentpc) {
        // We shouldn't be able to progress if we didn't have a postcode.
        this.$router.push('/give/whoami')
      }
    },
    postcodeSelect(pc) {
      const currentpc = this.$store.getters['compose/getPostcode']

      if (!currentpc || currentpc.id !== pc.id) {
        // The postcode has genuinely changed or been set for the first time.  We don't want to go through this code
        // if the postcode is the same, otherwise we'll reset the group (which might have been changed from the first,
        // for example in the give flow if you choose a different group.
        this.$store.dispatch('compose/setPostcode', pc)

        // If we don't have a group currently which is in the list near this postcode, choose the closest.  That
        // allows people to select further away groups if they wish.
        const groupid = this.$store.getters['compose/getGroup']

        if (pc && pc.groupsnear) {
          let found = false
          for (const group of pc.groupsnear) {
            if (parseInt(group.id) === parseInt(groupid)) {
              found = true
            }
          }

          if (!found) {
            this.group = pc.groupsnear[0].id
          } else {
            this.group = groupid
          }

          this.$store.dispatch('compose/setGroup', this.group)
        }
      }
    }
  },

  head() {
    return this.buildHead(
      'OFFER',
      'OFFER something to people nearby and see who wants it'
    )
  }
}
</script>
<style scoped>
.cg {
  flex-basis: 25%;
  flex-grow: 1;
}
</style>
