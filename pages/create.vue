<template>
  <v-container>
    <v-form ref="form" v-model="isValid">
      <v-row margin="100">
        <v-col>
          <h1>WebSRF_FORM</h1>
        </v-col>
        <v-sheet class="mx-auto" width="300">
          <v-form disabled>
            <v-text-field
              v-model="project.nomer"
              label="SRF No./ № заявки:"
              outlined
            ></v-text-field>
            <v-text-field
              v-model="project.order_date"
              label="Дата заявки:"
              outlined
            ></v-text-field>
          </v-form>
        </v-sheet>
      </v-row>
      <v-row>
        <v-col>
          <v-autocomplete
            v-model="selectedUser"
            :items="userList"
            item-text="full_name"
            item-value="id"
            label="Select or write user"
            return-object
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
          </div>
          <div v-else>
            <em>No additional information available for this user.</em>
          </div>
        </v-col>

        <v-col cols="6" class="text-right">
          <v-autocomplete
            v-model="selectedGeneralname"
            :items="uniqueGeneralnames"
            label="Search by Generalname"
            outlined
            @change="resetSelection"
            :rules="[requiredRule]"
            required
          ></v-autocomplete>

          <v-autocomplete
            v-model="selectedObject"
            :items="filteredObjects"
            item-text="name"
            item-value="uid"
            label="Search Related Objects"
            return-object
            outlined
            :rules="[requiredRule]"
          ></v-autocomplete>

          <div v-if="selectedObject" class="object-details">
            <h3>Selected Object Details:</h3>
            <p><strong>Name:</strong> {{ selectedObject.name }}</p>
            <p><strong>Code:</strong> {{ selectedObject.code }}</p>
            <p><strong>UID:</strong> {{ selectedObject.uid }}</p>
            <p>
              <strong>Generalname:</strong> {{ selectedObject.Generalname }}
            </p>
          </div>
        </v-col>
      </v-row>

      <v-row>
        <v-col>
          <v-sheet class="mx-auto">
            <v-form>
              <v-textarea
                v-model="project.supplier"
                label="Предлагаемые поставщики"
                outlined
                auto-grow
                clearable
                :rules="[requiredRule]"
              ></v-textarea>
              <v-textarea
                v-model="project.area"
                label="Участок"
                outlined
                auto-grow
                clearable
                :rules="[requiredRule]"
              ></v-textarea>
            </v-form>
          </v-sheet>
        </v-col>

        <v-col>
          <v-select
            v-model="project.type_expenditure"
            item-text="name"
            label="Вид статьи расходов"
            return-object
            outlined
            :items="expenditureTypes"
            item-value="id"
            required
            @change="fetchExpenditure"
          ></v-select>

          <div v-if="showClientAutocomplete">
            <v-autocomplete
              v-model="client_code"
              :items="clientList"
              item-text="name"
              item-value="bin"
              label="Клиент"
              return-object
              outlined
              @change="fetchClientDetails"
            ></v-autocomplete>
            <v-textarea
              v-model="project.documents"
              label="Ссылочный документ"
              outlined
              auto-grow
              clearable
            ></v-textarea>
            <div v-if="client_code">
              <p><strong>Selected Client Details:</strong></p>
              <p>Code: {{ client_code.code }}</p>
              <p>Name: {{ client_code.name }}</p>
              <p>BIN: {{ client_code.bin }}</p>
            </div>
          </div>
        </v-col>
      </v-row>

      <v-row>
        <v-col>
          <v-text-field
            v-model="project.currency"
            outlined
            label="currency"
            disabled
          ></v-text-field>
        </v-col>
        <v-col>
          <v-select
            v-model="project.extra_project"
            label="Project/Non-project"
            :items="['Project', 'Non-Project']"
            :rules="[requiredRule]"
            outlined
          ></v-select>
        </v-col>
      </v-row>

      <v-row>
        <v-col>
          <v-textarea
            v-model="project.description_works"
            label="Описание работ"
            auto-grow
            outlined
            :rules="[requiredRule]"
          ></v-textarea>
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="12">
          <v-data-table
            v-model="selected"
            :headers="headers"
            :items="tableData"
            item-value="id"
            class="elevation-1 rounded-0 no-padding"
            outlined
          >
            <!-- Верхний слот с инструментами -->
            <template v-slot:top>
              <v-toolbar flat>
                <v-toolbar-title
                  >Таблица с редактированием и диапазоном дат</v-toolbar-title
                >
                <v-spacer></v-spacer>
                <v-btn color="primary" @click="addRow">Добавить строку</v-btn>
                <v-btn color="error" @click="deleteLastRow">
                  <v-icon left>mdi-delete</v-icon> Удалить строку
                </v-btn>
              </v-toolbar>
            </template>

            <!-- Основное тело таблицы -->
            <template v-slot:body="{ items }">
              <tr v-for="(item, index) in items" :key="item.id">
                <td width="5%">
                  <v-text-field
                    v-model="item.id"
                    dense
                    outlined
                    disabled
                    border-e-lg
                    class="rounded-0"
                  ></v-text-field>
                </td>

                <td width="20%">
                  <v-text-field
                    v-model="item.description"
                    dense
                    class="rounded-0"
                    label="Описание"
                    outlined
                  ></v-text-field>
                </td>

                <td width="10%">
                  <v-menu
                    v-model="item.startDateMenu"
                    :close-on-content-click="false"
                    transition="scale-transition"
                    offset-y
                  >
                    <template v-slot:activator="{ on, attrs }">
                      <v-text-field
                        v-model="item.start_date"
                        readonly
                        dense
                        outlined
                        v-bind="attrs"
                        v-on="on"
                        class="rounded-0"
                        label="Начало"
                      ></v-text-field>
                    </template>
                    <v-date-picker
                      v-model="item.start_date"
                      no-title
                      scrollable
                      @change="item.startDateMenu = false"
                    >
                      <v-btn text color="primary">OK</v-btn>
                    </v-date-picker>
                  </v-menu>
                </td>

                <td width="10%">
                  <v-menu
                    v-model="item.endDateMenu"
                    :close-on-content-click="false"
                    transition="scale-transition"
                    offset-y
                  >
                    <template v-slot:activator="{ on, attrs }">
                      <v-text-field
                        v-model="item.end_date"
                        readonly
                        dense
                        outlined
                        v-bind="attrs"
                        v-on="on"
                        class="rounded-0"
                        label="Конец"
                      ></v-text-field>
                    </template>
                    <v-date-picker
                      v-model="item.end_date"
                      no-title
                      scrollable
                      @change="item.endDateMenu = false"
                    >
                      <v-btn text color="primary">OK</v-btn>
                    </v-date-picker>
                  </v-menu>
                </td>

                <td width="5%">
                  <v-text-field
                    v-model="item.quantity"
                    dense
                    outlined
                    class="rounded-0"
                    label="Количество"
                    type="number"
                  ></v-text-field>
                </td>

                <!-- <td width="5%">
                  <v-select
                    v-model="item.measure"
                    :items="measure"
                    item-text="name"
                    item-value="bin"
                    outlined
                    dense
                    class="rounded-0"
                    label="Ед. изм."
                  ></v-select>
                </td> */-->

                <td width="5%">
                  <v-text-field
                    v-model="item.price"
                    dense
                    outlined
                    type="number"
                    class="rounded-0"
                    label="Цена"
                  ></v-text-field>
                </td>

                <td width="45%">
                  <treeselect
                    v-model="item.nomenclatura_code"
                    :options="options"
                    :normalizer="normalizer"
                    class="mb-6 rounded-0"
                    :disable-branch-nodes="true"
                    :show-count="true"
                    :append-to-body="true"
                    label="Категория"
                  />
                </td>
              </tr>
            </template>
          </v-data-table>
        </v-col>
      </v-row>
      <v-row>
        <v-col>
          <v-card>
            <v-card-title class="d-flex justify-space-between">
              <span>Загрузка файлов</span>
              <v-btn color="primary" @click="addRowfile" outlined>
                <v-icon left>mdi-plus</v-icon> Добавить строку
              </v-btn>
            </v-card-title>
            <v-card-text>
              <v-data-table
                :headers="headersfor"
                :items="rows"
                class="elevation-1"
              >
                <template v-slot:body="{ items }">
                  <tbody>
                    <tr v-for="(item, index) in items" :key="index">
                      <td>{{ index + 1 }}</td>
                      <td>
                        <v-text-field
                          v-model="item.description"
                          label="Описание"
                          outlined
                          dense
                        />
                      </td>
                      <td>
                        <v-file-input
                          v-model="item.file"
                          label="Файл"
                          outlined
                          dense
                        />
                      </td>
                      <td>
                        <v-btn color="error" icon @click="removeRowfile(index)">
                          <v-icon>mdi-delete</v-icon>
                        </v-btn>
                      </td>
                    </tr>
                  </tbody>
                </template>
              </v-data-table>
            </v-card-text>
            <v-card-actions class="d-flex justify-space-between">
            </v-card-actions>
          </v-card>
        </v-col>
      </v-row>
      <v-dialog v-model="dialog" max-width="800px">
        <v-card>
          <v-card-title>Таблица с выбором</v-card-title>
          <v-card-text>
            <v-form ref="dialogForm" v-model="isDialogValid">
              <v-simple-table>
                <thead>
                  <tr>
                    <th>Имя</th>
                    <th>Выбор</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="(row, index) in matrixtable" :key="row.uid">
                    <td>{{ row.name }}</td>
                    <td>
                      <v-select
                        :items="row.items"
                        item-text="name"
                        item-value="uid"
                        v-model="selectedValues[index]"
                        :rules="[requiredRule]"
                        label="Выберите"
                      ></v-select>
                    </td>
                  </tr>
                </tbody>
              </v-simple-table>
            </v-form>
          </v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="red" text @click="dialog = false">Закрыть</v-btn>
            <v-btn color="green" text @click="saveSelection">Сохранить</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
      <v-btn color="primary" @click="dialog = true">Открыть таблицу</v-btn>

      <v-btn color="blue" @click="saveDraft">Сохранить в черновик</v-btn>
      <v-btn
        :disabled="!isValid || !isDialogValid"
        :loading="loading"
        color="success"
        class="ml-3"
        @click="submitForm"
      >
        Отправить заявку
      </v-btn>
    </v-form>
    <v-alert v-if="loading" type="info" dismissible>
      Отправка данных...
    </v-alert>

    <!-- Сообщение об успехе -->
    <v-alert v-if="successMessage" type="success" dismissible>
      {{ successMessage }}
    </v-alert>
  </v-container>
</template>

<script>
import axios from "axios";
import data from "../public/data/getcostcentertree.json";
import client from "../public/data/getclientlist.json";
import Treeselect from "@riophae/vue-treeselect";
import "@riophae/vue-treeselect/dist/vue-treeselect.css";
import service from "../public/data/getservicestree.json";
import measure from "../public/data/getmeasurelist.json";
import moment from "moment";
import matrix from "../public/data/getmatrixlistwithitems.json";
import { v4 as uuidv4 } from "uuid";
export default {
  components: { Treeselect },
  data() {
    return {
      isValid: false,
      loading: false,
      isDialogValid: false,
      successMessage: "",
      project: {
        uid: this.uuid,
        nomer: "",
        order_date: "",
        supplier: "",
        area: "",
        description_works: "",
        currency: "USD",
        documents: "",
        type_expenditure: "",
        date_shipment: moment(new Date()).format("YYYY-MM-DD"),
      },
      value: null,
      options: service,
      normalizer(node) {
        return {
          id: node.code,
          label: node.name,
          children:
            node.items && node.items.length > 0 ? node.items : undefined,
        };
      },
      selectedItems: [],
      headers: [
        { text: "Item #", value: "id" },
        { text: "Item description", value: "description" },
        { text: "Start Date", value: "start_date" },
        { text: "End Date", value: "end_date" },
        { text: "count", value: "quantity" },
        //{ text: "measure", value: "measure" },-->
        { text: "price", value: "price" },
        { text: "budgetcode", value: "nomenclatura_code" },
      ],
      dialog: false,
      matrixtable: matrix,
      selectedValues: [], // Хранение выбранных UID для каждой строки
      matrix: [], // Новый массив

      tableData: [
        {
          id: 1,
          description: null,
          start_date: null,
          end_date: null,
          quantity: null,
          // measure: null,
          price: null,
          nomenclatura_code: null,
        },
      ],
      selected: [],
      nextId: 2,
      date: new Date(Date.now() - new Date().getTimezoneOffset() * 60000)
        .toISOString()
        .substr(0, 10),

      modal: false,
      menu2: false,

      selectedUser: null,
      userList: [],
      detailedUserData: null, // выдает данные об юзере

      expenditureTypes: [],
      client_code: "", //дает данные клиент кода
      clientList: client,
      measure: measure,
      extra_project: "",
      selectedGeneralname: null,
      selectedObject: null, //выдает данные об выбранном проекте
      projectList: this.flattenData(data),
      loading: false,
      error: null,
      headersfor: [
        { text: "№", value: "id", align: "start" },
        { text: "Описание", value: "description" },
        { text: "Файл", value: "file" },
        { text: "Действия", value: "actions", sortable: false },
      ],
      rows: [], // Данные для таблицы
    };
  },
  computed: {
    uniqueGeneralnames() {
      return [...new Set(this.projectList.map((item) => item.Generalname))];
    },
    filteredObjects() {
      return this.projectList.filter(
        (item) => item.Generalname === this.selectedGeneralname
      );
    },
    showClientAutocomplete() {
      return (
        this.project.type_expenditure &&
        ["Reimbursable_by_Client", "Backcharge"].includes(
          this.project.type_expenditure.name
        )
      );
    },
  },
  mounted() {
    this.fetchUsers();
    this.fetchExpenditure();
  },
  created() {
    this.uuid = uuidv4(); // Генерация UUID при создании компонента
  },
  methods: {
    requiredRule(value) {
      return !!value || "Поле обязательно для заполнения.";
    },
    saveSelection() {
      // Генерация нового массива
      this.matrix = this.matrixtable.map((row, index) => {
        const selectedItem = row.items.find(
          (item) => item.uid === this.selectedValues[index]
        );
        return {
          element_name: row.namefull,
          value_name: selectedItem ? selectedItem.name : null,
          value_uid: selectedItem ? selectedItem.uid : null,
        };
      });
      this.dialog = false;
      console.log("Новый массив matrix:", this.matrix);
    },
    handleSelection(items) {
      console.log("Selected items:", items);
    },
    addRow() {
      this.tableData.push({
        id: this.nextId++,
        description: "",
        start_date: "",
        end_date: "",
        quantity: null,
        //measure: null,
        price: null,
        nomenclatura_code: null,
      });
    },
    deleteLastRow() {
      // Удаление последней строки, если она существует
      if (this.tableData.length > 0) {
        this.tableData.pop();
      }
    },
    addRowfile() {
      this.rows.push({
        id: this.rows.length + 1,
        description: "",
        file: null,
      });
    },
    // Удалить строку
    removeRowfile(index) {
      this.rows.splice(index, 1);
    },
    validateFile(item, index) {
      if (!item.file) {
        console.warn(`Файл в строке ${index + 1} не выбран`);
      }
    },
    convertFileToBase64(file) {
      return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.onload = () => resolve(reader.result.split(",")[1]);
        reader.onerror = (error) => reject(error);
        reader.readAsDataURL(file);
      });
    },
    async prepareRows() {
      const preparedRows = [];
      for (const [index, row] of this.rows.entries()) {
        if (!row.file) {
          throw new Error(`Файл в строке ${index + 1} не выбран`);
        }
        const base64File = await this.convertFileToBase64(row.file);
        preparedRows.push({
          item_number: index + 1,
          item_description: row.description,
          file_data: base64File,
          filename: row.file.name,
        });
      }
      return preparedRows;
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
    async fetchExpenditure() {
      try {
        const response = await axios.get(
          "http://192.168.0.163/api/v1/websrf/getexpenditure"
        );
        this.expenditureTypes = response.data;
      } catch (error) {
        console.error("Error fetching expenditures:", error);
      }
    },
    async handleUserSelection(user) {
      try {
        if (user) {
          const response = await axios.get(
            //`http://192.168.0.163/api/v1/1c/websrf/getuser?iin=${user.username}`
            `http://127.0.0.1:8000/api/organization/940419300162/`
          );
          this.detailedUserData = response.data;
        }
      } catch (error) {
        console.error("Error fetching detailed user data:", error);
      }
    },
    resetSelection() {
      this.selectedObject = null;
    },
    flattenData(data) {
      return data.reduce((acc, item) => {
        acc.push(item);
        if (item.items && item.items.length) {
          acc.push(...this.flattenData(item.items));
        }
        return acc;
      }, []);
    },

    async saveDraft() {
      // Убираем обязательность для черновика
      this.$refs.form.resetValidation();

      // Отправляем запрос
      await this.sendRequest("Черновик");
    },
    async submitForm() {
      // Включаем обязательность полей

      // Проверяем форму
      this.$refs.form.validate();

      if (this.isValid) {
        // Если форма валидна, отправляем запрос
        await this.sendRequest("Отправлено на согласование");
      }
    },
    async sendRequest(status) {
      this.loading = true;
      this.successMessage = "";

      const payload = {
        uid: this.uuid,
        area: this.project.area,
        supplier: JSON.stringify(this.tableData),
        nomer: this.project.order_number || null,
        datadoc: this.project.order_date || null,
        //org_name: this.detailedUserData.orgName,
        //autor_name: this.detailedUserData.FIO,
        //autor_iin: this.detailedUserData.IIN,
        //departament: this.detailedUserData.cost_name,
        //position: this.detailedUserData.post,
        //org_bin: this.detailedUserData.orgBIN,
        //Comment: this.project.description_works,
        //cost_name: this.selectedObject.name,
        //cost_code: this.selectedObject.code,
        //status: this.project.is_draft || null,
        //SpecialRequirementsOfTheClient: this.client_code.name || null,
        //extra_project: this.project.extra_project,
        //ReferenceDocument: this.project.documents || null,
        //TypeOfExpenditure: this.project.type_expenditure.name,
        area: JSON.stringify(this.detailedUserData),
        Comment: JSON.stringify(this.selectedUser),
        //user: this.selectedUser,
        //deteiluser: this.detailedUserData,
        //deteiluserjson: JSON.stringify(this.detailedUserData),
        //object: this.selectedObject,
        //date_shipment: this.project.date_shipment || null,
        //services: this.tableData || null,
        //matrix: JSON.stringify(this.matrix) || null,
        //refdocs: (await this.prepareRows()) || null,
        //uid: this.uuid,
        //area: this.project.area,
        //supplier: this.project.supplier,
        //nomer: this.project.order_number,
        //datadoc: this.project.order_date,
        //org_name: this.detailedUserData.orgName,
        //autor_name: this.detailedUserData.FIO,
        //autor_iin: this.detailedUserData.IIN,
        //departament: this.detailedUserData.CostName,
        //position: this.detailedUserData.Post,
        //org_bin: this.detailedUserData.orgBIN,
        //description_works: this.project.description_works,
        //cost_name: this.selectedObject.name,
        //cost_code: this.selectedObject.code,
        //is_draft: this.project.is_draft,
        //SpecialRequirementsOfTheClient: this.client_code.name || "",
        //extra_project: this.project.extra_project,
        //ReferenceDocument: this.project.documents,
        //area: this.project.area,
        //TypeOfExpenditure: this.project.type_expenditure.name,
        //date_shipment: this.project.date_shipment,
        //services: this.tableData,
        //matrix: this.matrix,
        //refdocs: await this.prepareRows(),
        //users: this.prepareUser(),
        //services: this.tableData,
        //matrix: this.matrix,
        //refdocs: await this.prepareRows(),
        //project: await this.prepareProject(),

        //tableData,
        //works: this.tableData,
        //docs: await this.handleAllRowsUpload(),
        //matrix: { privet: "aa" },
        status: status,
      };

      try {
        const response = await axios.post(
          "http://127.0.0.1:8000/expenditure/",
          payload
        );

        if (response.status === 200 || response.status === 201) {
          this.successMessage = "Данные успешно отправлены!";

          // Переход на главную страницу через 2 секунды
          setTimeout(() => {
            this.$router.push("/");
          }, 2000);
        }
      } catch (error) {
        console.error("Ошибка при отправке данных:", error);
      } finally {
        this.loading = false;
      }
    },
    async submitForm1() {
      try {
        const url =
          //"http://192.168.0.67/api/v1/websrf/create?iin=" +
          "http://127.0.0.1:8000/expenditures/";
        //this.detailedUserData.IIN;
        const data123 = {
          //  uid: this.uuid,
          //  area: this.project.area,
          //  supplier: this.project.supplier,
          //   number: this.project.order_number,
          // datadoc: this.project.order_date,
          //org_name: this.detailedUserData.orgName,
          //autor_name: this.detailedUserData.FIO,
          //autor_iin: this.detailedUserData.IIN,
          //departament: this.detailedUserData.CostName,
          //position: this.detailedUserData.Post,
          //org_bin: this.detailedUserData.orgBIN,
          // description_works: this.project.description_works,
          // cost_name: this.selectedObject.name,
          // cost_code: this.selectedObject.code,
          // is_draft: this.project.is_draft,
          //  SpecialRequirementsOfTheClient: this.client_code.name || "",
          //  extra_project: this.project.extra_project,
          // ReferenceDocument: this.project.documents,
          // area: this.project.area,
          //TypeOfExpenditure: this.project.type_expenditure.name,
          //date_shipment: this.project.date_shipment,
          //services: this.tableData,
          //matrix: this.matrix,
          //refdocs: await this.prepareRows(),
          //users: this.prepareUser(),
          services: JSON.stringify(this.tableData),
          matrix: this.matrix,
          refdocs: await this.prepareRows(),
          //project: await this.prepareProject(),

          //tableData,
          //works: this.tableData,
          //docs: await this.handleAllRowsUpload(),
          //matrix: { privet: "aa" },
        };
        const response = await axios.post(url, data123, {
          headers: {
            "Content-Type": "application/json",
          },
        });
        console.log("Заявка успешно отправлена:", response.data);
        alert("Заявка успешно отправлена!");
      } catch (error) {
        console.error("Ошибка при отправке заявки:", error);
        alert("Произошла ошибка при отправке.");
      }
    },
  },
};
</script>

<style>
.budget {
  padding: 0;
  margin: 0%;
  width: 100px;
  height: 40px;
}
.no-padding {
  padding: 0 !important;
  margin: 0 !important;
}
</style>
