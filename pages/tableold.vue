<template>
  <div>
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

              <td width="10%">description</td>

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
  </div>
</template>
