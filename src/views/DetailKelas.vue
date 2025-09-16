<template>
  <MainLayout>
    <div class="container mt-4">
      <div v-if="loading" class="text-center">
        <div class="spinner-border text-primary" role="status">
          <span class="visually-hidden">Loading...</span>
        </div>
      </div>

      <div v-else-if="kelas">
        <h2 class="fw-bold mb-4">{{ kelas.nama_kelas }}</h2>

        <div class="row">
          <!-- Gambar -->
          <div class="col-md-5 mb-3">
            <img
              v-if="kelas.gambar"
              :src="`http://localhost:8000/storage/${kelas.gambar}`"
              class="img-fluid rounded shadow-sm"
              alt="Gambar kelas"
              style="max-height: 300px; object-fit: cover;"
            />
            <div
              v-else
              class="d-flex align-items-center justify-content-center bg-light border rounded"
              style="height: 300px; font-style: italic; color: #888;"
            >
              Tidak ada gambar
            </div>
          </div>

          <!-- Detail -->
          <div class="col-md-7">
            <ul class="list-group mb-3">
              <li class="list-group-item">
                <strong>Gedung:</strong> {{ kelas.gedung?.nama_gedung || 'Tidak diketahui' }}
              </li>
              <li class="list-group-item">
                <strong>PIC:</strong> {{ kelas.pic || '-' }}
              </li>
              <li class="list-group-item">
                <strong>Layout:</strong> {{ kelas.layout || '-' }}
              </li>
              <li class="list-group-item">
                <strong>Standar Operasional:</strong> {{ kelas.standar_operasional || '-' }}
              </li>
            </ul>

            <!-- Tombol Aksi -->
            <div class="d-flex gap-2">
              <router-link
                :to="{ name: 'CreatePengaduan', params: { kelasId: kelas.id } }"
                class="btn btn-primary"
              >
                Buat Pengaduan untuk Kelas Ini
              </router-link>

              <router-link
                to="/lihat-kelas"
                class="btn btn-outline-secondary"
              >
                Kembali
              </router-link>
            </div>
          </div>
        </div>
      </div>

      <div v-else class="alert alert-danger text-center">
        Data kelas tidak ditemukan.
      </div>
    </div>
  </MainLayout>
</template>

<script>
import axios from "axios";
import MainLayout from "../layouts/MainLayout.vue";

export default {
  components: { MainLayout },
  props: ["id"], // pastikan route pakai props: true
  data() {
    return {
      kelas: null,
      loading: true,
    };
  },
  async mounted() {
    try {
      const kelasId = this.id || this.$route.params.id; // fallback kalau props tidak dikirim
      const res = await axios.get(`http://localhost:8000/api/kelas/${kelasId}`);
      console.log("Detail Kelas API response:", res.data);

      this.kelas = res.data.data; // ambil dari 'data'
    } catch (error) {
      console.error("Gagal ambil detail kelas:", error);
    } finally {
      this.loading = false;
    }
  },
};
</script>
