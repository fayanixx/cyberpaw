<template>
  <ion-page class="cyberpaw-page">
    <!-- Safe Area Header -->
    <header class="paw-safe-header">
      <div class="paw-navbar">
        <div class="paw-brand">
          <div class="brand-avatar">
            <i class="fa-solid fa-cat brand-cat"></i>
          </div>
          <div class="brand-titles">
            <span class="brand-text">CYBERPAW</span>
            <span class="brand-subtitle">iOS Security Terminal</span>
          </div>
        </div>
        <span class="brand-badge">v1.0</span>
      </div>

      <!-- Segment Selector (Encrypt / Decrypt) -->
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
    </header>

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
              <!-- Custom Styled Popover Trigger for Cipher Type -->
              <div class="field-item">
                <label class="field-title">Cipher Type</label>
                <div class="custom-select-trigger" @click="showCipherPicker = !showCipherPicker">
                  <span>{{ cipherType === 'caesar' ? 'Caesar Shift' : 'Vigenère Cipher' }}</span>
                  <i class="fa-solid fa-chevron-down caret-icon"></i>
                </div>
              </div>

              <!-- Key Input -->
              <div class="field-item">
                <label class="field-title">{{ cipherType === 'caesar' ? 'Numeric Shift' : 'Secret Key' }}</label>
                <div v-if="cipherType === 'caesar'" class="stepper-box">
                  <button type="button" class="step-btn" @click="shiftKey = Math.max(1, shiftKey - 1)">−</button>
                  <input type="number" v-model.number="shiftKey" min="1" max="25" class="stepper-field" />
                  <button type="button" class="step-btn" @click="shiftKey = Math.min(25, shiftKey + 1)">+</button>
                </div>
                <input 
                  v-else 
                  type="text" 
                  v-model="vigenereKey" 
                  placeholder="Keyword..." 
                  class="field-input uppercase"
                />
              </div>
            </div>
          </div>

          <!-- Input Card -->
          <div class="paw-card">
            <div class="card-head">
              <span class="card-tag">
                <i class="fa-solid fa-keyboard"></i> {{ mode === 'encrypt' ? 'Plaintext' : 'Ciphertext' }}
              </span>
              <div class="pill-group">
                <button type="button" class="pill-btn" @click="inputText = 'STEALTH CAT'">Stealth</button>
                <button type="button" class="pill-btn" @click="inputText = ''">Clear</button>
              </div>
            </div>
            <textarea 
              v-model="inputText" 
              class="field-textarea" 
              rows="4" 
              :placeholder="mode === 'encrypt' ? 'Type message to encrypt...' : 'Paste message to decrypt...'"
            ></textarea>
          </div>

          <!-- Output Card -->
          <div class="paw-card output-card">
            <div class="card-head">
              <span class="card-tag tag-white">
                <i class="fa-solid fa-shield-cat"></i> Output
              </span>
              <button type="button" class="pill-btn copy-btn" @click="copyResult" :disabled="!resultText">
                <i class="fa-regular fa-copy"></i> Copy
              </button>
            </div>
            <div class="result-display">
              {{ resultText || 'Result will appear here...' }}
            </div>
          </div>
        </div>

        <!-- TAB 2: INSPECTOR -->
        <div v-show="activeTab === 'inspector'" class="tab-pane">
          <div class="paw-card">
            <div class="card-head">
              <span class="card-tag"><i class="fa-solid fa-paw"></i> Letter Transformations</span>
            </div>
            <p class="tab-sub">Live cryptographic character mapping:</p>

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
              <p>Type characters in Console to inspect transformations.</p>
            </div>
          </div>
        </div>

        <!-- TAB 3: GUIDE -->
        <div v-show="activeTab === 'guide'" class="tab-pane">
          <div class="paw-card">
            <div class="card-head">
              <span class="card-tag"><i class="fa-solid fa-circle-info"></i> Cipher Mechanics</span>
            </div>

            <div class="info-block">
              <div class="info-title"><i class="fa-solid fa-paw"></i> Caesar Shift</div>
              <p class="info-desc">
                Substitutes each alphabet letter with one shifted $k$ places ahead. Loops automatically back from Z to A.
              </p>
            </div>

            <div class="info-block">
              <div class="info-title"><i class="fa-solid fa-shield-cat"></i> Vigenère Cipher</div>
              <p class="info-desc">
                Polyalphabetic substitution that repeats a secret keyword across the plaintext, shifting each character by its corresponding key offset.
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
            type="button" 
            :class="['ios-sheet-btn', cipherType === 'caesar' ? 'selected' : '']" 
            @click="cipherType = 'caesar'; showCipherPicker = false"
          >
            <span>Caesar Shift</span>
            <i v-if="cipherType === 'caesar'" class="fa-solid fa-check"></i>
          </button>
          <button 
            type="button" 
            :class="['ios-sheet-btn', cipherType === 'vigenere' ? 'selected' : '']" 
            @click="cipherType = 'vigenere'; showCipherPicker = false"
          >
            <span>Vigenère Cipher</span>
            <i v-if="cipherType === 'vigenere'" class="fa-solid fa-check"></i>
          </button>
          <button type="button" class="ios-sheet-btn cancel" @click="showCipherPicker = false">Cancel</button>
        </div>
      </div>

      <!-- Custom Themed Copy Toast Modal -->
      <div v-if="showToast" class="paw-toast-modal">
        <i class="fa-solid fa-paw toast-paw"></i>
        <span>Copied to Clipboard!</span>
      </div>
    </ion-content>

    <!-- Bottom Tab Navigation Bar -->
    <footer class="paw-footer-bar">
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
    </footer>
  </ion-page>
</template>

<script setup>
import { ref, computed } from 'vue';
import { IonPage, IonContent } from '@ionic/vue';

const activeTab = ref('console');
const mode = ref('encrypt');
const cipherType = ref('caesar');
const shiftKey = ref(3);
const vigenereKey = ref('PAW');
const inputText = ref('');
const showCipherPicker = ref(false);
const showToast = ref(false);

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

const resultText = computed(() => {
  if (!inputText.value) return '';
  const isEncrypt = mode.value === 'encrypt';

  if (cipherType.value === 'caesar') {
    return runCaesar(inputText.value, shiftKey.value, isEncrypt);
  } else {
    return runVigenere(inputText.value, vigenereKey.value, isEncrypt);
  }
});

const transformations = computed(() => {
  if (!inputText.value) return [];
  const textChars = inputText.value.slice(0, 16).split('');
  const outputChars = (resultText.value || '').slice(0, 16).split('');
  const isEncrypt = mode.value === 'encrypt';

  const cleanKey = vigenereKey.value.toUpperCase().replace(/[^A-Z]/g, '') || 'A';
  let vIndex = 0;

  return textChars.map((char, i) => {
    let label = 'Keep';
    const isLetter = /[a-zA-Z]/.test(char);

    if (isLetter) {
      if (cipherType.value === 'caesar') {
        label = `${isEncrypt ? '+' : '−'}${shiftKey.value}`;
      } else {
        const keyChar = cleanKey[vIndex % cleanKey.length];
        const s = keyChar.charCodeAt(0) - 65;
        label = `${keyChar} (${isEncrypt ? '+' : '−'}${s})`;
        vIndex++;
      }
    }

    return {
      orig: char === ' ' ? '␣' : char,
      transformed: outputChars[i] === ' ' ? '␣' : outputChars[i],
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
    }, 2200);
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

/* Safe Area Inset to stop top clipping */
.paw-safe-header {
  padding-top: max(env(safe-area-inset-top, 0px), 38px);
  background: #0a0a0a;
  border-bottom: 1px solid rgba(255, 255, 255, 0.08);
  flex-shrink: 0;
}

.paw-navbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 18px 8px;
}

.paw-brand {
  display: flex;
  align-items: center;
  gap: 12px;
}

.brand-avatar {
  width: 36px;
  height: 36px;
  border-radius: 10px;
  background: #141414;
  border: 1px solid #2a2a2a;
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
  letter-spacing: 0.5px;
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

/* iOS Style Segmented Control */
.mode-wrapper {
  padding: 8px 16px 14px;
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
  height: 38px;
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

/* Content Area */
.paw-content {
  --background: #000000;
  flex: 1;
}

.content-container {
  max-width: 580px;
  margin: 0 auto;
  padding: 16px;
}

/* Cards (iOS Apple-like Cards) */
.paw-card {
  background: #111113;
  border: 1px solid #222225;
  border-radius: 16px;
  padding: 16px;
  margin-bottom: 16px;
}

.output-card {
  border-color: #2e2e34;
  background: #141416;
}

.card-head {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
}

.card-tag {
  font-family: monospace;
  font-size: 0.78rem;
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
  gap: 12px;
}

.field-title {
  display: block;
  font-size: 0.75rem;
  text-transform: uppercase;
  color: #71717a;
  margin-bottom: 8px;
  font-weight: 600;
}

/* Custom Select Dropdown Trigger */
.custom-select-trigger {
  height: 46px;
  background: #1a1a1d;
  border: 1px solid #2c2c31;
  border-radius: 12px;
  padding: 0 14px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  font-size: 0.92rem;
  color: #ffffff;
  cursor: pointer;
  user-select: none;
}

.caret-icon {
  font-size: 0.75rem;
  color: #71717a;
}

.field-input {
  width: 100%;
  height: 46px;
  background: #1a1a1d;
  border: 1px solid #2c2c31;
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
  height: 46px;
  background: #1a1a1d;
  border: 1px solid #2c2c31;
  border-radius: 12px;
  overflow: hidden;
}

.step-btn {
  width: 44px;
  background: transparent;
  color: #ffffff;
  border: none;
  font-size: 1.25rem;
  cursor: pointer;
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

.pill-group {
  display: flex;
  gap: 8px;
}

.pill-btn {
  background: #1e1e24;
  border: 1px solid #2e2e38;
  color: #d4d4d8;
  font-size: 0.75rem;
  padding: 6px 12px;
  border-radius: 8px;
  cursor: pointer;
  font-weight: 600;
}

.copy-btn {
  background: #ffffff;
  color: #000000;
  border: none;
  padding: 6px 14px;
}

.field-textarea {
  width: 100%;
  background: #1a1a1d;
  border: 1px solid #2c2c31;
  border-radius: 12px;
  padding: 12px 14px;
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
  min-height: 64px;
  background: #09090b;
  border: 1px dashed #2f2f35;
  padding: 14px;
  border-radius: 12px;
}

/* Inspector Grid */
.tab-sub {
  font-size: 0.85rem;
  color: #a1a1aa;
  margin-bottom: 14px;
}

.step-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 10px;
}

.step-cell {
  background: #18181b;
  border: 1px solid #27272a;
  border-radius: 12px;
  padding: 10px 6px;
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
  padding: 36px 16px;
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
  padding-bottom: 14px;
  border-bottom: 1px solid #222225;
}

.info-title {
  font-weight: 700;
  font-size: 0.95rem;
  margin-bottom: 6px;
  display: flex;
  align-items: center;
  gap: 8px;
}

.info-desc {
  font-size: 0.88rem;
  line-height: 1.5;
  color: #a1a1aa;
}

/* iOS Bottom Action Sheet for Picker */
.ios-backdrop {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.7);
  backdrop-filter: blur(8px);
  -webkit-backdrop-filter: blur(8px);
  z-index: 999;
  display: flex;
  align-items: flex-end;
  justify-content: center;
  padding: 16px;
}

.ios-sheet {
  width: 100%;
  max-width: 480px;
  background: #1c1c1e;
  border-radius: 16px;
  overflow: hidden;
  border: 1px solid #2c2c2e;
}

.ios-sheet-header {
  padding: 14px;
  text-align: center;
  font-size: 0.8rem;
  color: #8e8e93;
  text-transform: uppercase;
  font-weight: 600;
  border-bottom: 1px solid #2c2c2e;
}

.ios-sheet-btn {
  width: 100%;
  height: 52px;
  background: transparent;
  border: none;
  border-bottom: 1px solid #2c2c2e;
  color: #ffffff;
  font-size: 1rem;
  font-weight: 600;
  padding: 0 20px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  cursor: pointer;
}

.ios-sheet-btn.selected {
  color: #38bdf8;
}

.ios-sheet-btn.cancel {
  border-bottom: none;
  color: #ef4444;
  justify-content: center;
  font-weight: 700;
}

/* Themed Toast Modal */
.paw-toast-modal {
  position: fixed;
  bottom: 84px;
  left: 50%;
  transform: translateX(-50%);
  background: #ffffff;
  color: #000000;
  padding: 10px 20px;
  border-radius: 30px;
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.6);
  display: flex;
  align-items: center;
  gap: 10px;
  font-weight: 700;
  font-size: 0.88rem;
  z-index: 1000;
}

.toast-paw {
  font-size: 1rem;
}

/* iOS Footer Navigation */
.paw-footer-bar {
  display: flex;
  background: #0a0a0a;
  border-top: 1px solid rgba(255, 255, 255, 0.08);
  padding: 8px 12px max(env(safe-area-inset-bottom, 0px), 16px);
  flex-shrink: 0;
}

.nav-item {
  flex: 1;
  height: 48px;
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
</style>