<template>
  <div class="py-4 sm:py-5">
    <div class="sm:grid sm:grid-cols-3 sm:gap-4">
      <dt class="font-medium text-gray-500 my-auto">
        {{ label }}
      </dt>
      <dd class="flex text-sm text-gray-900 sm:col-span-2">
        <div class="flex-grow">
          <input v-model="value" type="text" :name="field" @keyup.enter="submit(field, value)" class="block w-full border border-gray-300 rounded-md shadow-sm py-2 px-3 focus:outline-none focus:ring-sky-500 focus:border-sky-500 sm:text-sm" />
        </div>
        <div class="mx-5 flex-shrink-0 my-auto">
          <button type="button" class="rounded-md font-medium text-purple-600 hover:text-purple-500 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-purple-500" @click="submit(field, value)">
            Modifier
          </button>
        </div>
      </dd>
    </div>
    <div class="sm:grid sm:grid-cols-3 sm:gap-4" v-if="errorMessage">
      <div class="sm:col-start-2 sm:col-span-2">
        <span class="text-red-500 text-sm px-2">{{ errorMessage }}</span>
      </div>
    </div>
  </div>
</template>

<script>

import { useField, defineRule } from 'vee-validate';


defineRule('required', value => {
  if (!value || !value.length) {
    return 'Ce champ est requis';
  }

  return true;
});

defineRule('email', value => {
  // Field is empty, should pass
  if (!value || !value.length) {
    return true;
  }

  // Check if email
  if (!/[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,4}/.test(value)) {
    return 'Ce champ doit etre un email valide';
  }

  return true;
});

defineRule('exact', (value, count) => {
  // The field is empty so it should pass
  if (!value || !value.length) {
    return true;
  }
  
  const numericValue = value.length;
  if (numericValue != count) {
    return `Ce champ doit contenir exactement ${count} caractères`;
  }

  return true;
});


export default {

  emits: ['submit'],

  props: {
    label: String,
    field: String,
    defaultValue: String,
    rules: String,
  },

  methods: {
    submit(field, value) {
      if (this.errorMessage === undefined) {
        this.$emit('submit', field, value);
      }
    },
  },

  setup(props) {
    const { errorMessage, errors, value } = useField(props.field, props.rules);

    value.value = props.defaultValue;

    return {
      errorMessage,
      errors,
      value,
    };
  },

}

</script>