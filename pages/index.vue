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

      <div class="mt-5">
        <form
          v-on:submit.prevent="go"
          class="field has-addons has-addons-centered"
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
            <button class="button is-link is-success">
              Chercher
            </button>
          </div>
        </form>
      </div>

      <div class="container mt-4 has-background-grey-light p-2 r-2">
        <div class="tabs is-fullwidth is-boxed is-toggle">
          <ul>
            <li v-for="tab in endpoints" :key="tab.id">
              <a @click="get(tab.id)">
                <span>{{ tab.title }}</span>
              </a>
            </li>
          </ul>
        </div>

        <div class="card p-3 mt-3" v-if="c">
          <p class="title is-4">{{ c.title }}</p>

          <div v-html="c.html" class="htmlbox"></div>
        </div>
        <div class="card p-3 mt-3" v-else>
          <h1>Veuillez entrer votre recherche</h1>
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
      content: [],
      c: {},
      mot: "",
      errors: [],
      index: 0,
      endpoints: [
        {
          id: 0,
          url: "https://www.cnrtl.fr/definition/{q}",
          title: "Definition",
          selector: "#contentbox"
        },
        {
          id: 1,
          url: "https://www.cnrtl.fr/morphologie/{q}",
          title: "Morphologie",
          selector: "#contentbox"
        },
        {
          id: 2,
          url: "https://www.cnrtl.fr/etymologie/{q}",
          title: "Etymologie",
          selector: "#contentbox"
        },
        {
          id: 3,
          url: "https://www.cnrtl.fr/synonymie/{q}",
          title: "Synonymie",
          selector: "#contentbox"
        },
        {
          id: 4,
          url: "https://www.cnrtl.fr/antonymie/{q}",
          title: "Antonymie",
          selector: "#contentbox"
        },
        {
          id: 5,
          url: "https://www.cnrtl.fr/proxemie/{q}",
          title: "Proxemie",
          selector: "#contentbox"
        },
        {
          id: 6,
          url: "https://www.cnrtl.fr/concordance/{q}",
          title: "Concordance",
          selector: "#contentbox"
        }
      ]
    };
  },
  methods: {
     go() {
      self = this;
      self.content = [];
      self.endpoints.forEach(point => {
        axios
          .get(point.url.replace("{q}", self.mot))
          .then(async function(response) {
            var datasrc = parse(response.data);
            var data = datasrc.querySelector(point.selector);

            if (response.status != 200 || data == null) {
              self.addError(
                "Aucun resultats pour : " + point.title + " de " + self.mot
              );
            } else {
              var html = data;
              if (point.id == 0) {
                var opts = datasrc.querySelector("#vtoolbar");

                if (opts.childNodes[0].childNodes.length > 1) {
                  console.log(opts.childNodes[0].childNodes);
                  for (
                    var i = 1;
                    i < opts.childNodes[0].childNodes.length;
                    i++
                  ) {
                     await axios
                        .get(
                          self.endpoints[0].url.replace("{q}", self.mot) +
                            "/" +
                            i
                        )
                        .then(result => {
                         var rdata = parse(result.data);
                          html += "<hr>" + rdata.querySelector(point.selector);
                        })
                        .catch(function(error) {
                          console.log(error);
                          self.error = true;
                        });
                      //
                    
                  } 
                  
                }
              }

              self.content.push({
                id: point.id,
                html: html,
                title: point.title
              });
            }
          })
          .catch(function(error) {
            console.log(error);
            self.error = true;
          })
          .then(function() {
            // always executed
            if (point.id == 0) self.get();
          });
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
    },
    get(i = 0) {
      self = this;
      self.c = self.content.find(item => item.id === i);
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
  z-index: 99;
}
.title {
  font-family: "Quicksand", "Source Sans Pro", -apple-system, BlinkMacSystemFont,
    "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
}

.r-2 {
  border-radius: 2px;
}
.tlf_cdefinition {
  background-color: #98ec38a9;
}
.tlf_csyntagme {
  background-color: #c0fffaa9;
}
.tabs ul {
  flex-shrink: 1;
  flex-wrap: wrap;
  border-bottom-color: transparent;
}
.htmlbox {
  overflow-x: auto;
}
td img {
  height: 10px;
}
</style>
