<template>
  <div class="column">
    <div class="row">
      <button class="randomButton" @click="getRandomMeme()">Get Random Meme</button>
      <select @change="optionSelected($event)">
        <option :key="info.index" v-for="info in memeInfo" v-bind:value="info.index">{{info.name}}</option>
      </select>
    </div>
    <input
      class="topInput"
      v-bind:style="{ width: currentMeme.width /2 +'px'}"
      v-model="top_Caption"
      placeholder="Top Text"
      v-if="hasMeme"
    />
    <div class="container" id="imageWrapper">
      <div class="topText memeText">{{top_Caption}}</div>
      <img class="meme-image" v-bind:src="currentMeme.url" />
      <div class="bottomText memeText">{{bottom_Caption}}</div>
    </div>
    <input
      class="bottomInput"
      v-bind:style="{ width: currentMeme.width /2 +'px'}"
      v-model="bottom_Caption"
      placeholder="Bottom Text"
      v-if="hasMeme"
    />
    <button @click="getImage()" class="downloadButton" v-if="hasMeme">Download</button>
  </div>
</template>
<script>
import axios from "axios";
import htmlToImage from "html-to-image";
export default {
  data() {
    return {
      top_Caption: "",
      bottom_Caption: "",
      memes: [],
      memeInfo: [],
      currentMeme: {},
      hasMeme: false
    };
  },
  mounted() {
    this.getMemes();
  },
  methods: {
    getMemes() {
      axios.get("https://api.imgflip.com/get_memes").then(result => {
        this.memes = result.data.data.memes;
        let i = 0;
        this.memes.forEach(meme => {
          this.memeInfo.push({
            index: i,
            name: meme.name
          });
          i++;
        });
      });
    },

    getRandomMeme() {
      if (this.memes.length > 0) {
        const randomNumber = Math.floor(Math.random() * 100);
        this.currentMeme = this.memes[randomNumber];
        this.hasMeme = true;
      }
    },
    getImage() {
      const imageNode = document.getElementById("imageWrapper");
      htmlToImage
        .toPng(imageNode)
        .then(dataUrl => {
          var img = new Image();
          img.src = dataUrl;
          const downloadLink = document.createElement("a");
          downloadLink.href = dataUrl;
          downloadLink.download = `meme${new Date().toISOString()}.png`;
          document.body.appendChild(downloadLink);
          downloadLink.click();
          document.body.removeChild(downloadLink);
        })
        .catch(function(error) {
          throw error;
        });
    },
    optionSelected(event) {
      // eslint-disable-next-line no-console
      console.log(event.target.value);
      this.currentMeme = this.memes[event.target.value];
    }
  }
};
</script>

<style>
@import "./style/main.css";
</style>
