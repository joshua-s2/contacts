<template>
  <div rounded="lg" class="pa-8 fill-height">
    <v-progress-linear
      class="progress"
      background-color="secondary"
      rounded
      indeterminate
      height="10"
      :query="true"
    ></v-progress-linear>
    <h2 variant="text-h4 text-center mt-4">
      Loading...
    </h2>
  </div>
</template>
<style lang="scss" scoped>
.progress {
  border-radius: 9px;
}
</style>
