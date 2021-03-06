<template>
  <div>
    <v-app-bar flat color="#d3fgsg" v-if="$store.state.isUserLoggedIn">
      <v-toolbar-title>
        <router-link :to="{name: 'lectures'}" class="brand-black">Thinq</router-link>
      </v-toolbar-title>

      <v-menu
        bottom
        transition="scale-transition"
        :close-on-content-click="false"
        v-if="$store.state.isUserLoggedIn"
      >
        <template v-slot:activator="{ on }">
          <v-btn color="#ededed" v-on="on" style="margin-left: 30px;">
            Browse
            <v-icon small>mdi-menu-down</v-icon>
          </v-btn>
        </template>

        <v-card max-width="200px">
          <v-container fluid>
            <span class="subtitle ma-2 d-block font-weight-bold text-center">Featured</span>
            <div class="d-flex justify-center align-center flex-column ma-3">
              <v-btn depressed small text block :to="{name: `lectures`}">All Lectures</v-btn>
            </div>
            <v-divider />
            <span class="subtitle ma-2 d-block font-weight-bold text-center">Categories</span>
            <div
              class="d-flex justify-center align-center flex-column ma-1"
              v-for="category in categories"
              :key="category.id"
            >
              <v-btn
                depressed
                small
                text
                block
                :to="{path: `/lectures/categories/${category.id}`}"
              >{{category.name}}</v-btn>
            </div>
          </v-container>
        </v-card>
      </v-menu>

      <v-spacer></v-spacer>

      <v-menu
        bottom
        left
        transition="slide-x-reverse-transition"
        :close-on-content-click="false"
        v-if="$store.state.isUserLoggedIn"
      >
        <template v-slot:activator="{ on }">
          <v-btn color="indigo" v-on="on" icon style="margin-right: 0.3em">
            <v-avatar v-if="!$store.state.user.icon_url">
              <v-icon large>mdi-account-circle</v-icon>
            </v-avatar>
            <v-avatar v-else>
              <v-img :src="$store.state.user.icon_url" />
            </v-avatar>
          </v-btn>
        </template>

        <v-card max-width="200px">
          <v-container fluid>
            <div class="d-flex justify-center align-center flex-column ma-3">
              <router-link
                :to="{path: `/users/${($store.state.user.display_name).toLowerCase()}/${$store.state.user.id}/profile`}"
              >
                <v-avatar v-if="!$store.state.user.icon_url">
                  <v-icon large>mdi-account-circle</v-icon>
                </v-avatar>
                <v-avatar v-else>
                  <v-img :src="$store.state.user.icon_url" />
                </v-avatar>
              </router-link>
              <router-link
                class="d-block ma-2 bold"
                style="font-size: 16px;"
                :to="{path: `/users/${($store.state.user.display_name).toLowerCase()}/${$store.state.user.id}/profile`}"
              >
                <span>{{$store.state.user.display_name}}</span>
              </router-link>
              <v-btn
                depressed
                small
                block
                text
                :to="{path: `/users/${($store.state.user.display_name).toLowerCase()}/${$store.state.user.id}/profile`}"
              >View profile</v-btn>
            </div>
            <v-divider />
            <v-btn
              v-if="adminPermissions"
              class="my-3"
              depressed
              small
              text
              block
              :to="{name: 'admin-main'}"
            >Admin Panel</v-btn>
            <v-divider v-if="adminPermissions" />
            <div class="d-flex justify-center align-center flex-column ma-3">
              <v-btn
                v-if="permissions"
                class="ma-1"
                depressed
                small
                text
                block
                :to="{path: `/users/${($store.state.user.display_name).toLowerCase()}/${$store.state.user.id}/lectures`}"
              >My lectures</v-btn>
              <v-btn
                class="ma-1"
                depressed
                small
                text
                block
                :to="{path: `/users/${($store.state.user.display_name).toLowerCase()}/${$store.state.user.id}/edit`}"
              >Edit account</v-btn>

              <v-btn class="ma-1" depressed small text block @click="logout">Log out</v-btn>
            </div>
            <!-- <v-list-item @click="toPath">My lectures</v-list-item>
                <v-list-item @click="toPath">Settings</v-list-item>
                <v-list-item @click="logout">Log out</v-list-item>
            </v-list>-->
          </v-container>
        </v-card>
      </v-menu>
    </v-app-bar>
  </div>
</template>

<script>
import CategoryService from "@/services/CategoryService";
export default {
  data: () => ({
    categories: null,
    permissions: false,
    adminPermissions: false,
    isChecking: true
  }),
  watch: {
    $route: 'checkRoles'
  },
  created() {
    this.checkRoles();
    this.getCategories();
  },
  methods: {
    async logout() {
      this.$store.dispatch("setToken", null);
      this.$store.dispatch("setUser", null);
      this.$store.dispatch("setAuthorities", null);
      this.permissions = false;
      this.adminPermissions = false;
      this.$router.push({
        name: "login"
      });
    },
    async getCategories() {
      try {
        const response = await CategoryService.index();
        this.categories = response.data;
      } catch (err) {
        console.log(err);
      }
    },
    checkRoles() {
        const userAuthorities = this.$store.state.authorities;
        if (userAuthorities) {
          for (let i = 0; i < userAuthorities.length; i++) {
            if (
              userAuthorities[i] === "ROLE_LECTURER" ||
              userAuthorities[i] === "ROLE_MODERATOR" ||
              userAuthorities[i] === "ROLE_ADMIN"
            ) {
              this.permissions = true;
            } else {
              this.permissions = false;
            }
          }
          for (let i = 0; i < userAuthorities.length; i++) {
            if (userAuthorities[i] === "ROLE_ADMIN") {
              this.adminPermissions = true;
            } else {
              this.adminPermissions = false;
            }
          }
        }
    }
  }
};
</script>

<style scoped>
.brand-black {
  font-family: "Patrick Hand SC", cursive;
  color: #1b262c;
  opacity: 0.9;
  font-weight: 900 !important;
  font-size: 40px !important;
}
.brand {
  font-family: "Patrick Hand SC", cursive;
  color: white;
  font-weight: 900 !important;
  font-size: 40px !important;
}
.cta-btn-active {
  background-color: #3f72af !important;
  color: white !important;
  transition: 0.2s;
}
.cta-btn-hover {
  background-color: white !important;
  color: black !important;
  transition: 0.2s;
}
</style>