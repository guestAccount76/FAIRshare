<template>
  <div class="flex h-full w-full max-w-screen-xl flex-col items-center justify-center p-3 pr-5">
    <div class="flex h-full w-full flex-col">
      <span class="text-left text-lg font-medium">
        Select a license that defines the desired conditions for using your software
      </span>

      <el-divider> </el-divider>

      <div>
        <el-form
          :model="licenseForm"
          label-position="top"
          size="large"
          ref="licenseForm"
          @submit.prevent
          hide-required-asterisk
        >
          <el-form-item label="License" required prop="license">
            <template #label>
              <div class="flex">
                <span> License </span>
                <span class="px-1 text-red-500"> * </span>
                <form-help-content
                  popoverContent="<p class='text-sm'> Required. Selected license applies to all of your files <br /> If you want to upload some of your files under different licenses, please do so in separate uploads. <br /> If you cannot find the license you're looking for, include a relevant LICENSE file in your record and choose one of the <span class='italic'> Other </span> licenses available <span class='italic'> (Other (Open), Other (Attribution) </span>, etc.). <br /> The supported licenses in the list are harvested from <a onclick='window.ipcRenderer.send(`open-link-in-browser`, `https://opendefinition.org`)' class='text-url'> opendefinition.org </a> and <a onclick='window.ipcRenderer.send(`open-link-in-browser`, `https://spdx.org`)' class='text-url' > spdx.org </a>.</p>"
                ></form-help-content>
              </div>
            </template>
            <el-select
              v-model="licenseForm.license"
              filterable
              placeholder="Select a license"
              class="w-full"
              @change="licenseChange"
            >
              <el-option
                v-for="item in licenseOptions"
                :key="item.licenseId"
                :label="item.name"
                :value="item.licenseId"
              >
              </el-option>
            </el-select>

            <div class="flex w-full flex-row items-center">
              <span class="mx-1 text-sm italic text-zinc-600"> Suggestions: </span>
              <div class="flex-row">
                <el-tag
                  class="mx-2 cursor-copy transition-all hover:shadow-md"
                  size="small"
                  @click="pickSuggestedLicense('MIT')"
                  :type="licenseForm.license === 'MIT' ? '' : 'info'"
                >
                  MIT
                </el-tag>
                <el-tag
                  class="mx-1 cursor-copy transition-all hover:shadow-md"
                  size="small"
                  @click="pickSuggestedLicense('Apache-2.0')"
                  :type="licenseForm.license === 'Apache-2.0' ? '' : 'info'"
                >
                  Apache-2.0
                </el-tag>
              </div>
            </div>

            <div
              class="hover-underline-animation my-3 flex w-max cursor-pointer flex-row items-center text-primary-600"
              v-if="licenseForm.license != ''"
              @click="openLicenseDetails"
            >
              <span class="text-base font-medium"> Show license details </span>
              <Icon icon="grommet-icons:form-next-link" class="ml-2 h-5 w-5" />
            </div>

            <el-drawer
              v-if="showLicenseDetails"
              v-model="showLicenseDetails"
              :title="licenseTitle"
              direction="rtl"
            >
              <fade-transition>
                <div v-if="loading">
                  <div class="fixed bottom-2 right-3">
                    <Vue3Lottie :animationData="$helix_spinner" :width="80" :height="80" />
                  </div>
                </div>
              </fade-transition>
              <iframe
                sandbox
                :src="licenseHtmlUrl"
                class="h-full w-full transition-all"
                :class="loading ? 'opacity-0' : 'opacity-100'"
                @load="finishLoading"
              ></iframe>
            </el-drawer>
          </el-form-item>

          <div v-if="showLicenseGenerateQuestion">
            <p class="text-base">
              Do you want to create and add a license terms file into your dataset?
            </p>

            <div class="pb-3">
              <el-radio-group v-model="saveLicense" @change="showLicenseEditor">
                <el-radio label="Yes" size="large"> Yes </el-radio>
                <el-radio label="No" size="large"> No </el-radio>
              </el-radio-group>
            </div>
          </div>

          <div v-if="displayLicenseEditor" class="pb-5">
            <p class="py-2">Edit if required and continue</p>

            <v-md-editor
              v-model="draftLicense"
              height="400px"
              left-toolbar="undo redo clear | h bold italic strikethrough quote | ul ol table hr | link"
              right-toolbar="sync-scroll preview fullscreen"
            ></v-md-editor>
          </div>
        </el-form>
      </div>

      <div class="flex w-full flex-row justify-center space-x-4 py-2">
        <router-link
          :to="`/datasets/${this.$route.params.datasetID}/${this.$route.params.workflowID}/Code/createMetadata`"
          class=""
        >
          <button class="primary-plain-button">
            <el-icon><d-arrow-left /></el-icon> Back
          </button>
        </router-link>

        <button v-if="displayLicenseEditor" class="primary-button" @click="generateContinue">
          Generate license and Continue
          <el-icon> <d-arrow-right /> </el-icon>
        </button>

        <button
          v-else
          class="primary-button"
          @click="startCuration"
          id="continue"
          :disabled="licenseDisabled"
        >
          Continue
          <el-icon> <d-arrow-right /> </el-icon>
        </button>
      </div>
    </div>
    <app-docs-link url="curate-and-share/select-a-license" position="bottom-4" />
  </div>
</template>

<script>
import { useDatasetsStore } from "@/store/datasets";
import { useTokenStore } from "@/store/access.js";

import licensesJSON from "@/assets/supplementalFiles/licenses.json";

import { Icon } from "@iconify/vue";
import { ElMessage, ElNotification } from "element-plus";
import axios from "axios";

export default {
  name: "CodePickLicense",
  components: {
    Icon,
  },
  data() {
    return {
      loading: true,
      datasetStore: useDatasetsStore(),
      tokens: useTokenStore(),
      datasetID: this.$route.params.datasetID,
      workflowID: this.$route.params.workflowID,
      dataset: {},
      workflow: {},
      questions: [
        "Is the data being curated in accordance with the standards?",
        "Is the data being curated in accordance with the standards?",
      ],
      licenseForm: {
        license: "",
      },
      rules: {
        license: [
          {
            type: "string",
            required: true,
            message: "Please select your license",
            trigger: "change",
          },
        ],
      },
      licenseHtmlUrl: "",
      licenseTitle: "",
      showLicenseDetails: false,
      loadingLicenseDetails: false,
      licenseOptions: licensesJSON.licenses,
      spinnerGlobal: null,
      showLicenseGenerateQuestion: false,
      originalLicense: "",
      saveLicense: "",
      displayLicenseEditor: false,
      draftLicense: "",
    };
  },
  computed: {
    licenseDisabled() {
      let disabled = false;

      if (this.licenseForm.license === "") {
        return true;
      }

      if (this.showLicenseGenerateQuestion) {
        disabled = true;
      }

      if (disabled && this.saveLicense === "") {
        return true;
      } else if (disabled && this.saveLicense === "No") {
        return false;
      }

      return disabled;
    },
  },
  methods: {
    finishLoading() {
      this.loadingLicenseDetails = false;
      this.loading = false;
    },
    async openLicenseDetails() {
      this.loading = true;

      this.licenseHtmlUrl = "/";
      const licenseId = this.licenseForm.license;

      //get license object
      const licenseObject = await this.licenseOptions.find(
        (license) => license.licenseId === licenseId
      );

      this.licenseHtmlUrl = licenseObject.reference;
      this.licenseTitle = licenseObject.name;

      this.showLicenseDetails = true;
      this.loadingLicenseDetails = true;
    },

    pickSuggestedLicense(license) {
      this.licenseForm.license = license;
      this.licenseChange(license);
    },
    licenseChange(_license) {
      this.showLicenseGenerateQuestion = true;
      this.displayLicenseEditor = false;
      this.saveLicense = "";
      this.draftLicense = "";
    },
    async showLicenseEditor() {
      if (this.saveLicense === "Yes") {
        const licenseId = this.licenseForm.license;

        // get license object
        const licenseObject = this.licenseOptions.find(
          (license) => license.licenseId === licenseId
        );

        const licensejson = licenseObject.detailsUrl;

        const response = await axios
          .get(`${this.$server_url}/utilities/requestjson`, {
            params: {
              url: licensejson,
            },
          })
          .then((response) => {
            return response.data;
          })
          .catch((error) => {
            console.error(error);
            return "ERROR";
          });

        this.draftLicense = response.licenseText;

        this.displayLicenseEditor = true;
      } else {
        this.displayLicenseEditor = false;
      }
    },
    generateContinue() {
      if (this.draftLicense.trim() == "") {
        ElMessage({
          message: "Your license text cannot be empty",
          type: "error",
        });
        return;
      }

      this.dataset.data.Code.questions.license = this.licenseForm.license;
      this.dataset.data.general.questions.license = this.licenseForm.license;

      // turn this to false after license is generated at the end of the workflow
      this.workflow.generateLicense = true;
      this.workflow.createLicenseSelect = this.saveLicense;
      this.workflow.licenseText = this.draftLicense;

      this.datasetStore.updateCurrentDataset(this.dataset);
      this.datasetStore.syncDatasets();

      this.$router.push(`/datasets/${this.datasetID}/${this.workflowID}/selectDestination`);
    },
    startCuration() {
      this.$refs.licenseForm.validate((valid) => {
        console.log(valid);
        if (valid) {
          this.dataset.data.Code.questions.license = this.licenseForm.license;
          this.dataset.data.general.questions.license = this.licenseForm.license;

          this.workflow.createLicenseSelect = this.saveLicense;

          this.datasetStore.updateCurrentDataset(this.dataset);
          this.datasetStore.syncDatasets();

          this.$router.push(`/datasets/${this.datasetID}/${this.workflowID}/selectDestination`);
        }
      });
    },
    async prefillGithubLicense() {
      // get a list of contributors for the repo
      const tokenObject = await this.tokens.getToken("github");
      const GithubAccessToken = tokenObject.token;

      const selectedRepo = this.workflow.github.repo;

      let response = "";

      ElNotification({
        title: "Info",
        message: "Requesting license information from GitHub...",
        position: "top-right",
        type: "info",
      });

      response = await axios
        .get(`${process.env.VUE_APP_GITHUB_SERVER_URL}/repos/${selectedRepo}`, {
          params: {
            accept: "application/vnd.github.v3+json",
          },
          headers: {
            Authorization: `Bearer  ${GithubAccessToken}`,
          },
        })
        .then((response) => {
          return response.data;
        })
        .catch((error) => {
          ElNotification({
            title: "Error",
            message: "Something went wrong.",
            position: "top-right",
            type: "error",
          });

          console.error(error);
          return "ERROR";
        });

      if (
        response !== "ERROR" &&
        "license" in response &&
        response.license != null &&
        "spdx_id" in response.license &&
        (response.license.spdx_id != null || response.license.spdx_id != "")
      ) {
        this.originalLicense =
          this.licenseForm.license =
          this.dataset.data.Code.questions.license =
            response.license.spdx_id;
        // this.licenseForm.license = this.dataset.data.Code.questions.license;
        // this.originalLicense = this.licenseForm.license;

        ElNotification({
          title: "Success",
          message: "License information found in GitHub",
          position: "top-right",
          type: "info",
        });
      } else {
        ElNotification({
          title: "Error",
          message: "Could not find license information in GitHub",
          position: "top-right",
          type: "error",
        });
      }
    },
  },
  async mounted() {
    this.dataset = await this.datasetStore.getCurrentDataset();

    this.workflow = this.dataset.workflows[this.workflowID];

    this.datasetStore.showProgressBar();
    this.datasetStore.setProgressBarType("zenodo");
    this.datasetStore.setCurrentStep(4);

    this.workflow.currentRoute = this.$route.path;

    if (
      "general" in this.dataset.data &&
      "questions" in this.dataset.data.general &&
      "license" in this.dataset.data.general.questions
    ) {
      this.licenseForm.license = this.dataset.data.general.questions.license;
      this.originalLicense = this.licenseForm.license;
      this.showLicenseGenerateQuestion = true;
      this.saveLicense = this.workflow.createLicenseSelect;

      if (this.saveLicense === "Yes") {
        this.displayLicenseEditor = true;
        this.draftLicense = this.workflow.licenseText;
      } else {
        this.displayLicenseEditor = false;
      }
    } else {
      if ("source" in this.workflow) {
        if (this.workflow.source.type === "github") {
          await this.prefillGithubLicense();
        }
        if (this.workflow.source.type === "local") {
          this.licenseForm.license = "";
          this.originalLicense = "";
        }
      } else {
        this.licenseForm.license = "";
        this.originalLicense = "";
      }
    }
  },
};
</script>
