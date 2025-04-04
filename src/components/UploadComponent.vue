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
        <option value="yolov11s" :disabled="isVideo">YOLOv11s</option>
        <option value="yolov11m" :disabled="isVideo">YOLOv11m</option>
        <option value="yolov11l" :disabled="isVideo">YOLOv11l</option>
        <option value="yolov11x" :disabled="isVideo">YOLOv11x</option>
      </select>
      <div class="form-text mt-1">
        <strong>Nota:</strong> modelos maiores (como <code>yolov11x</code>) oferecem melhores resultados, mas também
        exigem <strong>mais tempo de processamento e mais recursos computacionais</strong>.<br>
        Para <strong>vídeos</strong>, devido ao alto consumo de CPU/GPU, apenas o modelo <code>yolov11n</code> está
        disponível.<br><br>
        <strong>Aviso:</strong> esta aplicação pode cometer erros e <strong>não substitui o diagnóstico de um
          profissional capacitado</strong>.
      </div>
    </div>

    <div class="mb-3">
      <label for="thresholdRange" class="form-label">Confiança mínima: {{ threshold }}</label>
      <input type="range" class="form-range" id="thresholdRange" min="0" max="1" step="0.01" v-model="threshold">
    </div>

    <button class="btn btn-primary" @click="uploadFile">Enviar</button>

    <div v-if="loading" class="modal fade show d-block" style="background: rgba(0,0,0,0.5);">
      <div class="modal-dialog modal-dialog-centered">
        <div class="modal-content p-4">
          <h5 class="modal-title text-center">
            {{ isVideo ? "Processando vídeo..." : "Processando imagem..." }}
          </h5>
          <div class="progress mt-3">
            <div class="progress-bar progress-bar-striped progress-bar-animated" role="progressbar"
              :style="{ width: progress + '%' }">
              {{ progress }}%
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

    <div v-if="csvUrl || pdfUrl" class="mt-3 text-center mb-5">
      <h4>Relatórios:</h4>
      <a v-if="csvUrl" :href="csvUrl" target="_blank" class="btn btn-outline-primary me-2">📄 Baixar CSV</a>
      <a v-if="pdfUrl" :href="pdfUrl" target="_blank" class="btn btn-outline-danger">📕 Baixar PDF</a>
    </div>

    <footer class="text-center text-muted mt-5 mb-4 small">
      <div>Desenvolvido por <strong>Halan Germano Bacca</strong> - PPGINFOS - UFSC - 2025</div>
      <div class="mt-2">
        <a href="https://github.com/halangbacca" target="_blank" class="me-3 text-decoration-none text-dark">
          <i class="bi bi-github"></i> GitHub
        </a>
        <a href="https://www.linkedin.com/in/halanbacca" target="_blank" class="text-decoration-none text-dark">
          <i class="bi bi-linkedin"></i> LinkedIn
        </a>
      </div>
    </footer>
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
      isVideo: false,
      progress: 0,
      ws: null,
    };
  },
  methods: {
    handleFileUpload(event) {
      this.file = event.target.files[0];
      this.processedFile = null;
      this.detectionsCount = {};
      this.csvUrl = null;
      this.pdfUrl = null;

      if (this.file && this.file.type.startsWith("video")) {
        this.isVideo = true;
        this.selectedModel = "yolov11n";
      } else {
        this.isVideo = false;
      }
    },
    iniciarWebSocket() {
      this.ws = new WebSocket("ws://localhost:8000/ws/progresso");
      this.ws.onmessage = (event) => {
        this.progress = parseInt(event.data);
      };
    },
    fecharWebSocket() {
      if (this.ws) {
        this.ws.close();
        this.ws = null;
      }
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
      this.progress = 0;
      this.loading = true;
      this.iniciarWebSocket();

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
        this.fecharWebSocket();
        this.loading = false;
        this.progress = 0;
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
  max-width: 400px;
}
</style>
