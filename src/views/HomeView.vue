<script setup>
import axios from 'axios';
import { ref } from 'vue';

const avatar = ref(null)
const uploadedImage = ref(null)
const imageId = ref(null)
const loading = ref(false)
const errorMessage = ref("")
const activeEffects = ref({
  grayscale: false,
  blur: 0,
  rotate: 0,
  flip: false,
  flop: false,
  modulate: {
    saturation: 0,
    brightness: 0
  }
})

const addBlur = () => {
  activeEffects.value.blur = 20
  applyEffect(activeEffects.value)
}
const toggleGreyscale = () => {
  activeEffects.value.grayscale = !activeEffects.value.grayscale
  applyEffect(activeEffects.value)
}
const addRotate = (deg) => {
  activeEffects.value.rotate = deg
  applyEffect(activeEffects.value)
}
const toggleFlip = () => {
  activeEffects.value.flip = !activeEffects.value.flip
  applyEffect(activeEffects.value)
}
const toggleFlop = () => {
  activeEffects.value.flop = !activeEffects.value.flop
  applyEffect(activeEffects.value)
}
const modulate = () => {
  activeEffects.value.modulate.saturation = 2
  activeEffects.value.modulate.brightness = 2
  applyEffect(activeEffects.value)
}

const reset = () => {
   activeEffects.value.blur = 0
   activeEffects.value.grayscale = false
   activeEffects.value.rotate = 0
   activeEffects.value.flip = false
   activeEffects.value.flop = false
   activeEffects.value.modulate.saturation = 0
   activeEffects.value.modulate.brightness = 0
   applyEffect(activeEffects.value)
} 
 
const onFileSelected = (e) => {
  console.log(e.target.files[0]?.name)
  avatar.value = e.target.files[0]
}

const handleUpload = async () => {
  try {
    const formData = new FormData()
    formData.append('avatar', avatar.value)
    loading.value = true
    errorMessage.value = ""
    const response = await axios.post(
      "http://localhost:5001/api/uploads/",
      formData
    );
    console.log(response.data);
    const imageUrl = response.data.data.url
    const id = response.data.data._id
    uploadedImage.value = imageUrl
    imageId.value = id
  } catch (error) {
    console.error(error);
    console.log(error.response.data.error)
    errorMessage.value = error.response.data.error
  } finally {
    console.log("Request completed");
    loading.value = false
  }
};

const applyEffect = async (effectObj) => {
    try {
    loading.value = true
    errorMessage.value = ""
    const response = await axios.post(
      "http://localhost:5001/api/uploads/transform",
      {
        imageId: imageId.value,
        effectObj
      }
    );
    console.log(response.data);
    const imageUrl = response.data.data
    uploadedImage.value = imageUrl
  } catch (error) {
    console.error(error);
    console.log(error.response.data.error)
    errorMessage.value = error.response.data.error
  } finally {
    console.log("Request completed");
    loading.value = false
  }
}

const downloadImage = async () => {
  try {
    // Fetch the image as a binary blob
    const response = await axios.get(uploadedImage.value, {
      responseType: 'blob'
    });
    
    // Create a local blob URL
    const blob = new Blob([response.data]);
    const link = document.createElement('a');
    link.href = window.URL.createObjectURL(blob);
    link.download = 'edited-image.jpg'; // File name for the download
    
    // Append to body, click it programmatically, then remove it
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
    
    // Clean up the object URL
    window.URL.revokeObjectURL(link.href);
  } catch (error) {
    console.error('Failed to download image:', error);
    errorMessage.value = "Failed to download image";
  }
};
</script>

<template>
  <main class="editor-container">
    <header class="header">
      <h1>Node & Sharp Image Editor</h1>
      <p class="subtitle">Upload, transform, and style your images seamlessly</p>
    </header>

    <!-- Error Banner -->
    <div v-if="errorMessage" class="error-banner">
      <p>{{ errorMessage }}</p>
    </div>

    <!-- Upload Section -->
    <form @submit.prevent="handleUpload" class="upload-form">
      <input type="file" @change="onFileSelected" class="file-input">
      <button type="submit" class="upload-btn">Upload Image</button>
    </form>

    <!-- Main Workspace -->
    <div class="workspace" v-if="uploadedImage || loading">
      <!-- Image Preview Area -->
      <div class="preview-card">
        <div v-if="loading" class="loader-state">
          <div class="spinner"></div>
          <p>Processing image...</p>
        </div>
        <div v-else-if="uploadedImage" class="image-wrapper">
          <img :src="uploadedImage" alt="Processed Image">
        </div>
      </div>

      <!-- Controls Panel -->
      <div class="controls-panel">
        <h3>Effects & Adjustments</h3>
        <div class="button-grid">
          <button 
            @click="toggleGreyscale"
            :class="{ active: activeEffects.grayscale }"
          >
            Grayscale
          </button>
          
          <button 
            @click="addBlur"
            :class="{ active: activeEffects.blur > 0 }"
          >
            Blur
          </button>

          <button 
            @click="addRotate(90)"
            :class="{ active: activeEffects.rotate === 90 }"
          >
            Rotate 90°
          </button>

          <button 
            @click="addRotate(180)"
            :class="{ active: activeEffects.rotate === 180 }"
          >
            Rotate 180°
          </button>

          <button 
            @click="toggleFlip"
            :class="{ active: activeEffects.flip }"
          >
            Flip
          </button>

          <button 
            @click="toggleFlop"
            :class="{ active: activeEffects.flop }"
          >
            Flop
          </button>

          <button 
            @click="modulate"
            :class="{ active: activeEffects.modulate.saturation === 2 }"
          >
            Modulate
          </button>

          <button @click="reset" class="reset-btn">
            Reset All
          </button>
         
        </div>
      </div>

       <div v-if="uploadedImage" class="download-container">
          <button @click="downloadImage" class="download-btn">
            Download Image
          </button>
        </div>
    </div>
  </main>
</template>

<style scoped>
/* Main Container Layout */
.editor-container {
  max-width: 900px;
  margin: 0 auto;
  padding: 40px 20px;
  font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  color: #f3f4f6;
}

.header {
  text-align: center;
  margin-bottom: 24px;
}

.header h1 {
  font-size: 1.8rem;
  font-weight: 700;
  color: #ffffff;
  margin-bottom: 6px;
}

.subtitle {
  color: #9ca3af;
  font-size: 0.95rem;
}

/* Error Banner */
.error-banner {
  background-color: rgba(239, 68, 68, 0.15);
  border: 1px solid #ef4444;
  color: #fca5a5;
  padding: 12px;
  border-radius: 8px;
  margin-bottom: 20px;
  text-align: center;
}

/* Upload Form */
.upload-form {
  display: flex;
  justify-content: center;
  gap: 12px;
  margin-bottom: 30px;
  background: #18181b;
  padding: 16px;
  border-radius: 12px;
  border: 1px solid #27272a;
}

.file-input {
  color: #a1a1aa;
  padding: 6px;
}

.upload-btn {
  background-color: #27272a;
  color: #f43f5e;
  border: 1px solid #3f3f46;
  padding: 8px 16px;
  border-radius: 6px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
}

.upload-btn:hover {
  background-color: #3f3f46;
  color: #fb7185;
}

/* Workspace Grid */
.workspace {
  display: grid;
  grid-template-columns: 1fr 300px;
  gap: 20px;
  align-items: start;
}

@media (max-width: 768px) {
  .workspace {
    grid-template-columns: 1fr;
  }
}

/* Preview Card */
.preview-card {
  background: #18181b;
  border: 1px solid #27272a;
  border-radius: 12px;
  padding: 20px;
  min-height: 420px;
  display: flex;
  justify-content: center;
  align-items: center;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.4);
}

.image-wrapper img {
  max-width: 100%;
  max-height: 400px;
  border-radius: 8px;
  object-fit: contain;
}

.loader-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
  color: #a1a1aa;
}

/* Controls Panel */
.controls-panel {
  background: #18181b;
  border: 1px solid #27272a;
  border-radius: 12px;
  padding: 20px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.4);
}

.controls-panel h3 {
  font-size: 0.85rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  color: #a1a1aa;
  margin-bottom: 14px;
}

.button-grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 10px;
}

/* Buttons */
button {
  background-color: #27272a;
  color: #e4e4e7;
  border: 1px solid #3f3f46;
  padding: 10px 14px;
  border-radius: 8px;
  font-weight: 500;
  font-size: 0.9rem;
  cursor: pointer;
  transition: all 0.2s ease;
  width: 100%;
  text-align: center;
}

button:hover {
  background-color: #3f3f46;
  color: #ffffff;
  border-color: #52525b;
}

/* Active State */
button.active {
  background-color: #4f46e5;
  color: #ffffff;
  border-color: #6366f1;
  box-shadow: 0 0 10px rgba(79, 70, 229, 0.4);
}

/* Reset Button distinction */
.reset-btn {
  background-color: rgba(239, 68, 68, 0.1);
  border-color: rgba(239, 68, 68, 0.3);
  color: #fca5a5;
  margin-top: 10px;
}

.reset-btn:hover {
  background-color: rgba(239, 68, 68, 0.2);
  color: #ffffff;
  border-color: #ef4444;
}
.download-container {
  margin-top: 16px;
  text-align: center;
}

.download-btn {
  display: inline-block;
  background-color: #10b981; /* Fresh emerald green for success/download */
  color: #ffffff;
  padding: 10px 20px;
  border-radius: 8px;
  font-weight: 600;
  text-decoration: none;
  transition: background-color 0.2s, transform 0.1s;
  box-shadow: 0 4px 12px rgba(16, 185, 129, 0.3);
}

.download-btn:hover {
  background-color: #059669;
  transform: translateY(-1px);
}
</style>