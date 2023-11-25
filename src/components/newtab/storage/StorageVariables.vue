<template>
  <div v-if="store.settings.dev" class="mt-6 flex">
    <ui-input
      v-model="state.query"
      :placeholder="t('common.search')"
      prepend-icon="riSearch2Line"
    />
    <div class="grow"></div>
    <ui-button
      variant="accent"
      style="min-width: 125px"
      class="ml-4"
      @click="editState.show = true"
    >
      Add variable
    </ui-button>
  </div>
  <ui-table
    item-key="id"
    :headers="tableHeaders"
    :items="variables"
    :search="state.query"
    class="mt-4 w-full"
  >
    <template #item-actions="{ item }">
      <v-remixicon
        v-if="showEdit(item)"
        name="riPencilLine"
        class="mr-4 inline-block cursor-pointer"
        @click="editVariable(item)"
      />
      <v-remixicon
        v-if="store.settings.dev"
        name="riDeleteBin7Line"
        class="inline-block cursor-pointer"
        @click="deleteVariable(item)"
      />
    </template>
  </ui-table>
  <ui-modal
    v-model="editState.show"
    :title="`${editState.type === 'edit' ? '编辑' : '添加'}变量`"
  >
    <ui-input v-model="editState.name" placeholder="输入名称" class="w-full" />
    <ui-input
      v-model="editState.label"
      placeholder="输入备注"
      class="w-full mt-4"
    />
    <ui-textarea
      v-model="editState.value"
      placeholder="变量值"
      class="mt-4 w-full"
    />
    <div class="mt-8 text-right">
      <ui-button class="mr-4" @click="editState.show = false">
        {{ t('common.cancel') }}
      </ui-button>
      <ui-button
        :disabled="!editState.name || editState.disabled"
        variant="accent"
        @click="saveVariable"
      >
        {{ t('common.save') }}
      </ui-button>
    </div>
  </ui-modal>
</template>
<script setup>
import { shallowReactive, watch } from 'vue';
import { useI18n } from 'vue-i18n';
import { useToast } from 'vue-toastification';
import { parseJSON } from '@/utils/helper';
import { useLiveQuery } from '@/composable/liveQuery';
import dbStorage from '@/db/storage';
import { useStore } from '@/stores/main';

const store = useStore();
const { t } = useI18n();
const toast = useToast();
const variables = useLiveQuery(() => dbStorage.variables.toArray());

const tableHeaders = [
  {
    value: 'name',
    filterable: true,
    text: t('common.name'),
    attrs: {
      class: 'w-2/12 text-overflow',
    },
  },
  {
    value: 'label',
    filterable: true,
    text: '备注',
    attrs: {
      class: 'w-5/12',
    },
  },
  {
    value: 'value',
    filterable: false,
    text: '变量值',
    attrs: {
      class: 'flex-1 line-clamp',
    },
  },
  {
    value: 'actions',
    filterable: false,
    sortable: false,
    text: '',
    attrs: {
      class: 'w-24',
    },
  },
];

const state = shallowReactive({
  id: '',
  query: '',
});
const editState = shallowReactive({
  type: '',
  name: '',
  label: '',
  value: '',
  show: false,
});

function deleteVariable({ id }) {
  dbStorage.variables.delete(id);
}
function editVariable({ id, name, label, value }) {
  state.id = id;
  editState.name = name;
  editState.label = label;
  editState.value =
    typeof value !== 'string' ? JSON.stringify(value, null, 2) : value;
  editState.type = 'edit';
  editState.show = true;
}
function saveVariable() {
  if (!editState.name) return;

  const trimmedName = editState.name.trim();
  const duplicateName = variables.value.some(
    ({ name, id }) => name.trim() === trimmedName && id !== state.id
  );

  if (duplicateName) {
    toast.error(`You alread add "${trimmedName}" variable`);
    return;
  }

  const varValue = parseJSON(editState.value, editState.value);

  if (editState.type === 'edit') {
    dbStorage.variables
      .update(state.id, {
        value: varValue,
        name: trimmedName,
        label: editState.label,
      })
      .then(() => {
        editState.show = false;
      });
  } else {
    dbStorage.variables
      .add({
        value: varValue,
        name: trimmedName,
        label: editState.label,
      })
      .then(() => {
        editState.show = false;
      });
  }
}
function showEdit({ name }) {
  if (store.settings.dev) {
    return true;
  }
  if (name.indexOf('greet_') !== -1) {
    return false;
  }
  return true;
}

watch(
  () => editState.show,
  (value) => {
    if (value) return;

    state.id = '';
    Object.assign(editState, {
      name: '',
      type: '',
      label: '',
      value: '',
      show: false,
    });
  }
);
</script>
