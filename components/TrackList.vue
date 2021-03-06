<template>
  <v-simple-table dense class="track-table no-select">
    <thead>
      <tr>
        <th style="width: 6em" class="pr-0 text-center" scope="col">#</th>
        <th style="width: 3em" class="pr-0 pl-0" scope="col"></th>
        <th scope="col">Title</th>
        <th style="width: 6.5em" class="text-center" scope="col">
          <v-icon class="text--primary" size="16">mdi-clock-outline</v-icon>
        </th>
      </tr>
    </thead>
    <tbody>
      <template v-for="(tracksOnDisc, discNumber) in tracksPerDisc">
        <tr
          v-if="Object.keys(tracksPerDisc).length > 1"
          :key="discNumber"
          class="disc-header"
        >
          <td colspan="4" class="text--secondary">
            <v-icon class="text--secondary">mdi-disc</v-icon>
            {{ $t('disc', { discNumber }) }}
          </td>
        </tr>
        <v-hover
          v-for="track in tracksOnDisc"
          v-slot="{ hover }"
          :key="track.Id"
        >
          <nuxt-link :to="`/item/${track.Id}/play`" tag="tr" event="dblclick">
            <td style="width: 6em" class="pr-0 text-center">
              <span v-if="hover">
                <v-btn icon nuxt :to="`/item/${track.Id}/play`">
                  <v-icon>mdi-play-circle-outline</v-icon>
                </v-btn>
              </span>
              <span v-else>{{ track.IndexNumber }}</span>
            </td>
            <td style="width: 3em" class="pr-0 pl-0 text-center">
              <v-btn icon>
                <v-icon>mdi-heart-outline</v-icon>
              </v-btn>
            </td>
            <td>
              <div class="d-flex align-center">
                <span>{{ track.Name }}</span>
                <div
                  v-if="!track.Artists.includes(track.AlbumArtist)"
                  class="ml-3"
                >
                  <nuxt-link
                    v-for="artist of track.ArtistItems"
                    :key="artist.Id"
                    tag="span"
                    class="link text--secondary"
                    :to="`/artist/${artist.Id}`"
                  >
                    {{ artist.Name }}
                  </nuxt-link>
                </div>
                <v-btn v-if="hover" icon class="ml-auto">
                  <v-icon>mdi-dots-horizontal</v-icon>
                </v-btn>
              </div>
            </td>
            <td class="text-center">{{ getRuntime(track.RunTimeTicks) }}</td>
          </nuxt-link>
        </v-hover>
      </template>
    </tbody>
  </v-simple-table>
</template>

<script lang="ts">
import { groupBy } from 'lodash';
import Vue from 'vue';
import { BaseItemDto, BaseItemDtoQueryResult } from '@jellyfin/client-axios';
import timeUtils from '~/mixins/timeUtils';

export default Vue.extend({
  mixins: [timeUtils],
  props: {
    item: {
      type: Object as () => BaseItemDto,
      required: true
    }
  },
  data() {
    return {
      tracks: [] as BaseItemDtoQueryResult
    };
  },
  computed: {
    tracksPerDisc: {
      get() {
        return groupBy(this.$data.tracks.Items, 'ParentIndexNumber');
      }
    }
  },
  async beforeMount() {
    const tracks = (
      await this.$api.items.getItems({
        userId: this.$auth.user.Id,
        parentId: this.item.Id,
        sortBy: 'SortName',
        sortOrder: 'Ascending'
      })
    ).data;

    this.tracks = tracks;
  },
  methods: {
    /**
     * @param {number} ticks The number of ticks to convert to track length
     * @returns {string} Returns the length of the track in the format XX:XX
     */
    getRuntime(ticks: number) {
      let seconds = this.ticksToMs(ticks) / 1000;
      const minutes = Math.floor(seconds / 60);
      seconds = Math.floor(seconds - minutes * 60);

      /**
       * Formats the second number
       * E.g. 7 -> 07
       *
       * @param {string} seconds Number to format
       * @returns {string} Formatted seconds number
       */
      function formatSeconds(seconds: string): string {
        return ('0' + seconds).slice(-2);
      }

      return `${minutes}:${formatSeconds(seconds.toString())}`;
    }
  }
});
</script>

<style lang="scss" scoped>
.link {
  cursor: pointer;
}
.link:hover {
  text-decoration: underline;
}
.v-data-table.track-table {
  background-color: transparent;
}
.v-data-table tr.disc-header:hover {
  background: transparent !important;
}
.no-select {
  user-select: none;
}
</style>
