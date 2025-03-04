
<template>
 <div id="app">
    <h1>Employee Registration Application</h1>

    <form @submit.prevent="submitForm">
      <input v-model="name" placeholder="Enter Name" @input="validateName" required />
      <span v-if="nameError" style="color: red;">{{ nameError }}</span>
      <br /><br />

      <input v-model="age" type="text" placeholder="Enter Age" @input="validateAge" required />
      <span v-if="ageError" style="color: red;">{{ ageError }}</span>
      <br /><br />

      <input v-model="empCode" placeholder="Enter Employee Code" @input="validateEmpCode" required />
      <span v-if="empCodeError" style="color: red;">{{ empCodeError }}</span>
      <br /><br />
     
    <div v-if="!image">
      <h2>Select an image</h2>
      <input type="file" @change="onFileChange">
    </div>
    <div v-else>
      <img :src="image" />
      <button v-if="!uploadURL" @click="removeImage">Remove image</button>
      <button v-if="!uploadURL" @click="uploadImage">Upload image</button>
    </div>
    <h2 v-if="uploadURL">Success! Image uploaded to bucket.</h2>
    </form>
</div>


</template>

<script>
//import Vue from 'vue/dist/vue.js';
import axios from "axios";

const MAX_IMAGE_SIZE = 1000000

/* ENTER YOUR ENDPOINT HERE
   FILES UPLOADED TO MY ENDPOINT ARE AUTOMATICALLY DELETED EVERY FEW HOURS */

const API_ENDPOINT = 'https://c1os4xvwji.execute-api.us-east-1.amazonaws.com/default/myfirstlambda'

export default {
  name: 'app',
    data () {
      return {
        image: '',
        uploadURL: '',
        nameError: "",
        ageError: "",
        empCodeError: "",
      }
    },
  methods: {
    onFileChange (e) {
      let files = e.target.files || e.dataTransfer.files
      if (!files.length) return
      this.createImage(files[0])
    },
    createImage (file) {
      // var image = new Image()
      let reader = new FileReader()
      reader.onload = (e) => {
        console.log('length: ', e.target.result.includes('data:image/jpeg'))
        if (!e.target.result.includes('data:image/jpeg')) {
          return alert('Wrong file type - JPG only.')
        }
        if (e.target.result.length > MAX_IMAGE_SIZE) {
          return alert('Image is loo large - 1Mb maximum')
        }
        this.image = e.target.result
      }
      reader.readAsDataURL(file)
    },
    removeImage: function (e) {
      console.log('Remove clicked', e)
      this.image = ''
    },
    uploadImage: async function (e) {
      console.log('Upload clicked', e)
      // Get the presigned URL
      const response = await axios({
        method: 'GET',
        url: API_ENDPOINT
      })
      console.log('Response: ', response.data)
      console.log('Uploading: ', this.image)
      let binary = atob(this.image.split(',')[1])
      let array = []
      for (var i = 0; i < binary.length; i++) {
        array.push(binary.charCodeAt(i))
      }
      let blobData = new Blob([new Uint8Array(array)], {type: 'image/jpeg'})
      console.log('Uploading to: ', response.data.uploadURL)
      const result = await fetch(response.data.uploadURL, {
        method: 'PUT',
        body: blobData
      })
      console.log('Result: ', result)
      // Final URL for the user doesn't need the query string params
      this.uploadURL = response.data.uploadURL.split('?')[0]
    },
    validateName() {
      const regex = /^[A-Za-z\s]+$/;
      this.nameError = regex.test(this.name) ? "" : "Name should contain only alphabets.";
    },
    validateAge() {
      const regex = /^[0-9]+$/;
      this.ageError = regex.test(this.age) ? "" : "Age should contain only numbers.";
    },
    validateEmpCode() {
      const regex = /^[A-Za-z0-9]+$/;
      this.empCodeError = regex.test(this.empCode) ? "" : "Employee Code should be alphanumeric only.";
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
body {
  background: #20262E;
  padding: 20px;
  font-family: sans-serif;
}

#app {
  background: #fff;
  border-radius: 4px;
  padding: 20px;
  transition: all 0.2s;
  text-align: center;
}

#logo {
  width: 100px;
}

h2 {
  font-weight: bold;
  margin-bottom: 15px;
}

h1, h2 {
  font-weight: normal;
  margin-bottom: 15px;
}
a {
  color: #42b983;
}
img {
  width: 30%;
  margin: auto;
  display: block;
  margin-bottom: 10px;
}
</style>
