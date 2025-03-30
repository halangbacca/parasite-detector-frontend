# 🧪 YOLOv11 - Detecção de Ovos de Parasitas (Frontend Vue)

Este é o frontend do projeto de detecção de ovos de parasitas utilizando modelos **YOLOv11**. A aplicação permite o envio de **imagens ou vídeos**, escolha do modelo de detecção, ajuste do nível de confiança e exibe os resultados com visualização, contagem e geração de relatórios em **PDF** e **CSV**.

## 🚀 Funcionalidades

- 📤 Envio de **imagem ou vídeo**
- 🤖 Escolha do modelo YOLOv11 (`n`, `s`, `m`, `l`, `x`)
- 🎯 Ajuste da **confiança mínima** (threshold)
- 🔍 Visualização do arquivo processado (imagem ou vídeo)
- 🧮 Contagem de ovos detectados por classe
- 📄 Download de relatórios em **CSV** e **PDF**

## 📦 Requisitos

- [Node.js](https://nodejs.org/) v16+
- Backend rodando em `http://localhost:8000` com suporte para a rota `/predict/`

> **Observação:** O backend deve estar preparado para retornar:
> - `download_url`: URL do arquivo processado
> - `csv_url`: URL para o relatório CSV
> - `pdf_url`: URL para o relatório PDF
> - `detections_count`: objeto com contagens por classe

## 🛠️ Instalação

```bash
# Clone o repositório
git clone https://github.com/halangbacca/parasite-detector-frontend.git
cd parasite-detector-frontend

# Instale as dependências
npm install
```

## ▶️ Execução

```bash
npm run dev
```

Acesse em: [http://localhost:8080](http://localhost:8080)

## 📁 Estrutura Principal

- `App.vue`: Componente principal com upload, seleção de modelo e exibição dos resultados
- `axios`: Utilizado para envio do arquivo ao backend
- `v-model`: Para seleção do modelo e confiança mínima
- `Bootstrap`: Utilizado para estilo básico

## 📷 Exemplo de Uso

1. Selecione uma imagem ou vídeo.
2. Escolha um modelo YOLOv11:
   - **yolov11n**: mais leve e rápido
   - **yolov11x**: mais pesado e preciso
3. Ajuste a confiança mínima (threshold).
4. Clique em **Enviar**.
5. Visualize o resultado com a contagem de ovos.
6. Faça download dos relatórios gerados.

## 📌 Observações

- Modelos maiores como `yolov11x` possuem melhor desempenho de detecção, porém são mais lentos.
- O vídeo ou imagem processado será exibido diretamente na interface.
- Ideal para análise automatizada em laboratórios ou projetos de pesquisa com parasitas.

## 🧑‍💻 Contribuindo

Pull Requests são bem-vindos! Sinta-se à vontade para sugerir melhorias ou reportar bugs.

## 📄 Licença

Este projeto está sob a licença [MIT](LICENSE).
