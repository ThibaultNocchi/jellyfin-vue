<template>
  <v-list color="transparent" two-line>
    <div v-if="items.length > 0">
      <v-list-item
        v-for="(item, index) in items"
        :key="`${item.Id}-${index}`"
        nuxt
        :to="getItemDetailsLink(item, 'Person')"
      >
        <v-list-item-avatar>
          <v-avatar color="card">
            <blurhash-image :item="item" icon-size="16" />
          </v-avatar>
        </v-list-item-avatar>
        <v-list-item-content>
          <v-list-item-title>{{ item.Name }}</v-list-item-title>
          <v-list-item-subtitle>
            {{ item.Role || item.Type }}
          </v-list-item-subtitle>
        </v-list-item-content>
      </v-list-item>
    </div>
    <div
      v-for="index in skeletonLength"
      v-else
      :key="index"
      class="d-flex align-center mt-5 mb-5"
    >
      <v-skeleton-loader type="avatar" class="ml-3 mr-3" />
      <v-skeleton-loader type="sentences" width="10em" class="pr-5" />
    </div>
  </v-list>
</template>

<script lang="ts">
import Vue from 'vue';
import { BaseItemPerson } from '@jellyfin/client-axios';
import { getItemDetailsLink } from '~/utils/items';

export default Vue.extend({
  props: {
    items: {
      type: Array,
      default: (): BaseItemPerson[] => {
        return [];
      }
    },
    skeletonLength: {
      type: Number,
      default: 0
    }
  },
  methods: {
    getItemDetailsLink
  }
});
</script>
