<template>
  <div>
    <bounce-loader :loading="isLoading" :color="'#68d391'" :size="100" />
    <AssetsTable v-if="!isLoading" :assets="assets" />
  </div>
</template>

<script>
import api from "@/api";
import AssetsTable from "@/components/AssetsTable";

export default {
  name: "Home",
  components: {
    AssetsTable
  },
  data() {
    return {
      assets: [],
      isLoading: false
    };
  },
  created() {
    this.isLoading = true;
    api
      .getAssets()
      .then(assets => (this.assets = assets))
      .finally(() => (this.isLoading = false));
  }
};
</script>
