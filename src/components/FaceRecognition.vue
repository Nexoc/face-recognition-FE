<template>
  <div class="face-recognition-container">
    <!-- Background decoration -->
    <div class="background-decoration">
      <div class="decoration-circle decoration-circle-1"></div>
      <div class="decoration-circle decoration-circle-2"></div>
      <div class="particles" ref="particles"></div>
    </div>

    <div class="main-container">
      <div class="recognition-card">

        <!-- Header -->
        <div class="card-header">
          <div class="header-icon">
            <svg viewBox="0 0 24 24" fill="currentColor" class="icon">
              <path d="M12 12c2.21 0 4-1.79 4-4s-1.79-4-4-4-4 1.79-4 4 1.79 4 4 4zm0 2c-2.67 0-8 1.34-8 4v2h16v-2c0-2.66-5.33-4-8-4z"/>
            </svg>
          </div>
          <h1 class="header-title">Face Recognition</h1>
          <p class="header-subtitle">Secure biometric authentication</p>
        </div>

        <!-- Video Container -->
        <div class="video-container">
          <div class="video-frame">
            <video
              ref="video"
              autoplay
              playsinline
              class="video-element"
              :class="{ 'video-ready': cameraReady }"
            ></video>


            <!-- Camera placeholder -->
            <div v-if="!cameraReady" class="camera-placeholder">
              <svg viewBox="0 0 24 24" fill="currentColor" class="camera-icon">
                <path d="M12 15c1.66 0 2.99-1.34 2.99-3L15 6c0-1.66-1.34-3-3-3S9 4.34 9 6v6c0 1.66 1.34 3 3 3zm5.3-3c0 3-2.54 5.1-5.3 5.1S6.7 15 6.7 12H5c0 3.41 2.72 6.23 6 6.72V22h2v-3.28c3.28-.48 6-3.3 6-6.72h-1.7z"/>
              </svg>
              <p>Initializing camera...</p>
            </div>

            <!-- Scan overlay -->
            <div
              class="scan-overlay"
              :class="{ 'scan-active': isCapturing }"
            ></div>

            <!-- Corner decorators -->
            <div class="corner-decorator corner-tl"></div>
            <div class="corner-decorator corner-tr"></div>
            <div class="corner-decorator corner-bl"></div>
            <div class="corner-decorator corner-br"></div>
          </div>
        </div>

        <canvas ref="canvas" width="600" height="450" style="display: none;"></canvas>

        <!-- Status Message -->
        <div
          v-if="statusMessage"
          class="status-message"
          :class="{
            'status-error': statusMessage.includes('error') || statusMessage.includes('failed'),
            'status-success': statusMessage.includes('completed') || statusMessage.includes('completed'),
            'status-info': !statusMessage.includes('error') && !statusMessage.includes('completed')
          }"
        >
          <div class="status-icon">
            <svg v-if="statusMessage.includes('error')" viewBox="0 0 24 24" fill="currentColor">
              <path d="M1 21h22L12 2 1 21zm12-3h-2v-2h2v2zm0-4h-2v-4h2v4z"/>
            </svg>
            <svg v-else-if="statusMessage.includes('completed')" viewBox="0 0 24 24" fill="currentColor">
              <path d="M9 16.17L4.83 12l-1.42 1.41L9 19 21 7l-1.41-1.41z"/>
            </svg>
            <svg v-else viewBox="0 0 24 24" fill="currentColor">
              <path d="M19 3H5c-1.1 0-2 .9-2 2v14c0 1.1.9 2 2 2h14c1.1 0 2-.9 2-2V5c0-1.1-.9-2-2-2zm-5 14H7v-2h7v2zm3-4H7v-2h10v2zm0-4H7V7h10v2z"/>
            </svg>
          </div>
          <span class="status-text">{{ statusMessage }}</span>
        </div>

        <!-- Progress Bar -->
        <div v-if="isCapturing" class="progress-container">
          <div class="progress-info">
            <span class="progress-label">Capturing photos...</span>
            <span class="progress-percent">{{ Math.round(progress) }}%</span>
          </div>
          <div class="progress-bar">
            <div
              class="progress-fill"
              :style="{ width: progress + '%' }"
            >
              <div class="progress-shimmer"></div>
            </div>
          </div>
        </div>

        <!-- Capture Button -->
        <button
          class="capture-button"
          :class="{
            'button-disabled': isCapturing || !cameraReady,
            'button-loading': isCapturing
          }"
          @click="startCapturing"
          :disabled="isCapturing || !cameraReady"
        >
          <span v-if="isCapturing" class="button-content">
            <div class="button-spinner"></div>
            <span>Capturing...</span>
          </span>

          <span v-else class="button-content">
            <svg viewBox="0 0 24 24" fill="currentColor" class="button-icon">
              <path d="M12 15c1.66 0 2.99-1.34 2.99-3L15 6c0-1.66-1.34-3-3-3S9 4.34 9 6v6c0 1.66 1.34 3 3 3zm5.3-3c0 3-2.54 5.1-5.3 5.1S6.7 15 6.7 12H5c0 3.41 2.72 6.23 6 6.72V22h2v-3.28c3.28-.48 6-3.3 6-6.72h-1.7z"/>
            </svg>
            <span>Start Capture (5 Photos)</span>
          </span>
        </button>

        <!-- Result -->
        <div v-if="result" class="result-container">
          <div class="result-header">
            <svg viewBox="0 0 24 24" fill="currentColor" class="result-icon">
              <path d="M9 16.17L4.83 12l-1.42 1.41L9 19 21 7l-1.41-1.41z"/>
            </svg>
            <span class="result-title">Authentication Successful</span>
          </div>
          <pre class="result-content">{{ result }}</pre>
        </div>
      </div>

      <!-- Footer -->
      <div class="footer">
        <p>🔒 Your biometric data is processed securely</p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

// Reactive references
const video = ref(null)
const canvas = ref(null)
const particles = ref(null)
const statusMessage = ref('')
const result = ref('')
const isCapturing = ref(false)
const progress = ref(0)
const cameraReady = ref(false)

// Lifecycle
onMounted(() => {
  initCamera()
  createParticles()
})

onUnmounted(() => {
  // Cleanup camera stream
  if (video.value && video.value.srcObject) {
    const tracks = video.value.srcObject.getTracks()
    tracks.forEach(track => track.stop())
  }
})

// Methods
async function initCamera() {
  try {
    const stream = await navigator.mediaDevices.getUserMedia({
      video: {
        width: { ideal: 640 },
        height: { ideal: 480 },
        facingMode: 'user'
      }
    })

    if (video.value) {
      video.value.srcObject = stream
      cameraReady.value = true
      statusMessage.value = 'Camera ready! Position your face in the frame'
    }
  } catch (err) {
    statusMessage.value = 'Camera error: ' + err.message
    console.error('Camera initialization error:', err)
  }
}

async function startCapturing() {
  if (isCapturing.value) return

  isCapturing.value = true
  result.value = ''
  progress.value = 0
  const blobs = []

  try {
    for (let i = 0; i < 5; i++) {
      statusMessage.value = `Taking photo ${i + 1} of 5...`
      const blob = await capturePhoto()
      blobs.push(blob)
      progress.value = ((i + 1) / 5) * 100
      if (i < 4) await delay(1000)
    }

    statusMessage.value = 'Uploading to server...'
    await sendPhotos(blobs)
  } catch (error) {
    statusMessage.value = 'Upload failed'
    result.value = 'Error: ' + error.message
  } finally {
    isCapturing.value = false
  }
}

function capturePhoto() {
  return new Promise(resolve => {
    const ctx = canvas.value.getContext('2d')
    ctx.drawImage(video.value, 0, 0, canvas.value.width, canvas.value.height)
    canvas.value.toBlob(blob => resolve(blob), 'image/jpeg', 0.8)
  })
}

function delay(ms) {
  return new Promise(resolve => setTimeout(resolve, ms))
}

async function sendPhotos(blobs) {
  const formData = new FormData()
  blobs.forEach((blob, index) => {
    formData.append('photos', blob, `face${index + 1}.jpg`)
  })

  try {
    const res = await fetch('http://localhost:8001/auth/register-or-login/face', {
      method: 'POST',
      body: formData
    })

    if (!res.ok) {
      throw new Error(`Server error: ${res.status}`)
    }

    const data = await res.json()

    // authentication data
    const queryParams = new URLSearchParams({
      token: data.token,
      userId: data.userId,
      email: data.email,
      name: data.name,
      faceId: data.faceId
    })

    // redirect
    setTimeout(() => {
      window.location.href = 'http://localhost:8080/face-login?' + queryParams.toString()
    }, 1000)
  } catch (e) {
    throw new Error('Upload failed: ' + e.message)
  }
}

function createParticles() {
  if (!particles.value) return

  for (let i = 0; i < 30; i++) {
    const particle = document.createElement('div')
    particle.className = 'particle'
    particle.style.left = Math.random() * 100 + '%'
    particle.style.top = Math.random() * 100 + '%'
    particle.style.animationDelay = Math.random() * 6 + 's'
    particle.style.animationDuration = (Math.random() * 3 + 3) + 's'
    particles.value.appendChild(particle)
  }
}
</script>

<style scoped>
.face-recognition-container {
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
  position: relative;
  overflow: hidden;
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
}

/* Background decorations */
.background-decoration {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  overflow: hidden;
  z-index: 1;
}

.decoration-circle {
  position: absolute;
  border-radius: 50%;
}

.decoration-circle-1 {
  top: -25%;
  right: -25%;
  width: 400px;
  height: 400px;
  background: linear-gradient(45deg, rgba(66, 153, 225, 0.2), rgba(159, 122, 234, 0.2));
}

.decoration-circle-2 {
  bottom: -25%;
  left: -25%;
  width: 400px;
  height: 400px;
  background: linear-gradient(45deg, rgba(102, 126, 234, 0.2), rgba(237, 100, 166, 0.2));
}

.particles {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}

.particle {
  position: absolute;
  width: 4px;
  height: 4px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 50%;
  animation: float 6s ease-in-out infinite;
}

@keyframes float {
  0%, 100% { 
    transform: translateY(0px) rotate(0deg); 
    opacity: 0.3; 
  }
  50% { 
    transform: translateY(-20px) rotate(180deg); 
    opacity: 0.8; 
  }
}

/* Main container */
.main-container {
  position: relative;
  z-index: 10;
  width: 100%;
  max-width: 420px;
}

.recognition-card {
  background: rgba(255, 255, 255, 0.95);
  border-radius: 24px;
  padding: 40px 32px;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.2);
  transition: all 0.3s ease;
}

.recognition-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 30px 60px rgba(0, 0, 0, 0.15);
}

/* Header */
.card-header {
  text-align: center;
  margin-bottom: 32px;
}

.header-icon {
  width: 60px;
  height: 60px;
  margin: 0 auto 16px;
  background: linear-gradient(135deg, #667eea, #764ba2);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
}

.icon {
  width: 24px;
  height: 24px;
}

.header-title {
  color: #2d3748;
  font-size: 28px;
  font-weight: 700;
  margin-bottom: 8px;
  background: linear-gradient(135deg, #667eea, #764ba2);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.header-subtitle {
  color: #718096;
  font-size: 16px;
  margin: 0;
}

/* Video container */
.video-container {
  position: relative;
  margin-bottom: 24px;
  border-radius: 16px;
  overflow: hidden;
}

.video-frame {
  position: relative;
  width: 100%;
  height: 240px;
  border-radius: 16px;
  overflow: hidden;
  background: linear-gradient(45deg, #f0f0f0, #e0e0e0);
  display: flex;
  align-items: center;
  justify-content: center;
}

.video-element {
  width: 100%;
  height: 100%;
  object-fit: cover;
  border-radius: 16px;
  display: none;
}

.video-element.video-ready {
  display: block;
}

.camera-placeholder {
  color: #a0aec0;
  text-align: center;
}

.camera-icon {
  width: 48px;
  height: 48px;
  margin-bottom: 8px;
}

.scan-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  border: 3px solid transparent;
  border-radius: 16px;
  transition: all 0.3s ease;
}

.scan-overlay.scan-active {
  border-color: #4299e1;
  box-shadow: 0 0 20px rgba(66, 153, 225, 0.3);
  animation: pulse 2s infinite;
}

@keyframes pulse {
  0%, 100% { 
    box-shadow: 0 0 20px rgba(66, 153, 225, 0.3); 
  }
  50% { 
    box-shadow: 0 0 30px rgba(66, 153, 225, 0.6); 
  }
}

.corner-decorator {
  position: absolute;
  width: 24px;
  height: 24px;
  border: 2px solid #4299e1;
  opacity: 0.6;
}

.corner-tl {
  top: 16px;
  left: 16px;
  border-right: none;
  border-bottom: none;
}

.corner-tr {
  top: 16px;
  right: 16px;
  border-left: none;
  border-bottom: none;
}

.corner-bl {
  bottom: 16px;
  left: 16px;
  border-right: none;
  border-top: none;
}

.corner-br {
  bottom: 16px;
  right: 16px;
  border-left: none;
  border-top: none;
}

/* Status message */
.status-message {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px 16px;
  margin-bottom: 20px;
  border-radius: 12px;
  font-size: 14px;
  font-weight: 500;
  transition: all 0.3s ease;
}

.status-info {
  background: linear-gradient(135deg, #bee3f8, #90cdf4);
  color: #2b6cb0;
  border: 1px solid rgba(43, 108, 176, 0.2);
}

.status-success {
  background: linear-gradient(135deg, #c6f6d5, #9ae6b4);
  color: #22543d;
  border: 1px solid rgba(34, 84, 61, 0.2);
}

.status-error {
  background: linear-gradient(135deg, #fed7d7, #feb2b2);
  color: #742a2a;
  border: 1px solid rgba(116, 42, 42, 0.2);
}

.status-icon {
  width: 20px;
  height: 20px;
  flex-shrink: 0;
}

.status-text {
  flex: 1;
}

/* Progress bar */
.progress-container {
  margin-bottom: 24px;
}

.progress-info {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 8px;
}

.progress-label {
  color: #4a5568;
  font-size: 14px;
  font-weight: 500;
}

.progress-percent {
  color: #4a5568;
  font-size: 14px;
  font-weight: 600;
}

.progress-bar {
  width: 100%;
  height: 8px;
  background: rgba(226, 232, 240, 0.8);
  border-radius: 4px;
  overflow: hidden;
  position: relative;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #4299e1, #667eea);
  border-radius: 4px;
  transition: width 0.3s ease;
  position: relative;
  overflow: hidden;
}

.progress-shimmer {
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255,255,255,0.4), transparent);
  animation: shimmer 1.5s infinite;
}

@keyframes shimmer {
  0% { left: -100%; }
  100% { left: 100%; }
}

/* Capture button */
.capture-button {
  width: 100%;
  padding: 16px 24px;
  background: linear-gradient(135deg, #4299e1, #667eea);
  color: white;
  border: none;
  border-radius: 16px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  position: relative;
  overflow: hidden;
}

.capture-button:hover:not(.button-disabled) {
  transform: translateY(-2px);
  box-shadow: 0 10px 25px rgba(66, 153, 225, 0.4);
}

.capture-button:active:not(.button-disabled) {
  transform: translateY(0);
}

.capture-button.button-disabled {
  opacity: 0.7;
  cursor: not-allowed;
  transform: none;
}

.button-content {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
}

.button-icon {
  width: 20px;
  height: 20px;
}

.button-spinner {
  width: 20px;
  height: 20px;
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-radius: 50%;
  border-top-color: white;
  animation: spin 1s ease-in-out infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

/* Result */
.result-container {
  margin-top: 24px;
  padding: 20px;
  background: linear-gradient(135deg, #f0fff4, #c6f6d5);
  border: 1px solid rgba(72, 187, 120, 0.2);
  border-radius: 16px;
  color: #22543d;
}

.result-header {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 12px;
}

.result-icon {
  width: 20px;
  height: 20px;
  color: #38a169;
}

.result-title {
  font-weight: 600;
  color: #22543d;
}

.result-content {
  font-family: 'SF Mono', 'Monaco', 'Cascadia Code', monospace;
  font-size: 13px;
  line-height: 1.5;
  color: #22543d;
  white-space: pre-wrap;
  max-height: 200px;
  overflow-y: auto;
  background: rgba(255, 255, 255, 0.3);
  padding: 12px;
  border-radius: 8px;
  margin: 0;
}

/* Footer */
.footer {
  text-align: center;
  margin-top: 24px;
}

.footer p {
  color: rgba(255, 255, 255, 0.8);
  font-size: 14px;
  margin: 0;
}

/* Responsive */
@media (max-width: 480px) {
  .face-recognition-container {
    padding: 16px;
  }

  .recognition-card {
    padding: 24px 20px;
  }

  .header-title {
    font-size: 24px;
  }
}
</style>