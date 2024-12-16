<template>
  <v-container>
    <v-autocomplete
      v-model="selectedItem"
      :items="items"
      label="Выберите элемент"
      item-text="name"
      item-value="id"
      return-object
      @change="onItemSelected"
    ></v-autocomplete>
  </v-container>
</template>

<script>
export default {
  data() {
    return {
      items: [
        { id: 1, name: "Элемент 1" },
        { id: 2, name: "Элемент 2" },
        { id: 3, name: "Элемент 3" },
      ],
      selectedItem: null, // выбранный элемент
    };
  },
  mounted() {
    // Автоматически выбираем первый элемент при загрузке
    if (this.items.length) {
      this.selectedItem = this.items[0];
    }
  },
  methods: {
    onItemSelected(value) {
      console.log("Выбран элемент:", value);
    },
  },
};
</script>
