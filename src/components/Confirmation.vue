<template>
  <div>
    <b-alert varian="success" show>{{ message }}</b-alert>
  </div>
</template>

<script>
export default {
  props: ['message'],
};
</script>
