<template>
  <div>
    <v-app-bar fixed flat dense class="second-toolbar">
      <span class="text-h6 hidden-sm-and-down">
        {{ genre.Name }}
      </span>
      <v-spacer />
      <v-fade-transition>
        <play-button v-if="!$fetchState.pending" :item="genre" />
      </v-fade-transition>
      <v-btn
        v-if="!$fetchState.pending"
        class="play-button mr-2"
        min-width="8em"
        outlined
        rounded
        nuxt
        :to="`./${genre.Id}/shuffle`"
      >
        {{ $t('playback.shuffleAll') }}
      </v-btn>
    </v-app-bar>
    <v-container class="after-second-toolbar">
      <v-row v-if="$fetchState.pending">
        <v-col cols="12" class="card-grid-container">
          <skeleton-card v-for="n in 24" :key="n" text />
        </v-col>
      </v-row>
      <item-grid
        v-if="genres.length"
        :items="genres"
        :loading="$fetchState.pending"
      />
      <v-row v-else-if="!$fetchState.pending" justify="center">
        <v-col cols="12" class="card-grid-container empty-card-container">
          <skeleton-card v-for="n in 24" :key="n" boilerplate text />
        </v-col>
        <div class="empty-message text-center">
          <h1 class="text-h5">
            {{ $t('libraryEmpty') }}
          </h1>
        </div>
      </v-row>
    </v-container>
  </div>
</template>

<script lang="ts">
import Vue from 'vue';
import { mapStores } from 'pinia';
import { BaseItemDto, SortOrder, ItemFields } from '@jellyfin/client-axios';
import { Context } from '@nuxt/types';
import { isValidMD5 } from '~/utils/items';
import { authStore, itemsStore, pageStore } from '~/store';

export default Vue.extend({
  validate(ctx: Context) {
    return isValidMD5(ctx.route.params.itemId);
  },
  async asyncData({ params, $api, route }) {
    const items = itemsStore();
    const auth = authStore();

    const itemId = params.itemId;
    const item = (
      await $api.userLibrary.getItem({
        userId: auth.currentUserId,
        itemId
      })
    ).data;

    let genres = (
      await $api.items.getItems({
        genreIds: [item.Id as string],
        includeItemTypes: [route.query.type.toString()],
        recursive: true,
        sortBy: ['SortName'],
        sortOrder: [SortOrder.Ascending],
        fields: Object.values(ItemFields),
        userId: auth.currentUserId
      })
    ).data.Items;

    genres = items.addCollection(item, genres || []);

    return { item, genres };
  },
  data() {
    return {
      item: {} as BaseItemDto,
      genres: [] as BaseItemDto[]
    };
  },
  head() {
    return {
      title: this.page.title
    };
  },
  computed: {
    ...mapStores(pageStore)
  },
  mounted() {
    this.page.title = this.item.Name || '';
  }
});
</script>

<style lang="scss" scoped>
@import '~vuetify/src/styles/styles.sass';

.second-toolbar {
  top: 56px;
}

@media #{map-get($display-breakpoints, 'md-and-up')} {
  .second-toolbar {
    top: 64px;
  }
}

@media #{map-get($display-breakpoints, 'lg-and-up')} {
  .second-toolbar {
    left: 256px !important;
  }
}

.after-second-toolbar {
  padding-top: 60px;
}

.genre-toolbar {
  width: 100%;
}

.scroller {
  max-height: 100%;
}

.card-grid-container {
  display: grid;
}

@media #{map-get($display-breakpoints, 'sm-and-down')} {
  .card-grid-container {
    grid-template-columns: repeat(3, minmax(calc(100% / 3), 1fr));
  }
}

@media #{map-get($display-breakpoints, 'sm-and-up')} {
  .card-grid-container {
    grid-template-columns: repeat(4, minmax(calc(100% / 4), 1fr));
  }
}

@media #{map-get($display-breakpoints, 'lg-and-up')} {
  .card-grid-container {
    grid-template-columns: repeat(6, minmax(calc(100% / 6), 1fr));
  }
}

@media #{map-get($display-breakpoints, 'xl-only')} {
  .card-grid-container {
    grid-template-columns: repeat(8, minmax(calc(100% / 8), 1fr));
  }
}
</style>
