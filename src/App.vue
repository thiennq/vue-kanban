<template>
  <div id="app">
    <kanban-board :stages="stages" :blocks="blocks" @update-block="updateBlock">
      <div v-for="block in blocks" :slot="block.id" :key="block.id">
    <div>
      <strong>id:</strong> {{ block.id }}
    </div>
    <div>
      {{ block.title }}
    </div>
  </div>
    </kanban-board>
  </div>
</template>

<script>
import Vue from "vue";
import vueKanban from "vue-kanban";
import Hamoni from "hamoni-sync";
const ACCOUNT_ID = '58d2c9b9-9732-4391-bd8d-8a7101ebbbb8';
const APP_ID = 'bab4dfafefb84acab457f3d5be7670e1';

Vue.use(vueKanban);

export default {
  name: "app",
  data() {
    return {
      stages: ["on-hold", "in-progress", "needs-review", "approved"],
      blocks: [],
      listPrimitive: null
    };
  },
  methods: {
    updateBlock(id, status) {
      let block = this.blocks[id];
      this.listPrimitive.update(id, { id, title: block.title, status });
    }
  },
  mounted: async function() {
    let hamoni;

    const response = await fetch("https://api.sync.hamoni.tech/v1/token", {
      method: "POST",
      headers: {
        "Content-Type": "application/json; charset=utf-8"
      },
      body: JSON.stringify({ accountId: ACCOUNT_ID, appId: APP_ID })
    });
    const token = await response.json();
      hamoni = new Hamoni(token);

      hamoni
        .connect()
        .then(() => {
          hamoni
            .get("board-12")
            .then(listPrimitive => {
              this.listPrimitive = listPrimitive;
              this.blocks = listPrimitive.getAll();
              listPrimitive.onItemUpdated(item => {
                this.blocks.splice(item.index, 1, item.value);
              });
            })
            .catch(error => {
              if (error == "Error getting state from server") {
                hamoni
                  .createList("board-12", blocks)
                  .then(listPrimitive => {
                    this.listPrimitive = listPrimitive;
                    this.blocks = listPrimitive.getAll();
                    listPrimitive.onItemUpdated(item => {
                      this.blocks.splice(item.index, 1, item.value);
                    });
                  })
                  .catch(console.log);
              }
            });
        })
        .catch(console.log);
    
  }
};

const blocks = [
  {
    id: 0,
    status: "approved",
    title: "Buy coffee machine"
  },
  {
    id: 1,
    status: "in-progress",
    title: "Find better AirBnB options"
  },
  {
    id: 2,
    status: "on-hold",
    title: "Approve Q3 budget"
  },
  {
    id: 3,
    status: "approved",
    title: "Travel to Guaraguara"
  },
  {
    id: 4,
    status: "needs-review",
    title: "Add Redux to the app"
  },
  {
    id: 5,
    status: "approved",
    title: "Well, Sleep all day 👩‍🎤"
  },
  {
    id: 6,
    status: "in-progress",
    title: "Find language exchange partner"
  }
];
</script>

<style lang="css">
@import "./assets/kanban.css";
</style>
