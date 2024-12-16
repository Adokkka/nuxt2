<template>
  <v-container>
    <v-row id="up_page">
      <v-col
        cols="12"
        lg="6"
        sm="12"
        class="d-flex align-center"
        style="gap: 10px"
      >
        <v-btn @click="goBack">
          <v-icon>mdi-arrow-left</v-icon>
        </v-btn>
        <h1 style="margin: 0">WEBSRF</h1>
      </v-col>
      <v-col
        cols="12"
        lg="6"
        md="6"
        sm="12"
        style="
          text-align: right;
          width: 100%;
          padding-top: 20px;
          padding-right: 30px;
        "
      >
        <v-btn
          class="green white--text"
          style="
            height: 40px !important;
            font-size: 10px;
            border-radius: 5px;
            margin: 5px;
            width: 35%;
          "
          @click="save"
        >
          <v-icon right dark>mdi-content-save </v-icon>
        </v-btn>

        <v-btn
          class="red white--text"
          style="
            height: 40px !important;
            font-size: 10px;
            border-radius: 5px;
            margin: 5px;
            width: 30%;
          "
          @click="refresh"
        >
          <v-icon right dark>mdi-refresh</v-icon>
        </v-btn>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {};
  },
  methods: {
    goBack() {
      this.$router.back();
    },
  },
};
</script>

<style scoped>
/* Add styles if needed */
</style>
