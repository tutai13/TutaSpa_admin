<template>
  <div class="review-management">
    <!-- Header Section -->
    <div class="page-header">
      <div class="container">
        <div class="header-content">
          <div class="header-info">
            <div class="header-icon">
              <i class="fas fa-star-half-alt"></i>
            </div>
            <div>
              <h1 class="page-title">Quản lý đánh giá khách hàng</h1>
              <p class="page-subtitle">Theo dõi và quản lý phản hồi từ khách hàng</p>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div class="container">
      <!-- Advanced Filter Section -->
      <div class="filter-section">
        <div class="filter-card">
          <div class="filter-header">
            <h3><i class="fas fa-filter me-2"></i>Bộ lọc nâng cao</h3>
          </div>
          <div class="filter-content">
            <div class="filter-row">
              <div class="filter-group">
                <label class="filter-label">Trạng thái đánh giá</label>
                <div class="status-buttons">
                  <button
                    class="status-btn"
                    :class="{ active: filter === 'all' }"
                    @click="filter = 'all'"
                  >
                    <i class="fas fa-list me-2"></i>Tất cả
                  </button>
                  <button
                    class="status-btn warning"
                    :class="{ active: filter === 'chuaduyet' }"
                    @click="filter = 'chuaduyet'"
                  >
                    <i class="fas fa-clock me-2"></i>Chưa duyệt
                  </button>
                  <button
                    class="status-btn success"
                    :class="{ active: filter === 'daduyet' }"
                    @click="filter = 'daduyet'"
                  >
                    <i class="fas fa-check me-2"></i>Đã duyệt
                  </button>
                </div>
              </div>

              <div class="filter-group">
                <label class="filter-label">Tên người dùng</label>
                <div class="search-input">
                  <i class="fas fa-search"></i>
                  <input
                    type="text"
                    placeholder="Nhập tên người dùng..."
                    v-model="searchName"
                  />
                </div>
              </div>

              <div class="filter-group">
                <label class="filter-label">Từ ngày</label>
                <input type="date" class="date-input" v-model="startDate" />
              </div>

              <div class="filter-group">
                <label class="filter-label">Đến ngày</label>
                <input type="date" class="date-input" v-model="endDate" />
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Service Tabs Section -->
      <div class="tabs-section">
        <button class="toggle-tabs-btn" @click="toggleTabs">
          <i class="fas fa-th-grid me-2"></i>
          <span>Lọc theo dịch vụ</span>
          <i
            class="fas fa-chevron-down toggle-icon"
            :class="{ rotated: showTabs }"
          ></i>
        </button>

        <transition name="slide-down">
          <div v-if="showTabs" class="service-tabs">
            <button
              class="service-tab"
              :class="{ active: selectedDichVu === 'all' }"
              @click="selectedDichVu = 'all'"
            >
              Tất cả dịch vụ
            </button>
            <button
              v-for="dv in dichVuTabs"
              :key="dv"
              class="service-tab"
              :class="{ active: selectedDichVu === dv }"
              @click="selectedDichVu = dv"
            >
              {{ dv }}
            </button>
          </div>
        </transition>
      </div>

      <!-- Reviews List -->
      <div class="reviews-section">
        <div v-if="danhSachLoc.length === 0" class="empty-state">
          <div class="empty-icon">
            <i class="fas fa-comments"></i>
          </div>
          <h3>Không có đánh giá phù hợp</h3>
          <p>Thử thay đổi bộ lọc để xem thêm đánh giá</p>
        </div>

        <div v-else class="reviews-grid">
          <div
            v-for="dg in danhSachLoc"
            :key="dg.id"
            class="review-card"
          >
            <div class="review-header">
              <div class="service-info">
                <div class="service-badge">
                  <i class="fas fa-spa me-1"></i>
                  {{ dg.dichVu?.tenDichVu || "Không rõ" }}
                </div>
                <div class="review-date">{{ formatDate(dg.ngayTao) }}</div>
              </div>
              
              <div class="review-status">
                <span
                  class="status-badge"
                  :class="dg.daDuyet ? 'approved' : 'pending'"
                >
                  <i
                    :class="
                      dg.daDuyet
                        ? 'fas fa-check-circle'
                        : 'fas fa-hourglass-start'
                    "
                  ></i>
                  {{ dg.daDuyet ? "Đã duyệt" : "Chưa duyệt" }}
                </span>
              </div>
            </div>

            <div class="review-body">
              <div class="user-info">
                <div class="user-avatar">
                  <i class="fas fa-user"></i>
                </div>
                <div class="user-details">
                  <div class="user-name">
                    <span v-if="dg.anDanh" class="anonymous">
                      <i class="fas fa-user-secret me-1"></i>Ẩn danh
                    </span>
                    <span v-else>{{ dg.user?.name || "Chưa rõ" }}</span>
                  </div>
                  <div class="rating">
                    <i
                      v-for="n in 5"
                      :key="n"
                      class="fas fa-star"
                      :class="
                        n <= dg.soSao ? 'star-filled' : 'star-empty'
                      "
                    ></i>
                    <span class="rating-text">{{ dg.soSao }}/5</span>
                  </div>
                </div>
              </div>

              <div class="review-content">
                <p>{{ dg.noiDung || "(Không có nội dung)" }}</p>
              </div>
            </div>

            <div class="review-actions">
              <button
                v-if="!dg.daDuyet"
                @click="duyetDanhGia(dg.id)"
                class="action-btn approve-btn"
                title="Duyệt đánh giá"
              >
                <i class="fas fa-check"></i>
                <span>Duyệt</span>
              </button>
              <button
                @click="toggleTrangThai(dg.id)"
                class="action-btn toggle-btn"
                :title="dg.isActive ? 'Ẩn đánh giá' : 'Hiển thị lại đánh giá'"
              >
                <i
                  :class="dg.isActive ? 'fas fa-eye-slash' : 'fas fa-eye'"
                ></i>
                <span>{{ dg.isActive ? 'Ẩn' : 'Hiện' }}</span>
              </button>
            </div>
          </div>
        </div>
      </div>

      <!-- Reviews Table -->
<!-- <div class="reviews-section">
  <div v-if="danhSachLoc.length === 0" class="empty-state">
    <div class="empty-icon">
      <i class="fas fa-comments"></i>
    </div>
    <h3>Không có đánh giá phù hợp</h3>
    <p>Thử thay đổi bộ lọc để xem thêm đánh giá</p>
  </div>

  <div v-else class="table-responsive">
    <table class="table table-striped table-hover align-middle">
      <thead class="table-primary">
        <tr>
          <th>#</th>
          <th>Dịch vụ</th>
          <th>Ngày tạo</th>
          <th>Người dùng</th>
          <th>Số sao</th>
          <th>Nội dung</th>
          <th>Trạng thái</th>
          <th>Hoạt động</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(dg, index) in danhSachLoc" :key="dg.id">
          <td>{{ index + 1 }}</td>
          <td>{{ dg.dichVu?.tenDichVu || "Không rõ" }}</td>
          <td>{{ formatDate(dg.ngayTao) }}</td>
          <td>
            <span v-if="dg.anDanh">
              <i class="fas fa-user-secret text-muted me-1"></i>Ẩn danh
            </span>
            <span v-else>{{ dg.user?.name || "Chưa rõ" }}</span>
          </td>
          <td>
            <i
              v-for="n in 5"
              :key="n"
              class="fas fa-star"
              :class="n <= dg.soSao ? 'text-warning' : 'text-muted'"
            ></i>
            <span class="ms-1">{{ dg.soSao }}/5</span>
          </td>
          <td>{{ dg.noiDung || "(Không có nội dung)" }}</td>
          <td>
            <span
              class="badge"
              :class="dg.daDuyet ? 'bg-success' : 'bg-warning text-dark'"
            >
              {{ dg.daDuyet ? "Đã duyệt" : "Chưa duyệt" }}
            </span>
          </td>
          <td>
            <button
              v-if="!dg.daDuyet"
              @click="duyetDanhGia(dg.id)"
              class="btn btn-sm btn-success me-1"
            >
              <i class="fas fa-check"></i>
            </button>
            <button
              @click="toggleTrangThai(dg.id)"
              class="btn btn-sm btn-secondary"
            >
              <i
                :class="dg.isActive ? 'fas fa-eye-slash' : 'fas fa-eye'"
              ></i>
            </button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</div> -->


      <!-- Charts Section -->
      <div class="charts-section">
        <div class="section-header">
          <h2><i class="fas fa-chart-bar me-2"></i>Thống kê đánh giá</h2>
        </div>
        <div class="charts-grid">
          <div class="chart-card">
            <div class="chart-header">
              <h3><i class="fas fa-chart-column me-2"></i>Dịch vụ được đánh giá nhiều</h3>
            </div>
            <div class="chart-content">
              <BarChart :data="chartDataSoLuong" :options="chartOptions" />
            </div>
          </div>
          <div class="chart-card">
            <div class="chart-header">
              <h3><i class="fas fa-star me-2"></i>Dịch vụ được đánh giá tốt</h3>
            </div>
            <div class="chart-content">
              <BarChart :data="chartDataTrungBinh" :options="chartOptions" />
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from "vue";
import BarChart from "@/components/BarChart.vue";
import axiosClient from "../utils/axiosClient";

const showTabs = ref(false);
const danhSach = ref([]);
const filter = ref("all");
const searchName = ref("");
const startDate = ref("");
const endDate = ref("");
const selectedDichVu = ref("all");

const chartDataSoLuong = ref({ labels: [], datasets: [] });
const chartDataTrungBinh = ref({ labels: [], datasets: [] });

const chartOptions = {
  responsive: true,
  plugins: { legend: { display: false } },
  scales: { y: { beginAtZero: true } },
};

onMounted(async () => await loadDanhSach());

const toggleTabs = () => {
  showTabs.value = !showTabs.value;
  selectedDichVu.value = "all";
};

const loadDanhSach = async () => {
  try {
    const res = await axiosClient.get("DanhGia/admin");
    danhSach.value = res;
    updateCharts();
  } catch (err) {
    console.error("Lỗi khi tải đánh giá:", err);
  }
};

const duyetDanhGia = async (id) => {
  if (confirm("Bạn có chắc muốn duyệt đánh giá này?")) {
    try {
      await axiosClient.put(`DanhGia/duyet/${id}`);
      await loadDanhSach();
    } catch (err) {
      console.error("Lỗi khi duyệt:", err);
    }
  }
};

const toggleTrangThai = async (id) => {
  if (confirm("Bạn có chắc muốn thay đổi trạng thái đánh giá này?")) {
    try {
      await axiosClient.put(`DanhGia/toggle/${id}`);
      await loadDanhSach();
    } catch (err) {
      console.error("Lỗi khi thay đổi trạng thái:", err);
    }
  }
};

const formatDate = (dateStr) => new Date(dateStr).toLocaleString("vi-VN");

const dichVuTabs = computed(() => {
  const tenDVs = danhSach.value
    .map((dg) => dg.dichVu?.tenDichVu)
    .filter(Boolean);
  return [...new Set(tenDVs)];
});

const danhSachLoc = computed(() => {
  return danhSach.value.filter((d) => {
    const matchFilter =
      filter.value === "all" ||
      (filter.value === "chuaduyet" && !d.daDuyet) ||
      (filter.value === "daduyet" && d.daDuyet);

    const matchSearch =
      !searchName.value ||
      (!d.anDanh &&
        d.user?.name?.toLowerCase().includes(searchName.value.toLowerCase()));

    const createdAt = new Date(d.ngayTao);
    const start = startDate.value ? new Date(startDate.value) : null;
    const end = endDate.value ? new Date(endDate.value + "T23:59:59") : null;

    const matchDate =
      (!start || createdAt >= start) && (!end || createdAt <= end);

    const matchDichVu =
      selectedDichVu.value === "all" ||
      d.dichVu?.tenDichVu === selectedDichVu.value;

    return matchFilter && matchSearch && matchDate && matchDichVu;
  });
});

const updateCharts = () => {
  const groupByService = {};
  danhSach.value.forEach((dg) => {
    const ten = dg.dichVu?.tenDichVu || "Không rõ";
    if (!groupByService[ten]) groupByService[ten] = { count: 0, total: 0 };
    groupByService[ten].count++;
    groupByService[ten].total += dg.soSao;
  });

  const labels = Object.keys(groupByService);
  const counts = labels.map((ten) => groupByService[ten].count);
  const averages = labels.map((ten) =>
    (groupByService[ten].total / groupByService[ten].count).toFixed(2)
  );

  chartDataSoLuong.value = {
    labels,
    datasets: [
      { label: "Số lượt đánh giá", backgroundColor: "#667eea", data: counts },
    ],
  };

  chartDataTrungBinh.value = {
    labels,
    datasets: [
      { label: "Điểm trung bình", backgroundColor: "#f093fb", data: averages },
    ],
  };
};
</script>

<style scoped>
/* Global Styles */
.review-management {
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}

.container {
  max-width: 1400px;
  margin: 0 auto;
  padding: 0 2rem;
}

/* Header Section */
.page-header {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  padding: 3rem 0;
  color: white;
}

.header-content {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.header-info {
  display: flex;
  align-items: center;
  gap: 1.5rem;
}

.header-icon {
  width: 60px;
  height: 60px;
  background: rgba(255, 255, 255, 0.2);
  border-radius: 15px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.8rem;
  backdrop-filter: blur(10px);
}

.page-title {
  font-size: 2.2rem;
  font-weight: 700;
  margin: 0;
  margin-bottom: 0.5rem;
}

.page-subtitle {
  font-size: 1.1rem;
  opacity: 0.9;
  margin: 0;
}

/* Filter Section */
.filter-section {
  margin-top: -2rem;
  margin-bottom: 2rem;
  position: relative;
  z-index: 10;
}

.filter-card {
  background: white;
  border-radius: 20px;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.1);
  overflow: hidden;
  backdrop-filter: blur(10px);
}

.filter-header {
  background: linear-gradient(135deg, #f8fafc 0%, #e2e8f0 100%);
  padding: 1.5rem 2rem;
  border-bottom: 1px solid #e2e8f0;
}

.filter-header h3 {
  margin: 0;
  color: #1e293b;
  font-size: 1.3rem;
  font-weight: 600;
}

.filter-content {
  padding: 2rem;
}

.filter-row {
  display: grid;
  grid-template-columns: 2fr 2fr 1fr 1fr;
  gap: 2rem;
  align-items: end;
}

.filter-group {
  display: flex;
  flex-direction: column;
  gap: 0.8rem;
}

.filter-label {
  font-weight: 600;
  color: #374151;
  font-size: 0.95rem;
  margin: 0;
}

.status-buttons {
  display: flex;
  gap: 0.5rem;
  flex-wrap: wrap;
}

.status-btn {
  padding: 0.8rem 1.2rem;
  border: 2px solid #e5e7eb;
  background: white;
  border-radius: 12px;
  font-weight: 500;
  transition: all 0.3s ease;
  cursor: pointer;
  display: flex;
  align-items: center;
  color: #6b7280;
}

.status-btn.active {
  border-color: #667eea;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  transform: translateY(-2px);
  box-shadow: 0 8px 25px rgba(102, 126, 234, 0.4);
}

.status-btn.warning.active {
  background: linear-gradient(135deg, #f59e0b 0%, #d97706 100%);
  border-color: #f59e0b;
  box-shadow: 0 8px 25px rgba(245, 158, 11, 0.4);
}

.status-btn.success.active {
  background: linear-gradient(135deg, #10b981 0%, #059669 100%);
  border-color: #10b981;
  box-shadow: 0 8px 25px rgba(16, 185, 129, 0.4);
}

.search-input {
  position: relative;
}

.search-input i {
  position: absolute;
  left: 1rem;
  top: 50%;
  transform: translateY(-50%);
  color: #9ca3af;
}

.search-input input {
  width: 100%;
  padding: 0.9rem 1rem 0.9rem 2.5rem;
  border: 2px solid #e5e7eb;
  border-radius: 12px;
  font-size: 0.95rem;
  transition: all 0.3s ease;
}

.search-input input:focus {
  outline: none;
  border-color: #667eea;
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
}

.date-input {
  padding: 0.9rem 1rem;
  border: 2px solid #e5e7eb;
  border-radius: 12px;
  font-size: 0.95rem;
  transition: all 0.3s ease;
}

.date-input:focus {
  outline: none;
  border-color: #667eea;
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
}

/* Tabs Section */
.tabs-section {
  margin-bottom: 2rem;
}

.toggle-tabs-btn {
  background: white;
  border: 2px solid #e5e7eb;
  padding: 1rem 1.5rem;
  border-radius: 12px;
  font-weight: 500;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 0.8rem;
  transition: all 0.3s ease;
  font-size: 1rem;
  color: #374151;
}

.toggle-tabs-btn:hover {
  border-color: #667eea;
  background: #f8fafc;
}

.toggle-icon {
  transition: transform 0.3s ease;
}

.toggle-icon.rotated {
  transform: rotate(180deg);
}

.service-tabs {
  margin-top: 1rem;
  display: flex;
  flex-wrap: wrap;
  gap: 0.8rem;
}

.service-tab {
  background: white;
  border: 2px solid #e5e7eb;
  padding: 0.8rem 1.2rem;
  border-radius: 12px;
  cursor: pointer;
  font-weight: 500;
  transition: all 0.3s ease;
  color: #6b7280;
}

.service-tab.active,
.service-tab:hover {
  border-color: #667eea;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  transform: translateY(-2px);
  box-shadow: 0 8px 25px rgba(102, 126, 234, 0.4);
}

/* Reviews Section */
.reviews-section {
  margin-bottom: 3rem;
}

.empty-state {
  background: white;
  border-radius: 20px;
  padding: 4rem 2rem;
  text-align: center;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
}

.empty-icon {
  width: 80px;
  height: 80px;
  background: linear-gradient(135deg, #f3f4f6 0%, #e5e7eb 100%);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 2rem;
  color: #9ca3af;
  margin: 0 auto 1.5rem;
}

.empty-state h3 {
  color: #374151;
  margin-bottom: 0.5rem;
}

.empty-state p {
  color: #6b7280;
  margin: 0;
}

.reviews-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(400px, 1fr));
  gap: 1.5rem;
}

.review-card {
  background: white;
  border-radius: 20px;
  padding: 1.5rem;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.08);
  transition: all 0.3s ease;
  border: 1px solid #f1f5f9;
}

.review-card:hover {
  transform: translateY(-8px);
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.12);
}

.review-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 1.5rem;
}

.service-info {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.service-badge {
  background: linear-gradient(135deg, #e0f2fe 0%, #b3e5fc 100%);
  color: #0277bd;
  padding: 0.5rem 1rem;
  border-radius: 20px;
  font-size: 0.85rem;
  font-weight: 600;
  display: flex;
  align-items: center;
  width: fit-content;
}

.review-date {
  font-size: 0.8rem;
  color: #6b7280;
  margin-left: 0.5rem;
}

.status-badge {
  padding: 0.5rem 1rem;
  border-radius: 20px;
  font-size: 0.8rem;
  font-weight: 600;
  display: flex;
  align-items: center;
  gap: 0.3rem;
}

.status-badge.approved {
  background: linear-gradient(135deg, #d1fae5 0%, #a7f3d0 100%);
  color: #059669;
}

.status-badge.pending {
  background: linear-gradient(135deg, #fef3c7 0%, #fde68a 100%);
  color: #d97706;
}

.review-body {
  margin-bottom: 1.5rem;
}

.user-info {
  display: flex;
  align-items: center;
  gap: 1rem;
  margin-bottom: 1rem;
}

.user-avatar {
  width: 45px;
  height: 45px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
  font-size: 1.1rem;
}

.user-details {
  display: flex;
  flex-direction: column;
  gap: 0.3rem;
}

.user-name {
  font-weight: 600;
  color: #1f2937;
}

.anonymous {
  color: #6b7280;
  font-style: italic;
}

.rating {
  display: flex;
  align-items: center;
  gap: 0.3rem;
}

.star-filled {
  color: #fbbf24;
}

.star-empty {
  color: #d1d5db;
}

.rating-text {
  font-size: 0.85rem;
  color: #6b7280;
  margin-left: 0.5rem;
}

.review-content {
  background: #f8fafc;
  border-radius: 12px;
  padding: 1rem;
  border-left: 4px solid #667eea;
}

.review-content p {
  margin: 0;
  color: #374151;
  line-height: 1.6;
  font-style: italic;
}

.review-actions {
  display: flex;
  gap: 0.8rem;
}

.action-btn {
  padding: 0.8rem 1.2rem;
  border: none;
  border-radius: 12px;
  font-weight: 500;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 0.5rem;
  font-size: 0.9rem;
  transition: all 0.3s ease;
}

.approve-btn {
  background: linear-gradient(135deg, #10b981 0%, #059669 100%);
  color: white;
}

.approve-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 25px rgba(16, 185, 129, 0.4);
}

.toggle-btn {
  background: linear-gradient(135deg, #6b7280 0%, #4b5563 100%);
  color: white;
}

.toggle-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 25px rgba(107, 114, 128, 0.4);
}

/* Charts Section */
.charts-section {
  margin-top: 4rem;
}

.section-header {
  margin-bottom: 2rem;
}

.section-header h2 {
  color: white;
  font-size: 1.8rem;
  font-weight: 700;
  display: flex;
  align-items: center;
}

.charts-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(500px, 1fr));
  gap: 2rem;
}

.chart-card {
  background: white;
  border-radius: 20px;
  box-shadow: 0 15px 35px rgba(0, 0, 0, 0.1);
  overflow: hidden;
}

.chart-header {
  background: linear-gradient(135deg, #f8fafc 0%, #e2e8f0 100%);
  padding: 1.5rem 2rem;
  border-bottom: 1px solid #e2e8f0;
}

.chart-header h3 {
  margin: 0;
  color: #1e293b;
  font-size: 1.2rem;
  font-weight: 600;
  display: flex;
  align-items: center;
}

.chart-content {
  padding: 2rem;
}

/* Animations */
.slide-down-enter-active,
.slide-down-leave-active {
  transition: all 0.3s ease;
  transform-origin: top;
}

.slide-down-enter-from,
.slide-down-leave-to {
  opacity: 0;
  transform: scaleY(0);
}

/* Responsive */
@media (max-width: 1200px) {
  .filter-row {
    grid-template-columns: 1fr 1fr;
    gap: 1.5rem;
  }
}

@media (max-width: 768px) {
  .container {
    padding: 0 1rem;
  }
  
  .page-header {
    padding: 2rem 0;
  }
  
  .page-title {
    font-size: 1.8rem;
  }
  
  .filter-row {
    grid-template-columns: 1fr;
    gap: 1.5rem;
  }
  
  .reviews-grid {
    grid-template-columns: 1fr;
  }
  
  .charts-grid {
    grid-template-columns: 1fr;
  }}
</style>
