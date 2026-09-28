<template>
  <ion-page class="cyberpaw-page">
    <!-- 0. Cat Themed Loading Splash Screen -->
    <transition name="fade">
      <div v-if="isLoading" class="paw-splash-screen">
        <div class="splash-box">
          <div class="splash-icon-wrapper">
            <i class="fa-solid fa-cat splash-cat"></i>
            <div class="scanner-ring"></div>
          </div>
          <h1 class="splash-title">CYBERPAW</h1>
          <p class="splash-sub">Readying cryptographic buffers...</p>
          <div class="splash-bar">
            <div class="splash-progress"></div>
          </div>
        </div>
      </div>
    </transition>

    <!-- 1. Header (Adaptive for Mobile Notch & Desktop Browser) -->
    <header class="paw-header">
      <div class="header-inner">
        <div class="paw-navbar">
          <div class="paw-brand">
            <div class="brand-avatar">
              <i class="fa-solid fa-cat brand-cat"></i>
            </div>
            <div class="brand-titles">
              <span class="brand-text">CYBERPAW</span>
              <span class="brand-subtitle">Cryptographic Security Tool</span>
            </div>
          </div>
          <div class="brand-meta">
            <span class="brand-badge">v1.2</span>
            <span class="brand-badge mode-badge">{{ cipherLabel }}</span>
          </div>
        </div>

        <!-- Mode Segment (Console only) -->
        <div class="mode-wrapper" v-if="activeTab === 'console'">
          <div class="mode-segmented">
            <button 
              type="button" 
              :class="['mode-tab', mode === 'encrypt' ? 'active' : '']" 
              @click="mode = 'encrypt'"
            >
              <i class="fa-solid fa-lock"></i>
              <span>Encrypt</span>
            </button>
            <button 
              type="button" 
              :class="['mode-tab', mode === 'decrypt' ? 'active' : '']" 
              @click="mode = 'decrypt'"
            >
              <i class="fa-solid fa-lock-open"></i>
              <span>Decrypt</span>
            </button>
          </div>
        </div>
      </div>
    </header>

    <!-- 2. Main Content Canvas -->
    <ion-content class="paw-content" :fullscreen="false">
      <div class="content-container">

        <!-- TAB 1: CONSOLE -->
        <div v-show="activeTab === 'console'" class="tab-pane">
          <!-- Parameters Card -->
          <div class="paw-card">
            <div class="card-head">
              <span class="card-tag"><i class="fa-solid fa-sliders"></i> Parameters</span>
            </div>

            <div class="form-row">
              <!-- Custom Select Dropdown Trigger -->
              <div class="field-item">
                <label class="field-title">Cipher Algorithm</label>
                <div class="custom-select-trigger" @click="showCipherPicker = !showCipherPicker">
                  <span>{{ cipherLabel }}</span>
                  <i class="fa-solid fa-chevron-down caret-icon"></i>
                </div>
              </div>

              <!-- Parameter Input Dynamic Fields -->
              <div class="field-item" v-if="cipherType === 'caesar'">
                <label class="field-title">Numeric Shift (1-25)</label>
                <div class="stepper-box">
                  <button type="button" class="step-btn" @click="shiftKey = Math.max(1, shiftKey - 1)">−</button>
                  <input type="number" v-model.number="shiftKey" min="1" max="25" class="stepper-field" />
                  <button type="button" class="step-btn" @click="shiftKey = Math.min(25, shiftKey + 1)">+</button>
                </div>
              </div>

              <div class="field-item" v-else-if="cipherType === 'vigenere'">
                <label class="field-title">Secret Keyword</label>
                <input 
                  type="text" 
                  v-model="vigenereKey" 
                  placeholder="e.g. PAW" 
                  class="field-input uppercase"
                />
              </div>

              <div class="field-item" v-else-if="cipherType === 'railfence'">
                <label class="field-title">Rails / Rows (2-10)</label>
                <div class="stepper-box">
                  <button type="button" class="step-btn" @click="railKey = Math.max(2, railKey - 1)">−</button>
                  <input type="number" v-model.number="railKey" min="2" max="10" class="stepper-field" />
                  <button type="button" class="step-btn" @click="railKey = Math.min(10, railKey + 1)">+</button>
                </div>
              </div>

              <div class="field-item" v-else>
                <label class="field-title">Mode Constraint</label>
                <div class="static-pill-field">
                  <i class="fa-solid fa-circle-check"></i>
                  <span>No key required</span>
                </div>
              </div>
            </div>
          </div>

          <!-- Input Card -->
          <div class="paw-card">
            <div class="card-head">
              <span class="card-tag">
                <i class="fa-solid fa-keyboard"></i> {{ mode === 'encrypt' ? 'Plaintext Message' : 'Ciphertext Input' }}
              </span>
              <div class="pill-group">
                <button type="button" class="pill-btn" @click="inputText = 'THE QUICK BROWN CAT'">Preset</button>
                <button type="button" class="pill-btn" @click="inputText = ''">Clear</button>
              </div>
            </div>
            <textarea 
              v-model="inputText" 
              class="field-textarea" 
              rows="4" 
              :placeholder="mode === 'encrypt' ? 'Type text here to encrypt...' : 'Paste encrypted text here to decrypt...'"
            ></textarea>
          </div>

          <!-- Output Card -->
          <div class="paw-card output-card">
            <div class="card-head">
              <span class="card-tag tag-white">
                <i class="fa-solid fa-shield-cat"></i> Output Result
              </span>
              <button type="button" class="pill-btn copy-btn" @click="copyResult" :disabled="!resultText">
                <i class="fa-regular fa-copy"></i> Copy
              </button>
            </div>
            <div class="result-display">
              {{ resultText || 'Result will appear here in real time...' }}
            </div>
          </div>
        </div>

        <!-- TAB 2: INSPECTOR -->
        <div v-show="activeTab === 'inspector'" class="tab-pane">
          <div class="paw-card">
            <div class="card-head">
              <span class="card-tag"><i class="fa-solid fa-paw"></i> Dynamic Character Mapping</span>
            </div>
            <p class="tab-sub">Real-time inspection of the first 16 characters:</p>

            <div v-if="transformations.length > 0" class="step-grid">
              <div v-for="(item, idx) in transformations" :key="idx" class="step-cell">
                <div class="char-original">{{ item.orig }}</div>
                <i class="fa-solid fa-arrow-down-long step-icon"></i>
                <div class="char-transformed">{{ item.transformed }}</div>
                <span class="char-shift">{{ item.shiftLabel }}</span>
              </div>
            </div>
            <div v-else class="empty-state">
              <i class="fa-solid fa-cat empty-icon"></i>
              <p>Type characters in Console to view live transformations.</p>
            </div>
          </div>
        </div>

        <!-- TAB 3: GUIDE -->
        <div v-show="activeTab === 'guide'" class="tab-pane">
          <div class="paw-card">
            <div class="card-head">
              <span class="card-tag"><i class="fa-solid fa-book"></i> Cryptography Knowledge Hub</span>
            </div>
            <p class="tab-sub">Clean, jargon-free overview of all supported ciphers:</p>

            <div class="info-block">
              <div class="info-header">
                <span class="info-badge">Shift</span>
                <span class="info-name">1. Caesar Cipher</span>
              </div>
              <p class="info-desc">
                Shifts each letter along the alphabet by a fixed numeric key. For example, with a shift of 3, <strong>A</strong> becomes <strong>D</strong>, and <strong>Z</strong> wraps around back to <strong>C</strong>.
              </p>
              <div class="info-example">
                <code>Formula: C = (P + k) mod 26</code>
              </div>
            </div>

            <div class="info-block">
              <div class="info-header">
                <span class="info-badge">Keyed</span>
                <span class="info-name">2. Vigenère Cipher</span>
              </div>
              <p class="info-desc">
                Uses a repeating secret word (e.g., <em>PAW</em>). Each letter in the key provides a different shift for the matching character in your message, offering stronger resistance against frequency attacks.
              </p>
              <div class="info-example">
                <code>Key: P (+15), A (+0), W (+22)</code>
              </div>
            </div>

            <div class="info-block">
              <div class="info-header">
                <span class="info-badge">Symmetric</span>
                <span class="info-name">3. ROT13</span>
              </div>
              <p class="info-desc">
                A special case of the Caesar cipher with a fixed shift of 13. Because the English alphabet has 26 letters, running ROT13 twice on any message restores the original plaintext.
              </p>
            </div>

            <div class="info-block">
              <div class="info-header">
                <span class="info-badge">Binary</span>
                <span class="info-name">4. Base64 Encoding</span>
              </div>
              <p class="info-desc">
                Converts text bytes into an ASCII radix-64 representation using letters, digits, and standard punctuation. Commonly used across web protocols and authentication headers.
              </p>
            </div>

            <div class="info-block no-border">
              <div class="info-header">
                <span class="info-badge">Transposition</span>
                <span class="info-name">5. Rail Fence Cipher</span>
              </div>
              <p class="info-desc">
                Rearranges the order of characters by writing them diagonally down and up across imaginary rows (rails) like a zigzag fence, then reading off each row sequentially.
              </p>
            </div>
          </div>
        </div>

      </div>

      <!-- iOS Custom Action Sheet Modal for Cipher Picker -->
      <div v-if="showCipherPicker" class="ios-backdrop" @click="showCipherPicker = false">
        <div class="ios-sheet" @click.stop>
          <div class="ios-sheet-header">Select Algorithm</div>
          <button 
            v-for="opt in cipherOptions" 
            :key="opt.value" 
            type="button" 
            :class="['ios-sheet-btn', cipherType === opt.value ? 'selected' : '']" 
            @click="cipherType = opt.value; showCipherPicker = false"
          >
            <div class="btn-left">
              <i :class="opt.icon"></i>
              <span>{{ opt.name }}</span>
            </div>
            <i v-if="cipherType === opt.value" class="fa-solid fa-check"></i>
          </button>
          <button type="button" class="ios-sheet-btn cancel" @click="showCipherPicker = false">Cancel</button>
        </div>
      </div>

      <!-- Custom Themed Copy Toast Modal -->
      <transition name="toast-pop">
        <div v-if="showToast" class="paw-toast-modal">
          <i class="fa-solid fa-cat toast-cat"></i>
          <span>Copied to Clipboard!</span>
        </div>
      </transition>
    </ion-content>

    <!-- 3. Bottom Tab Navigation Bar -->
    <footer class="paw-footer-bar">
      <div class="footer-inner">
        <button 
          type="button" 
          :class="['nav-item', activeTab === 'console' ? 'active' : '']" 
          @click="activeTab = 'console'"
        >
          <i class="fa-solid fa-terminal"></i>
          <span>Console</span>
        </button>

        <button 
          type="button" 
          :class="['nav-item', activeTab === 'inspector' ? 'active' : '']" 
          @click="activeTab = 'inspector'"
        >
          <i class="fa-solid fa-paw"></i>
          <span>Inspector</span>
        </button>

        <button 
          type="button" 
          :class="['nav-item', activeTab === 'guide' ? 'active' : '']" 
          @click="activeTab = 'guide'"
        >
          <i class="fa-solid fa-book-open"></i>
          <span>Guide</span>
        </button>
      </div>
    </footer>
  </ion-page>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';
import { IonPage, IonContent } from '@ionic/vue';

const isLoading = ref(true);
const activeTab = ref('console');
const mode = ref('encrypt');
const cipherType = ref('caesar');
const shiftKey = ref(3);
const vigenereKey = ref('PAW');
const railKey = ref(3);
const inputText = ref('');
const showCipherPicker = ref(false);
const showToast = ref(false);

const cipherOptions = [
  { value: 'caesar', name: 'Caesar Cipher', icon: 'fa-solid fa-arrow-right-arrow-left' },
  { value: 'vigenere', name: 'Vigenère Cipher', icon: 'fa-solid fa-key' },
  { value: 'rot13', name: 'ROT13', icon: 'fa-solid fa-arrows-rotate' },
  { value: 'base64', name: 'Base64 Encoding', icon: 'fa-solid fa-code' },
  { value: 'railfence', name: 'Rail Fence (Zigzag)', icon: 'fa-solid fa-bars-staggered' }
];

const cipherLabel = computed(() => {
  const match = cipherOptions.find(o => o.value === cipherType.value);
  return match ? match.name : 'Caesar Cipher';
});

onMounted(() => {
  setTimeout(() => {
    isLoading.value = false;
  }, 1600);
});

// --- CIPHER ALGORITHMS ---

// 1. Caesar
const runCaesar = (str, shift, encrypt = true) => {
  const normShift = ((shift % 26) + 26) % 26;
  const effectiveShift = encrypt ? normShift : (26 - normShift) % 26;

  return str.split('').map(char => {
    const code = char.charCodeAt(0);
    if (code >= 65 && code <= 90) {
      return String.fromCharCode(((code - 65 + effectiveShift) % 26) + 65);
    } else if (code >= 97 && code <= 122) {
      return String.fromCharCode(((code - 97 + effectiveShift) % 26) + 97);
    }
    return char;
  }).join('');
};

// 2. Vigenere
const runVigenere = (str, keyword, encrypt = true) => {
  const cleanKey = keyword.toUpperCase().replace(/[^A-Z]/g, '') || 'A';
  let keyIndex = 0;

  return str.split('').map(char => {
    const code = char.charCodeAt(0);
    let shift = cleanKey.charCodeAt(keyIndex % cleanKey.length) - 65;
    if (!encrypt) shift = (26 - shift) % 26;

    if (code >= 65 && code <= 90) {
      keyIndex++;
      return String.fromCharCode(((code - 65 + shift) % 26) + 65);
    } else if (code >= 97 && code <= 122) {
      keyIndex++;
      return String.fromCharCode(((code - 97 + shift) % 26) + 97);
    }
    return char;
  }).join('');
};

// 3. ROT13
const runRot13 = (str) => {
  return runCaesar(str, 13, true);
};

// 4. Base64
const runBase64 = (str, encrypt = true) => {
  try {
    if (encrypt) {
      return btoa(unescape(encodeURIComponent(str)));
    } else {
      return decodeURIComponent(escape(atob(str)));
    }
  } catch {
    return '[Invalid Base64 sequence]';
  }
};

// 5. Rail Fence
const runRailFence = (text, rails, encrypt = true) => {
  if (rails <= 1 || !text) return text;

  if (encrypt) {
    const fence = Array.from({ length: rails }, () => []);
    let rail = 0;
    let direction = 1;

    for (const char of text) {
      fence[rail].push(char);
      rail += direction;
      if (rail === 0 || rail === rails - 1) direction = -direction;
    }
    return fence.flat().join('');
  } else {
    const len = text.length;
    const pattern = new Array(len);
    let rail = 0;
    let direction = 1;

    for (let i = 0; i < len; i++) {
      pattern[i] = rail;
      rail += direction;
      if (rail === 0 || rail === rails - 1) direction = -direction;
    }

    const result = new Array(len);
    let textIndex = 0;

    for (let r = 0; r < rails; r++) {
      for (let i = 0; i < len; i++) {
        if (pattern[i] === r) {
          result[i] = text[textIndex++];
        }
      }
    }
    return result.join('');
  }
};

// Result Computation
const resultText = computed(() => {
  if (!inputText.value) return '';
  const isEncrypt = mode.value === 'encrypt';

  switch (cipherType.value) {
    case 'caesar':
      return runCaesar(inputText.value, shiftKey.value, isEncrypt);
    case 'vigenere':
      return runVigenere(inputText.value, vigenereKey.value, isEncrypt);
    case 'rot13':
      return runRot13(inputText.value);
    case 'base64':
      return runBase64(inputText.value, isEncrypt);
    case 'railfence':
      return runRailFence(inputText.value, railKey.value, isEncrypt);
    default:
      return inputText.value;
  }
});

// Transformations Mapping
const transformations = computed(() => {
  if (!inputText.value) return [];
  const textChars = inputText.value.slice(0, 16).split('');
  const outputChars = (resultText.value || '').slice(0, 16).split('');
  const isEncrypt = mode.value === 'encrypt';

  const cleanKey = vigenereKey.value.toUpperCase().replace(/[^A-Z]/g, '') || 'A';
  let vIndex = 0;

  return textChars.map((char, i) => {
    let label = 'Direct';
    const isLetter = /[a-zA-Z]/.test(char);

    if (cipherType.value === 'caesar') {
      label = isLetter ? `${isEncrypt ? '+' : '−'}${shiftKey.value}` : 'Keep';
    } else if (cipherType.value === 'vigenere') {
      if (isLetter) {
        const keyChar = cleanKey[vIndex % cleanKey.length];
        const s = keyChar.charCodeAt(0) - 65;
        label = `${keyChar} (${isEncrypt ? '+' : '−'}${s})`;
        vIndex++;
      }
    } else if (cipherType.value === 'rot13') {
      label = isLetter ? '±13' : 'Keep';
    } else if (cipherType.value === 'base64') {
      label = 'Radix-64';
    } else if (cipherType.value === 'railfence') {
      label = `Rail`;
    }

    return {
      orig: char === ' ' ? '␣' : char,
      transformed: outputChars[i] === ' ' ? '␣' : (outputChars[i] || '—'),
      shiftLabel: label
    };
  });
});

const copyResult = async () => {
  if (resultText.value) {
    await navigator.clipboard.writeText(resultText.value);
    showToast.value = true;
    setTimeout(() => {
      showToast.value = false;
    }, 2000);
  }
};
</script>

<style scoped>
.cyberpaw-page {
  --background: #000000;
  background: #000000;
  color: #ffffff;
  display: flex;
  flex-direction: column;
  height: 100vh;
  font-family: -apple-system, BlinkMacSystemFont, "SF Pro Text", "Segoe UI", Roboto, sans-serif;
  -webkit-font-smoothing: antialiased;
}

/* 0. Themed Loading Splash Screen */
.paw-splash-screen {
  position: fixed;
  inset: 0;
  background: #000000;
  z-index: 9999;
  display: flex;
  align-items: center;
  justify-content: center;
}

.splash-box {
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
  padding: 24px;
}

.splash-icon-wrapper {
  position: relative;
  width: 80px;
  height: 80px;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 20px;
}

.splash-cat {
  font-size: 2.8rem;
  color: #ffffff;
}

.scanner-ring {
  position: absolute;
  inset: 0;
  border-radius: 50%;
  border: 2px dashed rgba(255, 255, 255, 0.3);
  animation: spin 3s linear infinite;
}

@keyframes spin {
  100% { transform: rotate(360deg); }
}

.splash-title {
  font-family: monospace;
  font-size: 1.6rem;
  font-weight: 800;
  letter-spacing: 4px;
  margin-bottom: 4px;
}

.splash-sub {
  font-size: 0.72rem;
  letter-spacing: 2px;
  color: #71717a;
  margin-bottom: 20px;
}

.splash-bar {
  width: 180px;
  height: 3px;
  background: #18181b;
  border-radius: 4px;
  overflow: hidden;
  margin-bottom: 12px;
}

.splash-progress {
  width: 100%;
  height: 100%;
  background: #ffffff;
  animation: loadProgress 1.6s ease-in-out forwards;
}

@keyframes loadProgress {
  0% { transform: translateX(-100%); }
  100% { transform: translateX(0%); }
}

.splash-status {
  font-size: 0.75rem;
  color: #52525b;
  display: flex;
  align-items: center;
  gap: 6px;
}

/* 1. Header (Adaptive Padding: Notch-aware on Mobile, Clean on Desktop) */
.paw-header {
  background: #0a0a0c;
  border-bottom: 1px solid rgba(255, 255, 255, 0.08);
  flex-shrink: 0;
  padding-top: calc(env(safe-area-inset-top, 24px) + 8px);
}

@media screen and (min-width: 768px) {
  .paw-header {
    padding-top: 14px;
  }
}

.header-inner {
  max-width: 720px;
  margin: 0 auto;
  width: 100%;
}

.paw-navbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 18px;
}

.paw-brand {
  display: flex;
  align-items: center;
  gap: 12px;
}

.brand-avatar {
  width: 38px;
  height: 38px;
  border-radius: 10px;
  background: #141416;
  border: 1px solid #27272a;
  display: flex;
  align-items: center;
  justify-content: center;
}

.brand-cat {
  font-size: 1.25rem;
  color: #ffffff;
}

.brand-titles {
  display: flex;
  flex-direction: column;
}

.brand-text {
  font-family: monospace;
  font-weight: 800;
  font-size: 1.15rem;
  letter-spacing: 2px;
  line-height: 1.2;
}

.brand-subtitle {
  font-size: 0.72rem;
  color: #71717a;
}

.brand-meta {
  display: flex;
  gap: 6px;
}

.brand-badge {
  font-size: 0.7rem;
  background: #18181b;
  color: #a1a1aa;
  border: 1px solid #27272a;
  padding: 4px 8px;
  border-radius: 6px;
  font-weight: 600;
}

.mode-badge {
  color: #ffffff;
  border-color: #3f3f46;
}

/* Segmented Control */
.mode-wrapper {
  padding: 6px 18px 14px;
}

.mode-segmented {
  display: flex;
  background: #18181b;
  border-radius: 12px;
  padding: 3px;
  border: 1px solid #27272a;
}

.mode-tab {
  flex: 1;
  height: 40px;
  background: transparent;
  color: #a1a1aa;
  border: none;
  font-size: 0.88rem;
  font-weight: 600;
  border-radius: 9px;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.mode-tab.active {
  background: #ffffff;
  color: #000000;
  box-shadow: 0 3px 8px rgba(0, 0, 0, 0.4);
}

/* 2. Main Content Canvas & Balanced Grid */
.paw-content {
  --background: #000000;
  flex: 1;
}

.content-container {
  max-width: 720px;
  margin: 0 auto;
  padding: 20px 18px;
}

.paw-card {
  background: #101013;
  border: 1px solid #222226;
  border-radius: 16px;
  padding: 18px;
  margin-bottom: 18px;
}

.output-card {
  border-color: #2e2e36;
  background: #131317;
}

.card-head {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
}

.card-tag {
  font-family: monospace;
  font-size: 0.8rem;
  text-transform: uppercase;
  color: #a1a1aa;
  letter-spacing: 1px;
  display: flex;
  align-items: center;
  gap: 8px;
  font-weight: 600;
}

.tag-white {
  color: #ffffff;
}

.form-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 14px;
}

@media (max-width: 480px) {
  .form-row {
    grid-template-columns: 1fr;
  }
}

.field-title {
  display: block;
  font-size: 0.75rem;
  text-transform: uppercase;
  color: #71717a;
  margin-bottom: 8px;
  font-weight: 600;
}

.custom-select-trigger {
  height: 48px;
  background: #18181c;
  border: 1px solid #2c2c33;
  border-radius: 12px;
  padding: 0 14px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  font-size: 0.92rem;
  color: #ffffff;
  cursor: pointer;
}

.caret-icon {
  font-size: 0.75rem;
  color: #71717a;
}

.field-input {
  width: 100%;
  height: 48px;
  background: #18181c;
  border: 1px solid #2c2c33;
  border-radius: 12px;
  padding: 0 14px;
  font-size: 0.92rem;
  color: #ffffff;
  outline: none;
  box-sizing: border-box;
}

.field-input.uppercase {
  text-transform: uppercase;
}

.stepper-box {
  display: flex;
  height: 48px;
  background: #18181c;
  border: 1px solid #2c2c33;
  border-radius: 12px;
  overflow: hidden;
}

.step-btn {
  width: 46px;
  background: transparent;
  color: #ffffff;
  border: none;
  font-size: 1.25rem;
  cursor: pointer;
}

/* Remove default browser up/down arrows from number input */
.stepper-field::-webkit-outer-spin-button,
.stepper-field::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

.stepper-field[type="number"] {
  -moz-appearance: textfield;
  appearance: textfield;
}

.stepper-field {
  flex: 1;
  background: transparent;
  border: none;
  color: #ffffff;
  text-align: center;
  font-size: 1.05rem;
  font-family: monospace;
  font-weight: 700;
  outline: none;
}

.static-pill-field {
  height: 48px;
  background: #141417;
  border: 1px solid #242429;
  border-radius: 12px;
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 0 14px;
  font-size: 0.85rem;
  color: #71717a;
}

.pill-group {
  display: flex;
  gap: 8px;
}

.pill-btn {
  background: #1a1a20;
  border: 1px solid #2d2d38;
  color: #d4d4d8;
  font-size: 0.75rem;
  padding: 6px 14px;
  border-radius: 8px;
  cursor: pointer;
  font-weight: 600;
}

.copy-btn {
  background: #ffffff;
  color: #000000;
  border: none;
  padding: 6px 16px;
}

.field-textarea {
  width: 100%;
  background: #18181c;
  border: 1px solid #2c2c33;
  border-radius: 12px;
  padding: 14px;
  font-size: 0.95rem;
  line-height: 1.5;
  color: #ffffff;
  outline: none;
  resize: none;
  box-sizing: border-box;
}

.result-display {
  font-family: monospace;
  font-size: 1.05rem;
  color: #ffffff;
  line-height: 1.5;
  word-break: break-all;
  min-height: 68px;
  background: #09090b;
  border: 1px dashed #2f2f38;
  padding: 14px;
  border-radius: 12px;
}

/* Inspector Tab */
.tab-sub {
  font-size: 0.85rem;
  color: #a1a1aa;
  margin-bottom: 16px;
}

.step-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 12px;
}

@media (min-width: 600px) {
  .step-grid {
    grid-template-columns: repeat(8, 1fr);
  }
}

.step-cell {
  background: #16161a;
  border: 1px solid #27272f;
  border-radius: 12px;
  padding: 10px 4px;
  text-align: center;
}

.char-original {
  font-family: monospace;
  font-weight: 700;
  font-size: 1rem;
  color: #71717a;
}

.step-icon {
  font-size: 0.75rem;
  color: #52525b;
  margin: 6px 0;
}

.char-transformed {
  font-family: monospace;
  font-weight: 800;
  font-size: 1.1rem;
  color: #ffffff;
}

.char-shift {
  display: block;
  font-size: 0.65rem;
  color: #a1a1aa;
  margin-top: 6px;
}

.empty-state {
  text-align: center;
  padding: 40px 16px;
  color: #71717a;
}

.empty-icon {
  font-size: 2.4rem;
  margin-bottom: 12px;
  color: #27272a;
}

/* Guide Styles */
.info-block {
  margin-bottom: 18px;
  padding-bottom: 16px;
  border-bottom: 1px solid #202025;
}

.info-block.no-border {
  border-bottom: none;
  margin-bottom: 0;
  padding-bottom: 0;
}

.info-header {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 6px;
}

.info-badge {
  font-size: 0.65rem;
  text-transform: uppercase;
  background: #1a1a20;
  border: 1px solid #2c2c36;
  color: #a1a1aa;
  padding: 2px 6px;
  border-radius: 4px;
  font-weight: 700;
}

.info-name {
  font-weight: 700;
  font-size: 0.95rem;
  color: #ffffff;
}

.info-desc {
  font-size: 0.88rem;
  line-height: 1.5;
  color: #a1a1aa;
}

.info-example {
  margin-top: 8px;
}

.info-example code {
  font-family: monospace;
  font-size: 0.8rem;
  background: #09090c;
  border: 1px solid #1f1f26;
  padding: 4px 8px;
  border-radius: 6px;
  color: #e4e4e7;
}

/* Picker Action Sheet Modal */
.ios-backdrop {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.75);
  backdrop-filter: blur(8px);
  -webkit-backdrop-filter: blur(8px);
  z-index: 1000;
  display: flex;
  align-items: flex-end;
  justify-content: center;
  padding: 16px;
}

@media (min-width: 768px) {
  .ios-backdrop {
    align-items: center;
  }
}

.ios-sheet {
  width: 100%;
  max-width: 440px;
  background: #1a1a1e;
  border-radius: 18px;
  overflow: hidden;
  border: 1px solid #2c2c33;
}

.ios-sheet-header {
  padding: 14px;
  text-align: center;
  font-size: 0.78rem;
  color: #8e8e93;
  text-transform: uppercase;
  font-weight: 600;
  border-bottom: 1px solid #2c2c33;
}

.ios-sheet-btn {
  width: 100%;
  height: 52px;
  background: transparent;
  border: none;
  border-bottom: 1px solid #26262d;
  color: #ffffff;
  font-size: 0.95rem;
  font-weight: 600;
  padding: 0 20px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  cursor: pointer;
}

.btn-left {
  display: flex;
  align-items: center;
  gap: 12px;
}

.btn-left i {
  color: #71717a;
  width: 18px;
}

.ios-sheet-btn.selected {
  color: #ffffff;
  background: #24242b;
}

.ios-sheet-btn.cancel {
  border-bottom: none;
  color: #ef4444;
  justify-content: center;
  font-weight: 700;
}

/* Toast Pop */
.paw-toast-modal {
  position: fixed;
  bottom: 84px;
  left: 50%;
  transform: translateX(-50%);
  background: #ffffff;
  color: #000000;
  padding: 10px 22px;
  border-radius: 30px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.7);
  display: flex;
  align-items: center;
  gap: 10px;
  font-weight: 700;
  font-size: 0.88rem;
  z-index: 1001;
}

.toast-cat {
  font-size: 1.1rem;
}

/* 3. Footer Navigation Bar */
.paw-footer-bar {
  background: #0a0a0c;
  border-top: 1px solid rgba(255, 255, 255, 0.08);
  flex-shrink: 0;
  padding-bottom: calc(env(safe-area-inset-bottom, 12px) + 6px);
}

@media screen and (min-width: 768px) {
  .paw-footer-bar {
    padding-bottom: 14px;
  }
}

.footer-inner {
  max-width: 720px;
  margin: 0 auto;
  display: flex;
  padding: 6px 14px 0;
}

.nav-item {
  flex: 1;
  height: 46px;
  background: transparent;
  border: none;
  color: #71717a;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 4px;
  font-size: 0.72rem;
  font-weight: 600;
  cursor: pointer;
  transition: color 0.2s ease;
}

.nav-item i {
  font-size: 1.25rem;
}

.nav-item.active {
  color: #ffffff;
}

/* Transitions */
.fade-enter-active, .fade-leave-active {
  transition: opacity 0.5s ease;
}
.fade-enter-from, .fade-leave-to {
  opacity: 0;
}

.toast-pop-enter-active, .toast-pop-leave-active {
  transition: all 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
}
.toast-pop-enter-from, .toast-pop-leave-to {
  opacity: 0;
  transform: translate(-50%, 20px) scale(0.9);
}
</style>