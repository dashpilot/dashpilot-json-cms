<template>
<div>

  <input type="file" :id="'fileInput-'+rand" accept="image/*" @change="uploadImage" style="display: none;">


  <button class="btn btn-outline-dark mb-2" @click="chooseImage()"><i class="fas fa-images"></i> &nbsp;Upload Image

    <template v-if="loading">
      <i class="fas fa-spinner fa-spin"></i>
    </template>
  </button>


  <ul class="list-group mb-3">
    <template v-for="(img, i) in gallery">
      <li class="list-group-item" :class="{ 'sorting': sorting == img.filename }">
        <div class="row">
          <div class="col-1 preview-col">

            <div class="image-box" :style="{ backgroundImage: 'url('+ config.image_preview_url + img.filename + ')' }"></div>

          </div>
          <div class="col-4">

            <template v-if="image_title">
              <input type="text" class="form-control mb-0" :placeholder="image_title" v-model="img.title">
            </template>

          </div>
          <div class="col-4">

            <template v-if="image_alt">
              <input type="text" class="form-control mb-0" :placeholder="image_alt" v-model="img.alt">
            </template>

          </div>
          <div class="col-1"></div>
          <div class="col-1 text-end trash-col">

            <i class="fas fa-trash-alt" @click="deleteImage(img.filename)"></i>
            &nbsp;
          </div>
          <div class="col-1 text-end sort-col">

            <div>
              <i class="fas fa-caret-up" @click="moveUp(img.filename, i)"></i>
            </div>
            <div>
              <i class="fas fa-caret-down" @click="moveDown(img.filename, i)"></i>
            </div>

          </div>
        </div>
      </li>
    </template>
  </ul>


</div>
</template>

<script>
export default {
  data() {
    return {
      rand: Math.floor(Math.random() * 99999999999999999) + 1,
      loading: false,
      sorting: false,
    }
  },
  props: {
    gallery: {
      type: Array,
      default: [],
    },
    mykey: {},
    id: {},
    config: {
      type: Object,
      default: {},
    },
    image_width: {
      type: Number,
      default: 800,
    },
    image_title: {
      type: String
    },
    image_alt: {
      type: String
    },
  },
  methods: {
    chooseImage() {
      document.getElementById('fileInput-' + this.rand).click();
    },
    uploadImage(e) {

      const date = new Date().toJSON().slice(0, 10).replaceAll('-', '');
      const filename = date + "-" + Math.floor(Math.random() * 999999999) + ".jpg";


      // this.$emit('update', filename, this.id);

      var myapp = this;
      myapp.loading = true;

      var width = this.image_width;
      var imgUpload = new Image();
      imgUpload.onload = function() {
        var canvas = document.createElement('canvas'),
          ctx = canvas.getContext("2d"),
          oc = document.createElement('canvas'),
          octx = oc.getContext('2d');
        canvas.width = width; // destination canvas size
        canvas.height = canvas.width * imgUpload.height / imgUpload.width;
        var cur = {
          width: Math.floor(imgUpload.width * 0.5),
          height: Math.floor(imgUpload.height * 0.5)
        }
        oc.width = cur.width;
        oc.height = cur.height;
        octx.drawImage(imgUpload, 0, 0, cur.width, cur.height);
        while (cur.width * 0.5 > width) {
          cur = {
            width: Math.floor(cur.width * 0.5),
            height: Math.floor(cur.height * 0.5)
          };
          octx.drawImage(oc, 0, 0, cur.width * 2, cur.height * 2, 0, 0, cur.width, cur.height);
        }
        ctx.drawImage(oc, 0, 0, cur.width, cur.height, 0, 0, canvas.width, canvas.height);
        var base64Image = canvas.toDataURL('image/jpeg')

        console.log(base64Image);
        setTimeout(() => {

          if (myapp.config.save_url) {
            myapp.postData(myapp.config.save_url, {
                "type": "image",
                "path": filename,
                "content": base64Image
              })
              .then(data => {
                console.log(data); // JSON data parsed by `data.json()` call
                myapp.$emit('update', filename, myapp.id, myapp.mykey);
                myapp.loading = false;
              });
          } else {
            myapp.$emit('update', base64Image, myapp.id, myapp.mykey);
            myapp.loading = false;
          }

        }, 1000);

      }
      imgUpload.src = URL.createObjectURL(e.target.files[0]);

    },
    deleteImage(value) {
      if (confirm("Are you sure you want to delete this image?")) {
        let arr = this.gallery;
        arr = arr.filter(x => x.filename !== value)
        console.log(arr)
        this.$emit('update:gallery', arr);

        this.postData(this.config.image_delete_url, {
            "type": "image",
            "path": value
          })
          .then(data => {
            if (data.ok) {
              console.log(data);
            } else {
              console.log(data);
              alert("Error: " + data.message)
            }

          });

      }
    },
    moveUp(filename, i) {
      this.sorting = filename;

      setTimeout(() => {
        let newIndex = i - 1;
        if (i > 0) {
          this.array_move(this.gallery, i, newIndex)
        }
      }, 200);

    },
    moveDown(filename, i) {
      this.sorting = filename;

      setTimeout(() => {
        let newIndex = i + 1;
        if (this.gallery.length - 1 !== i) {
          this.array_move(this.gallery, i, newIndex)
        }
      }, 300);

    }
  }

}
</script>

<style scoped>
.image-box {
  width: 48px;
  height: 48px;
  border: 1px solid #DDD;
  background-size: cover;
  background-position: center center;
  border-radius: 4px;
  display: block;
}

.fa-trash-alt {
  margin-top: 15px;
  cursor: pointer;
}

.preview-col,
.sort-col {
  padding-left: 5px;
}

.trash-col {
  padding-right: 0px;
}

.sorting {
  background-color: #FDF3D1;
}

.fas:hover {
  color: #777;
  cursor: pointer;
}

.fa-trash-alt:hover {
  color: #CF5543;
  cursor: pointer;
}

.fa-caret-up,
.fa-caret-down {
  line-height: 0;
  margin: 0;
  padding: 0;
}

.form-control {
  margin-top: 5px;
}
</style>
