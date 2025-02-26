<template>
  <TransitionRoot appear :show="isOpen" as="template">
    <Dialog as="div" @close="setIsOpen(false, 'outside')">
      <div class="fixed inset-0 z-10 overflow-y-auto">
        <div class="min-h-screen px-4 text-center">
          <TransitionChild
            as="template"
            enter="duration-300 ease-out"
            enter-from="opacity-0"
            enter-to="opacity-100"
            leave="duration-200 ease-in"
            leave-from="opacity-100"
            leave-to="opacity-0"
          >
            <DialogOverlay class="fixed inset-0 bg-zinc-800/30" />
          </TransitionChild>

          <span class="inline-block h-screen align-middle" aria-hidden="true"> &#8203; </span>

          <TransitionChild
            as="template"
            enter="duration-300 ease-out"
            enter-from="opacity-0 scale-95"
            enter-to="opacity-100 scale-100"
            leave="duration-200 ease-in"
            leave-from="opacity-100 scale-100"
            leave-to="opacity-0 scale-95"
          >
            <div
              class="my-8 ml-[9rem] inline-block w-full max-w-lg transform overflow-hidden rounded-2xl bg-white p-6 text-left align-middle shadow-xl transition-all"
            >
              <div class="flex flex-col items-start justify-start">
                <div class="mb-0 flex w-full justify-center">
                  <Vue3Lottie
                    animationLink="https://assets1.lottiefiles.com/packages/lf20_8zle4p5u.json"
                    :width="100"
                    :height="100"
                    :loop="1"
                  />
                </div>
                <div class="my-2 flex w-full flex-col">
                  <DialogTitle
                    as="h3"
                    class="text-left text-xl font-medium leading-6 text-gray-900"
                  >
                    {{ localTitle }}
                  </DialogTitle>

                  <div class="mt-2 w-full">
                    <slot>
                      <p class="text-left text-base text-gray-500">
                        {{ localContent }}
                      </p>
                    </slot>
                  </div>
                </div>
              </div>

              <div class="mt-4 flex justify-center space-x-4">
                <button
                  v-if="localShowCancelButton"
                  type="button"
                  class="primary-plain-button"
                  @click="setIsOpen(false, 'cancel')"
                >
                  {{ localCancelButtonText }}
                </button>
                <button
                  type="button"
                  class="danger-button"
                  @click="setIsOpen(false, 'confirm')"
                  :disabled="confirmDisabled"
                >
                  {{ localConfirmButtonText }}
                </button>
              </div>
            </div>
          </TransitionChild>
        </div>
      </div>
    </Dialog>
  </TransitionRoot>
</template>

<script>
import {
  Dialog,
  DialogOverlay,
  DialogTitle,
  TransitionRoot,
  TransitionChild,
} from "@headlessui/vue";

export default {
  name: "WarningPrompt",
  components: {
    Dialog,
    DialogOverlay,
    DialogTitle,
    TransitionRoot,
    TransitionChild,
  },
  data() {
    return {
      isOpen: false,
      localTitle: this.title,
      localContent: this.content,
      localConfirmButtonText: this.confirmButtonText,
      localCancelButtonText: this.cancelButtonText,
      localShowCancelButton: this.showCancelButton,
    };
  },
  props: {
    title: {
      required: false,
      type: String,
      default: "Message Title",
    },
    content: {
      required: false,
      type: String,
      default: "Message Content",
    },
    confirmButtonText: {
      required: false,
      type: String,
      default: "OK",
    },
    cancelButtonText: {
      required: false,
      type: String,
      default: "Cancel",
    },
    showCancelButton: {
      required: false,
      type: Boolean,
      default: true,
    },
    confirmDisabled: {
      required: false,
      type: Boolean,
      default: false,
    },
  },
  emits: [
    "messageClosed",
    "messageConfirmed",
    "messageCancel",
    "messageOutsideClicked",
    "messageClosed",
  ],
  methods: {
    setIsOpen(val, type) {
      if (!val) {
        this.$emit("messageClosed");
        if (type === "confirm") {
          this.$emit("messageConfirmed");
        }
        if (type === "cancel") {
          this.$emit("messageCancel");
          this.$emit("messageClosed");
        }
        if (type === "outside") {
          this.$emit("messageOutsideClicked");
          this.$emit("messageClosed");
        }
      }
      this.isOpen = val;
    },
    setTitle(val) {
      this.localTitle = val;
    },
    setContent(val) {
      this.localContent = val;
    },
    show() {
      this.setIsOpen(true, "show");
    },
  },
  mounted() {},
};
</script>

<style></style>
