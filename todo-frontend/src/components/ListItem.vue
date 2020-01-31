<template>
  <div>
    <!-- @click odnosno v-on:click je Vue direktiva koja se okida kada se klike misem
      na element, addEventLister je analogni koncept u js-u.
      mi kada kliknemo pozivamo metodu iz methods objekta u script sekciji i prosledjujemo joj item,
      itemm koji dobijamo iz v-for direktive prolaskom kroz iterabilnu listu
    v-bind:key ili :key sluzi samo da jedinstveno identifikuje element u v-for-u
    :variant je primary, success, danger... bootstrap helper klasa
    -->
    <!-- v-for dirketiva sluzi da renderuje nesto sto je itreabilno bez kopiranja koda, npr. -->
    <!-- Pokazati ovaj primer kako radi: <div v-for="item in 5" :key="item.id">{{ item }}</div> -->
    <b-list-group-item
      style="cursor: pointer;"
      @click="changeStatus(item)"
      v-for="(item, id) in list.itemi"
      :key="id"
      id="list-item"
      class="d-flex justify-content-between align-items-center"
      :variant="(item.status == '0')?'info':'success'"
    >
      <!-- :syle ili v-bind:syle, sluzi da se u prop ubaci neki dinamicki content,
      npr. :style = '"color": stildugmeta', gde bi stilldugmeta bio neki prop ili data objekat-->
      <div
        :style="(item.status == '0')?'textDecoration: none;':'textDecoration: line-through'"
      >{{id + 1}}. {{item.content}}</div>
      <div>
        <!-- click.stop sluzi da spreci Event bubbling -->
        <b-badge @click.stop="deleteItem(item)" href="#" variant="danger" pill>X</b-badge>
      </div>
    </b-list-group-item>

    <b-input-group class="mt-3" style="z-index: 0" prepend="New item">
      <!-- v-model direktiva sluzi za 2way data binding, sve sto se unese u inputu, automatski
      je uneto u newListTitle data objektu, vazi i obrnuto-->
      <b-form-input v-model="newItem"></b-form-input>
      <b-input-group-append>
        <b-button @click="addItem(list)" size="sm" text="Add" variant="success">Add</b-button>
      </b-input-group-append>
    </b-input-group>
  </div>
</template>

<script>
export default {
  // Props, saljemo property-je iz roditelja u dete, ovo je dete komponenta
  props: {
    list: {
      type: Object
    }
  },
  // Data funkcija, sadrzi nase reaktivne objekte, svaka promena nad ovim objektima se automatski
  // reflektuje i u template
  data() {
    return {
      newItem: ""
    };
  },
  methods: {
    async deleteItem(item) {
      await this.$http.delete(`${this.$store.state.baseUrl}/itemi/${item.id}`);
      // Signal roditelju da se desio neki event
      this.$emit("update-lists");
      // VAZNO primetiti da se iz metoda mora sve pristupati preko this, jer Vue sve binduje na this
    },
    async changeStatus(item) {
      const resBody = {
        status: item.status == "0" ? "1" : "0",
        content: item.content,
        list_id: item.list_id
      };
      await this.$http.put(
        `${this.$store.state.baseUrl}/itemi/${item.id}`,
        resBody
      );
      // Signal roditelju da se desio neki event
      this.$emit("update-lists");
    },
    async addItem(list) {
      // Obicna validacija
      if (this.newItem.length < 1) return;
      const resBody = { content: this.newItem, status: 0, list_id: list.id };
      await this.$http.post(`${this.$store.state.baseUrl}/itemi`, resBody);
      this.newItem = "";
      // Signal roditelju da se desio neki event
      this.$emit("update-lists");
    }
  }
};
</script>

<style scoped>
</style>

Kreirajte List.vue komponentu
<template>
  <div>
    <b-container>
      <b-row>
        <!-- v-for dirketiva sluzi da renderuje nesto sto je itreabilno bez kopiranja koda, npr. -->
        <!-- Pokazati ovaj primer kako radi: <div v-for="item in 5" :key="item.id">{{ item }}</div> -->
        <!-- :key sluzi da jedinstveno indentifikuje item u listi -->
        <b-col xs="12" sm="6" md="4" v-for="list in lists" :key="list.id">
          <div class="controls">
            <!-- Kada se klikne na ovaj badge koji sluzi za brisanje, poziva  se metoda delteList, .prevent sluzi da
            ne bi se izvrsio href-->
            <b-badge @click.prevent="deleteList(list)" href="#" variant="danger" pill>X</b-badge>
          </div>
          <b-card-group deck>
            <b-card class="mt-5 mx-1" bg-variant="light" no-body :header="list.title">
              <b-list-group class="mt-3" flush>
                <!-- Ovde prosledjujemo listu itema ListItem komponenti koja u sebi
                renderuje stavke u odnosu na listu i osluskujemo na event
                update-lists koji ako se desi pozivamo updateUi metodu-->
                <ListItem @update-lists="updateUi" :list="list" />
              </b-list-group>
            </b-card>
          </b-card-group>
        </b-col>
        <b-col xs="12" sm="6" md="4">
          <b-card-group deck>
            <b-card class="mt-5 mx-1" no-body>
              <b-input-group prepend="New list">
                <!-- v-model direktiva sluzi za 2way data binding, sve sto se unese u inputu, automatski
                je uneto u newListTitle data objektu, vazi i obrnuto-->
                <b-form-input v-model="newListTitle"></b-form-input>
                <b-input-group-append>
                  <b-button @click="addList" size="sm" text="Add" variant="success">Add</b-button>
                </b-input-group-append>
              </b-input-group>
            </b-card>
          </b-card-group>
        </b-col>
      </b-row>
    </b-container>
  </div>
</template>

<script>
import ListItem from "@/components/ListItem.vue";
export default {
  components: {
    ListItem
  },

  methods: {
    // this.$store.state.baseUrl uzimanje URL-a do backa
    async updateUi() {
      const response = await this.$http.get(
        `${this.$store.state.baseUrl}/liste.json`
      );
      this.lists = response.data;
    },
    async deleteList(list) {
      await this.$http.delete(`${this.$store.state.baseUrl}/liste/${list.id}`);
      this.updateUi();
    },

    async addList() {
      if (this.newListTitle.length < 1) return;
      await this.$http.post(`${this.$store.state.baseUrl}/liste`, {
        title: this.newListTitle
      });
      this.newListTitle = "";
      this.updateUi();
    }
  },
  async mounted() {
    this.updateUi();
  },
  data() {
    return {
      lists: [],
      newListTitle: ""
    };
  }
};
</script>

<style scoped>
.controls {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  position: absolute;
  width: 100%;
  top: 40px;
  left: 0;
  z-index: 1;
}
@media (max-width: 576px) {
  .controls {
    padding: 0 10px;
  }
}
</style>
