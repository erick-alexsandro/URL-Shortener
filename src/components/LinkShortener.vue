<template>
  <div
    style="height: 100%; width: 80%; margin: auto"
    class="d-flex align-self-center justify-center align-center text-center"
  >
    <v-container>
      <v-row>
        <v-col cols="12">
          <div class="title">
            <h1>Link Shortener Tool</h1>
            <p>
              by
              <a
                class="text-decoration-none text-white"
                href="https://github.com/erick-alexsandro"
                target="_blank"
                >erick-alexsandro</a
              >
            </p>
          </div>
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="12">
          <div class="link-input d-flex ga-3">
            <v-text-field
              clearable
              label="Link"
              v-model="link"
              prepend-icon="mdi-link-edit"
              variant="outlined"
              rounded
            ></v-text-field>
            <v-btn
              @click="shortenLink"
              icon="mdi-check-bold"
              size="large"
            ></v-btn>
          </div>
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="12">
          <div class="shortened-link d-flex flex-column ga-3 justify-center">
            <a v-if="shortenedLink" :href="shortenedLink" target="_blank">{{
              shortenedLink
            }}</a>
            <div class="copy-button">
              <v-btn
                v-if="shortenedLink"
                @click="copyToClipboard"
                prepend-icon="mdi-content-copy"
              >
                <template v-slot:prepend>
                  <v-icon></v-icon>
                </template>
                {{ copyButtonText }}
              </v-btn>
            </div>
            <v-alert
              v-if="showAlert"
              closable
              :title="error"
              :text="errorText"
              type="error"
              @click:close="closeAlert"
            ></v-alert>
          </div>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from "vue";
const link = ref("");
const shortenedLink = ref("");
const showAlert = ref(false);

const error = ref("");
const errorText = ref("");

const closeAlert = () => {
  showAlert.value = false;
  error.value = "";
  errorText.value = "";
};

const token = import.meta.env.VITE_APP_TOKEN;

const apiUrl = `https://api.tinyurl.com/create?api_token=${token}`;

const shortenLink = async () => {
  try {
    showAlert.value = false; // Reset alert state
    if (link.value) {
      if (link.value.includes("https://")) {
        const response = await fetch(apiUrl, {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify({
            url: link._value,
            domain: "tinyurl.com",
            description: "string",
          }),
        });
        const data = await response.json();
        shortenedLink.value = data.data.tiny_url;
      } else {
        error.value = "The link must contain https://";
        errorText.value = "The link must contain https://";
        showAlert.value = true;
      }
    } else {
      error.value = "No value provided";
      errorText.value = "No value provided";
      showAlert.value = true;
    }
  } catch (error) {
    console.error(error);
    error.value = "An error occurred";
    errorText.value = "Please try again later";
    showAlert.value = true;
  }
};

const copyButtonText = ref("Copy to clipboard");

const copyToClipboard = async () => {
  try {
    await navigator.clipboard.writeText(shortenedLink.value);
    copyButtonText.value = "Copied!";
    setTimeout(() => {
      copyButtonText.value = "Copy to clipboard";
    }, 2000);
  } catch (err) {
    console.error("Failed to copy: ", err);
    error.value = "Copy failed";
    errorText.value = "Please try again";
    showAlert.value = true;
  }
};
</script>
