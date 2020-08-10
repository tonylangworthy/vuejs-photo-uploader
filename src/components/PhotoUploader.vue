<template>
  <div class="uk-container">
    <h1>Photo Uploader Demo</h1>
    <hr class="uk-divider-icon">
    <div class="uk-grid-small uk-width-2-3@l uk-width-1-1@s" uk-grid>
        <div class="uk-margin-bottom">
            <div uk-form-custom>
                <input type="file" accept="image/*" @change="loadFiles($event.target.files)" multiple>
                <button class="uk-button uk-button-default uk-border-rounded uk-width-1-1" :disabled="isSaving" type="button" tabindex="-1">
                    <span uk-icon="plus"></span> Add Files
                </button>
            </div>
        </div>
        <div class="uk-margin-bottom">
            <button class="uk-button uk-button-primary uk-border-rounded uk-width-1-1" :disabled="isSaving" @click="uploadImages">
                <span uk-icon="upload"></span> Upload
            </button>
        </div>
    </div>
    <div class="uk-grid-small" uk-grid>
        <progress id="js-progressbar" class="uk-progress" :value="uploadProgress" max="100" v-if="isSaving"></progress>
    </div>
    <div class="uk-flex uk-flex-left uk-height-viewport">
        <div class="uk-width-5-6@m border-top uk-padding-small">
            <div class="uk-grid-small" uk-grid>
                <div v-if="uploadList.length === 0" class="js-upload uk-placeholder uk-text-center">
                    <span uk-icon="icon: cloud-upload"></span>
                    <span class="uk-text-middle"> Select photos to upload by using the <strong>Add Files</strong> button</span>
                </div>
                <div v-else class="uk-overflow-auto">
                    <table class="uk-table uk-table-small uk-table-hover uk-table-middle uk-table-divider">
                        <thead>
                            <tr>
                                <th class=""></th>
                                <th class="uk-table-expand"></th>
                                <th class="uk-table-expand">Name</th>
                                <th class="uk-table-expand">Size</th>
                                <th v-bind:class="{ 'uk-hidden': isSuccess }">Cancel</th>
                                <th v-if="isSuccess">Delete</th>
                            </tr>
                        </thead>
                        <draggable 
                          v-model="uploadList" 
                          ghost-class="ghost" 
                          tag="tbody" 
                          handle=".handle"
                          @change="onDraggableChange($event)">
                            <tr v-for="(image, index) in uploadList" :key="index" @click="showImagePreview(image)">
                                <td class="handle"><button class="uk-button uk-button-text"><span uk-icon="icon: table"></span></button></td>
                                <td><img class="uk-preserve-width" :src="image.url" width="80" alt=""></td>
                                <td class="uk-text-truncate">{{ image.name }}</td>
                                <td class="uk-table-expand">{{ image.size }}</td>
                                <td v-bind:class="{ 'uk-hidden': isSuccess }" class="small-delete-button">
                                    <button class="uk-button uk-button-default uk-border-rounded uk-width-1-1" @click="cancelImage(index)">
                                        <span uk-icon="icon: ban"></span>
                                    </button>
                                </td>
                                <td v-if="isSuccess" class="small-delete-button">
                                    <button class="uk-button uk-button-danger uk-border-rounded uk-width-1-1" @click="deleteImage(index)">
                                        <span uk-icon="icon: trash"></span>
                                    </button>
                                </td>
                            </tr>
                        </draggable>
                    </table>
                </div>
            </div>
        </div>
        <div class="uk-visible@m uk-width-1-4 border-top border-left">
            <div v-if="currentSelection">
                <div class="uk-media-card-top uk-height-viewport">
                    <img :src="currentSelection.url" alt="" />
                <div class="uk-card-body uk-text-left">
                    <p><strong>Name</strong><br>{{ currentSelection.name }}</p>
                    <p><strong>Current Size</strong><br>{{ currentSelection.size }}</p>
                    <p><strong>Original Size</strong><br>{{ currentSelection.originalSize }}</p>
                    <p><strong>Last Modified</strong><br>{{ currentSelection.lastModified }}</p>
                </div>
                </div>
            </div>
            <div v-else>
                <div class="uk-card uk-card-default uk-card-body uk-text-center uk-height-viewport">
                    <h3 class="uk-card-title"><span uk-icon="icon: image; ratio: 3"></span></h3>
                    <p>No photos selected.</p>
                </div>
            </div>
        </div>
    </div>
  </div>
</template>

<script>
import draggable from 'vuedraggable'
import * as axios from 'axios'

const STATUS_INITIAL = 0, STATUS_SAVING = 1, STATUS_SUCCESS = 2, STATUS_FAILED = 3;

export default {
  name: 'PhotoUploader',
  components: {
      draggable
  },
  data() {
    return {
        tempList: [],
        uploadList: [],
        uploadError: null,
        uploadPercentage: 0,
        currentStatus: null,
        currentSelection: null,
        counter: '',
        dragging: false,
        enabled: true
        }
    },
    computed: {
        isInitial() {
            return this.currentStatus === STATUS_INITIAL;
        },
        isSaving() {
            return this.currentStatus === STATUS_SAVING;
        },
        isSuccess() {
            return this.currentStatus === STATUS_SUCCESS;
        },
        isFailed() {
            return this.currentStatus === STATUS_FAILED;
        },
        uploadProgress() {
            return this.uploadPercentage
        }
    },
	methods: {
        reset() {
            // reset form to initial state
            this.currentStatus = STATUS_INITIAL;
            this.uploadList = [];
            this.uploadError = null;
        },
		change({coordinates, canvas}) {
			console.log(coordinates, canvas)
        },
        reset() {
			this.image = null
		},
        loadFiles(files) {
            let images = []

            // Convert FileList to array
            files.forEach(f => {
                images.push(f)
            })
            
            this.resizeImages(images)
        },
        showImagePreview(image) {
            this.currentSelection = image
        },
        resizeImages (images) {
            console.log('resizing images...')
            // console.log(img.naturalWidth, img.naturalHeight)
            const width = 800
            

            Promise.all(images.map(i => {
                return new Promise((resolve, reject) => {

                    let preview = new Image()
                    preview.src = URL.createObjectURL(i)

                    preview.onload = () => {
                        const height = width * (preview.naturalHeight / preview.naturalWidth)
                        console.log(height)
                        const cvs = document.createElement('canvas')
                        cvs.width = width
                        cvs.height = height
                        const ctx = cvs.getContext('2d')
                        ctx.drawImage(preview, 0, 0, width, height)

                        ctx.canvas.toBlob(function(blob) {
                            console.log('inside toBlob')
                            const resizedFile = new File([blob], preview.name, {
                                type: preview.type,
                                lastModified: Date.now(),
                            })
                            const imageData = {
                                file: resizedFile,
                                originalSize: i.size, 
                                size: resizedFile.size, 
                                name: i.name, 
                                lastModified: resizedFile.lastModified}
                            resolve(imageData)

                        }, preview.type, 1)
                        URL.revokeObjectURL(preview.src)
                    }
                })
            })).then((photos) => {
                photos.forEach((photo) => {
                    console.log('file: ', photo)
                    let imageData = {
                        data: photo.file,
                        url: URL.createObjectURL(photo.file),
                        name: photo.name,
                        originalSize: this.getFileSize(photo.originalSize),
                        size: this.getFileSize(photo.size),
                        lastModified: this.getDateTimeFromMilliseconds(photo.lastModified)
                    }
                    this.uploadList.push(imageData)
                    console.log('imageData', imageData)
                })
            })

            console.log('outside toBlob')
        },
        getFileSize (number) {
            if(number < 1024) {
                return number + 'bytes';
            } else if(number >= 1024 && number < 1048576) {
                return (number/1024).toFixed(1) + 'KB';
            } else if(number >= 1048576) {
                return (number/1048576).toFixed(1) + 'MB';
            }
        },
        getDateTimeFromMilliseconds (ms) {
            const dateNow = new Date(ms)
            // const month = dateNow.getMonth()
            // const day = dateNow.getDate()
            // const year = dateNow.getFullYear()
            return dateNow.toString()
        },
        uploadImages (e) {
            console.log('uploading images')
            this.currentStatus = STATUS_SAVING;
            const formData = new FormData()
            const BASE_URL = "http://localhost:8083"
            this.uploadList.forEach((d, i) => {
                formData.append('file', d.data, d.name)
                console.log(i + ' - ' + d.name)
            })

            for (var value of formData.values()) {
                // console.log(value); 
            }

            setTimeout(() => {
               axios.post(`${BASE_URL}/photos`, formData, {
                headers: {
                'Content-Type': 'multipart/form-data'
                },
                onUploadProgress: function( progressEvent ) {
                    console.log(progressEvent)
                    this.uploadPercentage = parseInt( Math.round( ( progressEvent.loaded / progressEvent.total ) * 100 ))
                }.bind(this)
                })
                .then(x => {
                    // this.uploadList = [].concat(x)
                    this.currentStatus = STATUS_SUCCESS
                })
                .catch(err => {
                    this.uploadError = err.response
                    this.currentStatus = STATUS_FAILED
                })
            }, 1000)
        },
        deleteImage (idx) {
            console.log(`Photo ${idx} has been set for deletion.`)
        },
        cancelImage (idx) {
            console.log('removed index ', idx)
            // const imageListIndex = this.imageList.indexOf(img)
            // console.log("image Index: " + imageListIndex)
            const lastImage = this.uploadList.length - 1
            console.log('lastImage', lastImage)
            if(idx == lastImage) {
                console.log('match!')
                this.showImagePreview(null)
            }
            this.uploadList.splice(idx, 1)
        },
        handleMouseOver (event) {
            console.log(event.target)
        },
        onDraggableChange (e) {
            console.log('dragged', e)
            for(let i = 0; i < this.uploadList.length; i++) {
                console.log('index: ' + i + ' - name: ' + this.uploadList[i].data.name)
            }
        }
    },
     mounted() {
      this.reset();
    }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
    .preview-card > img {
        object-fit: cover;
    }
    .crop-container {
        max-width: 400px;
        max-height: 300px;
    }
    .border-top {
        border-top: 1px solid #E4E4E4;
    }
    .border-left {
        border-left: 1px solid #E4E4E4;
    }
    .ghost {
        opacity: 0.5;
        background: #c8ebfb;
    }
    .handle {
        margin-right: 1px;
        min-width: 20px;
    }
    .small-delete-button {
        min-width: 100px;
    }
    /* .hovering {
        cursor: -moz-grab;
        cursor: -webkit-grab;
        cursor: grab;
    } */
</style>
