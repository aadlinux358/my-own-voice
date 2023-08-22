<template>
  <q-page padding>
    <div class="column justify-center items-center q-my-md">
      <div class="text-h4">
        Voice Diary
      </div>
    </div>

    <div class="row justify-center items-center q-my-md">
      <q-btn :color="recording ? 'red' : 'primary'"
             round
             size="xl"
             :icon="recording ? 'mic_off' : 'mic'"
             @click="toggleRecording"></q-btn>
    </div>
    <div class="row justify-center items-center q-my-md">
      <div v-if="selectedAudio?.audioUrl">
        Playing: {{ selectedAudio?.fileName }}
      </div>
      <audio controls
             autoplay
             :src="selectedAudio?.audioUrl"></audio>
    </div>
    <div class="row justify-center items-center q-my-md">
      {{ audios.size === 0 ? 'No diaries' : 'Audio diaries' }}
    </div>
    <q-list>
      <q-item v-ripple
              v-for="[k, v] of audios"
              :key="k">
        <q-item-section avatar>

          <q-icon color="primary"
                  name="audiotrack" />
        </q-item-section>
        <q-item-section>{{ v.fileName }}</q-item-section>
        <q-item-section side>
          <div class="row q-gutter-sm">

            <q-btn icon="play_arrow"
                   round
                   size="sm"
                   @click="playAudio(v)"
                   color="primary" />
            <q-icon size="md"
                    :color="v.isFavorite ? 'red' : 'grey'"
                    :name="v.isFavorite ? 'favorite' : 'favorite_border'"
                    @click="v.isFavorite = !v.isFavorite" />
            <!--
            <q-btn icon="favorite_border"
                   round
                   size="sm"
                   :color="v.isFavorite ? 'red' : 'grey'"
                   @click="v.isFavorite = !v.isFavorite" />
              -->

            <q-btn icon="delete"
                   round
                   size="sm"
                   @click="deleteAudio(k)"
                   color="red" />
          </div>
        </q-item-section>
      </q-item>
    </q-list>
    <div class="text-h5">
      Favorites
    </div>
    <div v-for="[k, v] of audios"
         :key="k">
      <div v-if="v.isFavorite">
        {{ v.fileName }} is favorite: {{ v.isFavorite }}
      </div>
    </div>
  </q-page>
</template>

<script setup lang="ts">
import {ref, onMounted, Ref} from 'vue';
export interface AudioFile {
  fileName: string;
  audioUrl: string;
  isFavorite: boolean
}
const mediaRecorder: Ref<MediaRecorder | null> = ref(null)
const chunks: Ref<BlobPart[]> = ref([])
const audios: Ref<Map<string, AudioFile>> = ref(new Map())
const recording = ref(false)
const selectedAudio: Ref<AudioFile | null> = ref(null)
function startRecording() {
  if (mediaRecorder.value) {
    mediaRecorder.value.start()
    console.log(mediaRecorder.value.state)
    console.log('recorder started')
    recording.value = true
  } else {
    alert('Can not start recording.')
  }
}

function stopRecording() {
  if (mediaRecorder.value) {
    mediaRecorder.value.stop()
    console.log(mediaRecorder.value.state)
    console.log('recorder stoped')
    recording.value = false
  }
}
function toggleRecording() {
  if (recording.value) {
    stopRecording()
  } else {
    startRecording()
  }
}

function playAudio(audio: AudioFile) {
  selectedAudio.value = audio
}

function deleteAudio(key: string) {
  selectedAudio.value = null
  audios.value.delete(key)
}

onMounted(() => {
  if (navigator.mediaDevices?.getUserMedia) {
    navigator.mediaDevices.getUserMedia({audio: true})
      .then((stream) => {
        mediaRecorder.value = new MediaRecorder(stream)
        mediaRecorder.value.ondataavailable = (e: BlobEvent) => {
          chunks.value.push(e.data)
        }
        mediaRecorder.value.onstop = (e) => {
          console.log("Data available after MediaRecorder.stop() called.")
          const audioName = prompt('Enter audio file name: ', `Unnamed audio ${new Date().toString()}`)
          const blob = new Blob(chunks.value, {type: 'audio/ogg; codecs=opus'})
          const audioURL = window.URL.createObjectURL(blob)
          chunks.value = []
          audios.value.set(audioName!, {fileName: audioName!, audioUrl: audioURL, isFavorite: false})
        }
      }).catch((err) => console.log('The following getUserMedia error occured: ' + err))
  } else {
    alert('getUserMedia not supported on your browser!')
  }

})
</script>
