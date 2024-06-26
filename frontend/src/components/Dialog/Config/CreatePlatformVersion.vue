<script setup lang="ts">
import configApi from "@/services/api/config";
import storeConfig from "@/stores/config";
import type { Events } from "@/types/emitter";
import type { Emitter } from "mitt";
import { inject, ref } from "vue";

// Props
const show = ref(false);
const configStore = storeConfig();
const emitter = inject<Emitter<Events>>("emitter");
const fsSlugToCreate = ref();
const slugToCreate = ref();
emitter?.on("showCreatePlatformVersionDialog", ({ fsSlug = "", slug = "" }) => {
  fsSlugToCreate.value = fsSlug;
  slugToCreate.value = slug;
  show.value = true;
});

// Functions
function addVersionPlatform() {
  configApi
    .addPlatformVersionConfig({
      fsSlug: fsSlugToCreate.value,
      slug: slugToCreate.value,
    })
    .then(() => {
      configStore.addPlatformVersion(fsSlugToCreate.value, slugToCreate.value);
    })
    .catch(({ response, message }) => {
      emitter?.emit("snackbarShow", {
        msg: `${response?.data?.detail || response?.statusText || message}`,
        icon: "mdi-close-circle",
        color: "red",
        timeout: 4000,
      });
    });
  closeDialog();
}

function closeDialog() {
  show.value = false;
}
</script>
<template>
  <v-dialog v-model="show" max-width="500px" :scrim="true">
    <v-card>
      <v-toolbar density="compact" class="bg-terciary">
        <v-row class="align-center" no-gutters>
          <v-col cols="10">
            <v-icon icon="mdi-gamepad-variant" class="ml-5" />
            <v-icon icon="mdi-approximately-equal" class="ml-1 text-romm-gray" />
            <v-icon icon="mdi-controller" class="ml-1 text-romm-accent-1" />
          </v-col>
          <v-col>
            <v-btn
              @click="closeDialog"
              class="bg-terciary"
              rounded="0"
              variant="text"
              icon="mdi-close"
              block
            />
          </v-col>
        </v-row>
      </v-toolbar>
      <v-divider class="border-opacity-25" :thickness="1" />

      <v-card-text>
        <v-row class="pa-2 align-center" no-gutters>
          <v-text-field
            @keyup.enter=""
            v-model="fsSlugToCreate"
            label="Platform version"
            variant="outlined"
            required
            hide-details
          />
          <v-icon icon="mdi-menu-right" class="mx-2 text-romm-gray" />
          <v-text-field
            class="text-romm-accent-1"
            @keyup.enter=""
            v-model="slugToCreate"
            label="Main platform"
            color="romm-accent-1"
            base-color="romm-accent-1"
            variant="outlined"
            required
            hide-details
          />
        </v-row>
        <v-row class="justify-center pa-2" no-gutters>
          <v-btn @click="closeDialog" class="bg-terciary">Cancel</v-btn>
          <v-btn
            @click="addVersionPlatform()"
            class="text-romm-green bg-terciary ml-5"
          >
            Confirm
          </v-btn>
        </v-row>
      </v-card-text>
    </v-card>
  </v-dialog>
</template>
