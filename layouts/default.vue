<template>
  <v-app dark>
    <v-navigation-drawer
      v-model="drawer"
      :mini-variant="miniVariant"
      :clipped="clipped"
      fixed
      app
    >
      <v-list>
        <v-list-item
          v-for="link in links"
          :key="link.text"
          router
          :to="link.route"
          active-class="border"
        >
          <v-list-item-action>
            <v-icon>{{ link.icon }}</v-icon>
          </v-list-item-action>
          <v-list-item-content>
            <v-list-item-title>{{ link.text }}</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
        <v-list-group prepend-icon="mdi-account">
          <template #activator>
            <v-list-item-title>{{ "WEBSRF" }}</v-list-item-title></template
          >
          <v-list-item-content>
            <v-list-item
              v-for="i in account"
              :key="i.text"
              router
              :to="i.route"
              active-class="border"
            >
              <v-list-item-icon>
                <v-icon>{{ i.icon }}</v-icon>
              </v-list-item-icon>
              <v-list-item-title>{{ i.text }}</v-list-item-title>
            </v-list-item>
          </v-list-item-content>
        </v-list-group>
      </v-list>
    </v-navigation-drawer>
    <v-app-bar :clipped-left="clipped" inverted-scroll fixed app>
      <v-app-bar-nav-icon @click.stop="drawer = !drawer" />
    </v-app-bar>
    <v-main>
      <v-container>
        <nuxt />
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
//import Vue from "vue";
//import VueTreeList from "vue-tree-list";

//Vue.use(VueTreeList);
export default {
  name: "DefaultLayout",
  data: () => ({
    drawer: true,
    title: "Vuetify.js",
    clipped: false,
    miniVariant: false,
  }),
  computed: {
    links() {
      return [
        {
          icon: "mdi-apps",
          text: "main",
          route: "/",
        },
      ];
    },
    account() {
      return [
        {
          icon: "mdi-folder",
          text: "list of websrf",
          route: "/list",
        },
        {
          icon: "mdi-plus",
          text: "create websrf",
          route: "/create",
        },
      ];
    },
  },
};
</script>
