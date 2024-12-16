<template>
  <v-container>
    <v-row>
      <v-col>
        <v-autocomplete
          v-model="selectedUser"
          :items="userList"
          item-text="full_name"
          item-value="id"
          label="Select or write user"
          return-object
          auto-select-first
          outlined
          @change="handleUserSelection"
          :rules="[requiredRule]"
        ></v-autocomplete>

        <div v-if="detailedUserData" :rules="[requiredRule]">
          <p><strong>Selected User Details:</strong></p>
          <p>Full Name: {{ detailedUserData.FIO }}</p>
          <p>IIN: {{ detailedUserData.IIN }}</p>
          <p>Organization Name: {{ detailedUserData.orgName }}</p>
          <p>Organization BIN: {{ detailedUserData.orgBIN }}</p>
          <p>Cost Name: {{ detailedUserData.CostName }}</p>
          <p>Cost Code: {{ detailedUserData.CostCode }}</p>
          <p>Position: {{ detailedUserData.Post }}</p>
          <p>Position: {{ selectedUser }}</p>
        </div>
        <div v-else>
          <em>No additional information available for this user.</em>
        </div>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      selectedUser: null,
      userList: [],
      detailedUserData: null,
    };
  },
  mounted() {
    this.fetchUsers();
    this.olduid = "c5a1f5da-233f-412c-84ac-efeee2039dad";
    //alert(this.project.uid);
    this.getolinfo(this.olduid);
  },
  methods: {
    async getolinfo(olduid) {
      const response = await axios.get(
        `http://127.0.0.1:8000/expenditure/${olduid}/`
      );
      this.selectedUser = JSON.parse(response.data.Comment);

      console.log(this.selectedUser, "hello");
      // alert(this.selectedUser);
    },
    async fetchUsers() {
      try {
        this.loading = true;
        const response = await axios.get("http://intra.isker.kz/api/v1/users");
        this.userList = response.data.map((user) => ({
          ...user,
          full_name: `${user.last_name} ${user.first_name}`.trim(),
        }));
      } catch (error) {
        console.error("Error fetching users:", error);
        this.error = "Failed to load user data.";
      } finally {
        this.loading = false;
      }
    },
    async handleUserSelection(user) {
      try {
        if (user) {
          const response = await axios.get(
            // `http://192.168.0.163/api/v1/1c/websrf/getuser?iin=${user.username}`
            `http://127.0.0.1:8000/api/organization/${user.username}/`
          );
          this.detailedUserData = response.data || null;
        }
      } catch (error) {
        console.error("Error fetching detailed user data:", error);
      }
    },
  },
};
</script>
