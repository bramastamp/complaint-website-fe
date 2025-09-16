<template>
  <MainLayout>
    <div class="container p-4 font-poppins">
      <h2 class="text-2xl fw-semibold mb-4">Pilih Gedung</h2>

      <!-- Daftar Gedung -->
      <div class="d-flex flex-wrap gap-3">
        <button
          v-for="gedung in gedungList"
          :key="gedung.id"
          class="btn btn-outline-primary shadow-sm px-4 py-2 fw-semibold"
          @click="selectGedung(gedung.id, gedung.nama_gedung)"
        >
          {{ gedung.nama_gedung }}
        </button>
      </div>

      <!-- Loading Indicator -->
      <div v-if="loadingKelas" class="mt-5 text-center">
        <div class="spinner-border text-primary" role="status">
          <span class="visually-hidden">Memuat kelas...</span>
        </div>
        <p class="mt-2">Memuat kelas di {{ selectedGedung?.nama_gedung }}...</p>
      </div>

      <!-- Daftar Kelas -->
      <div v-else-if="kelasList.length > 0" class="mt-5">
        <h3 class="h5 fw-semibold mb-3">
          Kelas di {{ selectedGedung?.nama_gedung }}
        </h3>
        <ul class="list-group shadow-sm rounded">
          <li
            v-for="kelas in kelasList"
            :key="kelas.id"
            class="list-group-item list-group-item-action kelas-item"
            @click="goToDetail(kelas.id)"
          >
            {{ kelas.nama_kelas }}
          </li>
        </ul>
      </div>

      <!-- Jika tidak ada kelas -->
      <div v-else-if="selectedGedung && kelasKosong" class="mt-5 alert alert-warning">
        {{ selectedGedung?.nama_gedung }} belum memiliki kelas.
      </div>
    </div>
  </MainLayout>
</template>

<script>
import axios from "axios";
import MainLayout from "../layouts/MainLayout.vue";

export default {
  components: { MainLayout },
  data() {
    return {
      gedungList: [],
      kelasList: [],
      selectedGedung: null,
      loadingKelas: false,
      kelasKosong: false,
    };
  },
  async mounted() {
    try {
      const res = await axios.get("http://localhost:8000/api/gedung");
      this.gedungList = res.data.data || [];
    } catch (error) {
      console.error("Gagal ambil gedung:", error);
    }
  },
  methods: {
    async selectGedung(id, namaGedung) {
      this.loadingKelas = true;
      this.kelasList = [];
      this.kelasKosong = false;
      this.selectedGedung = { id, nama_gedung: namaGedung };

      try {
        const res = await axios.get(`http://localhost:8000/api/gedung/${id}/kelas`);
        this.kelasList = res.data.kelas || [];
        this.kelasKosong = this.kelasList.length === 0;
      } catch (error) {
        console.error("Gagal ambil kelas:", error);
        this.kelasKosong = true;
      } finally {
        this.loadingKelas = false;
      }
    },
    goToDetail(id) {
      this.$router.push({ name: "DetailKelas", params: { id } });
    },
  },
};
</script>

<style scoped>
/* Import Google Fonts */
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@400;500;600;700&display=swap');

.font-poppins {
  font-family: 'Poppins', sans-serif;
}

/* Hover efek untuk daftar kelas */
.kelas-item {
  cursor: pointer;
  transition: background-color 0.2s ease, transform 0.15s ease;
}
.kelas-item:hover {
  background-color: #f0f4ff;
  transform: translateX(4px);
}
</style>
