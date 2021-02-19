<template>
  <td>
    <label>
      <template v-if="(typeof link !== 'undefined')">
        <a
          v-if="link"
          :href="link"
          target="_blank"
        >{{placeholder}}</a>
        <span v-else>{{placeholder}}</span>
      </template>
      <span v-else>
        {{placeholder}}<template v-if="unit">, <b>{{unit}}</b></template>
      </span>
      <input
        :type="type"
        :value="value"
        :placeholder="placeholder"
        @input="$emit('update:value', $event.target.value);$emit('updat-storage')"
      >
    </label>
  </td>
</template>

<script>
export default {
  props: {
    type: {
      type: String,
      required: false,
      default: 'text'
    },
    placeholder: {
      type: String,
      required: false
    },
    unit: {
      type: String,
      required: false
    },
    value: {
      type: [String, Number],
      required: false
    },
    link: {
        type: String,
        required: false
    },
  },
}
</script>

<style lang="less" scoped>

</style>
