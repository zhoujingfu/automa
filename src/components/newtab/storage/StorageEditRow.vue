<template>
  <ui-modal :model-value="modelValue" persist custom-content>
    <ui-card
      padding="p-0"
      class="flex w-full max-w-xl flex-col"
      style="height: 600px"
    >
      <p class="p-4 font-semibold">添加数据项</p>
      <div class="scroll flex-1 overflow-auto px-4 pb-4">
        <ui-input
          v-for="(val, key) in state.rowData"
          :key="key"
          v-model="state.rowData[key]"
          class="mt-1 w-full"
          :label="key"
          placeholder="please enter"
        />
      </div>
      <div class="p-4 text-right">
        <ui-button class="mr-4" @click="clearTempTables(true)">
          {{ t('common.cancel') }}
        </ui-button>
        <ui-button :disabled="state.disabled" variant="accent" @click="saveRow">
          {{ t('common.save') }}
        </ui-button>
      </div>
    </ui-card>
  </ui-modal>
</template>
<script setup>
import { reactive, toRaw, watch } from 'vue';
import { useI18n } from 'vue-i18n';
import cloneDeep from 'lodash.clonedeep';
import { useToast } from 'vue-toastification';

const props = defineProps({
  modelValue: {
    type: Boolean,
    default: false,
  },
  rowData: {
    type: Object,
    default: () => {},
  },
});
const emit = defineEmits(['update:modelValue', 'save']);

const { t } = useI18n();

const toast = useToast();

const state = reactive({
  disabled: false,
  rowData: {},
});

function saveRow() {
  for (const key in state.rowData) {
    if (state.rowData[key] === '') {
      toast.error('请检查数据项');
      return;
    }
  }
  const rawState = toRaw(state);
  emit('save', rawState.rowData);
}
function clearTempTables(close = false) {
  state.rowData = {};
  if (close) {
    emit('update:modelValue', false);
  }
}

watch(
  () => props.modelValue,
  () => {
    if (props.modelValue) {
      Object.assign(state, {
        rowData: cloneDeep(props.rowData),
      });
    } else {
      clearTempTables();
    }
  }
);
</script>
