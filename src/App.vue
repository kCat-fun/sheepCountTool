<script setup lang="ts">
import { ref } from 'vue';
import Swal from 'sweetalert2';

import _errorSound from '~/assets/sounds/error-sound.mp3'
import _countUpSound from '~/assets/sounds/count-up-sound.mp3'

const isCounting = ref(false);
const sheepCount = ref(0);

const errorSound = new Audio(_errorSound);
const countUpSound = new Audio(_countUpSound);

// 初期化
const SpeechRecognition = (window as any).SpeechRecognition || (window as any).webkitSpeechRecognition;
const recognition = new SpeechRecognition();
// 言語設定を日本語に
recognition.lang = 'ja-JP';
// 解析中の結果を表示する
recognition.interimResults = false;
// 認識のたびに継続的に結果を返す
recognition.continuous = true;

const result = ref('');

let time: number = 0;

const countRecordingSheepCount = () => {
  if (isCounting.value) {
    return;
  }

  isCounting.value = true;

  try {
    // 音声認識開始
    recognition.start();

    recognition.onresult = (event: any) => {
      const results = event.results;
      for (let i = event.resultIndex; i < results.length; i++) {
        const transcript = results[i][0].transcript;
        if (transcript != '') {
          result.value = transcript;
          console.log('認識結果:', transcript);
        }
        if ((transcript.includes('羊が') || transcript.includes('ひつじが')) && transcript.includes('匹')) {
          const match = transcript.match(/\d+/);
          if (match) {
            const number = parseInt(match[0], 10);
            if (new Date().getTime() - time < 2000 && number !== sheepCount.value + 1) {
              break;
            }
            time = new Date().getTime();
            if (number == sheepCount.value + 1) {
              sheepCount.value++;
              countUpSound.play();
              break;
            } else {
              errorSound.play();
              Swal.fire({
                icon: 'error',
                title: '数え間違いです',
                text: `${number}匹目ではなく、${sheepCount.value + 1}匹目です。`,
              });
            }
          }
        }
      }
    };

    recognition.onend = () => {
      if (isCounting.value) {
        // 音声認識を継続
        recognition.start();
      } else {
        console.log('音声認識が終了しました。');
      }
    };

    recognition.onerror = (error: any) => {
      console.error('音声認識エラー:', error);
      if (isCounting.value) {
        // 再試行
        recognition.start();
      } else {
        Swal.fire({
          icon: 'warning',
          title: 'エラー',
          text: '音声認識が停止しました。',
        });
      }
    };
  } catch (error) {
    recognition.stop();
    console.error(error);
    Swal.fire({
      icon: 'error',
      title: 'エラー',
      text: String(error),
    });
  }
};

const resetRecordingSheepCount = () => {
  isCounting.value = false; // 停止フラグをセット
  recognition.stop(); // 音声認識を停止
  sheepCount.value = 0;
  result.value = '';
};
</script>

<template>
  <div>
    <div class="text-center">
      <h1>睡眠サポート用羊カウンター</h1>
      <p>快適な睡眠を実現するために<br />羊を数える際のサポートをします</p>
    </div>
    <div class="button-group">
      <button @click="countRecordingSheepCount">スタート</button>
      <button @click="resetRecordingSheepCount">リセット</button>
    </div>
    <div class="sheep-counter">
      <p id="status">
        Status: {{ isCounting ? 'Counting' : 'Stopped' }}
      </p>
      <p>羊の数: <span class="sheepCount">{{ sheepCount }}</span>匹</p>
      <p>認識結果: {{ result }}</p>
    </div>
  </div>
</template>

<style scoped>
h1 {
  font-size: 26px;
  margin-bottom: 10px;
}

.button-group {
  display: flex;
  justify-content: center;
  gap: 20px;
}

.sheep-counter {
  margin-top: 20px;
  font-size: 18px;
  text-align: center;
}

.sheepCounter {
  font-weight: bold;
}
</style>
