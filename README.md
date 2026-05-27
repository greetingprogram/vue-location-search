# vue-location-search

A Vuetify 3 combobox component for searching locations by city, state, or ZIP code. Supports US, Canada, and Mexico postal formats with smart pattern detection and a pluggable async search function.

---

## Features

- Search by city name, `City, ST`, `ST, City`, state abbreviation, or ZIP/postal code
- Smart pattern detection passes structured context to your search function
- Built-in postal code validation for **US**, **Canada**, and **Mexico**
- Debounced input with configurable delay and minimum character threshold
- Fully controlled via `v-model`
- Customizable label, variant, and density (Vuetify props)

---

## Requirements

- Vue 3
- Vuetify 3
- lodash-es (for debounce)

---

## Installation

Copy `LocationSearch.vue` into your project's components directory. No package installation needed beyond the peer dependencies above.

---

## Usage

```vue
<script setup>
import { ref } from 'vue'
import LocationSearch from './components/LocationSearch.vue'

const country = ref('US')
const location = ref(null)

async function searchLocations(query, country, pattern) {
  // Call your API here. `pattern` contains the parsed search intent.
  const response = await fetch(`/api/locations?q=${encodeURIComponent(query)}&country=${country}`)
  return response.json() // must return array of objects with a `display` property
}
</script>

<template>
  <LocationSearch
    v-model="location"
    :country="country"
    :search-fn="searchLocations"
    @selected="loc => console.log('Selected:', loc)"
  />
</template>
```

---

## Props

| Prop | Type | Default | Description |
|---|---|---|---|
| `modelValue` | `Object \| String` | `null` | The selected location object (use with `v-model`) |
| `country` | `String` | `'US'` | Country code for postal validation: `'US'`, `'CA'`, or `'MX'` |
| `searchFn` | `Function` | *(required)* | Async function called with `(query, country, pattern)` — must return an array of location objects |
| `debounceMs` | `Number` | `400` | Debounce delay in milliseconds |
| `minChars` | `Number` | `2` | Minimum characters before search is triggered |
| `label` | `String` | `'City, State or ZIP'` | Input label |
| `variant` | `String` | `'outlined'` | Vuetify input variant |
| `density` | `String` | `'compact'` | Vuetify input density |

---

## Events

| Event | Payload | Description |
|---|---|---|
| `update:modelValue` | `Object \| null` | Emitted when the selection changes (used by `v-model`) |
| `selected` | `Object \| null` | Emitted when a location is selected or cleared |

---

## The `searchFn` Signature

```js
async function searchFn(query, country, pattern) { ... }
```

The `pattern` argument contains the parsed search intent so your backend can optimize the query:

```js
// Examples:
{ type: 'zip',        value: '74101' }
{ type: 'partial-zip', value: '741' }
{ type: 'city-state', city: 'Tulsa', state: 'OK' }
{ type: 'state-city', state: 'OK', cityFilter: 'Tul' }
{ type: 'state',      value: 'OK' }
{ type: 'city',       value: 'Tulsa' }
{ type: 'general',    value: 'Tul' }
```

Your function must return an array of objects that each have at minimum a `display` string property:

```js
[
  { display: 'Tulsa, OK', city: 'Tulsa', state: 'OK', zip: '74101' },
  ...
]
```

You can include any additional properties — they will be passed through on selection.

---

## License

MIT
