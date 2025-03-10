<template>
  <div class="container mt-5">
    <h2 class="text-center">Envie uma imagem</h2>

    <div class="mb-3">
      <input type="file" class="form-control" @change="handleFileUpload">
    </div>

    <div class="mb-3">
      <label for="thresholdRange" class="form-label">Confiança mínima: {{ threshold }}</label>
      <input type="range" class="form-range" id="thresholdRange" min="0" max="1" step="0.01" v-model="threshold">
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

    <div v-if="Object.keys(detectionsCount).length > 0" class="mt-4 text-center">
      <h3>Ovos Detectados:</h3>
      <ul class="list-group">
        <li v-for="(count, className) in detectionsCount" :key="className" class="list-group-item">
          <strong>{{ className }}</strong>: {{ count }} ovos detectados
        </li>
      </ul>
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
      threshold: 0.5,
      loading: false,
      detectionsCount: {}  // Inicializa a contagem de detecções
    };
  },
  methods: {
    handleFileUpload(event) {
      this.file = event.target.files[0];
      this.processedImage = null;
      this.detectionsCount = {};  // Resetar a contagem ao enviar uma nova imagem
    },
    async uploadFile() {
      if (!this.file) {
        alert("Selecione um arquivo primeiro!");
        return;
      }

      let formData = new FormData();
      formData.append("file", this.file);

      const url = `http://localhost:8000/predict/?threshold=${this.threshold}`;

      this.loading = true;

      try {
        const response = await axios.post(url, formData, {
          headers: { "Content-Type": "multipart/form-data" }
        });

        // Processar resposta JSON
        this.detectionsCount = response.data.detections_count;

        // Converter imagem hexadecimal de volta para exibição
        const imageHex = response.data.image;
        const binaryData = new Uint8Array(imageHex.match(/.{1,2}/g).map(byte => parseInt(byte, 16)));
        this.processedImage = URL.createObjectURL(new Blob([binaryData], { type: "image/jpeg" }));
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
