<template>
  <div id="app">
    <button
      v-if="updateExists"
      @click="refreshApp"
      class="btn"
    >New version available! Click to update</button>
    <div v-else>There's no update yet.</div>
  </div>
</template>

<script>
export default {
  name: "app",
  data() {
    return {
      refreshing: false,
      worker: null,
      updateExists: false
    };
  },

  created() {
    // Listen for swUpdated event and display refresh snackbar as required.
    document.addEventListener("swUpdated", this.showRefreshUI, { once: true });
    // Refresh all open app tabs when a new service worker is installed.
    navigator.serviceWorker &&
      navigator.serviceWorker.addEventListener(
        //triggered by registration.claim
        "controllerchange",
        () => {
          if (this.refreshing) return;
          this.refreshing = true;
          console.log("controllerchange triggered, -> auto refresh!!");
          window.location.reload();
        }
      );
  },

  methods: {
    showRefreshUI(e) {
      // Display a button inviting the user to refresh/reload the app due
      // to an app update being available.
      // The new service worker is installed, but not yet active.
      // Store the ServiceWorkerRegistration instance for later use.
      if (e.detail) {
        this.worker = e.detail;
        this.updateExists = true;
        console.log("Showing refresh button.");
        console.log(this.worker);
      } else {
        console.warn("No worker data found!");
      }
    },
    refreshApp() {
      console.log("skipWaiting started");
      console.log(this.worker);
      // Handle a user tap on the update app button.
      this.updateExists = false;
      // Protect against missing registration.waiting.
      if (!this.worker) {
        console.warn("No worker data found when trying to refresh!");
        return;
      }
      this.worker.postMessage({ type: "SKIP_WAITING" });
      console.log("skipWaiting finished");
    }
  }
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e51;
  margin-top: 60px;
}

.btn {
  border: 1px solid #3a3a3a;
  border-radius: 4px;
  padding: 0.75rem;
  background-color: #3a3a3a;
  color: #fafafa;
  font-size: 1rem;
  font-family: "Avenir", Helvetica, Arial, sans-serif;
}
</style>
