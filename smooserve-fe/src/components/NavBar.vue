<script setup>
import { ref, computed, onMounted, watch } from "vue";
import { useRoute, useRouter } from "vue-router";
import { getAuth, onAuthStateChanged, signOut } from "firebase/auth";
import { getDocumentIdByEmail } from "@/helper/helperFunctions.js";

//login and logout things
const isLoggedIn = ref(false)

const auth = getAuth();
const router = useRouter();

const route = useRoute();
const CSPid = ref();

const role = ref("");

onMounted(() => {
  onAuthStateChanged(auth, async (user) => {
    if (user) {
      isLoggedIn.value = true;
      // check if is student
      let data = await getDocumentIdByEmail(user.email, "Users");
      role.value = data.role;
    } else {
      isLoggedIn.value = false;
    }
  })
})

const handleSignOut = () => {
  signOut(auth).then(() => {
    router.replace({ name: "Home" })
  })
}

const logoUrl = computed(() => {
  return `layout/images/logo-white.png`;
});



//menu
const items = ref([
  {
    label: "HOME",
    icon: "pi pi-home",
    routeName: "Home",
  },
  {
    label: "ABOUT US",
    icon: "pi pi-info-circle",
    routeName: "About",
  },
  {
    label: "NEAR ME",
    icon: "pi pi-map-marker",
    routeName: "gMap",
  },
  {
    label: "SMOOSERVE SHOP",
    icon: "pi pi-shopping-cart",
    routeName: "Shop",
  },

]);
</script>

<template>
  <div class="grid align-items-center justify-content-center">
    <div class="col-12 md:col-12 lg:col-10">
      <Menubar :model="items" :pt="{
        action: ({ props, state, context }) => ({
          class: context.active
            ? 'bg-primary-50 border-round-sm'
            : context.focused
              ? 'bg-primary-100 border-round-sm'
              : undefined,
        }),
      }" class="mb-3  surface-0 border-none">
        <template #start>
          <img alt="logo" :src="logoUrl" height="60" class="mr-2" />
        </template>
        <template #item="{ label, item, props, root, hasSubmenu }" class="p-menuitem-active">
          <router-link v-if="item.routeName" v-slot="routerProps" :to="{ name: item.routeName }" custom :exact="true">
            <a :href="routerProps.href" v-bind="props.action">
              <span v-bind="props.icon" />
              <span v-bind="props.label">{{ label }}</span>
            </a>
          </router-link>
          <a v-else :href="item.url" :target="item.target" v-bind="props.action">
            <span v-bind="props.icon" />
            <span v-bind="props.label">{{ label }}</span>
            <span :class="[
              hasSubmenu &&
              (root ? 'pi pi-fw pi-angle-down' : 'pi pi-fw pi-angle-right'),
            ]" v-bind="props.submenuicon" />
          </a>
        </template>
        <template #end>
          <a class='line-remove' v-if="isLoggedIn" @click="handleSignOut">
            <router-link :to="{ name: 'Home' }"><i class="pi pi-sign-out icon-spacing pr-2"></i>
              <span style="text-decoration: none;">LOG OUT</span></router-link>
          </a>
          <router-link v-if="isLoggedIn && (role == 'student')" :to="{ name: 'Profile' }"><i class="pi pi-user px-4" style="font-size: 1.2rem"></i></router-link>
          <router-link v-if="isLoggedIn && (role == 'csp')" :to="{ name: 'CSPSignup' }"><i class="pi pi-user px-4" style="font-size: 1.2rem"></i></router-link>
          <a class='line-remove' v-if="!isLoggedIn">
            <i class="navbar-icon pi pi-sign-in px-2"></i><router-link :to="{ name: 'Login' }"><span class="pr-4">LOG
                IN</span></router-link>
          </a>
        </template>
      </Menubar>
    </div>
  </div>
</template>
<style>
a {
  text-decoration: none !important;
  color: black !important;
}
</style>