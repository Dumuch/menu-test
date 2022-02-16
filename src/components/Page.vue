<template>
  <main>
    <div class="container">
      <component-sidebar></component-sidebar>
      <div class="content">
        <section class="section-fundraising">
          <h1 class="title title-h1 section-fundraising__title">
            Создание сбора
          </h1>

          <form action="" class="form">
            <fieldset class="fieldset">
              <legend class="title form__title">Информация о сборе</legend>
              <div class="wrapper__form__label-input">
                <label
                  class="title form__label title--size-s title--color-gray"
                  for="fundraising-name"
                  >Название сбора
                </label>
                <input
                  class="input form__input"
                  id="fundraising-name"
                  type="text"
                />
              </div>
            </fieldset>

            <fieldset class="fieldset fieldset--flex-column">
              <legend class="title form__title">Целевые группы</legend>
              <span
                class="title form__label title--size-s title--color-gray"
                for="name"
                >Укажите, кому помогает ваша организация
              </span>

              <div
                class="wrapper__choose-list"
                v-for="chooseGroup in filterChooseGroupsList"
                :key="chooseGroup.id"
              >
                <ul class="list choose-list form__list">
                  <li
                    class="title choose-item title--size-s"
                    v-for="item in chooseGroup.list"
                    :key="item.id"
                  >
                    {{ item.name }}
                  </li>
                </ul>
                <button
                  @click.prevent="deleteLastChooseGroup(chooseGroup.id)"
                  class="button-close choose-list__button"
                >
                  Удалить целевую группу
                </button>
              </div>

              <div class="wrapper__button__menu-list">
                <ul class="list menu-list" v-if="isShowMenuList">
                  <li class="menu-list__item" v-if="beforeGroup">
                    <button
                      @click.prevent="beforeMenuList(beforeGroup.id)"
                      type="button"
                      class="menu-list__button menu-list__button--before"
                    >
                      {{ beforeGroup.name }}
                    </button>
                  </li>
                  <li
                    class="menu-list__item"
                    v-for="item in menuList"
                    :key="item.id"
                  >
                    <button
                      v-if="isGroups(item)"
                      class="menu-list__button menu-list__button--next"
                      @click.prevent="onChooseGroup(item)"
                    >
                      {{ item.name }}
                    </button>

                    <button
                      @click.prevent.once="onChooseGroup(item)"
                      type="button"
                      v-else
                      class="menu-list__button"
                    >
                      {{ item.name }}
                    </button>
                  </li>
                </ul>

                <button
                  @click.prevent="openMenuList"
                  class="button-choose form__button-choose"
                >
                  <span class="button-choose__plus"></span>
                  Выбрать еще
                </button>
              </div>
            </fieldset>

            <div class="line form__line"></div>
            <button
              @click.prevent="submit"
              type="submit"
              class="button form__button button--color-orange"
            >
              cохранить и продолжить
            </button>
          </form>
        </section>
      </div>
    </div>
  </main>
</template>

<script>
import Sidebar from "./Sidebar.vue";

export default {
  data() {
    return {
      groups: [],
      chooseGroups: [],
      chooseGroupsList: [],
      activeIdGoup: null,
      isShowMenuList: false,
      breadCrumbs: [],
      index: 0,
    };
  },

  components: {
    "component-sidebar": Sidebar,
  },

  async mounted() {
    this.groups = await fetch("/data.js")
      .then((r) => r.json())
      .then((r) => r.groups);
  },

  methods: {
    isGroups(item) {
      if (item.groups.length !== 0) {
        return true;
      } else {
        return false;
      }
    },

    openMenuList() {
      this.isShowMenuList = !this.isShowMenuList;
      this.chooseGroups = [];
      this.breadCrumbs = [];
      this.index = 0;
    },

    deleteLastChooseGroup(id) {
      this.chooseGroupsList.map((item, index) => {
        if (item.id === id) {
          item.list.pop();
          if (item.list.length === 0) {
            this.chooseGroupsList.splice(index, 1);
          }
        }
      });

      this.chooseGroups = [];
    },

    deepSearch(arr, item) {
      for (var i = 0; i < arr.length; i++) {
        if (
          arr[i].groups.length !== 0 &&
          this.deepSearch(arr[i].groups, item)
        ) {
          this.breadCrumbs.push(arr[i]);
          return this.breadCrumbs;
        } else if (arr[i].id === item) {
          this.breadCrumbs.push(arr[i]);
          return this.breadCrumbs;
        }
      }
      return false;
    },

    onChooseGroup(item, isPrev = true) {
      const arr = this.groups;
      this.breadCrumbs = [];
      if (item.groups.length !== 0) {
        this.chooseGroups = item.groups;
        if (isPrev) {
          this.index++;
        }
      }
      const res = arr.find((val) => val.id === item.id);
      if (res) {
        this.activeIdGoup = res.id;
      }

      if (this.chooseGroupsList.length !== 0) {
        let group = this.chooseGroupsList.find(
          (item) => item.id === this.activeIdGoup
        );

        if (group) {
          if (!group.list.find((group) => group.id === item.id)) {
            group.list = this.deepSearch(arr, item.id).reverse();
          } else {
            this.deepSearch(arr, item.id).reverse();
          }
        } else {
          const obj = {
            id: this.activeIdGoup,
            list: this.deepSearch(arr, item.id).reverse(),
          };
          this.chooseGroupsList.push(obj);
        }
      } else {
        const obj = {
          id: this.activeIdGoup,
          list: this.deepSearch(arr, item.id).reverse(),
        };
        this.chooseGroupsList.push(obj);
      }
    },

    beforeMenuList() {
      if (this.index >= 2) {
        this.index = this.index - 1;
        this.onChooseGroup(this.breadCrumbs[this.index - 1], false);
      } else if (this.index == 1) {
        this.index = 0;
        this.chooseGroups = [];
      }
    },

    submit() {
      const arr = [];
      this.chooseGroupsList.forEach((item) => {
        item.list.forEach((item) => arr.push(item.id));
      });

      console.log(arr.reverse());
    },
  },

  computed: {
    menuList() {
      if (this.chooseGroups.length !== 0) {
        return this.chooseGroups;
      } else {
        return this.groups;
      }
    },

    filterChooseGroupsList() {
      return this.chooseGroupsList;
    },

    beforeGroup() {
      return this.breadCrumbs[this.index - 1];
    },
  },
};
</script>
