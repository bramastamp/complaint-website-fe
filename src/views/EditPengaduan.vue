<template>
  <MainLayout>
    <div class="form-container container">
      <h2 class="mb-4 fw-semibold">Edit Pengaduan</h2>

      <form @submit.prevent="updatePengaduan" class="p-4 rounded bg-white" enctype="multipart/form-data">
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
          <label for="isi" class="form-label">Isi Pengaduan</label>
          <textarea
            id="isi"
            v-model="form.isi"
            class="form-control"
            rows="5"
            required
          ></textarea>
        </div>

        <div class="mb-3">
          <label for="gambar" class="form-label">Upload Gambar (opsional)</label>
          <input
            type="file"
            id="gambar"
            class="form-control"
            accept="image/*"
            @change="handleFileUpload"
          />
        </div>

        <!-- Preview gambar -->
        <div v-if="previewUrl" class="mb-3">
          <p>Preview:</p>
          <img :src="previewUrl" class="img-fluid rounded" style="max-height:200px" />
        </div>

        <button type="submit" class="btn btn-success w-100">Update Pengaduan</button>
      </form>
    </div>
  </MainLayout>
</template>

<script>
import axios from 'axios'
import Swal from 'sweetalert2'
import MainLayout from '../layouts/MainLayout.vue'

export default {
  components: {
    MainLayout
  },
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
.form-container {
  font-family: 'Poppins', sans-serif;
}

form {
  background-color: #fff;
  border-radius: 12px;
  box-shadow: 6px 6px 20px 0px rgba(0, 0, 0, 0.1);
  -webkit-box-shadow: 6px 6px 20px 0px rgba(0, 0, 0, 0.1);
  -moz-box-shadow: 6px 6px 20px 0px rgba(0, 0, 0, 0.1);
}

input,
textarea,
select {
  font-size: 0.95rem;
  border-radius: 8px;
}

button {
  border-radius: 8px;
  font-weight: 500;
  letter-spacing: 0.5px;
}

h2 {
  font-weight: 600;
}
</style>
