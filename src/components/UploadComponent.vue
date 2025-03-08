<template>
  <div class="container mt-5">
    <h2 class="text-center">Envie uma imagem</h2>

    <div class="mb-3">
      <input type="file" class="form-control" @change="handleFileUpload">
    </div>

    <button class="btn btn-primary" @click="uploadFile">Enviar</button>

    <div v-if="loading" class="modal fade show d-block" style="background: rgba(0,0,0,0.5);">
      <div class="modal-dialog modal-dialog-centered">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title">Processando Imagem...</h5>
          </div>
          <div class="modal-body text-center">
            <div class="spinner-border text-primary" role="status">
              <span class="visually-hidden">Carregando...</span>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div v-if="processedImage" class="mt-4 text-center">
      <h3>Imagem Processada:</h3>
      <img :src="processedImage" class="img-fluid border rounded shadow" style="max-width: 500px; height: auto;"
        alt="Imagem detectada">
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      file: null,
      processedImage: null,
      loading: false
    };
  },
  methods: {
    handleFileUpload(event) {
      this.file = event.target.files[0];
      this.processedImage = null;
    },
    async uploadFile() {
      if (!this.file) {
        alert("Selecione um arquivo primeiro!");
        return;
      }

      let formData = new FormData();
      formData.append("file", this.file);

      this.loading = true;

      try {
        const response = await axios.post("http://localhost:8000/predict/", formData, {
          headers: { "Content-Type": "multipart/form-data" },
          responseType: "blob"
        });

        this.processedImage = URL.createObjectURL(response.data);
      } catch (error) {
        console.error("Erro ao enviar arquivo:", error);
        alert("Erro ao processar a imagem.");
      } finally {
        this.loading = false;
      }
    }
  }
};
</script>

<style>
.modal {
  display: block;
  z-index: 1050;
}

.modal-dialog {
  max-width: 300px;
}
</style>