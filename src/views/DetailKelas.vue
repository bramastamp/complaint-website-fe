<template>
  <MainLayout>
    <div class="container mt-4 font-poppins">
      <!-- Loading -->
      <div v-if="loading" class="text-center my-5">
        <div class="spinner-border text-primary" role="status">
          <span class="visually-hidden">Loading...</span>
        </div>
        <p class="mt-3 text-muted">Memuat detail kelas...</p>
      </div>

      <!-- Data Kelas -->
      <div v-else-if="kelas">
        <h2 class="fw-bold mb-4">{{ kelas.nama_kelas }}</h2>

        <div class="row">
          <!-- Gambar -->
          <div class="col-md-5 mb-3">
            <img
              v-if="kelas.gambar"
              :src="`http://localhost:8000/storage/${kelas.gambar}`"
              class="img-fluid rounded shadow-sm border"
              alt="Gambar kelas"
              style="max-height: 300px; object-fit: cover"
            />
            <div
              v-else
              class="d-flex align-items-center justify-content-center bg-light border rounded shadow-sm"
              style="height: 300px; font-style: italic; color: #888"
            >
              Tidak ada gambar
            </div>
          </div>

          <!-- Detail -->
          <div class="col-md-7">
            <ul class="list-group shadow-sm rounded mb-4">
              <li
                class="list-group-item d-flex justify-content-between align-items-center"
              >
                <strong>Gedung:</strong>
                <span>{{
                  kelas.gedung?.nama_gedung || 'Tidak diketahui'
                }}</span>
              </li>
              <li
                class="list-group-item d-flex justify-content-between align-items-center"
              >
                <strong>PIC:</strong>
                <span>{{ kelas.pic || '-' }}</span>
              </li>
              <li
                class="list-group-item d-flex justify-content-between align-items-center"
              >
                <strong>Layout:</strong>
                <span>{{ kelas.layout || '-' }}</span>
              </li>
              <li
                class="list-group-item d-flex justify-content-between align-items-center"
              >
                <strong>Standar Operasional:</strong>
                <span>{{ kelas.standar_operasional || '-' }}</span>
              </li>
            </ul>

            <!-- Tombol Aksi -->
            <div class="d-flex gap-2">
              <router-link
                :to="{ name: 'CreatePengaduan', params: { kelasId: kelas.id } }"
                class="btn btn-primary shadow-sm"
              >
                Buat Pengaduan untuk Kelas Ini
              </router-link>

              <router-link
                to="/lihat-kelas"
                class="btn btn-outline-secondary shadow-sm"
              >
                Kembali
              </router-link>
            </div>
          </div>
        </div>
      </div>

      <!-- Jika Tidak Ada Data -->
      <div v-else class="alert alert-danger text-center mt-5">
        Data kelas tidak ditemukan.
      </div>
    </div>
  </MainLayout>
</template>

<script>
import axios from 'axios';
import MainLayout from '../layouts/MainLayout.vue';

export default {
  components: { MainLayout },
  props: ['id'], // pastikan route pakai props: true
  data() {
    return {
      kelas: null,
      loading: true,
    };
  },
  async mounted() {
    try {
      const kelasId = this.id || this.$route.params.id;
      const res = await axios.get(`http://localhost:8000/api/kelas/${kelasId}`);
      console.log('Detail Kelas API response:', res.data);

      this.kelas = res.data.data;
    } catch (error) {
      console.error('Gagal ambil detail kelas:', error);
    } finally {
      this.loading = false;
    }
  },
};
</script>

<style scoped>
/* Import Google Fonts */
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@400;500;600;700&display=swap');

.font-poppins {
  font-family: 'Poppins', sans-serif;
}
</style>
