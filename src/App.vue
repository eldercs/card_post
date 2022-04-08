<template>
  <div class = "input-group">
    <input
      type="text"
      placeholder="Введите ID категории"
      class="input-search form-control"
      v-model="id_category"
    />
    <button type="button" class="btn btn-secondary" @click="search">
      Загрузить
    </button>
  </div>
  <form @submit.prevent="saveJSON">
    <div class="main-list">
      <div v-for="(card, index) in data_cards.attributes" :key="card.id">
        <div class="redactor">
          <p>
            name:<input
              type="text"
              class="form-control"
              v-model="data_cards.attributes[index].name"
            />
          </p>
          <p>
            dectionary_id:<input
              type="text"
              class="form-control"
              v-model="data_cards.attributes[index].dectionary_id"
            />
          </p>
          <p>
            description:
            <textarea
              name=""
              id=""
              cols="30"
              rows="10"
              class="form-control"
              v-model="data_cards.attributes[index].description"
            ></textarea>
          </p>
          <p>
            group_id:<input
              type="text"
              class="form-control"
              v-model="data_cards.attributes[index].group_id"
            />
          </p>
          <p>
            group_name:<input
              type="text"
              class="form-control"
              v-model="data_cards.attributes[index].group_name"
            />
          </p>
          <p>
            type:<input
              type="text"
              class="form-control"
              v-model="data_cards.attributes[index].type"
            />
          </p>
          <p>
            collection:<span v-if="data_cards.attributes[index].is_required"
              >*</span
            ><input
              type="text"
              class="form-control"
              v-model="data_cards.attributes[index].collections"
              @click = "activeElem = index"
            />
          </p>
          <span
            class="errors"
            v-if="errors[index] && !data_cards.attributes[index].collections"
            >Заполните поле!</span
          >
          <div :class="{'nope': (activeElem != index || data_cards.attributes[index].collections === undefined || startSearch(data_cards.attributes[index].collections)==0)}" class="autoComplit border-top-0 border-right-0 border-left-0">
            <img src="./img/close.svg" width="25" class = "close" :class="{'nope': (data_cards.attributes[index].collections == undefined)}" @click = "activeElem = 'none'" />
            <div
              v-for="s in startSearch(data_cards.attributes[index].collections)"
              :key="s"
            >
              <span @click="choiceSearch(s, index)">
                {{ s }}
              </span>
            </div>
          </div>
        </div>
      </div>
    </div>
    <button class="btn subButton btn-secondary" :class="{nope: data_cards == ''}" type="submit">
      Сохранить JSON
    </button>
    <p v-if="errors.length >= 1">Исправьте ошибки</p>
  </form>
  <form @submit.prevent="loadJSON">
    <div class = "loaderJson">
      <textarea name="" id="" rows="12" class="form-control" placeholder="Сюда JSON" v-model="data_cardsJson"></textarea>
    </div>
    <button type="submit" class="btn btn-secondary">Загрузить из JSON</button>
  </form>
</template>

<script>
import { computed } from "@vue/runtime-core";
//17034410
export default {
  name: "App",
  data() {
    return {
      data_cards: "",
      data_cardsJson: "",
      id_category: "17034410",
      collections: "",
      query: [],
      errors: [],
      activeElem: "",
      checkNull: ""
    };
  },

  methods: {
    async search() {
      let api = await fetch(
        `https://api-dev.shop-delivery.ru/int_api/ozon/v3/category/attribute`,
        {
          method: "POST",
          body: JSON.stringify({
            attribute_type: "ALL",
            category_id: [this.id_category],
            language: "DEFAULT",
          }),
          headers: {
            "Content-type": "application/json; charset=UTF-8",
          },
        }
      );
      let data = await api.json();
      this.collection(this.id_category);
      this.data_cards = data.result[0];
      this.data_cards.id_cat = this.id_category;
    },
    async collection(id_category) {
      let api = await fetch(
        `https://api-dev.shop-delivery.ru/int_api/ozon/v2/category/attribute/values`,
        {
          method: "POST",
          body: JSON.stringify({
            attribute_id: 10096,
            category_id: id_category,
            language: "DEFAULT",
            last_value_id: 0,
            limit: 20,
            query: "",
          }),
          headers: {
            "Content-type": "application/json; charset=UTF-8",
          },
        }
      );
      this.collections = await api.json();
    },
    saveJSON() {
      this.errors = [];
      for (let key in this.data_cards.attributes) {
        let checkError = this.data_cards.attributes;
        if (
          checkError[key].is_required === true &&
          checkError[key].collections === undefined ||
          checkError[key].collections === ""
        ) {
          this.errors[key] = "обазательное поле";
        }
      }
      if (this.errors.length == 0) {
        this.data_cardsJson = JSON.stringify(this.data_cards);
      }
    },
    loadJSON() {
      this.data_cards = JSON.parse(this.data_cardsJson);
      this.collection(this.data_cards.id_cat);
    },
    choiceSearch(s, index) {
      this.data_cards.attributes[index].collections = s;
    },
    startSearch(item) {
      if (item !== undefined && this.collections.length >= 20) {
        const a = this.collections;
        const resultFilter = [];
        for (let key in a) {
          resultFilter.push(a[key].value);
        }
        if(resultFilter.length < 1){
          return 0
        }
        return resultFilter
          .filter((el) => el.toLowerCase().includes(item.toLowerCase()))
          .splice(0, 4);
      }
    },
  },
};
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
textarea {
  max-width: 900px;
  width: 100%;
  resize: none;
  margin:0 auto;
}

.input-group{
  max-width: 900px;
  width:100%;
  margin: 0 auto;
}
.input-search {
  margin-right: 10px;
}
.main-list {
  display: flex;
  flex-wrap: wrap;
  max-width:1200px;
  width:100%;
  margin:20px auto;
}
.errors {
  position: absolute;
  color: red;
  bottom:5px;
  left:10px;
}
.close{
  position:absolute;
  top: 0;
  right:0;
}
.redactor {
  position:relative;
  font-size: 14px;
  width: 200px;
  border: 1px solid grey;
  padding-bottom:30px;
  textarea {
    width: 180px;
  }
  p{
    text-align:left;
    margin:5px;
  }
}
.collection-text {
  position: relative;
  text-align: center;
}
.collection {
  position: absolute;
  width: 150px;
  top: 20px;
  left: 20px;
}
.autoComplit{
  position:absolute;
  display:flex;
  z-index:20;
  background-color:white;
  max-width:198px;
  width:100%;
  border: 1px solid black;
  padding-right:20px;
  /* justify-content: space-between; */
  flex-wrap: wrap;
  div{
    border:1px solid black;
    border-radius: 10px;
    margin:5px;
    padding:5px;
  }
}
.nope{
  display:none;
}
.loaderJson{
  margin: 20px auto;
}
</style>
