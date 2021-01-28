<template>
  <div v-if="isDrizzleInitialized" id="app">
    <div class="section">
      <Voting />
    </div>

    <div class="section">
      <Toast />
    </div>
  </div>

  <div v-else>Loading...</div>
</template>

<script>
import Voting from "./Voting";
import Toast from "./Toast";
import { mapGetters } from "vuex";

export default {
  name: "app",
  components: {
    Voting,
    Toast,
  },

  computed: mapGetters("drizzle", ["isDrizzleInitialized"]),
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
