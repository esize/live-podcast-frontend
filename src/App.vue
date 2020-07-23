<template>
  <div id="app">
    <div v-if="results.length > 0" class="results-container">
      <h1>
        Results:
        <span class="close" @click="clearResults">&times;</span>
      </h1>
      <ul>
        <li
          v-for="(result, index) in results"
          :key="result"
          class="special-card"
        >{{index + 1}}. {{result}}</li>
      </ul>
    </div>
    <div v-else>
      <Modal btnText="Open Modal" openByDefault v-if="!me.name">
        <form action @submit.prevent="createUser(inputName)">
          <label for="name">Enter your name:</label>
          <input id="name" type="text" v-model="inputName" />
          <button type="submit">Start</button>
        </form>
      </Modal>

      <div v-if="me.name">
        <SlickList lockAxis="y" v-model="me.albums" @input="changed = true" class="grid-container">
          <div class="list">
            <h2>{{me.name}}</h2>
            <SlickItem
              v-for="(item, index) in me.albums"
              :index="index"
              :key="index"
              :disabled="inputDisabled"
              class="slick-item"
            >{{index +1 }}. {{ item.title }}</SlickItem>
          </div>
          <div class="list" v-for="user in users" :key="user.name">
            <div v-if="user.name !== me.name">
              <h2>{{user.name}}</h2>
              <ul>
                <li
                  v-for="album in user.albums"
                  :key="album.title"
                  class="socket-item"
                >{{album.title}}</li>
              </ul>
            </div>
          </div>
        </SlickList>
        <button @click="tabulate" class="results-container">Calculate Results</button>
      </div>
    </div>
  </div>
</template>

<script>
import { SlickList, SlickItem } from "vue-slicksort";
import io from "socket.io-client";
import Modal from "./components/Modal";

export default {
  name: "App",
  components: {
    SlickItem,
    SlickList,
    Modal,
  },
  data() {
    return {
      inputDisabled: false,
      changed: false,
      socket: io("localhost:5000"),
      me: {},
      inputName: "",
      users: {},
      results: [],
    };
  },
  watch: {
    changed: function () {
      this.socket.emit("UPDATE", {
        uuid: localStorage.uuid,
        name: this.me.name,
        albums: this.me.albums,
      });
      this.changed = false;
    },
  },
  mounted() {
    if (localStorage.uuid && localStorage.name) {
      this.me = newUserObject(localStorage.name);
      this.socket.emit("REFRESH", "");
    }
    this.socket.on("UPDATE", (data) => {
      this.users = data;
    });
    this.socket.on("set id", (id) => {
      localStorage.uuid = id;
      localStorage.name = this.me.name;
    });
    this.socket.on("DELETE TOKEN", () => {
      localStorage.uuid = "";
      window.location.reload();
    });
    this.socket.on("TABULATING", () => {
      this.inputDisabled = true;
    });
    this.socket.on("RESULTS", (data) => {
      let localResults = [];
      for (let result in data) {
        if (data[result]) {
          localResults.push(data[result]);
        }
      }
      this.results = localResults;
    });
  },
  methods: {
    tabulate() {
      this.socket.emit("TABULATE", "");
    },
    createUser(name) {
      const newUser = newUserObject(name);
      this.me = newUser;
      this.socket.emit("CREATE_USER", newUser);
      this.inputName = "";
    },
    clearResults() {
      let doClear = window.confirm(
        "Are you sure you want to clear the results?"
      );
      if (doClear) {
        this.socket.emit("CLEAR_RESULTS");
      }
    },
  },
};
function newUserObject(name) {
  return {
    name,
    albums: [
      {
        title: "Zentropy",
        cover: "",
        order: 0,
      },
      {
        title: "I'm Wide Awake, It's Morning",
        cover: "",
        order: 1,
      },
      {
        title: "If You're Feeling Sinister",
        cover: "",
        order: 2,
      },
      {
        title: "In My Room",
        cover: "",
        order: 3,
      },
    ],
  };
}
</script>

<style lang="css">
@import url("https://fonts.googleapis.com/css2?family=Roboto&display=swap");

input[type="text"],
select {
  width: 100%;
  padding: 12px 20px;
  margin: 8px 0;
  display: block;
  border: 1px solid #ccc;
  border-radius: 4px;
  box-sizing: border-box;
}

input[type="submit"] {
  width: 100%;
  background-color: #4caf50;
  color: white;
  padding: 14px 20px;
  margin: 8px 0;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

input[type="submit"]:hover {
  background-color: #45a049;
}

button {
  background-color: #515050;
  border: none;
  color: white;
  padding: 10px 32px;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 16px;
  margin: 4px 2px;
  cursor: pointer;
}

body {
  margin: 0;
  background-color: #f7f8fb;
  font-family: "Roboto", sans-serif;
}

.results-container {
  margin: 2em;
}

h2 {
  font-size: 2em;
}

.list {
  margin: 1.5em;
}

.slick-item,
.socket-item,
.special-card {
  padding: 2em;
  background-color: #ffffff;

  -webkit-touch-callout: none; /* iOS Safari */
  -webkit-user-select: none; /* Safari */
  -khtml-user-select: none; /* Konqueror HTML */
  -moz-user-select: none; /* Old versions of Firefox */
  -ms-user-select: none; /* Internet Explorer/Edge */
  user-select: none; /* Non-prefixed version, currently
                                  supported by Chrome, Edge, Opera and Firefox */

  border-radius: 0.3em;
  box-shadow: 10px 10px 30px rgba(0, 0, 0, 0.1);
  margin-top: 1em;
}

.special-card .clear {
  text-align: center !important;
}

ul {
  list-style-type: none;
  padding: 0;
}

.grid-container {
  display: grid;
  grid-gap: 2em;
  grid-template-columns: repeat(auto-fit, minmax(350px, 1fr));
}
</style>