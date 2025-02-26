<template>
  <div
    class="relative flex h-full w-full max-w-screen-xl flex-col items-center justify-center p-3 px-5"
    v-loading="loading"
  >
    <div class="flex h-full w-full flex-col items-start justify-start">
      <span class="text-left text-lg font-medium">
        Select a FAIR repository to share your research software
      </span>

      <el-divider class="my-4"> </el-divider>

      <div class="w-full">
        <span class="mb-2 w-full">
          Would you like to upload your dataset to a repository through FAIRshare?
        </span>

        <div class="py-1">
          <el-radio v-model="uploadToRepo" label="Yes" size="large"> Yes </el-radio>
          <el-radio v-model="uploadToRepo" label="No" size="large"> No </el-radio>
          <el-radio v-model="uploadToRepo" label="None" size="large" border class="!hidden">
            None
          </el-radio>
        </div>
      </div>

      <div v-if="uploadToRepo !== 'None'" class="flex h-full w-full flex-col items-center">
        <transition name="fade" mode="out-in" appear>
          <div v-if="uploadToRepo === 'Yes'" class="flex h-full w-full flex-col items-center">
            <line-divider class="w-full py-3"></line-divider>

            <span class="text-center">
              Based on your dataset requirements, we suggest uploading your data to one of these
              repositories.
            </span>
            <span class="text-center text-sm"> Please click one of the following options: </span>

            <div class="my-8 grid grid-cols-3 gap-8">
              <div>
                <div class="flex flex-col items-center justify-center">
                  <div
                    class="single-check-box flex h-[200px] w-[200px] cursor-pointer flex-col items-center justify-evenly rounded-lg p-4 shadow-md transition-all"
                    :class="{ 'selected-repo': repoID === 'zenodo' }"
                    @click="selectRepo($event, 'zenodo')"
                  >
                    <img
                      src="https://api.iconify.design/simple-icons/zenodo.svg"
                      alt=""
                      class="mb-3 h-16 w-16"
                    />
                    <span class="mx-5 text-lg"> Zenodo </span>
                  </div>
                  <div
                    class="hover-underline-animation my-5 flex w-max cursor-pointer flex-row items-center text-primary-600"
                    v-if="repoID === 'zenodo'"
                    @click="openWebsite('https://zenodo.org')"
                  >
                    <span class="font-medium"> Learn more... </span>
                    <Icon icon="grommet-icons:form-next-link" class="ml-2 h-5 w-5" />
                  </div>
                </div>
              </div>

              <div>
                <el-popover placement="bottom" trigger="hover" content="Coming soon...">
                  <template #reference>
                    <div>
                      <div
                        class="disabled-card single-check-box pointer-events-none flex h-[200px] w-[200px] cursor-pointer flex-col items-center justify-evenly rounded-lg p-4 text-stone-400 shadow-md transition-all"
                        :class="{ 'selected-repo': repoID === 'figshare' }"
                        @click="selectRepo($event, 'figshare')"
                      >
                        <img
                          src="https://www.digital-science.com/wp-content/uploads/2020/11/Figshare-no-padding.png"
                          alt=""
                          class="mb-3 h-16 opacity-50"
                        />
                        <span class="mx-5 text-lg"> Figshare </span>
                      </div>
                    </div>
                  </template>
                </el-popover>
              </div>

              <div>
                <el-popover placement="bottom" trigger="hover" content="Coming soon...">
                  <template #reference>
                    <div>
                      <div
                        class="disabled-card single-check-box pointer-events-none flex h-[200px] w-[200px] cursor-pointer flex-col items-center justify-evenly rounded-lg p-4 text-stone-400 shadow-md transition-all hover:shadow-lg"
                        :class="{
                          'selected-repo': repoID === 'softwareheritage',
                        }"
                        @click="selectRepo($event, 'softwareheritage')"
                      >
                        <img
                          src="https://www.softwareheritage.org/wp-content/uploads/2015/08/swh-logo.png"
                          alt=""
                          class="mb-3 h-16 opacity-50"
                        />
                        <span class="mx-5 w-full text-center text-lg"> Software Heritage </span>
                      </div>
                    </div>
                  </template>
                </el-popover>
              </div>
            </div>

            <fade-transition>
              <div
                v-if="repoID === 'zenodo' && workflow.source.type === 'local'"
                class="mb-8 flex flex-col"
              >
                <p class="mb-4">
                  Is your dataset already published on Zenodo or would you like to create a new
                  Zenodo publication?
                </p>

                <div class="flex flex-row items-center justify-center">
                  <el-radio
                    v-model="newVersion"
                    label="false"
                    size="large"
                    border
                    @change="selectZenodoDeposition"
                  >
                    Create a new Zenodo deposition
                  </el-radio>
                  <el-radio
                    v-model="newVersion"
                    label="true"
                    size="large"
                    border
                    @change="selectZenodoDeposition"
                  >
                    Create a new version of an existing Zenodo deposition
                  </el-radio>
                </div>
              </div>
            </fade-transition>

            <fade-transition>
              <ConnectZenodo
                v-if="
                  repoID === 'zenodo' && showSelectZenodoDeposition && !validZenodoTokenAvailable
                "
                :statusChangeFunction="showConnection"
              ></ConnectZenodo>
            </fade-transition>

            <fade-transition>
              <div v-if="showSelectZenodoDeposition && validZenodoTokenAvailable">
                <!-- Add a card style background here -->

                <div class="flex flex-col items-center">
                  <div
                    class="mt-4 mb-8 flex w-full flex-col items-start justify-center rounded-xl border border-zinc-100 px-4 py-4 shadow-xl"
                    v-if="'id' in selectedDeposition"
                  >
                    <h3 class="font-semibold">Currently selected Zenodo deposition</h3>
                    <line-divider class="my-1 w-full" />
                    <p class="py-1 text-base">
                      <span class="font-medium">Title:</span>
                      {{ selectedDeposition.metadata.title }}
                    </p>
                    <p class="py-1 text-base">
                      <span class="font-medium">Last published:</span>
                      {{ selectedDeposition.metadata.publication_date }}
                    </p>
                    <p class="py-1 text-base">
                      <span class="font-medium">DOI: </span>
                      <span @click="openWebsite(selectedDeposition.doi_url)" class="text-url">
                        {{ selectedDeposition.doi_url }}
                      </span>
                    </p>
                  </div>
                  <button
                    class="secondary-plain-button mb-5 w-max"
                    @click="showZenodoDepositionSelectorModal"
                  >
                    Select
                    {{ "id" in selectedDeposition ? "another" : "" }} Zenodo deposition
                  </button>
                </div>
              </div>
            </fade-transition>

            <general-dialog
              ref="generalDialog"
              title="Select your Zenodo deposition"
              @messageConfirmed="selectedDeposition = selectedDepositionFromList"
            >
              <div class="mt-4 flex flex-col">
                <fade-transition>
                  <div v-if="loadedZenodoDepositions">
                    <Listbox v-model="selectedDepositionFromList" v-if="loadedZenodoDepositions">
                      <div class="relative mt-1">
                        <ListboxButton
                          class="relative h-[45px] w-full cursor-default rounded-lg border border-gray-200 bg-white py-2 pl-3 pr-10 text-left shadow-md focus:outline-none sm:text-sm"
                        >
                          <span class="block truncate">{{
                            selectedDepositionFromList.metadata.title
                          }}</span>
                          <span
                            class="pointer-events-none absolute inset-y-0 right-0 flex items-center pr-2"
                          >
                            <Icon
                              class="h-6 w-6 text-gray-400"
                              icon="heroicons-outline:selector"
                              aria-hidden="true"
                            />
                          </span>
                        </ListboxButton>

                        <transition
                          enter-active-class="transition duration-100 ease-in"
                          enter-to-class="opacity-100"
                          enter-from-class="opacity-0"
                          leave-active-class="transition duration-100 ease-in"
                          leave-from-class="opacity-100"
                          leave-to-class="opacity-0"
                        >
                          <ListboxOptions
                            class="absolute mt-1 max-h-60 w-full overflow-auto rounded-md bg-white py-1 text-base shadow-lg ring-1 ring-black ring-opacity-5 focus:outline-none sm:text-sm"
                          >
                            <ListboxOption
                              v-slot="{ active, selected }"
                              v-for="deposition in filteredDepositions"
                              :key="deposition.id"
                              :value="deposition"
                              as="template"
                            >
                              <li
                                :class="[
                                  active ? 'bg-zinc-100 text-zinc-900' : 'text-gray-900',
                                  selected ? 'bg-secondary-100 text-gray-900' : 'text-gray-900',
                                  'relative cursor-default select-none border-t border-b border-zinc-50 py-2 pl-10 pr-4',
                                ]"
                              >
                                <div class="flex flex-col">
                                  <div class="flex pb-1">
                                    <span
                                      :class="[
                                        selected ? 'font-medium' : 'font-normal',
                                        'mr-2 block truncate ',
                                      ]"
                                    >
                                      {{ deposition.title }}
                                    </span>
                                    <el-tag size="small">
                                      {{ deposition.metadata.upload_type }}
                                    </el-tag>
                                  </div>
                                  <span
                                    :class="[
                                      selected ? 'font-medium' : 'font-normal',
                                      'block truncate text-xs',
                                    ]"
                                  >
                                    {{ deposition.doi }}
                                  </span>
                                </div>
                                <span
                                  v-if="selected"
                                  class="absolute inset-y-0 left-0 flex items-center pl-3 text-secondary-600"
                                >
                                  <el-icon size="18"><select-icon /></el-icon>
                                </span>
                              </li>
                            </ListboxOption>
                          </ListboxOptions>
                        </transition>
                      </div>
                    </Listbox>
                    <div class="mt-4 px-4">
                      <line-divider class="my-2" />
                      <p class="py-1 text-sm">
                        <span class="font-medium">Title:</span>
                        {{ selectedDepositionFromList.metadata.title }}
                      </p>
                      <p class="py-1 text-sm">
                        <span class="font-medium">Last published:</span>
                        {{ selectedDepositionFromList.metadata.publication_date }}
                      </p>
                      <p class="py-1 text-sm">
                        <span class="font-medium">DOI: </span>
                        <span
                          @click="openWebsite(selectedDepositionFromList.doi_url)"
                          class="text-url"
                        >
                          {{ selectedDepositionFromList.doi_url }}
                        </span>
                      </p>
                      <p class="py-1 text-sm">
                        <span class="font-medium">Description:</span>
                        {{ selectedDepositionFromList.metadata.description }}
                      </p>
                      <p class="py-1 text-sm">
                        <span class="font-medium">Upload type:</span>
                        {{ selectedDepositionFromList.metadata.upload_type }}
                      </p>
                      <p class="py-1 text-sm">
                        <span class="font-medium">License:</span>
                        {{ selectedDepositionFromList.metadata.license }}
                      </p>
                    </div>
                  </div>
                  <div v-else class="flex h-full w-full items-center justify-center">
                    <Vue3Lottie :animationData="$helix_spinner" :width="200" :height="200" />
                  </div>
                </fade-transition>
                <fade-transition> </fade-transition>
              </div>
            </general-dialog>

            <fade-transition>
              <div
                class="w-max-content flex flex-row justify-center space-x-4 py-6"
                v-if="newVersion === 'false'"
              >
                <router-link
                  :to="`/datasets/${this.$route.params.datasetID}/${this.$route.params.workflowID}/Code/pickLicense`"
                  class=""
                >
                  <button class="primary-plain-button">
                    <el-icon><d-arrow-left /></el-icon> Back
                  </button>
                </router-link>

                <button
                  class="primary-button"
                  @click="addMetadata"
                  :disabled="repoID === ''"
                  id="continue"
                >
                  Continue <el-icon> <d-arrow-right /> </el-icon>
                </button>
              </div>
            </fade-transition>
            <fade-transition>
              <div
                class="w-max-content flex flex-row justify-center space-x-4 py-6"
                v-if="
                  newVersion === 'true' &&
                  'metadata' in selectedDeposition &&
                  Object.keys(selectedDeposition.metadata).length !== 0
                "
              >
                <router-link
                  :to="`/datasets/${this.$route.params.datasetID}/${this.$route.params.workflowID}/Code/pickLicense`"
                  class=""
                >
                  <button class="primary-plain-button">
                    <el-icon><d-arrow-left /></el-icon> Back
                  </button>
                </router-link>

                <button
                  class="primary-button"
                  @click="saveSelectedVersionDetails"
                  :disabled="repoID === ''"
                  id="continueNewVersion"
                >
                  Continue <el-icon> <d-arrow-right /> </el-icon>
                </button>
              </div>
            </fade-transition>
          </div>
          <div v-else class="flex w-full justify-center space-x-4 px-5 py-4">
            <button class="primary-plain-button" size="medium" @click="saveSkip">
              <el-icon><d-arrow-left /></el-icon>
              Back
            </button>

            <button class="primary-button" @click="showRepoUploadWarning">
              Continue
              <el-icon><d-arrow-right /></el-icon>
            </button>
            <warning-confirm
              ref="warningConfirmSkipUploadToRepo"
              title="Warning"
              @messageConfirmed="skipUploadToRepo"
              confirmButtonText="Yes, I want to skip"
            >
              <p class="text-center text-base text-gray-500">
                Are you sure you want to skip uploading this dataset to a data repository? This is
                not recommended according to the FAIR guidelines.
              </p>
            </warning-confirm>
          </div>
        </transition>
      </div>
    </div>
    <app-docs-link url="curate-and-share/select-github-repo" position="bottom-4" />
    <error-confirm
      ref="errorConfirmNoAdminPermission"
      title="Invalid permissions"
      :showCancelButton="false"
    >
      <p class="text-center text-base text-gray-500">
        This repository does not have admin permissions for the currently authenticated user. Please
        contact the repository owner to grant admin permission to you.
      </p>
    </error-confirm>
  </div>
</template>

<script>
import ConnectZenodo from "@/components/serviceIntegration/ConnectZenodo";

import { useDatasetsStore } from "@/store/datasets";
import { useTokenStore } from "@/store/access";

import { Icon } from "@iconify/vue";
import axios from "axios";
import _ from "lodash";
import { Listbox, ListboxButton, ListboxOptions, ListboxOption } from "@headlessui/vue";

export default {
  name: "SelectRepositoryDestination",
  components: { Icon, Listbox, ListboxButton, ListboxOptions, ListboxOption, ConnectZenodo },
  data() {
    return {
      datasetStore: useDatasetsStore(),
      tokens: useTokenStore(),
      dataset: {},
      workflowID: this.$route.params.workflowID,
      datasetID: this.$route.params.datasetID,
      workflow: {},
      zenodoToken: "",
      loading: false,
      repoID: "",
      showSelectZenodoDeposition: false,
      loadedZenodoDepositions: false,
      showZenodoDepositionSelector: false,
      filteredDepositions: [],
      selectedDepositionFromList: {
        metadata: {},
      },
      selectedDeposition: { metadata: {} },
      uploadToRepo: "None",
      newVersion: "",
      loadingSpinner: false,
      validZenodoTokenAvailable: false,
    };
  },
  computed: {},
  methods: {
    selectRepo(event, repoID) {
      if (this.workflow.source.type === "github") {
        this.newVersion = "false";

        console.log(this.workflow.github.fullObject.originalObject.permissions.admin);

        if (!this.workflow.github.fullObject.originalObject.permissions.admin) {
          console.log("No admin permissions on this repo");
          this.$refs.errorConfirmNoAdminPermission.show();

          return;
        }
      }

      this.repoID = repoID;

      if (event && event.detail === 2) {
        this.addMetadata();
      }
    },
    async showConnection(status) {
      console.log(status);
      if (status === "connected") {
        this.validZenodoTokenAvailable = true;
      }
    },
    showZenodoDepositionSelectorModal() {
      this.loadingSpinner = true;
      this.$refs.generalDialog.show();
      this.getAllDepositions();
    },
    saveSelectedVersionDetails() {
      this.addMetadata("zenodo-new-version");
    },
    async getAllDepositions() {
      const tokenObject = await this.tokens.getToken("zenodo");
      const zenodoToken = tokenObject.token;

      this.loadedZenodoDepositions = false;
      const response = await axios
        .get(`${this.$server_url}/zenodo/depositions`, {
          params: {
            access_token: zenodoToken,
          },
        })
        .then((response) => {
          return response.data;
        })
        .catch((error) => {
          console.error(error);
          return "ERROR";
        });

      this.filteredDepositions = response.filter((deposition) => {
        return deposition.submitted === true;
      });

      this.selectedDepositionFromList = this.filteredDepositions[0];

      if (!_.isEqual(this.selectedDeposition, { metadata: {} })) {
        if (!_.isEqual(this.selectedDeposition, this.selectedDepositionFromList)) {
          this.selectedDepositionFromList = this.selectedDeposition;
        }
      }

      this.loadingSpinner = false;
      this.showZenodoDepositionSelector = true;
      this.loadedZenodoDepositions = true;
    },
    async selectZenodoDeposition() {
      if (this.newVersion === "true") {
        if (this.validZenodoTokenAvailable) {
          this.getAllDepositions();
        }
        this.showSelectZenodoDeposition = true;
      } else {
        this.showSelectZenodoDeposition = false;
      }
    },
    openWebsite(url) {
      window.ipcRenderer.send("open-link-in-browser", url);
    },
    addMetadata(type) {
      this.dataset.destinationSelected = true;

      if (!("destination" in this.workflow)) {
        this.workflow.destination = {};
      }

      if (!this.workflow.destination[this.repoID]) {
        this.workflow.destination[this.repoID] = {
          id: this.repoID,
          questions: {},
          status: {},
        };
      }

      if (this.workflow.destination.name === this.repoID) {
        //do nothing
        this.workflow.destination.name = this.repoID;
      } else {
        // warn the user that they are changing repos (add a sweetalert or something)
        this.workflow.destination.name = this.repoID;
      }

      this.dataset.meta.destination = this.repoID;

      if (this.uploadToRepo === "Yes") {
        this.workflow.uploadToRepo = true;
      } else {
        this.workflow.uploadToRepo = false;
      }

      if (type === "zenodo-new-version") {
        this.workflow.destination[this.repoID].selectedDeposition = this.selectedDeposition;
      }

      if (this.newVersion === "true") {
        this.workflow.destination[this.repoID].newVersion = true;
      } else {
        this.workflow.destination[this.repoID].newVersion = false;
      }

      this.datasetStore.updateCurrentDataset(this.dataset);
      this.datasetStore.syncDatasets();

      // using the zenodo one here. I don't think this matters since we aren't uploading anything
      // Might change it to its own page later
      const routerPath = `/datasets/${this.datasetID}/${this.workflowID}/zenodo/metadata`;
      this.$router.push(routerPath);
    },
    async saveSkip() {
      if (this.uploadToRepo === "Yes") {
        this.workflow.uploadToRepo = true;
      } else {
        this.workflow.uploadToRepo = false;
      }

      await this.datasetStore.updateCurrentDataset(this.dataset);
      await this.datasetStore.syncDatasets();

      const routerPath = `/datasets/${this.datasetID}/${this.workflowID}/Code/pickLicense`;
      this.$router.push(routerPath);
    },
    showRepoUploadWarning() {
      this.$refs.warningConfirmSkipUploadToRepo.show();
    },
    async skipUploadToRepo() {
      if (this.uploadToRepo === "Yes") {
        this.workflow.uploadToRepo = true;
      } else {
        this.workflow.uploadToRepo = false;
      }

      await this.datasetStore.updateCurrentDataset(this.dataset);
      await this.datasetStore.syncDatasets();

      let routerPath = "";

      if ("source" in this.workflow) {
        if (this.workflow.source.type === "github") {
          routerPath = `/datasets/${this.datasetID}/${this.workflowID}/github/upload`;
        }
        if (this.workflow.source.type === "local") {
          routerPath = `/datasets/${this.dataset.id}/${this.workflowID}/localNoUpload/summary`;
        }
      } else {
        routerPath = `/datasets/${this.dataset.id}/${this.workflowID}/localNoUpload/summary`;
      }

      this.$router.push({ path: routerPath });
    },
  },
  async mounted() {
    this.loading = true;

    this.dataset = await this.datasetStore.getCurrentDataset();
    this.workflow = this.dataset.workflows[this.workflowID];

    const tokenObject = await this.tokens.getToken("zenodo");
    this.zenodoToken = tokenObject.token;

    const validZenodoConnection = await this.tokens.verifyZenodoConnection();

    if (validZenodoConnection) {
      this.validZenodoTokenAvailable = true;
    } else {
      this.validZenodoTokenAvailable = false;
    }

    this.datasetStore.showProgressBar();
    this.datasetStore.setProgressBarType("zenodo");
    this.datasetStore.setCurrentStep(5);

    this.workflow.currentRoute = this.$route.path;

    if ("uploadToRepo" in this.workflow) {
      this.uploadToRepo = this.workflow.uploadToRepo ? "Yes" : "No";
    } else {
      this.uploadToRepo = "None";
    }

    if (this.workflow.destination) {
      this.repoID = this.workflow.destination.name;

      if (this.repoID !== "") {
        if ("selectedDeposition" in this.workflow.destination[this.repoID]) {
          this.selectedDeposition = this.workflow.destination[this.repoID].selectedDeposition;
        }

        if (this.workflow.source.type === "local") {
          if (
            "newVersion" in this.workflow.destination[this.repoID] &&
            this.workflow.source.type === "local"
          ) {
            this.newVersion = this.workflow.destination[this.repoID].newVersion.toString();

            if (this.newVersion === "true") {
              this.showSelectZenodoDeposition = true;
            } else {
              this.showSelectZenodoDeposition = false;
            }
          } else {
            this.newVersion = "None";
            this.showSelectZenodoDeposition = false;
          }
        } else if (this.workflow.source.type === "github") {
          this.newVersion = "false";
          this.showSelectZenodoDeposition = false;
        }
      } else {
        this.newVersion = "None";
        this.showSelectZenodoDeposition = false;
      }
    }

    this.loading = false;
  },
};
</script>

<style scoped>
.single-check-box {
  @apply flex h-48 w-48 items-center justify-center transition-all;
}

.single-check-box:not(.disabled-card, .selected-repo):hover {
  @apply border border-secondary-500 shadow-lg shadow-secondary-500/50 transition-all;
}
</style>
