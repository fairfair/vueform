
<template>
  <div class="py-2 sm:py-3">
    <div class="sm:grid sm:grid-cols-3 sm:gap-4">
      <dt class="font-medium text-gray-500 my-auto" :class="labelTop ? 'col-span-3' : 'col-span-1'">
        {{ label }}
      </dt>
      <dd class="flex text-sm text-gray-900" :class="labelTop ? 'col-span-3' : 'col-span-2'">
        <div class="flex-grow range-wrap">
          <input v-model="value" type="range" :min="min" :max="max" :name="field" @keyup.enter="submit(field, value)" class="block w-full range py-2 px-3 sm:text-sm" />
          <output class="bubble"></output>
        </div>
        <div class="mx-5 flex-shrink-0 my-auto" v-if="submitButton">
          <button type="button" class="rounded-md font-semibold text-blue-600 hover:text-blue-500 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-purple-500" @click="submit(field, value)">
            Enregistrer
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
    min: Number,
    max: Number,
    defaultValue: Number,
    labelTop: {
      type: Boolean,
      default: false
    },
    rules: String,
    submitButton: {
      type: Boolean,
      default: true,
    },
  },

  mounted() {
    const allRanges = document.querySelectorAll(".range-wrap");
    allRanges.forEach(wrap => {
      const range = wrap.querySelector(".range");
      const bubble = wrap.querySelector(".bubble");

      range.addEventListener("input", () => {
        setBubble(range, bubble);
      });
      setBubble(range, bubble);
    });

    function setBubble(range, bubble) {
      const val = range.value;
      const min = range.min ? range.min : 0;
      const max = range.max ? range.max : 100;
      const newVal = Number(((val - min) * 100) / (max - min));
      bubble.innerHTML = val;

      // Sorta magic numbers based on size of the native UI thumb
      bubble.style.left = `calc(${newVal}% + (${8 - newVal * 0.15}px))`;
    }
  },

  methods: {
    submit(field, value) {
      if (this.errorMessage === undefined) {
        this.$emit('submit', field, parseInt(value));
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

<style>
.range-wrap {
  position: relative;
}
.range {
  width: 100%;
}
.bubble {
  background: black;
  color: white;
  padding: 4px 12px;
  position: absolute;
  border-radius: 4px;
  left: 50%;
  transform: translateX(-50%);
}
.bubble::after {
  content: "";
  position: absolute;
  width: 2px;
  height: 2px;
  background: black;
  top: -1px;
  left: 50%;
}
</style>
