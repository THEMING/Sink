<script setup>
import { createReusableTemplate, useMediaQuery, useUrlSearchParams, watchDebounced } from '@vueuse/core'
import { safeDestr } from 'destr'
import { Check, ChevronsUpDown } from 'lucide-vue-next'
import { VList } from 'virtua/vue'

const emit = defineEmits(['change'])

const [TriggerTemplate, TriggerComponent] = createReusableTemplate()
const [FilterTemplate, FilterComponent] = createReusableTemplate()

const isDesktop = useMediaQuery('(min-width: 640px)')

const links = ref([])
const isOpen = ref(false)
const selectedLinks = ref([])

async function getLinks() {
  links.value = await useAPI('/api/link/search')
}

onMounted(() => {
  getLinks()
})

watchDebounced(selectedLinks, (value) => {
  emit('change', 'slug', value.join(','))
}, { debounce: 500, maxWait: 1000 })

function restoreFilters() {
  const searchParams = useUrlSearchParams('history')
  if (searchParams.filters) {
    const filters = safeDestr(searchParams.filters)
    if (filters.slug) {
      selectedLinks.value = filters.slug.split(',')
    }
  }
}

onBeforeMount(() => {
  restoreFilters()
})
</script>

<template>
  <TriggerTemplate>
    <Button
      variant="outline"
      role="combobox"
      :aria-expanded="isOpen"
      class="flex justify-between px-3 w-full sm:w-48"
    >
      <div class="flex-1 text-left truncate" :class="selectedLinks.length ? 'text-foreground' : 'text-muted-foreground'">
        {{ selectedLinks.length ? selectedLinks.join(', ') : $t('dashboard.filter_placeholder') }}
      </div>
      <ChevronsUpDown class="ml-2 w-4 h-4 opacity-50 shrink-0" />
    </Button>
  </TriggerTemplate>
  <FilterTemplate>
    <Command v-model="selectedLinks" multiple>
      <CommandInput :placeholder="selectedLinks.length ? selectedLinks.join(', ') : $t('dashboard.filter_placeholder')" />
      <CommandEmpty>No link found.</CommandEmpty>
      <CommandList :class="{ 'max-h-none': !isDesktop }">
        <CommandGroup>
          <VList
            v-slot="{ item: link }"
            :data="links"
            :style="{ height: isDesktop ? '292px' : '420px' }"
          >
            <CommandItem
              :value="link.slug"
            >
              <Check
                :class="cn(
                  'mr-2 h-4 w-4',
                  selectedLinks.includes(link.slug) ? 'opacity-100' : 'opacity-0',
                )"
              />
              {{ link.comment }}
            </CommandItem>
          </VList>
        </CommandGroup>
      </CommandList>
    </Command>
  </FilterTemplate>
  <Popover v-if="isDesktop" v-model:open="isOpen">
    <PopoverTrigger as-child>
      <TriggerComponent />
    </PopoverTrigger>
    <PopoverContent class="p-0 w-full sm:w-48">
      <FilterComponent />
    </PopoverContent>
  </Popover>

  <Drawer v-else v-model:open="isOpen">
    <DrawerTrigger as-child>
      <TriggerComponent />
    </DrawerTrigger>
    <DrawerContent class="h-[500px]">
      <FilterComponent />
    </DrawerContent>
  </Drawer>
</template>
