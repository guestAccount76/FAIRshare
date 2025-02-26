<template>
  <!-- <div class="container mt-2">Current App Path: {{ appPath }}</div> -->
  <div class="flex flex-row bg-white">
    <AppSidebar :environment="environment"></AppSidebar>

    <AppContent>
      <router-view v-slot="{ Component }" class="pt-2">
        <transition name="fade" appear mode="out-in">
          <component :is="Component" />
        </transition>
      </router-view>
    </AppContent>

    <div
      class="fixed bottom-4 right-4 flex w-[230px] flex-col items-center justify-center rounded-lg border border-zinc-300 bg-white px-4 py-2 shadow-xl"
      v-if="showDownloadingMessage"
    >
      <div
        class="absolute top-2 right-2 cursor-pointer text-zinc-400 transition-all hover:text-zinc-700"
        @click="closeNotification"
      >
        <el-icon><circle-close-filled /></el-icon>
      </div>
      <Vue3Lottie
        animationLink="https://assets5.lottiefiles.com/private_files/lf30_t26law.json"
        :width="100"
        :height="100"
      />
      <p class="text-center text-sm">FAIRshare is downloading the latest version of the app...</p>
    </div>

    <div
      class="fixed bottom-4 right-4 flex w-[250px] flex-col items-center justify-center rounded-lg border border-zinc-300 bg-white px-6 py-3 shadow-xl"
      v-if="showRestartMessage"
    >
      <div
        class="absolute top-2 right-2 cursor-pointer text-zinc-400 transition-all hover:text-zinc-700"
        @click="closeNotification"
      >
        <el-icon><circle-close-filled /></el-icon>
      </div>
      <Vue3Lottie
        animationLink="https://assets8.lottiefiles.com/packages/lf20_dv9qkzg7.json"
        :width="50"
        :height="50"
      />
      <p class="py-3 text-center text-sm">
        {{
          platform == "darwin"
            ? "Update downloaded. It will be installed when you close and relaunch the app."
            : "Restart FAIRshare to install the latest version of the app."
        }}
      </p>
      <button class="primary-plain-button py-1 px-2" @click="restartAppForUpdate">
        {{ platform == "darwin" ? "Close FAIRshare" : "Restart FAIRshare" }}
      </button>
    </div>
  </div>
</template>

<script>
import AppSidebar from "./components/ui/AppSidebar.vue";
import AppContent from "./components/ui/AppContent.vue";

import { app } from "@electron/remote";
import semver from "semver";
import axios from "axios";
import axiosRetry from "axios-retry";
import { ElLoading } from "element-plus";
import Mousetrap from "mousetrap";

import { useDatasetsStore } from "./store/datasets";
import { useTokenStore } from "./store/access.js";
import { useConfigStore } from "./store/config.js";

const MIN_API_VERSION = "0.0.1";

export default {
  name: "App",
  components: {
    AppSidebar,
    AppContent,
  },
  data() {
    return {
      appPath: app.getAppPath(),
      allDatasets: useDatasetsStore(),
      config: useConfigStore(),
      tokens: useTokenStore(),
      loading: "",
      environment: "",
      showDownloadingMessage: false,
      showRestartMessage: false,
      platform: process.platform,
    };
  },
  methods: {
    async loadStores() {
      try {
        // Load all the projects
        await this.allDatasets.loadDatasets();

        // Load all the access tokens
        await this.tokens.loadTokens();

        // Load app config
        await this.config.loadConfig();

        // Run all the integrations checks
        this.tokens.verifyAllConnections();

        this.loading.close();
      } catch (error) {
        console.error("Error with loading stores...");
        console.error(error);
        this.loading.close();
      }
    },
    restartAppForUpdate() {
      window.ipcRenderer.send("restart-fairshare-for-update");
    },
    closeNotification() {
      this.showDownloadingMessage = false;
      this.showRestartMessage = false;
    },
  },
  mounted() {
    this.$track("App Launched", "OS", process.platform);
    this.$track("App Launched", "Version", app.getVersion());
    this.$track("App Launched", "Arch", process.arch);

    // disable the mouse back and forward buttons
    window.addEventListener("mouseup", (e) => {
      if (e.button === 3 || e.button === 4) {
        e.preventDefault();
      }
    });

    // disable the refresh button on macOS
    Mousetrap.bind("command+r", function () {
      if (process.env.NODE_ENV !== "development") {
        return false;
      }
    });

    // disable the refresh button on Windows
    Mousetrap.bind("ctrl+r", function () {
      if (process.env.NODE_ENV !== "development") {
        return false;
      }
    });

    const client = axios.create({ baseURL: `${this.$server_url}` });
    axiosRetry(client, { retries: 3 });

    this.loading = ElLoading.service({
      lock: true,
      text: "Connecting to backend systems and loading data...",
      background: "rgba(255, 255, 255, 0.95)",
    });

    client
      .get("/echo", {
        "axios-retry": {
          retries: 5,
          retryDelay: axiosRetry.exponentialDelay,
        },
      })
      .then((response) => {
        console.log(`Server Status: ${response.data}`);
        axios
          .get(`${this.$server_url}/api_version`)
          .then((response) => {
            if (semver.lte(semver.clean(MIN_API_VERSION), semver.clean(response.data))) {
              console.log("API version satisfied");

              this.loadStores();
            } else {
              // will need to change this to a more user friendly message
              alert("Invalid API version");
              this.loading.close();
            }
          })
          .catch((error) => {
            console.error(error);
          });
      })
      .catch((error) => {
        console.error(error);
      });

    axios
      .get(`${this.$server_url}/zenodo/env`)
      .then((response) => {
        if (response.data.search("sandbox") === -1) {
          this.environment = "production";
        } else {
          this.environment = "sandbox";
        }
      })
      .catch((error) => {
        console.error(error);
      });

    window.ipcRenderer.on("update-available", (_e, _arg) => {
      this.showDownloadingMessage = true;
      this.$track("App Update", "Update available", `${app.getVersion()}`);
      window.ipcRenderer.removeAllListeners("update-available");
    });

    window.ipcRenderer.on("update-downloaded", (_e, _arg) => {
      this.showDownloadingMessage = false;
      this.showRestartMessage = true;
      this.$track("App Update", "Update downloaded", `${app.getVersion()}`);
      window.ipcRenderer.removeAllListeners("update-downloaded");
    });

    console.log(`Current app path: ${this.appPath}`);

    this.$router.push({
      path: "/home",
    });
  },
};
</script>
