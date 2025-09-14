<template>
  <MainLayout>
  <div class="container mt-5 form-container">
    <h2 class="mb-4 fw-semibold">Kelola Tanggapan Pengaduan</h2>

    <!-- Detail pengaduan -->
    <div v-if="pengaduan" class="mb-4 p-3 rounded bg-white shadow-sm">
      <h5 class="fw-semibold">Detail Pengaduan</h5>
      <p><strong>ID:</strong> {{ pengaduan.id }}</p>
      <p><strong>Isi Pengaduan:</strong> {{ pengaduan.isi }}</p>
    </div>

    <!-- Daftar tanggapan -->
    <div v-if="tanggapans.length" class="mb-4 p-3 rounded bg-light">
      <h5 class="fw-semibold">Daftar Tanggapan</h5>
      <ul class="list-group">
        <li
          v-for="t in tanggapans"
          :key="t.id"
          class="list-group-item d-flex justify-content-between align-items-start"
        >
          <div>
            <p class="mb-1">{{ t.isi_tanggapan }}</p>
            <small class="text-muted">Oleh: {{ t.user?.name || 'Anonim' }}</small>
          </div>
          <span class="badge bg-secondary rounded-pill">{{
            new Date(t.created_at).toLocaleDateString()
          }}</span>
        </li>
      </ul>
    </div>

    <!-- Form tanggapan baru -->
    <h3 class="mb-3">Tambah Tanggapan Baru</h3>
    <form @submit.prevent="submitTanggapan" class="p-4 rounded bg-white shadow-sm">
      <div class="mb-3">
        <label for="isi" class="form-label">Isi Tanggapan</label>
        <textarea
          id="isi"
          v-model="form.isi_tanggapan"
          class="form-control"
          rows="4"
          required
        ></textarea>
      </div>

      <button type="submit" class="btn btn-success w-100">
        Kirim Tanggapan
      </button>
    </form>
  </div>
  </MainLayout>
</template>

<script>
import axios from "axios";
import MainLayout from "../layouts/MainLayout.vue";

export default {
  components: {
    MainLayout
  },
  props: ["pengaduanId"],
  data() {
    return {
      tanggapans: [],
      form: {
        pengaduan_id: this.$route.params.pengaduanId,
        isi_tanggapan: "",
      },
      pengaduan: null,
    };
  },
  async mounted() {
    await this.fetchPengaduan();
    await this.fetchTanggapans();
  },
  methods: {
    async fetchPengaduan() {
      const token = localStorage.getItem("token");
      try {
        const res = await axios.get(
          `http://localhost:8000/api/pengaduan/${this.$route.params.pengaduanId}`,
          {
            headers: { Authorization: `Bearer ${token}` },
          }
        );
        this.pengaduan = res.data;
      } catch (error) {
        alert(
          "Gagal mengambil data pengaduan: " +
            (error.response?.data?.message || error.message)
        );
      }
    },
    async fetchTanggapans() {
      const token = localStorage.getItem("token");
      try {
        const res = await axios.get(
          `http://localhost:8000/api/pengaduan/${this.$route.params.pengaduanId}/tanggapan`,
          {
            headers: { Authorization: `Bearer ${token}` },
          }
        );
        this.tanggapans = res.data;
      } catch (error) {
        alert("Gagal mengambil data tanggapan");
      }
    },
    async submitTanggapan() {
      const token = localStorage.getItem("token");
      try {
        await axios.post(
          `http://localhost:8000/api/tanggapan`,
          {
            pengaduan_id: this.form.pengaduan_id,
            isi_tanggapan: this.form.isi_tanggapan,
          },
          {
            headers: { Authorization: `Bearer ${token}` },
          }
        );
        alert("Tanggapan berhasil dikirim");
        this.form.isi_tanggapan = "";
        this.fetchTanggapans();
      } catch (error) {
        alert(
          "Gagal mengirim tanggapan: " +
            (error.response?.data?.message || error.message)
        );
      }
    },
  },
};
</script>

<style scoped>
.form-container {
  font-family: "Poppins", sans-serif;
}

form {
  background-color: #fff;
  border-radius: 12px;
  box-shadow: 6px 6px 20px 0px rgba(0, 0, 0, 0.1);
}

input,
textarea {
  font-size: 0.95rem;
  border-radius: 8px;
}

button {
  border-radius: 8px;
  font-weight: 500;
  letter-spacing: 0.5px;
}
</style>
