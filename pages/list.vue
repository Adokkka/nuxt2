<template>
  <v-container style="max-width: 100%">
    <v-row justify="center" align="center">
      <v-col sm="12" lg="12" md="12" style="margin-top: 20px">
        <v-row>
          <v-col cols="8"><h1>Список заявок</h1></v-col>
          <v-col cols="4" style="text-align: right">
            <v-btn class="primary" @click="generateAndNavigate">
              Создать заявку
              <v-icon right dark>mdi-plus</v-icon>
            </v-btn>
          </v-col>
        </v-row>

        <v-data-table
          :headers="headers"
          :items="ordersWithIndex"
          item-key="index"
          class="elevation-1"
          :search="search"
          :loading="loading"
          loading-text="Загрузка заявок..."
          :footer-props="footerProps"
        >
          <!-- Слот поиска -->
          <template #top>
            <v-text-field
              v-model="search"
              label="Поиск"
              class="mx-4"
              append-icon="mdi-magnify"
              single-line
              hide-details
            ></v-text-field>
          </template>

          <!-- Кастомный слот статуса -->
          <template #item.status="{ item }">
            
              {{ item.status }}
            </v-icon>
          </template>

          <!-- Слот действий -->
          <template #item.actions="{ item }">
            <v-btn
              v-if="item.status === 'Черновик'"
              color="primary"
              @click="editTask(item.uid)"
            >
              Edit
            </v-btn>
            <v-btn v-else color="secondary" @click="viewTask(item.uid)">
              View
            </v-btn>
          </template>
        </v-data-table>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import moment from "moment";
import { v4 as uuidv4 } from "uuid";
export default {
  data() {
    return {
      search: "",
      loglist: [],
      loading: false,
      footerProps: {
        "disable-items-per-page": true,
      },
    };
  },
  computed: {
    headers() {
      return [
        { text: "#", value: "index" },
        { text: "Отправитель", value: "autor_name" },
        { text: "Номер проекта", value: "cost_name" },
        { text: "Наименование проекта", value: "cost_code" },
        { text: "Вид расходов", value: "TypeOfExpenditure" },
        { text: "Клиент", value: "SpecialRequirementsOfTheClient" },
        { text: "Дата создания", value: "data_shipment" },
        { text: "Статус", value: "status" },
        { text: "Actions", value: "actions", sortable: false },
      ];
    },
    ordersWithIndex() {
      return this.loglist
        .sort((a, b) => b.id - a.id)
        .map((item, index) => ({
          ...item,
          index: index + 1,
        }));
    },
  },
  async created() {
    this.initialize();
  },
  methods: {
    generateAndNavigate() {
      //const uuid = uuidv4();
      this.$router.push(`/create`);
    },
    editTask(taskId) {
      // Переход на страницу редактирования задачи
      this.$router.push(`/edit/${taskId}`);
    },
    async initialize() {
      try {
        this.loading = true;
        const response = await this.$axios.$get(
          "http://127.0.0.1:8000/expenditure/author/940419300162/"
        );
        this.loglist = response.map((item) => ({
          ...item,
          start_date: moment(item.start_date).format("HH:mm - DD/MM/yyyy"),
          //status: item.status==="Черновик" ? "Отправлено на согласование" : "Черновик",
        }));
      } catch (error) {
        console.error("Ошибка при загрузке данных:", error);
      } finally {
        this.loading = false;
      }
    },
    openForm(item) {
      this.$router.push({
        path: "/create",
        query: { id: item.id },
      });
    },
  },
};
</script>
