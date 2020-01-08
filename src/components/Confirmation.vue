<template>
  <div>
    <b-alert
      variant="success"
      dismissible
      v-model="view"
      @dismissed='close()'
    >
      {{ message }}
    </b-alert>
  </div>
</template>

<script>
export default {
  props: ['message', 'value'],
  data() {
    return {
      view: false,
    };
  },
  methods: {
    close() {
      this.$emit('input', false);
    },
  },
  watch: {
    value() {
      this.view = this.value;
    },
  },
  mounted() {
    this.view = this.value;
  },
};
</script>
