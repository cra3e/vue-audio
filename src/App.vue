<template>
  <div class="app">

    <div class="logo-block">
      <img src="./assets/VUEaudio.png" height="50px"/>
    </div>

    <div class="button-container">
      <div v-for="(button, index) in buttons" :key="button" class="button-wrapper">
        <div><button
            @click="playSound(button)"
            :class="{ 'sound-button': true, active: isActiveButton[index] }"
        >
          {{ button }}
        </button>
        </div>
        <div>
        <button @click="toggleRepeat(index)" :class="{ 'repeat-button': !isRepeatingSound[index], 'repeat-button-active': isRepeatingSound[index] }">
          <svg xmlns="http://www.w3.org/2000/svg" xml:space="preserve" width="26" height="24" style="shape-rendering:geometricPrecision;text-rendering:geometricPrecision;image-rendering:optimizeQuality;fill-rule:evenodd;clip-rule:evenodd" viewBox="0 0 1.26 0.89"><path d="M.19.15h.91v.23h-.09V.25H.19l.08.08L.2.4 0 .2.2 0l.07.07-.08.08zm.88.59H.15V.51h.1v.14h.82L.99.56 1.06.5l.2.19-.2.2L.99.82l.08-.08z" style="fill:#8a8a8a;fill-rule:nonzero"/></svg></button></div>
      </div>
    </div>

    <div class="input-block">

      <select v-model="selectedButton" class="select-button">
        <option value="" disabled>PAD</option>
        <option v-for="button in buttons" :key="button" :value="button">{{ button }}</option>
      </select>

      <input type="file" @change="handleFileUpload" accept="audio/*" class="input-file" :disabled="!selectedButton" ref="fileInput"/>
      <label for="fileInput">
        <button @click="$refs.fileInput.click()" :disabled="!selectedButton" class="file-input-button">{{!selectedButton ? 'CHOOSE PAD' : 'UPLOAD SOUND'}}</button>
      </label>

    </div>

    <div class="file-list-box">
    <ul class="file-list">
      <li v-for="(file, index) in fileList" :key="index">
        {{ index + 1 }}: {{ file ? file.name : '-' }}
      </li>
    </ul>
    </div>
  </div>

</template>

<script>
export default {
  data() {
    return {
      buttons: [1, 2, 3, 4, 5, 6, 7, 8],
      selectedButton: "",
      soundFiles: {},
      selectedFile: "",
      fileList: Array(8).fill(null),
      isRepeatingSound: Array(8).fill(false),
      repeatIntervals: Array(8).fill(null),
      isActiveButton: Array(8).fill(false) // Добавлен массив флагов активности кнопок
    };
  },
  methods: {
    handleFileUpload(event) {
      const file = event.target.files[0];
      const buttonIndex = this.buttons.indexOf(this.selectedButton);

      if (buttonIndex !== -1 && file) {
        this.fileList.splice(buttonIndex, 1, file);
        const reader = new FileReader();
        reader.onload = () => {
          this.$set(this.soundFiles, this.selectedButton, reader.result);
        };
        reader.readAsDataURL(file);
      }
    },

    playSound(button) {
      const sound = new Audio(this.soundFiles[button]);
      sound.addEventListener('ended', () => {
        this.isActiveButton.splice(button - 1, 1, false); // Устанавливаем флаг активности кнопки в false после окончания проигрывания
      });
      sound.play();
      this.isActiveButton.splice(button - 1, 1, true); // Устанавливаем флаг активности кнопки в true
      this.selectedButton = button; // Устанавливаем выбранную кнопку в состояние активности
    },

    toggleRepeat(index) {
      this.isRepeatingSound.splice(index, 1, !this.isRepeatingSound[index]);
      if (this.isRepeatingSound[index]) {
        const sound = new Audio(this.soundFiles[this.buttons[index]]);
        sound.addEventListener('loadedmetadata', () => {
          const duration = sound.duration * 1000; // Длительность аудиофайла в миллисекундах
          this.repeatIntervals.splice(index, 1, setInterval(() => {
            this.playSound(this.buttons[index]);
          }, duration));
        });
      } else {
        clearInterval(this.repeatIntervals[index]);
        this.repeatIntervals.splice(index, 1, null);
      }
    },

    handleKeyPress(event) {
      const key = event.key;
      if (this.buttons.includes(parseInt(key))) {
        this.playSound(parseInt(key));
      }
    },
  },
  mounted() {
    window.addEventListener("keydown", this.handleKeyPress);
  },
  beforeDestroy() {
    window.removeEventListener("keydown", this.handleKeyPress);
  },
};
</script>

<style>
.app {
  display: flex;
  align-items: center;
  flex-direction: column;
  background-color: #333;
  padding: 20px;
  font-family: "helvetica", sans-serif;
}

.input-block {
  width: 440px;
  display: flex;
  flex-wrap: wrap;
}

.logo-block {
  width: 440px;
  padding: 20px;
  align-items: center;
  flex-direction: column;
  color: #fff;
  display: flex;
  flex-wrap: wrap;
}

.input-file {
  background-color: #212121;
  border-radius: 4px;
  color: #8a8a8a;
  width: 316px;
  height: 46px;
  display: flex;
  flex-wrap: wrap;
  border: 2px solid #fce979;
  margin-left: 5px;
  margin-right: 5px;
  margin-top: 10px;
  margin-bottom: 10px;
  font-size: 16px;
  cursor: pointer;
}

.input-file {
  display: none;
}

.file-input-button {
  background-color: #212121;
  border-radius: 4px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  color: #8a8a8a;
  width: 320px;
  height: 50px;
  flex-wrap: wrap;
  border: 2px solid #fce979;
  margin-left: 5px;
  margin-right: 5px;
  margin-top: 10px;
  margin-bottom: 10px;
  font-size: 16px;
  cursor: pointer;
}

.file-input-button:active {
  box-shadow: 0px 0px 5px 5px rgba(255, 255, 0, .2);
}

.file-input-button:disabled {
  border: 2px solid #f00;
  box-shadow: 0px 0px 5px 5px rgba(255, 0, 0, .2);
}

.select-button {
  background-color: #212121;
  border-radius: 4px;
  color: #8a8a8a;
  width: 100px;
  height: 50px;
  border: 2px solid #fce979;
  margin-left: 5px;
  margin-right: 5px;
  margin-top: 10px;
  margin-bottom: 10px;
  padding: 5px;
  font-size: 16px;
  cursor: pointer;
}

.select-button:active {
  border-radius: 4px;
  padding: 5px;
  background-color: #252525;
  border-color: #4ae754;
}

.button-container {
  width: 440px;
  display: flex;
  flex-wrap: wrap;
}

.button-wrapper {
  display: flex;
  align-items: center;
  flex-direction: column;
}

.sound-button {
  background-color: #212121;
  border-radius: 4px;
  color: #8a8a8a;
  width: 100px;
  height: 100px;
  border: 2px solid #fce979;
  padding: 20px;
  margin-left: 5px;
  margin-right: 5px;
  margin-top: 10px;
  margin-bottom: 5px;
  font-size: 24px;
  cursor: pointer;
}

.sound-button:hover {
  border-radius: 4px;
  background-color: #232323;
  border-color: #d0fc79;
}

.sound-button:active {
  border-radius: 4px;
  background-color: #252525;
  border-color: #4ae754;
  box-shadow: 0px 0px 5px 5px rgba(0, 255, 0, .2);
}

.repeat-button {
  background-color: #212121;
  border-radius: 4px;
  color: #8a8a8a;
  width: 100px;
  height: 30px;
  border: 2px solid #fce979;
  margin-left: 5px;
  margin-right: 5px;
  margin-top: 5px;
  margin-bottom: 10px;
  font-size: 16px;
  cursor: pointer;
}

.repeat-button-active {
  background-color: #252525;
  border-radius: 4px;
  color: #8a8a8a;
  width: 100px;
  height: 30px;
  border: 2px solid #4ae754;
  margin-left: 5px;
  margin-right: 5px;
  margin-top: 5px;
  margin-bottom: 10px;
  font-size: 16px;
  cursor: pointer;
  box-shadow: 0px 0px 5px 5px rgba(0, 255, 0, .2);
}

.file-list-box {
  display: flex;
  flex-direction: column;
  width: 440px;
}

.file-list {
  display: flex;
  flex-direction: column;
  background-color: #252525;
  border-radius: 4px;
  border: 2px solid #fce979;
  margin-left: 5px;
  margin-right: 5px;
  margin-top: 10px;
  margin-bottom: 10px;
  list-style: none;
  padding: 10px;
  color: #8a8a8a;;
}

.file-list li {
  margin-bottom: 5px;
}

.active {
  background-color: #252525;
  border-color: #4ae754;
  box-shadow: 0px 0px 5px 5px rgba(0, 255, 0, .2);
}

</style>
