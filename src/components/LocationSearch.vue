<template>
  <v-combobox
    v-model="internalSelection"
    :items="locations"
    :search="searchText"
    @update:search="onSearch"
    item-title="display"
    item-value="display"
    return-object
    :label="label"
    :variant="variant"
    :density="density"
    :loading="loading"
    :no-data-text="noDataText"
    clearable
    @update:model-value="onSelect"
    auto-select-first="exact"
    :hide-no-data="false"
    :menu-props="{ closeOnContentClick: true }"
    no-filter
    :error="!!validationError"
    :error-messages="validationError ? [validationError] : []"
  >
    <template #item="{ props, item }">
      <v-list-item
        v-bind="props"
        :title="item.raw.display"
        @click="selectItem(item.raw)"
      />
    </template>
  </v-combobox>
</template>

<script setup>
  import { ref, computed, watch } from 'vue'
  import { debounce } from 'lodash-es'

  const props = defineProps({
    modelValue: { type: [Object, String], default: null },
    country: { type: String, default: 'US' },
    searchFn: { type: Function, required: true },
    debounceMs: { type: Number, default: 400 },
    minChars: { type: Number, default: 2 },
    label: { type: String, default: 'City, State or ZIP' },
    variant: { type: String, default: 'outlined' },
    density: { type: String, default: 'compact' },
  })

  const emit = defineEmits(['update:modelValue', 'selected'])

  const internalSelection = ref(props.modelValue)
  const searchText = ref(props.modelValue?.display ?? '')
  const locations = ref([])
  const loading = ref(false)
  const validationError = ref('')

  const noDataText = computed(() => {
    if (validationError.value) return validationError.value
    if (loading.value) return 'Searching...'
    if (searchText.value?.length >= props.minChars) return 'No locations found'
    return 'Type to search'
  })

  function validateZip(text, country) {
    const t = text.trim()
    switch (country) {
      case 'US':
        if (/^\d{5}(-\d{4})?$/.test(t)) return { valid: true, partial: false }
        if (/^\d{1,4}$/.test(t)) return { valid: false, partial: true, error: null }
        if (/^\d{6,}/.test(t)) return { valid: false, partial: true, error: 'US ZIP codes are max 5 digits' }
        return { valid: false, partial: false }
      case 'CA':
        if (/^[A-Za-z]\d[A-Za-z]\s?\d[A-Za-z]\d$/.test(t)) return { valid: true, partial: false }
        if (/^[A-Za-z](\d([A-Za-z](\s?\d([A-Za-z]\d?)?)?)?)?$/.test(t) && t.length < 6)
          return { valid: false, partial: true, error: null }
        if (t.length > 7) return { valid: false, partial: true, error: 'Canada postal codes are max 7 chars (A1A 1A1)' }
        return { valid: false, partial: false }
      case 'MX':
        if (/^\d{5}$/.test(t)) return { valid: true, partial: false }
        if (/^\d{1,4}$/.test(t)) return { valid: false, partial: true, error: null }
        if (/^\d{6,}/.test(t)) return { valid: false, partial: true, error: 'Mexico postal codes are max 5 digits' }
        return { valid: false, partial: false }
      default:
        return { valid: /^\d{5}$/.test(t), partial: /^\d{1,5}$/.test(t) }
    }
  }

  function analyzePattern(text) {
    const t = text.trim()

    const stateOnly = t.match(/^([A-Z]{2}),?\s*$/i)
    if (stateOnly) return { type: 'state', value: stateOnly[1].toUpperCase() }

    const stateCity = t.match(/^([A-Z]{2}),?\s+(.+)$/i)
    if (stateCity) return { type: 'state-city', state: stateCity[1].toUpperCase(), cityFilter: stateCity[2].trim() }

    const cityState = t.match(/^([A-Za-z\s]+),\s*([A-Z]{2})$/i)
    if (cityState) return { type: 'city-state', city: cityState[1].trim(), state: cityState[2].toUpperCase() }

    const cityComma = t.match(/^([A-Za-z\s]+),\s*$/)
    if (cityComma) return { type: 'city', value: cityComma[1].trim() }

    const zip = validateZip(t, props.country)
    if (zip.valid) return { type: 'zip', value: t }
    if (zip.partial) return { type: 'partial-zip', value: t, error: zip.error }

    if (t.length >= props.minChars) return { type: 'general', value: t }

    return { type: 'none', value: '' }
  }

  async function performSearch(text) {
    const pattern = analyzePattern(text)

    if (pattern.type === 'partial-zip' && pattern.error) {
      validationError.value = pattern.error
      locations.value = []
      return
    }
    validationError.value = ''

    if (pattern.type === 'none') {
      locations.value = []
      return
    }

    loading.value = true
    try {
      const results = await props.searchFn(text, props.country, pattern)
      locations.value = Array.from(
        new Map((results || []).map(r => [r.display, r])).values(),
      )
    } catch (e) {
      console.error('Location search failed:', e)
      locations.value = []
    } finally {
      loading.value = false
    }
  }

  const debouncedSearch = debounce(performSearch, props.debounceMs)

  function onSearch(value) {
    searchText.value = value
    if (!value || value.length < props.minChars) {
      locations.value = []
      debouncedSearch.cancel()
      return
    }
    debouncedSearch(value)
  }

  function onSelect(value) {
    if (!value) {
      internalSelection.value = null
      searchText.value = ''
      locations.value = []
      emit('update:modelValue', null)
      emit('selected', null)
    } else if (typeof value === 'object') {
      selectItem(value)
    }
  }

  function selectItem(location) {
    internalSelection.value = location
    searchText.value = location.display
    emit('update:modelValue', location)
    emit('selected', location)
  }

  watch(
    () => props.modelValue,
    v => {
      if (v !== internalSelection.value) {
        internalSelection.value = v
        searchText.value = v?.display ?? ''
      }
    },
    { immediate: true },
  )

  watch(
    () => props.country,
    () => {
      searchText.value = ''
      internalSelection.value = null
      locations.value = []
      validationError.value = ''
      emit('update:modelValue', null)
    },
  )
</script>
