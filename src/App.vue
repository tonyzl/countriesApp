<script setup lang="ts">
import { onMounted, ref, watch } from "vue";
import PageHeader from "./components/PageHeader.vue";
import CountryList from "./components/CountryList.vue";
import axiosClient from "./utils/axios";
import { Country } from "./models/country.model";

const countries = ref<Country[]>([]);
const search = ref("");
const filteredCountries = ref<Country[]>([]);
const page = ref(1);
const itemsPerPage = ref(12);
const paginatedCountries = ref<Country[]>([]);
const totalItems = ref(0);

const fetchcountries = async () => {
  try {
    const { data } = await axiosClient.get("/all");
    countries.value = data;
    totalItems.value=countries.value.length
  } catch (error) {
    console.log(error);
  }
};

const filtercountries = () => {
  filteredCountries.value = countries.value.filter((country) =>
    country.name.common
      .toLocaleLowerCase()
      .includes(search.value.toLocaleLowerCase())
  );
};

const sliceCountries = (currentCountries: Country[]) => {
  const start = (page.value - 1) * itemsPerPage.value;
  const end = page.value * itemsPerPage.value;
  paginatedCountries.value = currentCountries.slice(start, end);
};

const changePage = (newPage:number) =>{
  page.value=newPage;
}

onMounted(() => {
  fetchcountries();
});

watch([countries,page,filteredCountries], () => {
  sliceCountries(
    filteredCountries.value.length <=0 
    ?countries.value
    :filteredCountries.value
    );
});
</script>

<template>
  <PageHeader />

  <div class="container px-6 mx-auto max-w--screen-lg">
    <div class="mb-8">
      <input
        class="w-full p-1 px-4 border border-gray-300 rounded"
        type="text"
        placeholder="Search by country name"
        v-model="search"
        @input="filtercountries" />
    </div>
    <div class="flex justify-center mb-8 space-x-6">
      <button
        :disabled="page <= 1"
        :class="{'opacity-20':page<=1}"
        @click="changePage(page-1)"
        class="px-2 border border-gray-300 rounded py-0.5 hover:bg-gray-200">
        previous
      </button>
      <button
        :disabled="page >= totalItems/itemsPerPage"
        :class="{'opacity-20':page>= totalItems/itemsPerPage}"
        @click="changePage(page+1)"
        class="px-2 border border-gray-300 rounded py-0.5 hover:bg-gray-200">
        next
      </button>
    </div>
    <CountryList
      :countries="
        paginatedCountries
      " />
  </div>
</template>

<!-- <div v-for="country in countries">{{country.name.common}}</div> -->
