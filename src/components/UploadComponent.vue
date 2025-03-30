<template>
  <div class="container mt-5">
    <h2 class="text-center">Envie uma imagem ou vídeo</h2>

    <div class="mb-3">
      <input type="file" class="form-control" accept="image/*,video/*" @change="handleFileUpload">
    </div>

    <div class="mb-3">
      <label for="modelSelect" class="form-label">Modelo YOLOv11:</label>
      <select id="modelSelect" v-model="selectedModel" class="form-select">
        <option disabled value="">Selecione um modelo</option>
        <option value="yolov11n">YOLOv11n</option>
        <option value="yolov11s">YOLOv11s</option>
        <option value="yolov11m">YOLOv11m</option>
        <option value="yolov11l">YOLOv11l</option>
        <option value="yolov11x">YOLOv11x</option>
      </select>
      <div class="form-text mt-1">
        <strong>Nota:</strong> quanto <strong>maior o modelo</strong> (ex: <code>yolov11x</code>), maior a sensibilidade
        e especificidade, mas também <strong>maior o tempo de processamento</strong>.
      </div>
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
            <h5 class="modal-title">Processando Arquivo...</h5>
          </div>
          <div class="modal-body text-center">
            <div class="spinner-border text-primary" role="status">
              <span class="visually-hidden">Carregando...</span>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div v-if="processedFile" class="mt-4 text-center">
      <h3>Arquivo Processado:</h3>
      <img v-if="file && file.type.startsWith('image')" :src="processedFile" class="img-fluid border rounded shadow"
        style="max-width: 500px; height: auto;" alt="Imagem processada">

      <video v-else controls class="img-fluid border rounded shadow" style="max-width: 500px; height: auto;">
        <source :src="processedFile" type="video/mp4" />
      </video>
    </div>

    <div v-if="Object.keys(detectionsCount).length > 0" class="mt-4 text-center">
      <h3>Ovos Detectados:</h3>
      <ul class="list-group">
        <li v-for="(count, className) in detectionsCount" :key="className" class="list-group-item">
          <strong>{{ className }}</strong>: {{ count }} ovos detectados
        </li>
      </ul>
    </div>

    <div v-if="csvUrl || pdfUrl" class="mt-3 text-center">
      <h4>Relatórios:</h4>
      <a v-if="csvUrl" :href="csvUrl" target="_blank" class="btn btn-outline-primary me-2">📄 Baixar CSV</a>
      <a v-if="pdfUrl" :href="pdfUrl" target="_blank" class="btn btn-outline-danger">📕 Baixar PDF</a>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      file: null,
      processedFile: null,
      threshold: 0.5,
      loading: false,
      detectionsCount: {},
      selectedModel: "yolov11n",
      csvUrl: null,
      pdfUrl: null,
    };
  },
  methods: {
    handleFileUpload(event) {
      this.file = event.target.files[0];
      this.processedFile = null;
      this.detectionsCount = {};
      this.csvUrl = null;
      this.pdfUrl = null;
    },
    async uploadFile() {
      if (!this.file) {
        alert("Selecione um arquivo primeiro!");
        return;
      }

      if (!this.selectedModel) {
        alert("Selecione um modelo YOLO!");
        return;
      }

      let formData = new FormData();
      formData.append("file", this.file);

      const url = `http://localhost:8000/predict/?threshold=${this.threshold}&model=${this.selectedModel}`;
      this.loading = true;

      try {
        const response = await axios.post(url, formData, {
          headers: { "Content-Type": "multipart/form-data" }
        });

        this.detectionsCount = response.data.detections_count;
        this.processedFile = response.data.download_url;
        this.csvUrl = response.data.csv_url;
        this.pdfUrl = response.data.pdf_url;

      } catch (error) {
        console.error("Erro ao enviar arquivo:", error);
        alert("Erro ao processar o arquivo.");
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
