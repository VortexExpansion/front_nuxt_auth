<template>
  <div>
    <v-progress-linear
      :active="loading"
      :indeterminate="loading"
      absolute
      top
      color="orange white-4"
    />
    <template v-if="topicsDetail">
      <!-- <v-row> -->
      <v-col>
        <v-card class="d-flex justify-space-between mb-6" flat tile>
          <v-card flat>
            <v-row>
              <v-col class="pt-0">
                <h1 class="mb-3">
                  {{ topicsDetail.subject }}
                </h1>
                <span
                  class="c-btn c-btn_main c-btn_sm c-btn_disable white--text"
                >
                  {{ topicsDetail.contents_type_nm }}
                </span>
              </v-col>
            </v-row>
          </v-card>
          <!-- For favorite -->
          <v-card flat>
            <div class="text-right mt-2">
              {{ topicsDetail.inst_ymdhi.slice(0, 10) }}
              <v-btn icon :color="favoriteColor" @click="onClickToggleFavorite">
                <v-icon x-large left> mdi-star </v-icon>
              </v-btn>
            </div>
          </v-card>
        </v-card>

        <!-- Wysiwyg contents -->
        <v-row
          v-if="topicsDetail && topicsDetail.contents"
          class="p-article_content"
        >
          <v-container fluid>
            <v-card class="mx-auto" max-width="7000">
              <v-card-title class="c-heading_h2" v-text="subtitle" />
              <v-card-text>
                <div class="text--primary" v-html="topicsDetail.contents"></div>
              </v-card-text>
            </v-card>
          </v-container>
        </v-row>

        <!-- extension contents -->
        <v-row
          v-for="(item, idx) in items"
          :key="idx"
          class="p-article_content"
        >
          <TopicsDetail v-if="item" v-bind="item" />
        </v-row>
      </v-col>
      <v-col></v-col>

      <!-- </v-row> -->
      <div class="text-center col mt-5">
        <button
          type="submit"
          class="c-btn c-btn_dark c-btn_icon c-text_white"
          @click="() => $router.push('/topics_list')"
        >
          {{ $t("common.back_to_listing") }}
          <v-icon class="icon c-text_white pr-2"> mdi-undo-variant </v-icon>
        </button>
      </div>
    </template>
  </div>
</template>
<script setup>
const { authUser, profile, logout } = useAuth();
const route = useRoute();
const topicsDetail = ref(null);
const loading = ref(true);
const favoriteResponse = ref(null);
const favoriteColor = ref("grey");
const snackbar = useSnackbar();

const items = computed(() => {
  if (!topicsDetail) {
    return [];
  }

  const { texts, positionPatterns, imageUrls, subtitles } = topicsDetail.value;
  return positionPatterns.map(({ key }, i) => ({
    text: texts?.[i],
    positionPatternKey: key,
    imageUrl: imageUrls?.[i]?.url ? `${imageUrls?.[i]?.url}?width=800` : null,
    subtitle: subtitles?.[i],
  }));
});

const onClickToggleFavorite = async () => {
  try {
    const request =
      favoriteColor.value === "grey"
        ? await $fetch(
            `${apiDomain.baseURL}/rcms-api/1/favorite/register`,
            {
              method: "POST",
              credentials: "include",
              server: false,
              headers: {
                "Content-Type": "application/json",
              },
              body: JSON.stringify({
                module_id: parseInt(route.params.slug),
                module_type: "topics",
              }),
            }
          )
        : await $fetch(
            `${apiDomain.baseURL}/rcms-api/1/favorite/delete`,
            {
              method: "POST",
              credentials: "include",
              server: false,
              headers: {
                "Content-Type": "application/json",
              },
              body: JSON.stringify({
                module_id: parseInt(route.params.slug),
                module_type: "topics",
              }),
            }
          );

    await request;
    favoriteColor.value = favoriteColor.value === "grey" ? "red" : "grey";
  } catch (error) {
    snackbar.add({
      type: "error",
      text: error?.response?._data?.errors?.[0]?.message || "An error occurred",
    });
  }
};

try {
  const response = await $fetch(
    `${apiDomain.baseURL}/rcms-api/1/content/details/${route.params.slug}`,
    {
      credentials: "include",
      server: false,
    }
  );
  const d = response.details;
  topicsDetail.value = {
    ...d,
    fileType: d?.ext_1?.key,
    fileUrl: d?.ext_2?.url,
    fileDownload: d?.ext_2?.dl_link,
    linkUrl: d?.ext_3?.url,
    linkTitle: d?.ext_3?.title,

    // for TopicsDetail
    positionPatterns: d?.ext_4,
    texts: d?.ext_7,
    imageUrls: d?.ext_5,
    subtitles: d?.ext_9,
  };

  const fav = await $fetch(
    `${apiDomain.baseURL}/rcms-api/1/favorite/list`,
    {
      credentials: "include",
      server: false,
      params: {
        member_id: parseInt(authUser.value.member_id),
        module_id: parseInt(route.params.slug),
        module_type: "topics",
      },
    }
  );
  favoriteResponse.value = fav;
  favoriteColor.value =
    favoriteResponse.value?.pageInfo?.totalCnt > 0 ? "red" : "grey";
  loading.value = false;
} catch (error) {
  snackbar.add({
      type: "error",
      text: error?.response?._data?.errors?.[0]?.message || "An error occurred",
    });
}
</script>