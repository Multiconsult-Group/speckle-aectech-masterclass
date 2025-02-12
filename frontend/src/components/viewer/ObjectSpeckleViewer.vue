<template lang="html">
  <v-card :class="`my-1 pa-0 ${localExpand ? 'elevation-3' : 'elevation-0'} my-0`">
    <v-card-title v-if="object">
      <v-chip color="" @click="toggleLoadExpand">
        <v-icon small class="mr-2">mdi-code-braces</v-icon>
        {{ keyName }}
        <span class="caption ml-2">
          {{ object.data.speckle_type ? object.data.speckle_type : 'Referenced Object' }}
        </span>
        <v-icon small class="ml-2">
          {{ localExpand ? 'mdi-minus' : 'mdi-plus' }}
        </v-icon>
      </v-chip>
      <v-btn
        icon
        small
        :href="`${serverUrl}/${streamId}/objects/${value.referencedId}`"
        target="_blank"
      >
        <v-icon small>mdi-open-in-new</v-icon>
      </v-btn>
    </v-card-title>
    <v-card-title v-if="!object">
      <v-chip color="" @click="toggleLoadExpand">
        <v-icon small class="mr-2">mdi-code-braces</v-icon>
        {{ keyName }}
        <span class="caption ml-2">Referenced Object</span>
        <v-icon small class="ml-2">
          {{ localExpand ? 'mdi-minus' : 'mdi-plus' }}
        </v-icon>
      </v-chip>
      <v-btn
        icon
        small
        target="_blank"
        :href="`${serverUrl}/streams/${streamId}/objects/${value.referencedId}`"
      >
        <v-icon small>mdi-open-in-new</v-icon>
      </v-btn>
    </v-card-title>
    <v-card-text v-if="localExpand" class="pr-0 pl-3">
      <v-skeleton-loader
        v-if="loading"
        type="list-item-three-line, list-item-three-line"
      ></v-skeleton-loader>
      <component
        :is="entry.type"
        v-for="(entry, index) in objectEntries"
        :key="index"
        :key-name="entry.key"
        :value="entry.value"
        :stream-id="streamId"
      ></component>
    </v-card-text>
  </v-card>
</template>
<script>
import {getObject} from "@/speckleUtils";

export default {
  name: 'ObjectSpeckleViewer',
  components: {
    ObjectListViewer: () => import('./ObjectListViewer'),
    ObjectSimpleViewer: () => import('./ObjectSimpleViewer'),
    ObjectValueViewer: () => import('./ObjectValueViewer')
  },
  props: {
    expand: {
      type: Boolean,
      default: false
    },
    value: {
      type: Object,
      default: null
    },
    keyName: {
      type: String,
      default: null
    },
    streamId: {
      type: String,
      default: null
    }
  },
  data() {
    return {
      localExpand: false,
      loading: true,
      object: null,
      serverUrl: process.env.VUE_APP_SERVER_URL
    }
  },
  computed: {
    objectEntries() {
      if (!this.object) return []
      let entries = Object.entries(this.object.data)
      let arr = []
      for (let [key, val] of entries) {
        if (key.startsWith('__')) continue
        if (key[0] === '@') key = key.substring(1)
        if (key === 'totalChildrenCount') key = 'total children count'
        if (key === 'speckle_type') key = 'speckle type'

        if (Array.isArray(val)) {
          arr.push({
            key,
            value: val,
            type: 'ObjectListViewer',
            description: `List (${val.length} elements)`
          })
        } else if (typeof val === 'object' && val !== null) {
          if (val.speckle_type && val.speckle_type === 'reference') {
            arr.push({
              key,
              value: val,
              type: 'ObjectSpeckleViewer'
            })
          } else {
            arr.push({
              key,
              value: val,
              type: 'ObjectSimpleViewer'
            })
          }
        } else {
          arr.push({
            key,
            value: val,
            type: 'ObjectValueViewer'
          })
        }
      }
      arr.sort((a, b) => {
        if (a.type === b.type) return 0
        if (a.type === 'ObjectValueViewer') return -1
        return 0
      })
      return arr
    }
  },
  async mounted() {
    this.loading = true
    this.localExpand = this.expand
    if(!this.localExpand){
      var res = await getObject(this.streamId,this.value.referencedId)
      delete res.data.stream.object.data.__closure
      this.object = res.data.stream.object
    }
    this.loading = false
  },
  methods: {
    toggleLoadExpand() {
      this.localExpand = !this.localExpand
    }
  }
}
</script>
