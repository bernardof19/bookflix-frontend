<template>
  <v-container>
    <v-sheet
      v-for="(m, idx) in models"
      :key="m.id"
      class="mx-auto mb-10"
      elevation="8"
    >
      <v-subheader>
        <h2>{{ m.genre }}</h2>
      </v-subheader>
      <v-slide-group v-model="sliderModel[idx]" class="pa-4" show-arrows>
        <v-slide-item v-for="book in m.books" :key="book.id">
          <v-card
            class="ma-4"
            height="200"
            width="150"
            @click="handleBookClick(book, idx)"
          >
            <v-img
              :src="book.cover"
              aspect-ratio="1"
              class="white--text align-end"
              height="200px"
            >
            </v-img>
          </v-card>
        </v-slide-item>
      </v-slide-group>
    </v-sheet>
    <v-dialog v-model="dialog" scrollable width="auto " height="auto">
      <v-card :loading="loading" max-width="600">
        <template slot="progress">
          <v-progress-linear
            color="deep-purple"
            height="10"
            indeterminate
          ></v-progress-linear>
        </template>

        <v-img
          class="mx-auto mt-5"
          height="200"
          width="150"
          :src="book.cover"
        ></v-img>

        <v-card-title>{{ book.title }}</v-card-title>

        <v-card-text>
          <div class="mb-4 text-subtitle-1">Written By: {{ book.author }}</div>

          <div>
            {{ book.description }}
          </div>
        </v-card-text>

        <v-divider class="mx-4"></v-divider>

        <v-card-title>Similar Books</v-card-title>

        <v-sheet
          v-if="book.relatedBooks.length > 0"
          class="mb-5"
          elevation="2"
          width="600px"
        >
          <v-slide-group v-model="model2" class="pa-4" show-arrows>
            <v-slide-item v-for="i in book.relatedBooks" :key="i.cover_i">
              <v-card class="ma-4" height="200" width="100">
                <v-img
                  :src="i.cover"
                  aspect-ratio="1"
                  class="white--text align-end"
                  height="200px"
                >
                </v-img>
                <v-row class="fill-height" align="center" justify="center">
                </v-row>
              </v-card>
            </v-slide-item>
          </v-slide-group>
        </v-sheet>

        <v-card-actions>
          <v-btn
            class="mx-auto"
            color="deep-purple lighten-2"
            text
            @click="readBook"
          >
            Read Now
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script>
export default {
  name: "Home",

  data: () => ({
    model: null,
    loading: false,
    book: {
      id: 0,
      title: "",
      description: "",
      cover: "",
      relatedBooks: [],
    },
    models: [],
    sliderModel: [],
    dialog: false,
    model2: null,
  }),

  mounted() {
    this.loadModel();
  },

  methods: {
    handleBookClick: function(book, idx) {
      this.book = book;
      this.sliderModel[idx] = book;
      this.dialog = true;
    },
    readBook: function() {
      this.$router.push("/read/" + this.book.id);
    },
    loadModel: function() {
      this.axios.get("/books?genre=Fiction").then((resp) => {
        this.models.push({
          id: 1,
          genre: "Fiction",
          books: resp.data,
        });
        this.axios.get("/books?genre=Fantasy").then((resp) => {
          this.models.push({
            id: 2,
            genre: "Fantasy",
            books: resp.data,
          });
          this.axios.get("/books?genre=Classical").then((resp) => {
            this.models.push({
              id: 4,
              genre: "Classical",
              books: resp.data,
            });
            this.axios.get("/books?genre=Adventure").then((resp) => {
            this.models.push({
              id: 4,
              genre: "Adventure",
              books: resp.data,
            });
          });
          });
        });
        for (let i = 0; i < this.models.length; i++) {
          this.sliderModel.push(i);
        }
      });
    },
  },
};
</script>
