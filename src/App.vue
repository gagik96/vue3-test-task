<template>
  <v-app>
    <v-main>
      <SiListDropdown
        v-model="items"
        :selectedValues="selectedValues"
        :maxDropdowns="7"
        :minDropdowns="2"
        :disableAdd="false"
        :disableAllDeleteBtns="false"
        @update:modelValue="handleModelValueUpdate"
      />
      <!-- The selected list objects for all dropdowns can be read from outside the component, ordered
      by the active dropdown order. Show them in de demo screen that uses the component -->
      <v-container class="w-25">
        Selected Values: {{ selectedValues.join(', ') }}
      </v-container>
    </v-main>
  </v-app>
</template>

<script setup lang="ts">
  import SiListDropdown from '@/components/si-listdropdown.vue'
  import { v4 as uuidv4 } from 'uuid';
  import { ref, computed } from 'vue';

  export interface IDropdown{
    id: number
    value: string
    disableDelete?: boolean
  }

  const items = ref([
    {
      id: uuidv4(),
      value: 'Title'
    },
    {
      id: uuidv4(),
      value: 'ID',
      disableDelete: true
    },
    {
      id: uuidv4(),
      value: 'Not Existing Value'
    },
    {
      id: uuidv4(),
      value: ''
    },
    {
      id: uuidv4(),
      value: ''
    }
  ])

  // The selected list objects for all dropdowns can be read from outside the component, ordered
  // by the active dropdown order. Show them in de demo screen that uses the component
  const selectedValues = computed(
    (): string[] => items.value.filter((item: IDropdown) => !!item.value).map((item: IDropdown) => item.value)
  );

  const handleModelValueUpdate = (updatedItems: IDropdown[]): void => {
    items.value = updatedItems
  }
</script>
