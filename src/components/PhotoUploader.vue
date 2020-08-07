<template>
  <div class="uk-container">
    <h1>Photo Uploader Demo</h1>
    <hr class="uk-divider-icon">
    <div class="uk-grid-small uk-width-2-3@l uk-width-1-1@s" uk-grid>
        <div class="uk-margin-bottom">
            <div uk-form-custom>
                <input type="file" accept="image/*" @change="loadFiles($event.target.name, $event.target.files)" multiple>
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
                <div v-if="imageList.length === 0" class="js-upload uk-placeholder uk-text-center">
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
                                <td><img class="uk-preserve-width" :src="image.attributes.url" width="80" alt=""></td>
                                <td class="uk-text-truncate">{{ image.data.name }}</td>
                                <td class="uk-table-expand">{{ image.attributes.size }}</td>
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
                    <img :src="currentSelection.data" alt="" />
                <div class="uk-card-body uk-text-left">
                    <p><strong>Name</strong><br>{{ currentSelection.name }}</p>
                    <p><strong>Size</strong><br>{{ currentSelection.size }} KB</p>
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
        imageList: [],
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
        loadFiles(fieldName, files) {
            // console.log('Loading files...')
            // console.info(fieldName)
            // Reference to the DOM input element
            this.counter = 1
            // console.log(files)
            // console.log('we have files!!')

            files.forEach(file => {
                
                // create a new FileReader to read this image and convert to base64 format
                const reader = new FileReader();
                reader.readAsDataURL(file)
                let imageData = ''
                // console.log('file loop')
                // Define a callback function to run, when FileReader finishes its job
                reader.onload = (e) => {
                    // Note: arrow function used here, so that "this.imageData" refers to the imageData of Vue component
                    // Read image as base64 and set to imageData
                    // console.log('reader.onload() fired')
                    const img = new Image()
                    img.src = e.target.result
                    
                    img.onload = () => {
                        // console.log('img.onload() fired')
                        imageData = this.resizeImage(img, file)
                        this.imageList.push(imageData)
                        console.log("imageData", imageData)
                    }
                    
                };
                
            });
        },
        showImagePreview(image) {
            this.currentSelection = image
        },
        resizeImage (img, file) {
            console.log('resizing image...')
            console.log(img.naturalWidth, img.naturalHeight)
            const width = 800
            const height = width * (img.naturalHeight / img.naturalWidth)
            let imageData = null

            const cvs = document.createElement('canvas')
            cvs.width = width
            cvs.height = height
            
            const ctx = cvs.getContext('2d')
            ctx.drawImage(img, 0, 0, width, height)
            
            ctx.canvas.toBlob((blob) => {
                console.log(URL.createObjectURL(blob))
                let objectUrl = URL.createObjectURL(blob)
                // let objectUrl = ctx.canvas.toDataURL(img, file.type)
                const fileObj = new File([blob], file.name, {
                    type: file.type,
                    lastModified: Date.now(),
                })
                let attributes = {
                    url: objectUrl,
                    size: file.size
                }
                this.uploadList.push({data: fileObj, attributes: attributes})
                console.log({data: fileObj, attributes: attributes})
            }, file.type, 1)
            // imageData = {
            //     // order: this.counter++,
            //     // data: ctx.canvas.toDataURL(img, file.type),
            //     data: objectUrl,
            //     name: file.name,
            //     size: file.size,
            //     isHovering: false,
            //     isSelected: false
            // }
            // console.log(imageData)



            return imageData

        },
        fileReader () {

        },
        uploadImages (e) {
            this.currentStatus = STATUS_SAVING;
            const formData = new FormData()
            const BASE_URL = "http://localhost:8083"
            this.uploadList.forEach((f, i) => {
                formData.append('file', f, f.name)
                console.log(i + ' - ' + f.name)
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
                    this.uploadList = [].concat(x)
                    this.currentStatus = STATUS_SUCCESS
                })
                .catch(err => {
                    this.uploadError = err.response
                    this.currentStatus = STATUS_FAILED
                })
            }, 1000)
        },
        deleteImage (idx) {

        },
        cancelImage (idx) {
            console.log(idx)
            // const imageListIndex = this.imageList.indexOf(img)
            // console.log("image Index: " + imageListIndex)
            this.imageList.splice(idx, 1)
                // URL.revokeObjectURL(attributes.url) // this prevents the photos from displaying
        },
        handleMouseOver (event) {
            console.log(event.target)
        },
        onDraggableChange (e) {
            console.log('dragged', e)
            for(let i = 0; i < this.imageList.length; i++) {
                console.log('index: ' + i + ' - name: ' + imageList[i].name)
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
