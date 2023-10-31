<script setup>
import { ref, computed, onMounted, watch } from "vue";
import axios from "axios";
import { useRoute } from "vue-router";
import { useToast } from "primevue/usetoast";
import CSPNavbar from "../CSPNavBar.vue";

const toast = useToast();

const route = useRoute();

const CSPid = route.params.id;

const list = ref([]);

const csp = ref([]);

const loading = ref(true);

const fontColor = ref("");
const buttonColor = ref("");
const buttonFontColor = ref("");
const bgColor = ref("");

// dropdown font
const selectedFont = ref();
const fontOptions = ref([
  { name: "Times New Roman" },
  { name: "Arial" },
  { name: "Verdana" },
  { name: "Lucida Console" },
  { name: "Courier" },
  { name: "monospace" },
  { name: "fantasy" },
]);

// options for button type
const value = ref(null);
const options = ref([
  { icon: "outlined", value: "outlined", outlined: true, rounded: false },
  { icon: "rounded", value: "rounded", outlined: false, rounded: true },
]);

function save() {
  loading.value = true;
  csp.value.settings.font["font-family"] = selectedFont.value.name;
  csp.value.settings.font["font-colour"] = fontColor.value;
  csp.value.settings.buttons["button-colour"] = buttonColor.value;
  csp.value.settings.buttons["button-font-colour"] = buttonFontColor.value;
  csp.value.settings.background["bg-colour"] = bgColor.value;

  axios
    .put("https://smooserve-be.vercel.app/api/csp/" + CSPid, csp.value)
    .then((response) => {
      toast.add({
        severity: "success",
        summary: "Saved",
        detail: response.statusText,
        life: 3000,
      });
    })
    .catch((error) => {
      console.log(error);
      toast.add({
        severity: "error",
        summary: "Error",
        detail: error,
        life: 3000,
      });
    })
    .finally(() => (loading.value = false));
}

onMounted(async () => {
  axios
    .get("https://smooserve-be.vercel.app/api/csp/" + CSPid)
    .then((response) => {
      csp.value = response.data;
      response.data.settings.urls
        ? (list.value = response.data.settings.urls)
        : (list.value = []);
      selectedFont.value = { name: csp.value.settings.font["font-family"] };
      fontColor.value = csp.value.settings.font["font-colour"];
      buttonColor.value = csp.value.settings.buttons["button-colour"];
      buttonFontColor.value = csp.value.settings.buttons["button-font-colour"];
      bgColor.value = csp.value.settings.background["bg-colour"];
    })
    .catch((error) => {
      console.log(error);
      toast.add({
        severity: "error",
        summary: "Error",
        detail: error,
        life: 3000,
      });
    })
    .finally(() => (loading.value = false));
});

watch(
  () => buttonColor,
  (newColor) => {
    // Watch for changes in the buttonColor value and update the button background color
    const buttons = document.querySelectorAll(".selectBtns");
    buttons.forEach((button) => {
      button.style.backgroundColor = newColor;
    });
  }
);
</script>
<template>
  <Toast></Toast>

  <div v-if="loading" class="card">
    <ProgressBar mode="indeterminate" style="height: 6px"></ProgressBar>
  </div>
  <div v-else class="surface-ground flex flex-column w-full h-full">
    <!-- menu -->
    <CSPNavbar />

    <!-- Background -->
    <div class="grid align-items-center justify-content-center mt-3">
      <div class="col-12 md:col-12 lg:col-6">
        <div class="card">
          <Fieldset legend="Background" :toggleable="true">
            <div class="flex flex-wrap gap-5 align-items-center p-4">
              <div class="flex flex-wrap gap-5 align-items-center">
                <div class="flex flex-column align-items-center">
                  <label for="cp-hex" class="font-bold block mb-3">
                    Colour
                  </label>
                  <ColorPicker
                    v-model="bgColor"
                    inputId="cp-hex"
                    format="hex"
                    class="mb-3"
                  />
                </div>
                <div class="align-items-center">
                  <InputText
                    type="text"
                    v-model="bgColor"
                    class="w-full md:w-9rem"
                  />
                </div>
              </div>
            </div>
          </Fieldset>
        </div>
      </div>
    </div>

    <!-- Fonts -->
    <div class="grid align-items-center justify-content-center mt-3">
      <div class="col-12 md:col-12 lg:col-6">
        <div class="card">
          <Fieldset legend="Fonts" :toggleable="true">
            <div class="flex flex-wrap gap-5 align-items-center p-4">
              <div class="flex flex-column gap-3 mb-3 w-full">
                <label for="font" class="font-bold block">Font</label>
                <Dropdown
                  v-model="selectedFont"
                  :options="fontOptions"
                  filter
                  optionLabel="name"
                  placeholder="Select a Font"
                  class="w-full md:w-14rem"
                  id="font"
                  :pt="{
                    root: { class: 'w-full md:w-14rem' },
                    item: ({ props, state, context }) => ({
                      class: context.selected
                        ? 'bg-primary'
                        : context.focused
                        ? 'bg-blue-100'
                        : undefined,
                    }),
                  }"
                >
                  <template #value="slotProps">
                    <div v-if="slotProps.value" class="flex align-items-center">
                      <div :style="{ fontFamily: slotProps.value.name }">
                        {{ slotProps.value.name }}
                      </div>
                    </div>
                    <span v-else>
                      {{ slotProps.placeholder }}
                    </span>
                  </template>
                  <template #option="slotProps">
                    <div class="flex align-items-center">
                      <div :style="{ fontFamily: slotProps.option.name }">
                        {{ slotProps.option.name }}
                      </div>
                    </div>
                  </template>
                </Dropdown>
              </div>
              <div class="flex flex-wrap gap-5 align-items-center">
                <div class="flex flex-column align-items-center">
                  <label for="cp-hex" class="font-bold block mb-3">
                    Colour
                  </label>
                  <ColorPicker
                    v-model="fontColor"
                    inputId="cp-hex"
                    format="hex"
                    class="mb-3"
                  />
                </div>
                <div class="align-items-center">
                  <InputText
                    type="text"
                    v-model="fontColor"
                    class="w-full md:w-9rem"
                  />
                </div>
              </div>
            </div>
          </Fieldset>
        </div>
      </div>
    </div>

    <!-- Buttons -->
    <div class="grid align-items-center justify-content-center mt-3">
      <div class="col-12 md:col-12 lg:col-6">
        <div class="card mb-3">
          <Fieldset legend="Buttons" :toggleable="true">
            <div class="flex flex-wrap gap-5 align-items-center p-4">
              <div class="flex flex-column gap-3 mb-3 w-full">
                <SelectButton
                  v-model="value"
                  :options="options"
                  optionLabel="value"
                  dataKey="value"
                  aria-labelledby="custom"
                >
                  <template #option="slotProps">
                    <Button
                      disabled
                      :rounded="slotProps.option.rounded"
                      :outlined="slotProps.option.outlined"
                      class="align-items-center justify-content-center selectBtns"
                    >
                      <span :style="{ backgroundColor: buttonColor }">
                        <i :class="'pi pi-' + slotProps.option.icon"></i>
                      </span>
                    </Button>
                  </template>
                </SelectButton>
              </div>
              <div class="flex flex-wrap gap-5 align-items-center">
                <div class="flex flex-column align-items-center">
                  <label for="cp-hex" class="font-bold block mb-3">
                    Button Colour
                  </label>
                  <ColorPicker
                    v-model="buttonColor"
                    inputId="cp-hex"
                    format="hex"
                    class="mb-3"
                  />
                </div>
                <div class="align-items-center">
                  <InputText
                    type="text"
                    v-model="buttonColor"
                    class="w-full md:w-9rem"
                  />
                </div>
              </div>
              <div class="flex flex-wrap gap-5 align-items-center">
                <div class="flex flex-column align-items-center">
                  <label for="cp-hex" class="font-bold block mb-3">
                    Button Font Colour
                  </label>
                  <ColorPicker
                    v-model="buttonFontColor"
                    inputId="cp-hex"
                    format="hex"
                    class="mb-3"
                  />
                </div>
                <div class="align-items-center">
                  <InputText
                    type="text"
                    v-model="buttonFontColor"
                    class="w-full md:w-9rem"
                  />
                </div>
              </div>
            </div>
          </Fieldset>
        </div>
        <Button
          rounded
          label="Save"
          @click="save()"
          class="w-full align-items-center justify-content-center"
          ><i class="pi pi-save px-2"></i>Save</Button
        >
      </div>
    </div>
  </div>
</template>

<style>
.card {
  border-radius: 12px;
}
</style>