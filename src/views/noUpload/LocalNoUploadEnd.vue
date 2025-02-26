<template>
  <div class="flex h-full w-full flex-col items-center justify-center p-3 pr-5">
    <div class="flex h-full w-full flex-col">
      <span class="text-left text-lg font-medium"> Final step </span>
      <span class="text-left"> All your requested data has been generated. </span>

      <el-divider class="my-4"> </el-divider>

      <div class="flex h-full flex-col items-center justify-center px-10">
        <p class="pb-5 text-center">
          It does not look like you have selected a data repository to upload to. This is not
          recommended if you are trying to make your dataset FAIR. You can come back to this page
          later and select a repository to make your dataset completely FAIR.
        </p>
        <div class="flex space-x-4">
          <router-link :to="`/datasets`">
            <button class="primary-plain-button">
              <el-icon><data-line /></el-icon> Go to the homepage
            </button>
          </router-link>
          <router-link :to="`/datasets/${datasetID}/${workflowID}/selectDestination`">
            <button class="secondary-plain-button">
              <el-icon><upload-filled /></el-icon> Upload to a repository
            </button>
          </router-link>
          <button class="primary-button" @click="openFileExplorer">
            <el-icon><star-icon /></el-icon> View generated files
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { useDatasetsStore } from "@/store/datasets";

import axios from "axios";

export default {
  name: "LocalNoUploadEnd",
  data() {
    return {
      datasetStore: useDatasetsStore(),
      dataset: {},
      folderPath: "",
      datasetID: this.$route.params.datasetID,
      workflowID: this.$route.params.workflowID,
      workflow: {},
      zenodoToken: "",
      published: false,
      zenodoDatasetID: "",
    };
  },
  computed: {},
  methods: {
    async sleep(ms) {
      return new Promise((resolve) => setTimeout(resolve, ms));
    },

    async openFileExplorer() {
      await axios
        .post(`${this.$server_url}/utilities/openfileexplorer`, {
          file_path: this.dataset.meta.locationPath,
        })
        .then((response) => {
          return response.data;
        })
        .catch((error) => {
          console.error(error);
          return "ERROR";
        });
    },
  },
  async mounted() {
    this.dataset = await this.datasetStore.getCurrentDataset();
    this.workflow = this.dataset.workflows[this.workflowID];

    this.datasetStore.showProgressBar();
    this.datasetStore.setProgressBarType("zenodo");
    this.datasetStore.setCurrentStep(7);

    this.workflow.currentRoute = this.$route.path;
  },
};
</script>

<style lang="postcss" scoped>
.blob {
  box-shadow: 0 0 0 0 rgba(52, 172, 224, 1);
  animation: pulse-blue 2s infinite;
}

@keyframes pulse-blue {
  0% {
    transform: scale(0.95);
    box-shadow: 0 0 0 0 rgba(52, 172, 224, 0.7);
  }

  70% {
    transform: scale(1);
    box-shadow: 0 0 0 10px rgba(52, 172, 224, 0);
  }

  100% {
    transform: scale(0.95);
    box-shadow: 0 0 0 0 rgba(52, 172, 224, 0);
  }
}
</style>
