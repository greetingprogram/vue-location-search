<script setup>
  import { ref } from 'vue'
  import LocationSearch from './components/LocationSearch.vue'

  const country = ref('US')
  const location = ref(null)

  function onSelected(loc) {
    console.log('Location selected:', loc)
  }

  const mockCities = [
    { display: 'Fort Worth, TX', city: 'Fort Worth', state: 'TX', zip: '76101' },
    { display: 'Tulsa, OK', city: 'Tulsa', state: 'OK', zip: '74101' },
    { display: 'Dallas, TX', city: 'Dallas', state: 'TX', zip: '75201' },
    { display: 'Denver, CO', city: 'Denver', state: 'CO', zip: '80201' },
    { display: 'Austin, TX', city: 'Austin', state: 'TX', zip: '73301' },
    { display: 'Houston, TX', city: 'Houston', state: 'TX', zip: '77001' },
    { display: 'Chicago, IL', city: 'Chicago', state: 'IL', zip: '60601' },
  ]

  async function mockSearch(query, country, pattern) {
    if (!query) return mockCities
    const q = query.toLowerCase()
    return mockCities.filter(item =>
      item.display.toLowerCase().includes(q) ||
      item.zip.startsWith(q),
    )
  }
</script>

<template>
  <v-app>
    <v-container>
      <v-card class="mx-auto mt-10" max-width="500" elevation="2">
        <v-card-title class="text-h5 pt-4 px-6">
          Location Search Demo
        </v-card-title>

        <v-card-text class="px-6 pb-6">
          <v-select
            v-model="country"
            :items="['US', 'CA', 'MX']"
            label="Country"
            variant="outlined"
            density="compact"
            class="mb-4"
          />

          <LocationSearch
            v-model="location"
            :country="country"
            :search-fn="mockSearch"
            @selected="onSelected"
          />

          <v-alert v-if="location" type="info" variant="tonal" class="mt-4">
            <strong>Selected:</strong> {{ location.display }}
            <pre class="text-caption mt-2">{{ JSON.stringify(location, null, 2) }}</pre>
          </v-alert>
        </v-card-text>
      </v-card>
    </v-container>
  </v-app>
</template>
