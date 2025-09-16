<template>
  <MainLayout>
    <div class="form-container container">
      <h2 class="mb-4 fw-semibold">Buat Pengaduan Baru</h2>

      <form
        @submit.prevent="submitForm"
        class="p-4 rounded bg-white"
        enctype="multipart/form-data"
      >
        <!-- Dropdown kelas (autofill tapi bisa diganti) -->
        <div class="mb-3">
          <label for="kelas" class="form-label">Pilih Kelas</label>
          <select id="kelas" v-model="kelas_id" class="form-select">
            <option disabled value="">Pilih kelas</option>
            <option v-for="item in kelasList" :key="item.id" :value="item.id">
              {{ item.nama_kelas }}
            </option>
          </select>
        </div>

        <div class="mb-3">
          <label for="kategori" class="form-label">Kategori</label>
          <select
            id="kategori"
            v-model="kategori_id"
            class="form-select"
            required
          >
            <option disabled value="">Pilih kategori</option>
            <option
              v-for="kategori in kategoriList"
              :key="kategori.id"
              :value="kategori.id"
            >
              {{ kategori.nama_kategori }}
            </option>
          </select>
        </div>

        <div class="mb-3">
          <label for="judul" class="form-label">Judul</label>
          <input
            type="text"
            id="judul"
            class="form-control"
            v-model="judul"
            required
          />
        </div>

        <div class="mb-3">
          <label for="isi" class="form-label">Isi Pengaduan</label>
          <textarea
            id="isi"
            class="form-control"
            rows="5"
            v-model="isi"
            required
          ></textarea>
        </div>

        <div class="mb-3">
          <label for="gambar" class="form-label">Upload Gambar</label>
          <input
            type="file"
            id="gambar"
            class="form-control"
            accept="image/*"
            @change="handleFileUpload"
          />
        </div>

        <!-- Preview gambar -->
        <div v-if="gambarPreview" class="mb-3">
          <p>Preview:</p>
          <img
            :src="gambarPreview"
            class="img-fluid rounded"
            style="max-height: 200px"
          />
        </div>

        <div class="form-check mb-4">
          <input
            class="form-check-input"
            type="checkbox"
            id="anonim"
            v-model="is_anonymous"
          />
          <label class="form-check-label" for="anonim">
            Kirim sebagai anonim
          </label>
        </div>

        <button type="submit" class="btn btn-primary w-100">
          Kirim Pengaduan
        </button>
      </form>
    </div>
  </MainLayout>
</template>

<script>
import axios from "axios";
import MainLayout from "../layouts/MainLayout.vue";
import Swal from "sweetalert2";

export default {
  components: { MainLayout },
  props: {
    kelasId: {
      type: [String, Number],
      default: null,
    },
  },
  data() {
    return {
      judul: "",
      isi: "",
      kategori_id: "",
      kelas_id: "",
      kelas: null,
      kelasList: [],
      is_anonymous: false,
      gambar: null,
      gambarPreview: null,
      kategoriList: [],
    };
  },
  methods: {
    handleFileUpload(event) {
      const file = event.target.files[0];
      if (file) {
        this.gambar = file;
        this.gambarPreview = URL.createObjectURL(file);
      }
    },
    async submitForm() {
      // ✅ Validasi manual dulu
      if (!this.kategori_id || !this.judul || !this.isi) {
        Swal.fire({
          icon: "warning",
          title: "Form belum lengkap",
          text: "Kategori, Judul, dan Isi pengaduan wajib diisi.",
        });
        return;
      }

      try {
        const token = localStorage.getItem("token");
        let formData = new FormData();

        formData.append("judul", this.judul);
        formData.append("isi", this.isi);
        formData.append("kategori_id", this.kategori_id);
        if (this.kelas_id) formData.append("kelas_id", this.kelas_id);
        formData.append("is_anonymous", this.is_anonymous ? 1 : 0);
        if (this.gambar) formData.append("gambar", this.gambar);

        await axios.post("http://localhost:8000/api/pengaduan", formData, {
          headers: {
            Authorization: `Bearer ${token}`,
            "Content-Type": "multipart/form-data",
          },
        });

        // ✅ Sweetalert sukses + redirect
        Swal.fire({
          icon: "success",
          title: "Berhasil!",
          text: "Pengaduan berhasil dikirim.",
          timer: 2000,
          showConfirmButton: false,
        }).then(() => {
          this.$router.push("/dashboard");
        });

        // reset form
        this.judul = "";
        this.isi = "";
        this.kategori_id = "";
        this.kelas_id = "";
        this.is_anonymous = false;
        this.gambar = null;
        this.gambarPreview = null;
      } catch (error) {
        Swal.fire({
          icon: "error",
          title: "Gagal",
          text:
            error.response?.data?.message ||
            "Gagal mengirim pengaduan. Silakan coba lagi.",
        });
        console.error(error.response?.data || error);
      }
    },
  },
  async created() {
    try {
      const resKategori = await axios.get("http://localhost:8000/api/kategori");
      this.kategoriList = resKategori.data;

      const resKelasAll = await axios.get("http://localhost:8000/api/kelas");
      this.kelasList = resKelasAll.data.data || [];

      if (this.kelasId) {
        const resKelas = await axios.get(
          `http://localhost:8000/api/kelas/${this.kelasId}`
        );
        this.kelas = resKelas.data.data;
        this.kelas_id = this.kelas.id;
      }
    } catch (error) {
      console.error("Gagal memuat data:", error);
    }
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
