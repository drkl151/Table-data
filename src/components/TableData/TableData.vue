<template>
  <div>
    <div class="search-block">
      <div class="wrapper__search-input">
        <img
          class="search-icon"
          src="./img/search-icon.svg"
          alt="search-icon"
        />
        <input
          class="search-input"
          placeholder="Search"
          type="text"
          v-model="searchText"
        />
      </div>
    </div>

    <div class="table-wrapper">
      <div class="table">
        <tbody v-if="searchName.length" class="tbody">
          <tr>
            <th>
              <input
                class="checkbox"
                @click="checkMainCheckbox"
                type="checkbox"
                v-model="mainChecked"
              />
            </th>
            <th
              class="sort-button"
              @click="
                (sortingCategory = 'number'), (sortedByNumber = !sortedByNumber)
              "
              :v-model="sortedByNumber"
            >
              Номер
            </th>
            <th
              class="sort-button"
              @click="
                (sortingCategory = 'name'), (sortedByName = !sortedByName)
              "
            >
              Название
            </th>
            <th>Описание</th>
          </tr>
          <tr v-for="dataObject in searchName" :key="dataObject.id">
            <td>
              <input
                class="checkbox"
                type="checkbox"
                v-model="dataObject.checked"
              />
            </td>
            <td>{{ dataObject.age }}</td>

            <td
              v-if="editingName !== dataObject.age"
              @dblclick="enterEditMode(dataObject, 'name')"
            >
              {{ dataObject.name }}
            </td>

            <td v-else>
              <input
                class="edit-input"
                v-model="dataObject.name"
                type="text"
                placeholder="Введите название"
              />
              <button  class="button-ok" @click="saveEdit">Ok</button>
              <button
                class="button button-cancel"
                @click="undoEditing(dataObject)"
              >
                Cancel
              </button>
            </td>

            <td
              v-if="editingSnippet !== dataObject.age"
              @dblclick="enterEditMode(dataObject, 'snippet')"
            >
              {{ dataObject.snippet }}
            </td>
            <td v-else>
              <input
                class="edit-input"
                v-model="dataObject.snippet"
                type="text"
                placeholder="Введите название"
              />
              <button class="button-ok"  @click="saveEdit">Ok</button>
              <button class="button button-cancel" @click="undoEditingSnippet(dataObject)">
                Cancel
              </button>
            </td>
          </tr>
        </tbody>
        <h4 class="error-no-results-found" v-else>
          По запросу ничего не найдено
        </h4>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import searchIcon from './img/search-icon.svg';

export default {
  data() {
    return {
      dataArray: [],
      searchText: '',
      sortingCategory: '',
      defaultName: '',
      defaultSnippet: '',
      sortedByNumber: false,
      sortedByName: false,
      mainChecked: false,
      editingName: null,
      editingSnippet: null,
      searchIcon,
    };
  },

  mounted() {
    axios
      .get(
        'https://mate-academy.github.io/phone-catalogue-static/api/phones.json'
      )
      .then(({ data }) => {
        this.dataArray = data.map((obj) => {
          return { ...obj, checked: false };
        });
      })
      .catch((error) => {
        console.log(error);
      });
    document.addEventListener('click', this.saveEdit);
  },
  updated() {
    if (this.dataArray.some((el) => el.checked === false)) {
      this.mainChecked = false;
    } else if (this.dataArray.every((el) => el.checked === true)) {
      this.mainChecked = true;
    }
  },

  methods: {
    sortByNumber(e1, e2) {
      return this.sortedByNumber ? e2.age - e1.age : e1.age - e2.age;
    },

    sortByName(e1, e2) {
      return this.sortedByName
        ? e1.name.toLowerCase() > e2.name.toLowerCase()
          ? 1
          : -1
        : e2.name.toLowerCase() > e1.name.toLowerCase()
        ? 1
        : -1;
    },

    checkMainCheckbox() {
      this.dataArray.forEach((item) => {
        item.checked = this.mainChecked ? false : true;
      });
    },

    enterEditMode(dataObject, key) {
      if (key === 'name') {
        this.editingName = dataObject.age;
        this.defaultName = dataObject.name;
      } else if (key === 'snippet') {
        this.editingSnippet = dataObject.age;
        this.defaultSnippet = dataObject.snippet;
      }
    },

    undoEditing(dataObject) {
      dataObject.name = this.defaultName;
      this.editingName = null;
    },

    undoEditingSnippet(dataObject) {
      dataObject.snippet = this.defaultSnippet;
      this.editingSnippet = null;
    },

    saveEdit(e) {
      const newValueName = this.dataArray.filter(
        (obj) => obj.age === this.editingName
      );
      const newValueSnippet = this.dataArray.filter(
        (obj) => obj.age === this.editingSnippet
      );
      const currentElementClass = e.srcElement.className;

      if (
        this.editingName !== null &&
        currentElementClass !== 'edit-input' &&
        currentElementClass !== 'button'
      ) {
        this.defaultName = newValueName.name;
        this.editingName = null;
      }
      if (
        this.editingSnippet !== null &&
        currentElementClass !== 'edit-input' &&
        currentElementClass !== 'button'
      ) {
        this.defaultSnippet = newValueSnippet.snippet;
        this.editingSnippet = null;
      }
    },
  },

  watch: {
    searchText() {
      console.log(this.dataArray);
      if (!this.dataArray.length) {
        this.noResultsFound = true;
      }
    },
  },

  computed: {
    sortTheList() {
      let sortResult = this.dataArray;
      const copyDataArray = [...this.dataArray];
      if (this.sortingCategory === 'number') {
        sortResult = copyDataArray.sort(this.sortByNumber);
      } else if (this.sortingCategory === 'name') {
        sortResult = copyDataArray.sort(this.sortByName);
      }
      return sortResult;
    },
    searchName() {
      return this.sortTheList.filter((el) =>
        el.name.toLowerCase().includes(this.searchText.toLowerCase())
      );
    },
  },
};
</script>


<style lang="scss">
@import './TableData.scss';
</style>
