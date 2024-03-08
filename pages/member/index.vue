<template>
  <div v-if="done">
    <template v-if="isDisallowed">
      <v-card>
        <v-card-text class="text-center">
          {{ $t("member.disallowed") }}
        </v-card-text>
      </v-card>
    </template>
    <template v-else>
      <v-row>
        <v-col class="col-sm-6 col-12 py-0">
          <v-autocomplete
            v-model="currName"
            :items="names"
            dense
            filled
            :label="$t('member.search_member')"
          />
        </v-col>
        <v-col class="col-sm-6 col-12 py-0">
          <v-autocomplete
            v-model="currDepartment"
            :items="departments"
            dense
            filled
            :label="$t('member.department')"
          />
        </v-col>
      </v-row>
      <v-data-table
        :headers="headers"
        :items="filteredItems"
        class="elevation-1 mt-5 c-table"
      >
        <template v-slot:item.name="{ item, value }">
          <NuxtLink :to="`/member/detail/${item.id}`" no-prefetch>
            {{ value }}
          </NuxtLink>
        </template>
        <template v-slot:item.phone="{ value }">
          <a :href="`tel:${value}`">{{ value }}</a>
        </template>
      </v-data-table>
    </template>
  </div>
</template>
<script setup>
const { authUser } = useAuth();
const userList = ref([]);
const currName = ref("");
const currDepartment = ref("");
const done = ref(false);
const isDisallowed = ref(false);
const perpage = ref(10);
const page = ref(1);
const pageCount = ref(0);
const snackbar = useSnackbar();

const headers = [
  { title: "Name", value: "name", sortable: false },
  { title: "Deparment", value: "department", sortable: false },
  { title: "Position", value: "position", sortable: false },
  { title: "Phone", value: "phone", sortable: false },
];

const names = computed(() => {
  return ["", ...userList.value.map(({ name }) => name)];
});

const departments = computed(() => {
  return [
    "",
    ...userList.value.map(({ department }) => department).filter((d) => d),
  ];
});

const filteredItems = computed(() => {
  return userList.value
    .filter(({ department }) =>
      currDepartment.value === "" ? true : department === currDepartment.value
    )
    .filter(({ name }) => (currName.value === "" ? true : name === currName.value));
});

console.log("Bhaijaan", filteredItems.value);

onMounted(async () => {
  try {
    const { list } = await $fetch(
      `${apiDomain.baseURL}/rcms-api/1/member/list`,
      {
        credentials: "include",
        server: false,
      }
    );
    console.log(list);
    userList.value = list.map((item) => ({
      name: `${item.name1} ${item.name2}`,
      department: item.department,
      position: item.position,
      phone: item.tel,
      id: item.member_id,
    }));
    console.log(userList.value);
  } catch (error) {
    snackbar.add({
      type: "error",
      text: error?.response?._data?.errors?.[0]?.message || "An error occurred",
    });
    if (error?.response?.status) {
      isDisallowed = true;
    }
  }
  done.value = true;
});
</script>