<template>
  <ion-page class="cyberpaw-page">
    <!-- Header -->
    <ion-header :translucent="false" class="ion-no-border">
      <ion-toolbar class="paw-toolbar">
        <ion-title>
          <div class="paw-brand">
            <i class="fa-solid fa-cat brand-cat"></i>
            <span class="brand-text">CYBERPAW</span>
            <span class="brand-badge">v1.0</span>
          </div>
        </ion-title>
      </ion-toolbar>

      <!-- Segment Selector -->
      <div class="mode-wrapper" v-if="activeTab === 'console'">
        <div class="mode-capsule">
          <button 
            type="button" 
            :class="['mode-tab', mode === 'encrypt' ? 'active' : '']" 
            @click="mode = 'encrypt'"
          >
            <i class="fa-solid fa-lock"></i> Encrypt
          </button>
          <button 
            type="button" 
            :class="['mode-tab', mode === 'decrypt' ? 'active' : '']" 
            @click="mode = 'decrypt'"
          >
            <i class="fa-solid fa-lock-open"></i> Decrypt
          </button>
        </div>
      </div>
    </ion-header>

    <ion-content class="ion-padding paw-content">
      <!-- TAB 1: CONSOLE -->
      <div v-show="activeTab === 'console'" class="tab-pane">
        <div class="paw-card">
          <div class="card-head">
            <span class="card-tag"><i class="fa-solid fa-sliders"></i> Parameters</span>
          </div>

          <div class="form-row">
            <div class="field-item">
              <label class="field-title">Cipher Type</label>
              <select v-model="cipherType" class="field-select">
                <option value="caesar">Caesar Shift</option>
                <option value="vigenere">Vigenère Cipher</option>
              </select>
            </div>

            <div class="field-item">
              <label class="field-title">{{ cipherType === 'caesar' ? 'Numeric Shift' : 'Secret Keyword' }}</label>
              <div v-if="cipherType === 'caesar'" class="stepper-box">
                <button type="button" class="step-btn" @click="shiftKey = Math.max(1, shiftKey - 1)">-</button>
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

        <!-- Input Field -->
        <div class="paw-card">
          <div class="card-head">
            <span class="card-tag">
              <i class="fa-solid fa-keyboard"></i> {{ mode === 'encrypt' ? 'Plaintext' : 'Ciphertext' }}
            </span>
            <div class="pill-group">
              <button type="button" class="pill-btn" @click="inputText = 'STEALTH CAT'">Preset</button>
              <button type="button" class="pill-btn" @click="inputText = ''">Clear</button>
            </div>
          </div>
          <textarea 
            v-model="inputText" 
            class="field-textarea" 
            rows="3" 
            :placeholder="mode === 'encrypt' ? 'Type message to encrypt...' : 'Paste scrambled message...'"
          ></textarea>
        </div>

        <!-- Output Field -->
        <div class="paw-card output-accent">
          <div class="card-head">
            <span class="card-tag tag-white">
              <i class="fa-solid fa-shield-cat"></i> Output
            </span>
            <button type="button" class="pill-btn light" @click="copyResult" :disabled="!resultText">
              <i class="fa-regular fa-copy"></i> Copy
            </button>
          </div>
          <div class="result-display">
            {{ resultText || 'Waiting for input...' }}
          </div>
        </div>
      </div>

      <!-- TAB 2: INSPECTOR -->
      <div v-show="activeTab === 'inspector'" class="tab-pane">
        <div class="paw-card">
          <div class="card-head">
            <span class="card-tag"><i class="fa-solid fa-paw"></i> Letter Conversions</span>
          </div>
          <p class="tab-sub">Inspect how each letter was transformed:</p>

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
            <p>Type text in the Console to view live transformations.</p>
          </div>
        </div>
      </div>

      <!-- TAB 3: GUIDE -->
      <div v-show="activeTab === 'guide'" class="tab-pane">
        <div class="paw-card">
          <div class="card-head">
            <span class="card-tag"><i class="fa-solid fa-circle-info"></i> How It Works</span>
          </div>

          <div class="info-block">
            <div class="info-title"><i class="fa-solid fa-paw"></i> Caesar Shift</div>
            <p class="info-desc">
              Shifts each letter down the alphabet by a chosen numerical key value. Wraps around automatically from Z to A.
            </p>
          </div>

          <div class="info-block">
            <div class="info-title"><i class="fa-solid fa-shield-cat"></i> Vigenère Cipher</div>
            <p class="info-desc">
              Uses an alphabetic keyword to apply dynamic, alternating Caesar shifts for each consecutive character.
            </p>
          </div>
        </div>
      </div>
    </ion-content>

    <!-- Footer Navigation -->
    <ion-footer class="ion-no-border">
      <div class="paw-nav-bar">
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
    </ion-footer>
  </ion-page>
</template>

<script setup>
import { ref, computed } from 'vue';
import { 
  IonPage, 
  IonHeader, 
  IonToolbar, 
  IonTitle, 
  IonContent, 
  IonFooter 
} from '@ionic/vue';

const activeTab = ref('console');
const mode = ref('encrypt');
const cipherType = ref('caesar');
const shiftKey = ref(3);
const vigenereKey = ref('PAW');
const inputText = ref('');

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
        label = `${isEncrypt ? '+' : '-'}${shiftKey.value}`;
      } else {
        const keyChar = cleanKey[vIndex % cleanKey.length];
        const s = keyChar.charCodeAt(0) - 65;
        label = `${keyChar} (${isEncrypt ? '+' : '-'}${s})`;
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
    alert('Copied to clipboard!');
  }
};
</script>

<style scoped>
.cyberpaw-page {
  --background: #000000;
  background: #000000;
  color: #ffffff;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
}

.paw-toolbar {
  --background: #050505;
  --border-width: 0 0 1px 0;
  --border-color: #222222;
  padding: 6px 4px;
}

.paw-brand {
  display: flex;
  align-items: center;
  gap: 8px;
}

.brand-cat {
  font-size: 1.15rem;
  color: #ffffff;
}

.brand-text {
  font-family: monospace;
  font-weight: 700;
  font-size: 1rem;
  letter-spacing: 2px;
}

.brand-badge {
  font-size: 0.65rem;
  background: #1a1a1a;
  color: #888888;
  border: 1px solid #333333;
  padding: 2px 6px;
  border-radius: 4px;
}

.mode-wrapper {
  background: #050505;
  padding: 8px 16px 12px;
  border-bottom: 1px solid #1a1a1a;
}

.mode-capsule {
  display: flex;
  background: #111111;
  border: 1px solid #2a2a2a;
  border-radius: 8px;
  padding: 3px;
}

.mode-tab {
  flex: 1;
  padding: 8px;
  background: transparent;
  color: #777777;
  border: none;
  font-size: 0.8rem;
  font-weight: 700;
  border-radius: 6px;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
  cursor: pointer;
  transition: 0.2s;
}

.mode-tab.active {
  background: #ffffff;
  color: #000000;
}

.paw-content {
  --background: #000000;
}

.paw-card {
  background: #0b0b0b;
  border: 1px solid #1c1c1c;
  border-radius: 12px;
  padding: 14px;
  margin-bottom: 14px;
}

.card-head {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}

.card-tag {
  font-family: monospace;
  font-size: 0.72rem;
  text-transform: uppercase;
  color: #888888;
  letter-spacing: 1px;
  display: flex;
  align-items: center;
  gap: 6px;
}

.tag-white {
  color: #ffffff;
}

.form-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px;
}

.field-title {
  display: block;
  font-size: 0.7rem;
  text-transform: uppercase;
  color: #666666;
  margin-bottom: 6px;
  font-family: monospace;
}

.field-select, .field-input, .field-textarea {
  width: 100%;
  background: #141414;
  border: 1px solid #2a2a2a;
  color: #ffffff;
  border-radius: 8px;
  padding: 9px;
  font-size: 0.85rem;
  outline: none;
  box-sizing: border-box;
}

.field-input.uppercase {
  text-transform: uppercase;
}

.stepper-box {
  display: flex;
  background: #141414;
  border: 1px solid #2a2a2a;
  border-radius: 8px;
  overflow: hidden;
}

.step-btn {
  background: transparent;
  color: #ffffff;
  border: none;
  padding: 8px 12px;
  cursor: pointer;
  font-weight: 700;
}

.stepper-field {
  width: 100%;
  text-align: center;
  background: transparent;
  border: none;
  color: #ffffff;
  font-family: monospace;
}

.pill-group {
  display: flex;
  gap: 6px;
}

.pill-btn {
  background: #1a1a1a;
  border: 1px solid #333333;
  color: #aaaaaa;
  font-size: 0.65rem;
  padding: 4px 8px;
  border-radius: 4px;
  cursor: pointer;
}

.pill-btn.light {
  background: #ffffff;
  color: #000000;
  border: none;
  font-weight: 700;
}

.field-textarea {
  resize: none;
}

.result-display {
  font-family: monospace;
  font-size: 0.95rem;
  color: #ffffff;
  word-break: break-all;
  min-height: 48px;
  background: #050505;
  border: 1px dashed #2a2a2a;
  padding: 12px;
  border-radius: 8px;
}

.tab-sub {
  font-size: 0.8rem;
  color: #777777;
  margin-bottom: 12px;
}

.step-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 8px;
}

.step-cell {
  background: #121212;
  border: 1px solid #222222;
  border-radius: 8px;
  padding: 8px;
  text-align: center;
}

.char-original {
  font-family: monospace;
  font-weight: 700;
  color: #777777;
}

.step-icon {
  font-size: 0.65rem;
  color: #444444;
  margin: 4px 0;
}

.char-transformed {
  font-family: monospace;
  font-weight: 700;
  color: #ffffff;
}

.char-shift {
  display: block;
  font-size: 0.6rem;
  color: #888888;
  margin-top: 4px;
}

.empty-state {
  text-align: center;
  padding: 24px;
  color: #666666;
}

.empty-icon {
  font-size: 2rem;
  margin-bottom: 8px;
  color: #333333;
}

.info-block {
  margin-bottom: 16px;
  padding-bottom: 12px;
  border-bottom: 1px solid #1a1a1a;
}

.info-title {
  font-weight: 700;
  font-size: 0.88rem;
  margin-bottom: 4px;
  display: flex;
  align-items: center;
  gap: 6px;
}

.info-desc {
  font-size: 0.8rem;
  line-height: 1.4;
  color: #999999;
}

.paw-nav-bar {
  display: flex;
  background: #050505;
  border-top: 1px solid #1a1a1a;
  padding: 6px 12px 14px;
}

.nav-item {
  flex: 1;
  background: transparent;
  border: none;
  color: #555555;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
  font-size: 0.65rem;
  cursor: pointer;
}

.nav-item i {
  font-size: 1.1rem;
}

.nav-item.active {
  color: #ffffff;
}
</style>