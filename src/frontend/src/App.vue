<template>
  <div class="page-loader" v-if="!isLoaded">
    <div style="background-color: #8cc271" class="cube first"></div>
    <div style="background-color: #69beeb" class="cube"></div>
    <div style="background-color: #f5aa39" class="cube"></div>
    <div style="background-color: #e9643b" class="cube last"></div>
  </div>
  <div id="head">
      <p style="cursor: pointer;">Идентификация Covid по звуку кашля <a target="_blank" style="color: #004fcf;"> </a></p>
      <img style="width: 100px" id="logovk" src="./assets/logovk.png">
  </div>

  <div id="up">
      <button id="button" onclick="document.getElementById('inputfile').click()">Выбрать файл</button>
      <input type='file' id="inputfile" style="display:none" @change="onFileChange">
      <button id="process" @click="onProcess">Проверить</button>
  </div>

  <div id="down">
      <div id="left">
        <audio controls v-if="url" :key="url">
          <source :src="url" alt="input file" type="audio/wav">
        </audio>
      </div>
      <div id="right">
        <h2 style="color: white">
          Вероятность положительного результата:
          <span v-if="res" v-bind:class="{low: low, low_mid: low_mid, mid: mid, mid_high: mid_high, high: high}">{{ res }}</span>
        </h2>
      </div>
  </div>
  <footer><p>© Барабанщиков Лев, Гамарник Александр, Кежаев Максим, 2022 </p></footer>
</template>

<script>

export default {
  name: 'App',
  data() {
    return {
      isLoaded: false,
      file: null,
      url: null,
      res: null,
      fd: null,
      low: null,
      low_mid: null,
      mid: null,
      mid_high: null,
      high: null,
    }
  },
  mounted() {
    document.onreadystatechange = () => {
      if (document.readyState == "complete") {
        this.isLoaded = true;
      }
    }
  },
  methods: {
    onFileChange(e) {
      this.file = e.target.files[0];
      console.log(this.file);
      this.url = URL.createObjectURL(this.file);
    },
    onProcess() {
      this.isLoaded = false;

      this.fd = new FormData();

      this.fd.append("audio", this.file);

      fetch("http://localhost:8000/api/predict/", {method: "POST", body: this.fd})
      .then(response => response.json())
      .then(data => {
        this.res = data["prob"]
        if (.3 - this.res >= 1e-9) {
          this.low = true;
          this.low_mid = false
          this.mid = false;
          this.mid_high = false;
          this.high = false
        } else if (.5 - this.res > 1e-9) {
          this.low = false;
          this.low_mid = true;
          this.mid = false;
          this.mid_high = false;
          this.high = false;
        } 
        else if (.7 - this.res >= 1e-9) {
          this.low = false;
          this.low_mid = false;
          this.mid = true;
          this.mid_high = false;
          this.high = false;
        } else if (.8 - this.res >= 1e-9) {
          this.low = false;
          this.low_mid = false;
          this.mid = false;
          this.mid_high = true
          this.high = false;
        } else {
          this.low = false;
          this.low_mid = false;
          this.mid = false;
          this.mid_high = false;
          this.high = true;
        }

        this.isLoaded = true;
      });
    },
  },
}
</script>

<style>

@keyframes left {
  40% {
    transform: translateX(-60px);
  }

  50% {
    transform: translateX(0);
  }
}

@keyframes right {
  40% {
    transform: translateX(60px);
  }

  50% {
    transform: translateX(0);
  }
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  padding-left: 10%;
  padding-right: 10%;
}

div {
  border-radius: 12px;
}

.page-loader {
  display: flex;
  justify-content: center;
  align-items: center;
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background-color: rgba(3, 3, 3, .3);
  z-index: 999;
}

.cube {
  width: 40px;
  height: 40px;
  margin-right: 10px;
}

.first {
  animation: left 1s infinite;
}

.last {
  animation: right 1s infinite .5s;
}

#head {
    padding: 10px;
    text-align: center;
    background-color: #0077ff;
    font-size: 30px;
    font-weight: bold;
    color: white;
    margin: 1%;
}

#up {
    padding: 10px 10px;
    margin: 1%;
    padding-left: 39%;
}

#logohcmut {
    width: 10%;
    height: 10%;
    cursor: pointer;
}

.logouit {
    width: 3%;
    height: 3%;
    cursor: pointer;
}

#button {
    display: block;
    background-color: white;
    border: 2px solid #074B80;
    color: black;
    padding: 15px 32px;
    text-align: center;
    text-decoration: none;
    display: inline-block;
    font-size: 16px;
    border-radius: 12px;
    margin-right: 10px;
}

#process {
    display: block;
    background-color: white;
    border: 2px solid #4CAF50;
    color: black;
    padding: 15px 32px;
    text-align: center;
    text-decoration: none;
    display: inline-block;
    font-size: 16px;
    border-radius: 12px;
}

#button:hover {
    background-color: #074B80;
    color: white;
    cursor: pointer;
}

#process:hover {
    background-color: #4CAF50;
    color: white;
    cursor: pointer;
}

#left {
    float: left;
    background: #222;
    width: 48%;
    height: 200px;
    margin: 1%;
    display: flex;
    align-items: center;
    justify-content: center;
}

#right {
    float: right;
    background: #222;
    width: 48%;
    height: 200px;
    margin: 1%;
    display: flex;
    align-items: center;
    justify-content: center;
}

.low {
    color: #00ff55;
    display: block;
    margin: 10px 10px;
}

.low_mid {
    color: #66f706;
    display: block;
    margin: 10px 10px;
}

.mid {
    color: #fffb00;
    display: block;
    margin: 10px 10px;
}

.mid_high {
    color: #ff7b00;
    display: block;
    margin: 10px 10px;
}

.high {
    color: #ff0000;
    display: block;
    margin: 10px 10px;
}

footer {
    margin: 1%;
    text-align: center;
    color: #074B80;
    font-size: 20px;
    margin-top: 18%;
}
</style>
