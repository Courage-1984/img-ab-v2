<template>
  <div class="modal-backdrop">
    <div class="modal">
      <h2>Export as GIF/Video</h2>
      <div style="color: #ff0; font-size: 0.95em; margin-bottom: 0.5em;">All debug logs are saved to <b>debug-log.txt</b> in the app folder for sharing and troubleshooting.</div>
      <div v-if="images && images.length === 2">
        <p><strong>Image 1:</strong> {{ images[0].name }}</p>
        <p><strong>Image 2:</strong> {{ images[1].name }}</p>
      </div>
      <div v-else>
        <p style="color: red;">Need at least two images to export.</p>
      </div>
      <form @submit.prevent="onExport">
        <label>
          Export Type:
          <select v-model="exportType">
            <option value="gif">GIF</option>
            <option value="video">Video (MP4)</option>
          </select>
        </label>
        <br />
        <label>
          Slider Speed (seconds):
          <input type="number" v-model.number="sliderSpeed" min="0.5" step="0.1" />
        </label>
        <br />
        <div class="modal-actions">
          <button type="button" @click="$emit('close')">Cancel</button>
          <button type="submit" :disabled="!canExport">Export</button>
        </div>
      </form>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, nextTick } from 'vue';
import html2canvas from 'html2canvas';

const props = defineProps({
  images: {
    type: Array,
    required: true
  }
});
const emit = defineEmits(['close', 'export']);

console.log('ExportGifVideoModal props:', props);

const exportType = ref('gif');
const sliderSpeed = ref(2.0);

const canExport = computed(() => props.images && props.images.length === 2);

async function onExport() {
  // Hide the modal
  const modal = document.querySelector('.modal-backdrop');
  if (modal) modal.style.display = 'none';

  // Remove all usage of imageCompareInstance
  // Only allow export if two images are present
  if (!props.images || props.images.length < 2) {
    if (modal) modal.style.display = '';
    alert('Need at least two images to export.');
    emit('close');
    return;
  }
  const imageCompareEl = document.querySelector('.image-compare');
  const imageEl = imageCompareEl.querySelector('.icv__img');
  if (!imageEl) {
    if (modal) modal.style.display = '';
    alert('No image found in comparison view.');
    emit('close');
    return;
  }
  const frames = [];
  const steps = 30;
  for (let i = 0; i <= steps; i++) {
    const pos = (i / steps) * 100;
    // imageCompareInstance.setSliderPosition(pos); // This line is removed
    await nextTick(); // Wait for Vue to update the DOM
    await new Promise(r => setTimeout(r, 50)); // Wait for browser to paint

    // Log computed styles after waiting
    const wrapper = imageCompareEl.querySelector('.icv__wrapper');
    const imgA = imageCompareEl.querySelector('.icv__img-a');
    const imgB = imageCompareEl.querySelector('.icv__img-b');
    if (wrapper) {
      console.log(`[AFTER WAIT] Frame ${i} wrapper style:`, getComputedStyle(wrapper).width, getComputedStyle(wrapper).height, wrapper.style.clipPath);
    }
    if (imgA) {
      console.log(`[AFTER WAIT] Frame ${i} imgA style:`, getComputedStyle(imgA).width, getComputedStyle(imgA).height);
    }
    if (imgB) {
      console.log(`[AFTER WAIT] Frame ${i} imgB style:`, getComputedStyle(imgB).width, getComputedStyle(imgB).height);
    }

    const canvas = await html2canvas(imageCompareEl);
    const frameDataUrl = canvas.toDataURL();
    console.log(`Captured frame ${i} at slider position ${pos}:`, frameDataUrl);
    frames.push(frameDataUrl);
  }
  if (modal) modal.style.display = '';
  // Log the export event data
  try {
    console.log('[ExportGifVideoModal] Emitting export event with:', {
      framesCount: frames.length,
      exportType: exportType.value,
      sliderSpeed: sliderSpeed.value,
      // Do not include imageCompareInstance or DOM elements
    });
  } catch (err) {
    console.error('[ExportGifVideoModal] Error logging export event data:', err);
  }
  emit('export', { frames, exportType: exportType.value, sliderSpeed: sliderSpeed.value });
  emit('close');
}
</script>

<style scoped>
.modal-backdrop {
  position: fixed;
  top: 0; left: 0; right: 0; bottom: 0;
  background: rgba(0,0,0,0.5);
  z-index: 1000;
  display: flex;
  align-items: center;
  justify-content: center;
}
.modal {
  background: #222;
  color: #fff;
  padding: 2rem;
  border-radius: 8px;
  min-width: 320px;
  box-shadow: 0 2px 16px rgba(0,0,0,0.4);
}
.modal-actions {
  margin-top: 1.5rem;
  display: flex;
  gap: 1rem;
  justify-content: flex-end;
}
button {
  padding: 0.5rem 1.2rem;
  border: none;
  border-radius: 4px;
  background: #444;
  color: #fff;
  cursor: pointer;
}
button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
</style> 