<template>
  <div id="read">
    <div>
      <iframe
        height="900"
        width="100%"
        :src="`lib/pdfjs/web/viewer.html?file=` + this.filepath"
      ></iframe>
    </div>
  </div>
</template>

<script>
export default {
  name: "Read",
  data() {
    return {
      filepath: "",
    };
  },
  mounted() {
    this.loadBook();
  },
  methods: {
    loadBook: function() {
      this.axios.get("/books/" + this.$route.params.id).then((resp) => {
        this.filepath = window.location.origin + "/" + resp.data.file;
      });
    },
  },
};
</script>
