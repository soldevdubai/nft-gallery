<template>
  <section class="hero is-dark homepage">
    <div class="hero-body">
      <div class="container">
        <h1 class="title is-1 homepage__title">
          <span class="text__stroked">Jew</span>
          <span>Bot</span>
        </h1>
        <h2 class="title is-1 is-flex is-flex-direction-column uppercase homepage__heading">
          <span>
            Jew Positive
          </span>
          <span class="text__stroked">
            JEWBOT native
          </span>
          <span>
            NFT plaftorm
          </span>
          <span class="title is-6 homepage__subtitle uppercase text-semibold">
            Built on
            <span class="text-bold text-primary">
              JEW Protocol
            </span>
          </span>
        </h2>
        <div class="buttons">
          <b-dropdown aria-role="list" class="mr-2">
            <template #trigger>
                <b-button
                  label="Create"
                  type="is-primary"
                />
            </template>
            <b-dropdown-item
              aria-role="listitem"
            >
              <router-link :to="{ name: 'rmrk'}">
                {{ $t('Classic') }}
              </router-link>
            </b-dropdown-item>
            <b-dropdown-item
              aria-role="listitem"
            >
              <router-link :to="{ name: 'simpleMint'}">
                {{ $t('Simple') }}
              </router-link>
            </b-dropdown-item>
          </b-dropdown>
          <b-button
              tag="router-link"
              to="/rmrk/gallery"
              type="is-primary"
          >
            Gallery
          </b-button>
          <b-button
              tag="router-link"
              to="/spotlight"
              type="is-primary"
          >
            Spotlight
          </b-button>
          <b-button
              tag="router-link"
              to="/about"
              type="is-primary"
          >
            About
          </b-button>
          <b-button
              tag="router-link"
              to="/rmrk/faq"
              type="is-primary"
          >
            Faq
          </b-button>
          <b-button
              tag="router-link"
              to="/grants"
              type="is-primary"
          >
            Grants
          </b-button>
        </div>
        <div class="homepage__box">
          <div class="homepage__box-content">
            <p>
              Would you like to get featured on our gallery page? Perhaps you have a business proposal in mind. Maybe you would like to be an ambassador. Go to this section for all of the above.
            </p>
            <b-button
              tag="router-link"
              to="/partnership"
              type="is-primary"
              class="homepage__button--wrapped"
            >
              Partnership & Ambassador Program
            </b-button>
            <div>
              <p>
                <span>
                  Would you like to know how our journey started?
                </span>
                <span>
                  If so, visit the About section.
                </span>
              </p>
              <p>
                <span>
                  Struggling with JEWBOT?
                </span>
                <span>
                  Head over to our <a href="/rmrk/faq">FAQ page.</a>
                </span>
              </p>
              <p>
                <span>
                  Found issue? Have Feedback?
                </span>
                <span>
                  Create issue on our TELEGRAM.
                </span>
              </p>
            </div>
          </div>
          <b-button
            tag="a"
            href="https://t.me/jewbotbsc"
            target="_blank"
            rel="noopener noreferrer"
            type="is-primary"
          >
            JEWBOT TELEGRAM
          </b-button>
        </div>
      </div>
    </div>
  </section>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import nftListWithSearch from '@/queries/nftListWithSearch.graphql';
import { denyList } from '@/constants';
import { getMany, update } from 'idb-keyval';
import { fetchNFTMetadata } from '../rmrk/utils';

const components = {
  Identity: () => import('@/components/shared/format/Identity.vue'),
};
@Component<Landing>({
  metaInfo() {
    return {
      meta: [
        { property: 'og:title', content: 'JEWBOT - JEWBOT NFT Market explorer'},
        { property: 'og:image', content: 'https://i.imgur.com/LCfi5yu.png'},
        { property: 'og:description', content: 'JEW POSITIVE NFT gallery on Kusama'},
        { property: 'twitter:title', content: 'JEWBOT - JEWBOT NFT Market Explorer' },
        { property: 'twitter:description', content: 'JEW POSITIVE NFT gallery on Kusama'},
        { property: 'twitter:image', content: 'https://i.imgur.com/LCfi5yu.png'},
      ]
    }
  },
  components
})
export default class Landing extends Vue {

  public publicCommunity: any = [
    ['🇦🇲 Armenia', 'https://t.me/kodadotarmenia'],
    // ['🇧🇩 Bengali', 'https://t.me/KodaDot_Bengali'], seems not active
    ['🇨🇿 Česko', 'https://t.me/joinchat/'],
    ['🇯🇵 Japan', 'https://t.me/'],
    ['🇵🇱 Polska', 'https://t.me/joinchat/'],
    ['🇵🇹 Portuguese', 'https://t.me/joinchat/'],
    ['🇷🇺 Russia', 'https://t.me/kodadotru'],
    ['🇪🇸 Spanish','https://t.me/joinchat/'],
    ['🇹🇷 Türkiye', 'https://t.me/KodaDotTR'],
    ['🌐 Global KodaDot Discord', 'https://discord.gg/'],
    ['🏗 Developers', 'https://discord.gg/'],
    // ['🇻🇳 Việt Nam', 'https://t.me/joinchat/'], seems not active
    ['Want to help translate?', 'https://github.com/kodadot/nft-gallery/tree/i18n/src/locales']
  ]

  public mounted() {
    this.fetchFirstGalleryPage();
  }

  public async fetchFirstGalleryPage() {
    const nfts = this.$apollo.query({
      query: nftListWithSearch,
      variables: {
        first: 12,
        offset: 0,
        denyList,
        search: []
      }
    });

    const {
      data: { nFTEntities: { nodes: nftList } }
    } = await nfts;

    const storedPromise = getMany(
      nftList.map(({ metadata }: any) => metadata)
    );

    const storedMetadata = await storedPromise;

    storedMetadata.forEach(async (m, i) => {
      if (!m) {
        try {
          const meta = await fetchNFTMetadata(nftList[i]);
          update(nftList[i].metadata, () => meta);
        } catch (e) {
          console.warn('[ERR] unable to get metadata');
        }
      }
    });


  }
}
</script>

<style lang="scss" scoped>
@import '@/styles/variables';

.homepage {
  content: '';
  width: 100%;
  height: 100%;

  @include desktop {
    background: url('https://i.imgur.com/LCfi5yu.png') center bottom;
    background-repeat: no-repeat;
    background-size: contain;
  }

  &__title {
    display: inline-flex;
    padding: 16px 32px;
    margin: 0 0 60px;
    text-transform: uppercase;
    border: 4px solid $primary;
  }

  &__heading {
    font-size: 4rem;
  }

  &__box {
    max-width: 600px;
    padding: 40px 48px;
    margin: 120px 0 132px;
    background-color: $scheme-main;
    border: 4px solid $primary;
    border-radius: 0;

    @include desktop {
      box-shadow: 28px -28px $black, 28px -28px 0 4px $primary;
    }
  }

  &__box-content {
    max-width: 464px;

    p, .button {
      margin-bottom: 32px;
    }

    p {
      display: flex;
      flex-direction: column;
    }
  }

  &__button {
    &--wrapped {
      height: auto;
      white-space: normal;
    }
  }
}

.subtitle {
  text-decoration: underline;
}

.truncate {
  flex: 1;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

li {
  list-style-type: square;
}
</style>
