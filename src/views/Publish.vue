<template>
  <v-container>
    <v-subheader>
      <h2 class="mx-auto">Publish a Book</h2>
    </v-subheader>
    <form>
      <v-text-field
        v-model="name"
        :error-messages="nameErrors"
        label="Title"
        required
        @input="$v.name.$touch()"
        @blur="$v.name.$touch()"
      ></v-text-field>
      <v-text-field
        v-model="author"
        :error-messages="authorErrors"
        label="Author"
        required
        @input="$v.author.$touch()"
        @blur="$v.author.$touch()"
      ></v-text-field>
      <v-text-field
        v-model="description"
        :error-messages="descriptionErrors"
        label="Description"
        required
        @input="$v.description.$touch()"
        @blur="$v.description.$touch()"
      ></v-text-field>
      <v-select
        v-model="select"
        :items="genres"
        :error-messages="selectErrors"
        label="Genre"
        required
        @change="$v.select.$touch()"
        @blur="$v.select.$touch()"
      ></v-select>
      <v-file-input
        truncate-length="15"
        label="Cover File"
        required
        v-on:change="handleCoverFile"
      ></v-file-input>
      <v-file-input
        truncate-length="15"
        label="Book File"
        required
        v-on:change="handleBookFile"
      ></v-file-input>
      <v-sheet v-if="similarsSelected.length > 0" class="mb-5" elevation="2">
        <v-subheader>
          <h2>Similars</h2>
        </v-subheader>
        <v-slide-group v-model="model2" class="pa-4" show-arrows>
          <v-slide-item v-for="i in similarsSelected" :key="i.cover_i">
            <v-card class="ma-4" height="200" width="100">
              <v-img
                :src="
                  `http://covers.openlibrary.org/b/id/` + i.cover_i + `-L.jpg`
                "
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
      <v-dialog transition="dialog-top-transition" max-width="800">
        <template v-slot:activator="{ on, attrs }">
          <v-btn color="primary" v-bind="attrs" v-on="on"
            >Add Similar Books</v-btn
          >
        </template>
        <template v-slot:default="dialog">
          <v-card>
            <v-toolbar color="primary" dark>Search Similar Books </v-toolbar>
            <v-card-text>
              <div class="mt-5">
                <v-text-field
                  label="Type the book name..."
                  solo
                  v-model="query"
                >
                  <template slot="append">
                    <v-btn icon @click="searchBooks"
                      ><v-icon>mdi-magnify</v-icon></v-btn
                    >
                  </template>
                </v-text-field>
              </div>
              <div v-if="isLoading" class="loader mx-auto mb-5"></div>
              <v-sheet
                v-if="items.length > 0"
                class="mx-auto"
                elevation="2"
                max-width="600"
              >
                <v-subheader>
                  <h2>Results</h2>
                </v-subheader>
                <v-slide-group v-model="model" class="pa-4" show-arrows>
                  <v-slide-item
                    v-for="i in items"
                    :key="i.cover_i"
                    v-slot="{ toggle }"
                  >
                    <v-card
                      class="ma-4"
                      height="300"
                      width="200"
                      @click="handleCardClick(i, toggle)"
                    >
                      <v-img
                        :src="
                          `http://covers.openlibrary.org/b/id/` +
                            i.cover_i +
                            `-L.jpg`
                        "
                        aspect-ratio="1"
                        class="white--text align-end"
                        height="300px"
                      >
                      </v-img>
                      <v-row
                        class="fill-height"
                        align="center"
                        justify="center"
                      >
                      </v-row>
                    </v-card>
                  </v-slide-item>
                </v-slide-group>

                <v-expand-transition>
                  <v-sheet v-if="model != null" height="200" tile>
                    <v-row class="fill-height" align="center" justify="center">
                      <h6 class="text-h6">Selected {{ similarBookTitle }}</h6>
                    </v-row>
                  </v-sheet>
                </v-expand-transition>
              </v-sheet>
            </v-card-text>
            <v-card-actions class="justify-end">
              <v-btn
                v-if="similarBookTitle != ''"
                text
                color="green"
                @click="selectSimilarBook(dialog)"
              >
                Confirm
              </v-btn>
              <v-btn text @click="dialog.value = false">Close</v-btn>
            </v-card-actions>
          </v-card>
        </template>
      </v-dialog>
      <v-checkbox
        v-model="checkbox"
        :error-messages="checkboxErrors"
        label="I agree with the terms and conditions"
        required
        @change="$v.checkbox.$touch()"
        @blur="$v.checkbox.$touch()"
      ></v-checkbox>

      <v-btn class="mr-4" @click="submit">
        submit
      </v-btn>
      <v-btn @click="clear">
        clear
      </v-btn>
    </form>

    <v-snackbar
      v-model="snackbar"
      color="success"
      :timeout="timeout"
    >
      {{ text }}

      <template v-slot:action="{ attrs }">
        <v-btn
          color="blue"
          text
          v-bind="attrs"
          @click="snackbar = false"
        >
          Close
        </v-btn>
      </template>
    </v-snackbar>
  </v-container>
</template>

<script>
import { validationMixin } from "vuelidate";
import { required, maxLength, email } from "vuelidate/lib/validators";

export default {
  name: "Publish",
  mixins: [validationMixin],

  validations: {
    name: { required, maxLength: maxLength(10) },
    email: { required, email },
    description: { required },
    author: { required },
    select: { required },
    checkbox: {
      checked(val) {
        return val;
      },
    },
  },

  data: () => ({
    name: "",
    author: "",
    email: "",
    description: "",
    bookFile: "",
    coverFile: "",
    select: null,
    genres: ["Classical", "Fiction", "Fantasy", "Adventure"],
    checkbox: false,
    isLoading: false,
    items: [],
    model: null,
    model2: null,
    search: null,
    tab: null,
    query: "",
    similarBookTitle: "",
    currentSelectedSimilar: null,
    similarsSelected: [],
    snackbar: false,
    text: 'Success!',
      timeout: 2000,
  }),

  computed: {
    checkboxErrors() {
      const errors = [];
      if (!this.$v.checkbox.$dirty) return errors;
      !this.$v.checkbox.checked && errors.push("You must agree to continue!");
      return errors;
    },
    selectErrors() {
      const errors = [];
      if (!this.$v.select.$dirty) return errors;
      !this.$v.select.required && errors.push("Item is required");
      return errors;
    },
    nameErrors() {
      const errors = [];
      if (!this.$v.name.$dirty) return errors;
      !this.$v.name.required && errors.push("Title is required.");
      return errors;
    },
    authorErrors() {
      const errors = [];
      if (!this.$v.author.$dirty) return errors;
      !this.$v.author.required && errors.push("Author is required.");
      return errors;
    },
    descriptionErrors() {
      const errors = [];
      if (!this.$v.description.$dirty) return errors;
      !this.$v.description.required && errors.push("Description is required.");
      return errors;
    },
    emailErrors() {
      const errors = [];
      if (!this.$v.email.$dirty) return errors;
      !this.$v.email.email && errors.push("Must be valid e-mail");
      !this.$v.email.required && errors.push("E-mail is required");
      return errors;
    },
  },

  methods: {
    submit() {
      this.$v.$touch();
      let book = {
        title: this.name,
        description: this.description,
        cover: "covers/" + this.coverFile,
        file: "books/" + this.bookFile,
        genre: this.select,
        author: this.author,
        relatedBooks: this.prepareRelatedBooks(),
      };
      this.axios.post("/books", book).then(() => {
        this.snackbar = true;
      });
    },
    clear() {
      this.$v.$reset();
      this.name = "";
      this.email = "";
      this.author = "";
      this.description = "";
      this.select = null;
      this.checkbox = false;
    },
    prepareRelatedBooks() {
      let related = [];
      for (let i = 0; i < this.similarsSelected.length; i++) {
        related.push({
          id: this.similarsSelected[i].cover_i,
          title: this.similarsSelected[i].title,
          author: this.similarsSelected[i].author_name[0],
          description: this.similarsSelected[i].title,
          genre: "",
          cover: `http://covers.openlibrary.org/b/id/` + this.similarsSelected[i].cover_i + `-L.jpg`,
        });
      }
      return related;
    },
    handleBookFile(event) {
      this.bookFile = event.name;
    },
    handleCoverFile(event) {
      this.coverFile = event.name;
    },
    handleCardClick(item, toggle) {
      toggle();
      this.similarBookTitle = item.title;
      this.currentSelectedSimilar = item;
    },
    selectSimilarBook(dialog) {
      this.similarsSelected.push(this.currentSelectedSimilar);
      console.log(this.similarsSelected);
      dialog.value = false;
    },
    searchBooks() {
      this.isLoading = true;

      // Lazily load input items
      this.axios
        .get("http://openlibrary.org/search.json?title=" + this.query)
        .then((res) => {
          console.log(res);
          this.items = res.data.docs.slice(0, 10);
        })
        .catch((err) => {
          console.log(err);
        })
        .finally(() => (this.isLoading = false));
    },
  },
};
</script>

<style lang="scss" scoped>
.loader {
  border: 16px solid #f3f3f3;
  border-top: 16px solid #3498db;
  border-radius: 50%;
  width: 36px;
  height: 36px;
  animation: spin 2s linear infinite;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}
</style>
