<template>
  <v-data-table
    :headers="headers"
    :items="desserts"
    sort-by="calories"
    class="elevation-1"
  >
    <template v-slot:top>
      <v-toolbar flat>
        <v-toolbar-title>My CRUD</v-toolbar-title>
        <v-divider class="mx-4" inset vertical></v-divider>
        <v-spacer></v-spacer>
        <v-dialog v-model="dialog" max-width="1200px">
          <template v-slot:activator="{ on, attrs }">
            <v-btn color="primary" dark class="mb-2" v-bind="attrs" v-on="on">
              New Item
            </v-btn>
          </template>
          <v-card>
            <v-card-title>
              <span class="text-h5">{{ formTitle }}</span>
            </v-card-title>

            <v-card-text>
              <v-container>
                <v-row>
                  <v-col cols="2" sm="6" md="4">
                    <v-text-field
                      v-model="editedItem.id"
                      label="ID"
                    ></v-text-field>
                  </v-col>
                  <v-col cols="12" sm="6" md="4">
                    <v-text-field
                      v-model="editedItem.description"
                      label="Description"
                    ></v-text-field>
                  </v-col>
                  <v-col cols="12" sm="6" md="4">
                    <v-text-field
                      v-model="editedItem.count"
                      label="Count"
                      type="number"
                    ></v-text-field>
                  </v-col>
                  <v-col cols="12" sm="6" md="4">
                    <v-text-field
                      v-model="editedItem.price"
                      label="Price"
                      type="number"
                    ></v-text-field>
                  </v-col>
                  <v-col cols="12" sm="6" md="4">
                    <v-menu
                      v-model="editedItem.startDateMenu"
                      :close-on-content-click="false"
                      transition="scale-transition"
                      offset-y
                    >
                      <template v-slot:activator="{ on, attrs }">
                        <v-text-field
                          v-model="editedItem.start_date"
                          readonly
                          dense
                          outlined
                          v-bind="attrs"
                          v-on="on"
                          class="rounded-0"
                          label="Start Date"
                        ></v-text-field>
                      </template>
                      <v-date-picker
                        v-model="editedItem.start_date"
                        no-title
                        scrollable
                        @change="editedItem.startDateMenu = false"
                      >
                        <v-btn text color="primary">OK</v-btn>
                      </v-date-picker>
                    </v-menu>
                  </v-col>
                  <v-col cols="12" sm="6" md="4">
                    <v-menu
                      v-model="editedItem.endDateMenu"
                      :close-on-content-click="false"
                      transition="scale-transition"
                      offset-y
                    >
                      <template v-slot:activator="{ on, attrs }">
                        <v-text-field
                          v-model="editedItem.end_date"
                          readonly
                          dense
                          outlined
                          v-bind="attrs"
                          v-on="on"
                          class="rounded-0"
                          label="End Date"
                        ></v-text-field>
                      </template>
                      <v-date-picker
                        v-model="editedItem.end_date"
                        no-title
                        scrollable
                        @change="editedItem.endDateMenu = false"
                      >
                        <v-btn text color="primary">OK</v-btn>
                      </v-date-picker>
                    </v-menu>
                  </v-col>
                  <v-col cols="12" sm="6" md="4">
                    <v-select
                      v-model="editedItem.measure"
                      :items="measureOptions"
                      label="Measure"
                      dense
                      outlined
                    ></v-select>
                  </v-col>
                  <v-col>
                    <treeselect
                      v-model="editedItem.nomenclatura_code"
                      :options="options"
                      :normalizer="normalizer"
                      class="mb-6 rounded-0"
                      :disable-branch-nodes="true"
                      :show-count="true"
                      :append-to-body="true"
                      label="Категория"
                    />
                  </v-col>
                </v-row>
              </v-container>
            </v-card-text>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" text @click="close"> Cancel </v-btn>
              <v-btn color="blue darken-1" text @click="save"> Save </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
        <v-dialog v-model="dialogDelete" max-width="500px">
          <v-card>
            <v-card-title class="text-h5"
              >Are you sure you want to delete this item?</v-card-title
            >
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" text @click="closeDelete"
                >Cancel</v-btn
              >
              <v-btn color="blue darken-1" text @click="deleteItemConfirm"
                >OK</v-btn
              >
              <v-spacer></v-spacer>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-toolbar>
    </template>
    <template v-slot:item.actions="{ item }">
      <v-icon small class="mr-2" @click="editItem(item)"> mdi-pencil </v-icon>
      <v-icon small @click="deleteItem(item)"> mdi-delete </v-icon>
    </template>
    <template v-slot:no-data>
      <v-btn color="primary" @click="initialize"> Reset </v-btn>
    </template>
  </v-data-table>
</template>
<script>
import Treeselect from "@riophae/vue-treeselect";
import "@riophae/vue-treeselect/dist/vue-treeselect.css";
export default {
  components: { Treeselect },
  data: () => ({
    dialog: false,
    dialogDelete: false,
    headers: [
      { text: "Item #", value: "id" },
      { text: "Item description", value: "description" },
      { text: "Start Date", value: "start_date" },
      { text: "End Date", value: "end_date" },
      { text: "count", value: "quantity" },
      { text: "measure", value: "measure" },
      { text: "price", value: "price" },
      { text: "budgetcode", value: "nomenclatura_code" },
    ],
    desserts: [],
    editedIndex: -1,
    editedItem: {
      id: 1,
      description: null,
      start_date: null,
      end_date: null,
      quantity: null,
      measure: null,
      price: null,
      nomenclatura_code: null,
    },
  }),

  computed: {
    formTitle() {
      return this.editedIndex === -1 ? "New Item" : "Edit Item";
    },
  },

  watch: {
    dialog(val) {
      val || this.close();
    },
    dialogDelete(val) {
      val || this.closeDelete();
    },
  },

  created() {
    this.initialize();
  },

  methods: {
    initialize() {
      this.desserts = [
        {
          id: 1,
          description: null,
          start_date: null,
          end_date: null,
          quantity: null,
          measure: null,
          price: null,
          nomenclatura_code: null,
        },
      ];
    },

    editItem(item) {
      this.editedIndex = this.desserts.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialog = true;
    },

    deleteItem(item) {
      this.editedIndex = this.desserts.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialogDelete = true;
    },

    deleteItemConfirm() {
      this.desserts.splice(this.editedIndex, 1);
      this.closeDelete();
    },

    close() {
      this.dialog = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      });
    },

    closeDelete() {
      this.dialogDelete = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      });
    },

    save() {
      if (this.editedIndex > -1) {
        Object.assign(this.desserts[this.editedIndex], this.editedItem);
      } else {
        this.desserts.push(this.editedItem);
      }
      this.close();
    },
  },
};
</script>
