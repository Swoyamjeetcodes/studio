<template>

  <div class="channel-selection-list">
    <div
      v-if="show('channelSelectionList', loading, 400)"
      class="loader-wrapper"
    >
      <StudioLargeLoader />
    </div>
    <template v-else>
      <KTextbox
        v-model="search"
        class="search-input"
        :label="$tr('searchText')"
        :appearanceOverrides="{ maxWidth: '350px' }"
      />
      <p
        v-if="!listChannels.length"
        class="empty-state"
      >
        {{ $tr('noChannelsFound') }}
      </p>
      <template v-else>
        <VCard
          v-for="channel in listChannels"
          :key="channel.id"
          flat
          hover
          class="list-card-hover"
        >
          <div class="selection-row">
            <KCheckbox
              :checked="selectedChannels.includes(channel.id)"
              :data-testid="`checkbox-${channel.id}`"
              class="selection-checkbox"
              @change="checked => handleCheckboxChange(checked, channel.id)"
            >
              <span class="visuallyhidden">{{ $tr('selectChannel') }}</span>
            </KCheckbox>
            <ChannelItem
              :channelId="channel.id"
              :data-testid="`channel-item-${channel.id}`"
              @click="handleSelectChannel"
            />
          </div>
        </VCard>
      </template>
    </template>
  </div>

</template>


<script>

  import sortBy from 'lodash/sortBy';
  import { mapGetters, mapActions } from 'vuex';
  import useKShow from 'kolibri-design-system/lib/composables/useKShow';
  import ChannelItem from './ChannelItem';
  import { ChannelListTypes } from 'shared/constants';
  import StudioLargeLoader from 'shared/views/StudioLargeLoader';

  function listTypeValidator(value) {
    // The value must match one of the ListTypes
    return Object.values(ChannelListTypes).includes(value);
  }

  export default {
    name: 'ChannelSelectionList',
    components: {
      ChannelItem,
      StudioLargeLoader,
    },
    setup() {
      const { show } = useKShow();

      return {
        show,
      };
    },
    props: {
      value: {
        type: Array,
        default() {
          return [];
        },
      },
      listType: {
        type: String,
        validator: listTypeValidator,
        default: Object.values(ChannelListTypes)[0],
      },
    },
    data() {
      return {
        loading: true,
        search: '',
      };
    },
    computed: {
      ...mapGetters('channel', ['channels']),
      selectedChannels: {
        get() {
          return this.value;
        },
        set(value) {
          this.$emit('input', value);
        },
      },
      listChannels() {
        return sortBy(
          this.channels.filter(
            channel =>
              channel[this.listType] &&
              channel.published &&
              (channel.name.toLowerCase().includes(this.search.toLowerCase()) ||
                channel.description.toLowerCase().includes(this.search.toLowerCase())),
          ),
          'name',
        );
      },
    },
    mounted() {
      this.loadChannelList({
        listType: this.listType,
        published: true,
      }).finally(() => {
        this.loading = false;
      });
    },
    methods: {
      ...mapActions('channel', ['loadChannelList']),
      handleCheckboxChange(checked, channelId) {
        if (checked) {
          this.selectedChannels = [
            channelId,
            ...this.selectedChannels.filter(id => id !== channelId),
          ];
          return;
        }

        this.selectedChannels = this.selectedChannels.filter(id => id !== channelId);
      },
      handleSelectChannel(channelId) {
        this.selectedChannels = this.selectedChannels.includes(channelId)
          ? this.selectedChannels.filter(id => id !== channelId)
          : [...this.selectedChannels, channelId];
      },
    },
    $trs: {
      searchText: 'Search for a channel',
      noChannelsFound: 'No channels found',
      selectChannel: 'Select channel',
    },
  };

</script>


<style lang="scss" scoped>

  .channel-selection-list {
    padding-bottom: 20px;
  }

  .loader-wrapper {
    padding-top: 16px;
  }

  .list-card-hover {
    padding: 0 12px;
    margin: 16px;
    box-shadow: 0 3px 5px 0 rgba(0, 0, 0, 0.2);
  }

  .selection-row {
    display: flex;
    align-items: center;
  }

  .selection-checkbox {
    flex-shrink: 0;
    margin: 0;
  }

  .search-input {
    margin-top: 16px;
  }

  .empty-state {
    margin-top: 16px;
    margin-bottom: 0;
    color: var(--v-grey-darken1);
  }

</style>
