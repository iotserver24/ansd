<script setup>
const env = useRuntimeConfig();
const episode_dialog = ref(false);
const infotab = ref(null);
const ep_tab = ref(null);
const selectedProvider = ref("Gogoanime");

const {
  data: anime,
  pending: aniPending,
  error: aniError,
} = await useFetch(
  `${env.public.API_URL}/api/${env.public.version}/info/${
    useRoute().params.id
  }`,
  {
    cache: "force-cache",
  }
);

useSeoMeta({
  ogTitle: anime.value?.title.userPreferred,
  ogDescription: anime.value?.description,
  ogImage: anime.value?.coverImage.large,
  ogUrl: useRoute().fullPath,
  twitterTitle: `${anime.value?.title.userPreferred} - AnimeVerse`,
  twitterDescription: anime.value?.description,
  twitterImage: anime.value?.coverImage.large,
  twitterCard: "summary",
});

useHead({
  htmlAttrs: {
    lang: "en",
  },
  title: anime.value?.title.userPreferred,
});

const countdown = ref();

const updateCountdown = () => {
  setInterval(() => {
    const currentTime = Math.floor(Date.now() / 1000);
    const remainingTime = anime?.value.nextair?.airingAt - currentTime;
    countdown.value = formatDuration(remainingTime);
  }, 1000);
};

const formatDuration = (duration) => {
  const days = Math.floor(duration / (60 * 60 * 24));
  const hours = Math.floor((duration % (60 * 60 * 24)) / (60 * 60));
  const minutes = Math.floor((duration % (60 * 60)) / 60);
  const seconds = duration % 60;

  return `${days} days, ${hours} hours, ${minutes} minutes, ${seconds} seconds`;
};

function getAiringDay() {
  const airingDate = new Date(anime?.value.nextair?.airingAt * 1000);
  const daysOfWeek = [
    "Sunday",
    "Monday",
    "Tuesday",
    "Wednesday",
    "Thursday",
    "Friday",
    "Saturday",
  ];
  const airingDay = daysOfWeek[airingDate.getDay()];
  return airingDay;
}

onMounted(() => {
  updateCountdown();
});

const { data: recmedAnime, pending: recmedPending } = useFetch(
  `${env.public.API_URL}/api/${env.public.version}/recommendations/${
    useRoute().params.id
  }`,
  {
    cache: "force-cache",
  }
);

const {
  data: epAni,
  pending: loadAni,
  error: epAniError,
} = useLazyFetch(
  `${env.public.API_URL}/api/v1/episode/${
    anime?.value.id_provider === null ? "''" : anime.value.id_provider.idGogo
  }`,
  {
    cache: "default",
  }
);
const {
  data: epAniDub,
  pending: loadAniDub,
  error: epDubAniError,
} = useLazyFetch(
  `${env.public.API_URL}/api/v1/episode/${
    anime?.value.id_provider === null ? "''" : anime.value.id_provider.idGogoDub
  }`,
  {
    cache: "default",
  }
);

const stringInstring = '""';
</script>

<template>
  <div v-if="aniPending" class="loadingBlock">
    <v-progress-circular :size="45" indeterminate />
  </div>
  <div v-else>
    <v-breadcrumbs>
      <template #prepend>
        <v-icon size="small" icon="mdi-home"></v-icon>
      </template>
      <v-breadcrumbs-item title="Home" to="/pwa" />
      <v-breadcrumbs-divider />
      <v-breadcrumbs-item :title="useRoute().params.id" />
    </v-breadcrumbs>
    <v-card>
      <v-img
        v-if="anime?.bannerImage !== null"
        :src="anime?.bannerImage"
        max-height="280px"
        cover=""
      >
        <template #placeholder>
          <v-sheet :color="anime?.coverImage.color" height="250px"></v-sheet>
        </template>
      </v-img>
      <v-sheet v-else :color="anime?.coverImage.color" height="250px" />
      <v-container>
        <div class="card-container">
          <div class="image-area">
            <img
              class="image-poster"
              :src="anime?.coverImage.large"
              :alt="anime?.title.userPreferred"
            />
            <div class="d-none d-lg-flex flex-column">
              <BookmarkButton
                :id="anime?.id"
                :title="anime?.title.userPreferred"
                :imgsrc="anime?.coverImage.large"
                :imgalt="anime?.id"
                :anime-color="anime?.coverImage.color"
                :year="anime?.year"
                :type="anime?.format"
                :total-ep="anime?.episodes ? anime?.episodes : 0"
                :status="anime?.status"
              />
            </div>
          </div>
          <div class="card-content">
            <div class="mt-2">
              <v-chip
                class="mr-1"
                label
                variant="elevated"
                :color="
                  !epAniDub ||
                  !epAniDub?.episodes ||
                  epAniDub?.episodes.length === 0
                    ? 'warning'
                    : 'success'
                "
              >
                {{
                  !epAniDub ||
                  !epAniDub?.episodes ||
                  epAniDub?.episodes.length === 0
                    ? "No Dub"
                    : "Dub"
                }}
              </v-chip>
              <v-chip
                class="mx-1"
                label
                variant="elevated"
                :color="
                  anime?.status === 'FINISHED'
                    ? 'success'
                    : anime?.status === 'RELEASING'
                    ? 'warning'
                    : anime?.status === 'NOT_YET_RELEASED'
                    ? 'info'
                    : anime?.status === 'CANCELLED'
                    ? 'danger'
                    : 'No data'
                "
              >
                {{
                  anime?.status === "FINISHED"
                    ? "Finished"
                    : anime?.status === "RELEASING"
                    ? "Currently Releasing"
                    : anime?.status === "NOT_YET_RELEASED"
                    ? "Not Released"
                    : anime?.status === "CANCELLED"
                    ? "Cancelled"
                    : "No data"
                }}
              </v-chip>
            </div>
            <h1 class="mt-2" style="line-height: 2rem; font-size: x-large">
              {{ anime?.title.userPreferred }}
            </h1>
            <p class="mb-2">{{ anime?.title.english }}</p>
            <p class="mb-2">{{ anime?.title.native }}</p>
            <div class="d-flex d-lg-none flex-column">
              <BookmarkButton
                :id="anime?.id"
                :title="anime?.title.userPreferred"
                :imgsrc="anime?.coverImage.large"
                :imgalt="anime?.id"
                :anime-color="anime?.coverImage.color"
                :year="anime?.year"
                :type="anime?.format"
                :total-ep="anime?.episodes ? anime?.episodes : 0"
                :status="anime?.status"
              />
            </div>
            <v-btn
              class="my-2"
              color="red"
              prepend-icon="mdi-play"
              :disabled="
                anime.id_provider === null ||
                anime.id_provider.idGogo.includes(stringInstring)
              "
              @click="episode_dialog = !episode_dialog"
            >
              {{
                anime.id_provider === null || anime.id_provider.idGogo === '""'
                  ? "Not available"
                  : "Watch Now"
              }}
            </v-btn>
            <ClientOnly>
              <v-dialog
                v-model="episode_dialog"
                scrim="#202020"
                max-width="500px"
                height="auto"
                scrollable
              >
                <v-card elevation="10">
                  <v-card-title>Episode</v-card-title>
                  <v-card elevation="0">
                    <v-tabs v-model="ep_tab" grow="">
                      <v-tab value="eplist"> Episode list </v-tab>
                    </v-tabs>
                    <v-card-text>
                      <v-window v-model="ep_tab">
                        <v-window-item value="eplist">
                          <div>
                            <v-select
                              v-model="selectedProvider"
                              clearable
                              label="Select streaming provider"
                              :items="[
                                'Gogoanime',
                                'Gogoanime (DUB)',
                              ]"
                              variant="solo"
                            ></v-select>
                            <v-list
                              v-if="selectedProvider == 'Gogoanime'"
                              lines="two"
                              height="300px"
                            >
                              <v-progress-circular
                                v-if="loadAni"
                                :size="40"
                                indeterminate
                              />
                              <div
                                v-else-if="
                                  !epAni ||
                                  !epAni?.episodes ||
                                  epAni?.episodes.length === 0
                                "
                              >
                                Episodes not found or not available...
                              </div>
                              <div v-else-if="epAniError">
                                Error found... try again...
                              </div>
                              <v-list-item
                                v-else
                                v-for="(episode, i) in epAni?.episodes"
                                :key="i"
                                :title="'Ep ' + episode.num"
                                @click="
                                  window.open(episode.url, '_blank')
                                "
                              >
                                <v-list-item-content>
                                  <v-list-item-title
                                    v-text="'Ep ' + episode.num"
                                  ></v-list-item-title>
                                  <v-list-item-subtitle
                                    v-text="'Upload: ' + episode.date"
                                  ></v-list-item-subtitle>
                                </v-list-item-content>
                                <v-list-item-icon>
                                  <v-icon icon="mdi-play"></v-icon>
                                </v-list-item-icon>
                              </v-list-item>
                            </v-list>
                            <v-list
                              v-if="selectedProvider == 'Gogoanime (DUB)'"
                              lines="two"
                              height="300px"
                            >
                              <v-progress-circular
                                v-if="loadAniDub"
                                :size="40"
                                indeterminate
                              />
                              <div
                                v-else-if="
                                  !epAniDub ||
                                  !epAniDub?.episodes ||
                                  epAniDub?.episodes.length === 0
                                "
                              >
                                Episodes not found or not available...
                              </div>
                              <div v-else-if="epDubAniError">
                                Error found... try again...
                              </div>
                              <v-list-item
                                v-else
                                v-for="(episode, i) in epAniDub?.episodes"
                                :key="i"
                                :title="'Ep ' + episode.num"
                                @click="
                                  window.open(episode.url, '_blank')
                                "
                              >
                                <v-list-item-content>
                                  <v-list-item-title
                                    v-text="'Ep ' + episode.num"
                                  ></v-list-item-title>
                                  <v-list-item-subtitle
                                    v-text="'Upload: ' + episode.date"
                                  ></v-list-item-subtitle>
                                </v-list-item-content>
                                <v-list-item-icon>
                                  <v-icon icon="mdi-play"></v-icon>
                                </v-list-item-icon>
                              </v-list-item>
                            </v-list>
                          </div>
                        </v-window-item>
                      </v-window>
                    </v-card-text>
                  </v-card>
                </v-dialog>
              </ClientOnly>
            </v-card>
            <v-divider></v-divider>
            <v-card-subtitle>Description</v-card-subtitle>
            <div class="text-break" v-html="anime?.description"></div>
            <v-divider></v-divider>
            <v-card-subtitle>Informations</v-card-subtitle>
            <v-list
              v-if="anime"
              class="text-break"
              lines="two"
              height="auto"
              style="flex-wrap: wrap"
            >
              <v-list-item title="Origin" v-text="anime?.origin"></v-list-item>
              <v-list-item
                title="Title Synonyms"
                v-text="
                  anime?.synonyms.length > 0 ? anime?.synonyms.join(', ') : '-'
                "
              ></v-list-item>
              <v-list-item
                title="Genres"
                v-text="anime?.genres.length > 0 ? anime?.genres.join(', ') : '-'"
              ></v-list-item>
              <v-list-item
                title="Themes"
                v-text="anime?.themes.length > 0 ? anime?.themes.join(', ') : '-'"
              ></v-list-item>
              <v-list-item
                title="Total Episodes"
                v-text="anime?.episodes ? anime?.episodes : '-'"
              ></v-list-item>
              <v-list-item title="Year" v-text="anime?.year"></v-list-item>
              <v-list-item
                title="Airing Day"
                v-if="anime?.nextair?.airingAt !== null"
                v-text="getAiringDay()"
              ></v-list-item>
              <v-list-item
                title="Next Episode"
                v-if="anime?.nextair?.airingAt !== null"
                v-text="countdown"
              ></v-list-item>
              <v-list-item
                title="Season"
                v-text="anime?.season ? anime?.season : '-'"
              ></v-list-item>
              <v-list-item
                title="Studios"
                v-text="
                  anime?.studios.length > 0 ? anime?.studios.join(', ') : '-'
                "
              ></v-list-item>
              <v-list-item
                title="Status"
                v-text="
                  anime?.status === 'FINISHED'
                    ? 'Finished'
                    : anime?.status === 'NOT_YET_RELEASED'
                    ? 'Not Released'
                    : anime?.status === 'CANCELLED'
                    ? 'Cancelled'
                    : anime?.status === 'RELEASING'
                    ? 'Releasing'
                    : 'No data'
                "
              />
            </v-list>
          </div>
        </div>
      </v-container>
    </v-card>
    <v-container>
      <v-row>
        <v-col cols="12" lg="3" md="4" sm="12">
          <ClientOnly>
            <v-card elevation="10" style="position: sticky; top: 20px;">
              <v-card-title>Episodes</v-card-title>
              <v-card-text>
                <v-list
                  lines="two"
                  height="auto"
                  v-for="(episode, i) in epAni?.episodes"
                  :key="i"
                  :title="'Ep ' + episode.num"
                  @click="window.open(episode.url, '_blank')"
                >
                  <v-list-item-content>
                    <v-list-item-title v-text="'Ep ' + episode.num"></v-list-item-title>
                    <v-list-item-subtitle v-text="'Upload: ' + episode.date"></v-list-item-subtitle>
                  </v-list-item-content>
                  <v-list-item-icon>
                    <v-icon icon="mdi-play"></v-icon>
                  </v-list-item-icon>
                </v-list>
              </v-card-text>
            </v-card>
          </ClientOnly>
        </v-col>
        <v-col cols="12" lg="9" md="8" sm="12">
          <v-card class="mb-2">
            <v-card-title>Recommendations</v-card-title>
            <v-card-text>
              <div v-if="recmedPending" class="loadingBlock">
                <v-progress-circular :size="45" indeterminate />
              </div>
              <div v-else-if="!recmedAnime || recmedAnime.length === 0">
                No recommendations available...
              </div>
              <div v-else>
                <v-row>
                  <v-col v-for="rec in recmedAnime" :key="rec.id" cols="12" md="6" lg="4">
                    <v-card>
                      <v-img :src="rec.coverImage.large" height="200px"></v-img>
                      <v-card-title>{{ rec.title.userPreferred }}</v-card-title>
                      <v-card-subtitle>{{ rec.description }}</v-card-subtitle>
                      <v-card-actions>
                        <v-btn :to="`/pwa/anime/${rec.id}`">More Info</v-btn>
                      </v-card-actions>
                    </v-card>
                  </v-col>
                </v-row>
              </div>
            </v-card-text>
          </v-card>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>
