<template>
  <div class="container mt-5">
    <h2>Edit Pengaduan</h2>

    <form @submit.prevent="updatePengaduan" class="mt-4">
      <div class="mb-3">
        <label for="judul" class="form-label">Judul</label>
        <input
          type="text"
          id="judul"
          v-model="form.judul"
          class="form-control"
          required
        />
      </div>

      <div class="mb-3">
        <label for="isi" class="form-label">Isi</label>
        <textarea
          id="isi"
          v-model="form.isi"
          class="form-control"
          rows="5"
          required
        ></textarea>
      </div>

      <div class="mb-3">
        <label for="gambar" class="form-label">Gambar (opsional)</label>
        <input
          type="file"
          id="gambar"
          class="form-control"
          @change="handleFileUpload"
        />
        <!-- Preview gambar lama -->
        <div v-if="previewUrl" class="mt-2">
          <p>Preview Gambar:</p>
          <img :src="previewUrl" alt="Preview" class="img-thumbnail" width="200" />
        </div>
      </div>

      <button type="submit" class="btn btn-success">Update</button>
    </form>
  </div>
</template>

<script>
import axios from 'axios'
import Swal from 'sweetalert2'

export default {
  data() {
    return {
      form: {
        judul: '',
        isi: '',
        gambar: null
      },
      previewUrl: null
    }
  },
  async mounted() {
    const id = this.$route.params.id
    const token = localStorage.getItem('token')

    try {
      const res = await axios.get(`http://localhost:8000/api/pengaduan/${id}`, {
        headers: {
          Authorization: `Bearer ${token}`
        }
      })
      this.form.judul = res.data.judul
      this.form.isi = res.data.isi

      if (res.data.gambar) {
        this.previewUrl = `http://localhost:8000/storage/${res.data.gambar}`
      }
    } catch (error) {
      console.error('Gagal mengambil data:', error)
    }
  },
  methods: {
    handleFileUpload(e) {
      this.form.gambar = e.target.files[0]
      this.previewUrl = URL.createObjectURL(this.form.gambar)
    },
    async updatePengaduan() {
      const id = this.$route.params.id
      const token = localStorage.getItem('token')

      try {
        const formData = new FormData()
        formData.append('judul', this.form.judul)
        formData.append('isi', this.form.isi)
        if (this.form.gambar) {
          formData.append('gambar', this.form.gambar)
        }

        await axios.post(
          `http://localhost:8000/api/pengaduan/${id}?_method=PUT`,
          formData,
          {
            headers: {
              Authorization: `Bearer ${token}`,
              'Content-Type': 'multipart/form-data'
            }
          }
        )

        Swal.fire({
          icon: 'success',
          title: 'Berhasil!',
          text: 'Pengaduan berhasil diperbarui',
          timer: 2000,
          showConfirmButton: false
        }).then(() => {
          this.$router.push('/dashboard')
        })
      } catch (error) {
        Swal.fire({
          icon: 'error',
          title: 'Oops...',
          text: 'Gagal memperbarui pengaduan!'
        })
        console.error(error)
      }
    }
  }
}
</script>

<style scoped>
.container {
  max-width: 600px;
}

.img-thumbnail {
  border: 1px solid #ddd;
  border-radius: 8px;
}
</style>
