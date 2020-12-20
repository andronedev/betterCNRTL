<template>
  <div class="home">
    <div>
      <section class="hero is-primary">
        <div class="hero-body">
          <div class="container">
            <h1 class="title">
              BetterCNRTL
            </h1>
            <h2 class="subtitle">
              CNRTL.fr avec une interface amélioré !
            </h2>
          </div>
        </div>
      </section>
      <div class="notif">
        <div
          v-for="error in errors"
          :key="errors.id"
          class="notification is-danger is-light"
        >
          <button
            class="delete"
            v-on:click="errors = errors.filter(item => item.id != error.id)"
          ></button>
          {{ error.msg }}
        </div>
      </div>

      <div class="MainForm">
        <form
          v-on:submit.prevent="go"
          class="field has-addons-centered has-addons"
        >
          <div class="control">
            <input
              class="input"
              type="text"
              placeholder="Rechercher un mot"
              v-model="mot"
            />
          </div>
          <div class="control">
            <button class="button is-link is-success" >
              Chercher
            </button>
          </div>
        </form>
      </div>
      <div class="container mt-4">
        <div
          v-if="contentHTML != ''"
          class="card p-3"
          v-html="contentHTML"
        ></div>
        <div v-else class="card p-3">
          <h1>En attente de recherche ^^</h1>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
const axios = require("axios");
import { parse } from "node-html-parser";

export default {
  data() {
    return {
      contentHTML: "",
      mot: "",
      errors: []
    };
  },
  methods: {
    go() {
      self = this;
      self.data = "";

      axios
        .get(`https://www.cnrtl.fr/definition/${self.mot}`)
        .then(function(response) {
          var data = parse(response.data);
          data = data.querySelector("#lexicontent");
          if (response.status != 200 || data == null) {
            self.contentHTML = "";
            self.addError("Aucun Resultats pour " + self.mot);
          } else {
            self.contentHTML = data;
          }
        })
        .catch(function(error) {
          console.log(error);
          self.data = "";
          self.error = true;
        })
        .then(function() {
          // always executed
        });
    },
    addError(err = "Erreur") {
      var e = self.errors.push({ msg: err, date: new Date() });
    },
    deleteNotifs() {
      var maxLifespan = 2500;
      self = this;
      // check once per second
      setInterval(function checkItems() {
        try {
          self.errors.forEach(function(item) {
            if (Date.now() - maxLifespan > item.date) {
              self.errors.shift(); // remove first item
            }
          });
        } catch (_) {}
      }, 1000);
    }
  },
  mounted() {
    this.deleteNotifs();
  }
};
</script>

<style>
.home {
  margin: 0 auto;
  min-height: 100vh;
}

.notif {
  position: absolute;
  right: 10px;
  top: 30px;
}
.title {
  font-family: "Quicksand", "Source Sans Pro", -apple-system, BlinkMacSystemFont,
    "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
}

.MainForm {
  padding-top: 15px;
}

.tlf_cdefinition {
  background-color: #98ec38a9;
}
.tlf_csyntagme {
  background-color: #c0fffaa9;
}
</style>
