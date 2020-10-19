<template>
  <v-app>
    <v-app-bar app color="primary" dark>
      <div class="d-flex align-center">
        <v-img
          alt="Vuetify Logo"
          class="shrink mr-2"
          contain
          src="https://cdn.vuetifyjs.com/images/logos/vuetify-logo-dark.png"
          transition="scale-transition"
          width="40"
        />
  
        <h3>HomeWorkMaid</h3>
  
      </div>
    </v-app-bar>
    <v-main>
      
      <v-file-input v-model="file" multiple @change="changeSelection"></v-file-input>
    
          <v-btn
              v-if="showFiles && showFiles.length"
              @click="newZip"
              color="primary">
              <a :href="downloadUrl"  ref="downloadLink" class="white--text">Download</a>
          </v-btn>
          <div v-for="f of showFiles" :key="f.Name">
          <span> {{ f.name }} </span> 
          <img :src="getImageContentString(f.content)" />
         <v-spacer></v-spacer>
         </div>
    </v-main>
  </v-app>
</template>

<script lang="ts">
import Vue from "vue";
import JsZip, { files } from 'jszip'; 

interface ShowFile{
  name: string;
  content: Blob;
}

export default Vue.extend({
  name: "App",


  data: () => ({
    
    downloadUrl:'',
    showFiles: [] as ShowFile[],
    file: [] as File[],
    fileSizes: [] as number[]
  }),
  methods:{
    changeSelection(files: File[]){
        this.fileSizes.length = 0;
        this.showFiles.length = 0;
        for(const f of files){
          this.resizeImage(f, 150,150).then(b=> this.showFiles.push({ name: f.name, content: b}));
        }
    },
    async getSize(f: File){
      const s = await f.arrayBuffer();
      return s.byteLength;
    },
    getImageContentString(blob:Blob): string {
      return URL.createObjectURL(blob);
    },
    resizeImage(file:File, maxWidth:number, maxHeight:number):Promise<Blob> {
      return new Promise<Blob>((resolve, reject) => {
        const image = new Image();
        image.src = URL.createObjectURL(file);
        image.onload = () => {
            const width = image.width;
            const height = image.height;
            
            if (width <= maxWidth && height <= maxHeight) {
                resolve(file);
            }

            let newWidth
            let newHeight;

            if (width > height) {
                newHeight = height * (maxWidth / width);
                newWidth = maxWidth;
            } else {
                newWidth = width * (maxHeight / height);
                newHeight = maxHeight;
            }

            const canvas = document.createElement('canvas');
            canvas.width = newWidth;
            canvas.height = newHeight;

            const context: any = canvas.getContext('2d');

            if(context != null) {
              context.drawImage(image, 0, 0, newWidth, newHeight);

              canvas.toBlob(resolve as BlobCallback, file.type);
            }
            else{
              reject();
            }
        };
        image.onerror = reject;
      });
    },
   newZip(){
       const zip = new JsZip();
       // zip.file(" ");
        for (const sf of this.showFiles){
            zip.file(sf.name, sf.content);
        }     
          zip.generateAsync({
            type:'blob'
          }).then(b => this.downloadUrl= this.getImageContentString(b))
    
   }
  }
});
</script>
