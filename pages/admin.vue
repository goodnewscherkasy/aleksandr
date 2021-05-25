<template>
  <div>
    <v-col cols="12">
      <v-file-input
        accept="audio/mp3"
        color="deep-purple accent-4"
        counter
        multiple
        placeholder="Выберите файл"
        prepend-icon="mdi-bookmark-music"
        v-model="File"

      >
        <template>
          <v-file-input
            counter
            label="File input"
            multiple
            show-size
          ></v-file-input>
        </template>

      </v-file-input>
    </v-col>
    <v-btn
      @click="addLocation(File)"
      color="blue darken-1"
      text
      type="submit"
    >
      Сохранить
    </v-btn>
  </div>
</template>

<script>
import Swal from 'sweetalert2'

export default {
  name: 'admin',
  data () {
    return {
      File: []
    }
  },
  methods: {
    async addLocation (File) {
      console.log(File)
      Swal.fire({
        title: 'Идет загрузка...',
        imageUrl: '352.gif',
        showConfirmButton: false
      })

      const promises = []
      const promisesName = []

      if (File) {
        for (let i = 0; i < File.length; i++) {
          // Создайте метаданные файла
          const storageRef = this.$fireStorage.ref()
          const metadata = {
            contentType: 'audio/mp3'
          }
          const nameTime = +new Date() + '.mp3'

          // Загрузить файл и метаданные в объект 'assets/images/***.jpg'
          const uploadTask = storageRef.child(`voices/${nameTime}`).put(File[i], metadata)

          promises.push(
            uploadTask
              .then(snapshot =>
                snapshot.ref.getDownloadURL()
              )
          )
          promisesName.push(
            nameTime
          )
        }
      }

      const URLs = await Promise.all(promises)
      const NameImages = await Promise.all(promisesName)

      const docRef = await this.$fireStore.collection('products').add({
        NameImages,
        arrayImages: URLs,
        name
      })
      try {
        const docAdded = await docRef
        await this.$fireStore.doc('products/' + `${docAdded.id}`).update({ id: `${docAdded.id}` })
      } catch (err) {
        return err
      }

      Swal.close()

      Swal.fire({
        position: 'top-end',
        icon: 'success',
        title: 'Ваша работа была сохранена',
        showConfirmButton: false,
        timer: 2000
      })
      // arrayImages = [];
      this.dialog = false
    }
  }
}
</script>

<style scoped>

</style>
