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
  <input 
    type="date" 
    class="date-input" 
    v-model="startDate" 
    :max="endDate" 
  />
</div>

<div class="filter-group">
  <label class="filter-label">Đến ngày</label>
  <input 
    type="date" 
    class="date-input" 
    v-model="endDate" 
    :min="startDate" 
  />
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

      <!-- Reviews Table -->
      <div class="reviews-section">
        <div v-if="danhSachLoc.length === 0" class="empty-state">
          <div class="empty-icon">
            <i class="fas fa-comments"></i>
          </div>
          <h3>Không có đánh giá phù hợp</h3>
          <p>Thử thay đổi bộ lọc để xem thêm đánh giá</p>
        </div>

        <div v-else class="table-section">
          <div class="table-header">
            <h3><i class="fas fa-table me-2"></i>Danh sách đánh giá ({{ danhSachLoc.length }} kết quả)</h3>
          </div>
          <div class="table-wrapper">
            <table class="reviews-table">
              <thead>
                <tr>
                  <th class="col-stt">#</th>
                  <th class="col-service">Dịch vụ</th>
                  <th class="col-user">Người dùng</th>
                  <th class="col-rating">Đánh giá</th>
                  <th class="col-content">Nội dung</th>
                  <th class="col-date">Ngày tạo</th>
                  <th class="col-status">Trạng thái</th>
                  <th class="col-actions">Thao tác</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(dg, index) in danhSachLoc" :key="dg.id" class="review-row">
                  <td class="stt-cell">{{ index + 1 }}</td>
                  
                  <td class="service-cell">
                    <div class="service-info">
                      <div class="service-badge">
                        <i class="fas fa-spa"></i>
                        <span>{{ dg.dichVu?.tenDichVu || "Không rõ" }}</span>
                      </div>
                    </div>
                  </td>
                  
                  <td class="user-cell">
                    <div class="user-info">
                      <div class="user-avatar">
                        <i class="fas fa-user"></i>
                      </div>
                      <div class="user-details">
                        <span v-if="dg.anDanh" class="anonymous">
                          <i class="fas fa-user-secret me-1"></i>Ẩn danh
                        </span>
                        <span v-else class="username">{{ dg.user?.name || "Chưa rõ" }}</span>
                      </div>
                    </div>
                  </td>
                  
                  <td class="rating-cell">
                    <div class="rating-display">
                      <div class="stars">
                        <i
                          v-for="n in 5"
                          :key="n"
                          class="fas fa-star"
                          :class="n <= dg.soSao ? 'star-filled' : 'star-empty'"
                        ></i>
                      </div>
                      <span class="rating-text">{{ dg.soSao }}/5</span>
                    </div>
                  </td>
                  
                  <td class="content-cell">
                    <div class="review-content">
                      <p>{{ dg.noiDung || "(Không có nội dung)" }}</p>
                    </div>
                  </td>
                  
                  <td class="date-cell">
                    <div class="date-display">
                      <i class="fas fa-calendar-alt"></i>
                      <span>{{ formatDate(dg.ngayTao) }}</span>
                    </div>
                  </td>
                  
                  <td class="status-cell">
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
                  </td>
                  
                  <td class="actions-cell">
                    <div class="action-buttons">
                      <button
                        v-if="!dg.daDuyet"
                        @click="duyetDanhGia(dg.id)"
                        class="action-btn approve-btn"
                        title="Duyệt đánh giá"
                      >
                        <i class="fas fa-check"></i>
                      </button>
                      <button
                        @click="toggleTrangThai(dg.id)"
                        class="action-btn toggle-btn"
                        :title="dg.isActive ? 'Ẩn đánh giá' : 'Hiển thị lại đánh giá'"
                      >
                        <i
                          :class="dg.isActive ? 'fas fa-eye-slash' : 'fas fa-eye'"
                        ></i>
                      </button>
                    </div>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>

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

/* Table Section */
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

.table-section {
  background: white;
  border-radius: 20px;
  box-shadow: 0 15px 35px rgba(0, 0, 0, 0.1);
  overflow: hidden;
}

.table-header {
  background: linear-gradient(135deg, #f8fafc 0%, #e2e8f0 100%);
  padding: 1.5rem 2rem;
  border-bottom: 1px solid #e2e8f0;
}

.table-header h3 {
  margin: 0;
  color: #1e293b;
  font-size: 1.3rem;
  font-weight: 600;
  display: flex;
  align-items: center;
}

.table-wrapper {
  overflow-x: auto;
  max-height: 70vh;
  overflow-y: auto;
}

.reviews-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 0.9rem;
}

.reviews-table thead {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  position: sticky;
  top: 0;
  z-index: 10;
}

.reviews-table th {
  padding: 1.2rem 1rem;
  text-align: left;
  font-weight: 600;
  font-size: 0.85rem;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  border-right: 1px solid rgba(255, 255, 255, 0.1);
}

.reviews-table th:last-child {
  border-right: none;
}

/* Column Widths */
.col-stt { width: 50px; text-align: center; }
.col-service { width: 180px; }
.col-user { width: 150px; }
.col-rating { width: 120px; }
.col-content { width: 300px; }
.col-date { width: 160px; }
.col-status { width: 120px; }
.col-actions { width: 100px; text-align: center; }

.reviews-table tbody tr {
  transition: all 0.3s ease;
  border-bottom: 1px solid #f1f5f9;
}

.reviews-table tbody tr:hover {
  background: linear-gradient(135deg, #f8fafc 0%, #f1f5f9 100%);
  transform: scale(1.001);
  box-shadow: inset 0 0 0 1px rgba(102, 126, 234, 0.1);
}

.reviews-table td {
  padding: 1rem;
  vertical-align: middle;
  border-right: 1px solid #f1f5f9;
}

.reviews-table td:last-child {
  border-right: none;
}

/* Cell Styles */
.stt-cell {
  text-align: center;
  font-weight: 600;
  color: #6b7280;
}

.service-info {
  display: flex;
  align-items: center;
}

.service-badge {
  background: linear-gradient(135deg, #e0f2fe 0%, #b3e5fc 100%);
  color: #0277bd;
  padding: 0.4rem 0.8rem;
  border-radius: 15px;
  font-size: 0.75rem;
  font-weight: 600;
  display: flex;
  align-items: center;
  gap: 0.3rem;
}

.user-info {
  display: flex;
  align-items: center;
  gap: 0.8rem;
}

.user-avatar {
  width: 35px;
  height: 35px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
  font-size: 0.9rem;
}

.username {
  font-weight: 600;
  color: #1f2937;
}

.anonymous {
  color: #6b7280;
  font-style: italic;
  font-size: 0.85rem;
}

.rating-display {
  display: flex;
  flex-direction: column;
  gap: 0.3rem;
}

.stars {
  display: flex;
  gap: 0.1rem;
}

.star-filled {
  color: #fbbf24;
}

.star-empty {
  color: #d1d5db;
}

.rating-text {
  font-size: 0.75rem;
  color: #6b7280;
  font-weight: 600;
}

.review-content {
  max-width: 300px;
}

.review-content p {
  margin: 0;
  color: #374151;
  line-height: 1.5;
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
  text-overflow: ellipsis;
}

.date-display {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  color: #6b7280;
  font-size: 0.85rem;
}

.date-display i {
  color: #9ca3af;
}

.status-badge {
  padding: 0.4rem 0.8rem;
  border-radius: 15px;
  font-size: 0.75rem;
  font-weight: 600;
  display: flex;
  align-items: center;
  gap: 0.3rem;
  width: fit-content;
}

.status-badge.approved {
  background: linear-gradient(135deg, #d1fae5 0%, #a7f3d0 100%);
  color: #059669;
}

.status-badge.pending {
  background: linear-gradient(135deg, #fef3c7 0%, #fde68a 100%);
  color: #d97706;
}

.action-buttons {
  display: flex;
  gap: 0.5rem;
  justify-content: center;
}

.action-btn {
  width: 35px;
  height: 35px;
  border: none;
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.3s ease;
  font-size: 0.9rem;
}

.approve-btn {
  background: linear-gradient(135deg, #10b981 0%, #059669 100%);
  color: white;
}

.approve-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 5px 15px rgba(16, 185, 129, 0.4);
}

.toggle-btn {
  background: linear-gradient(135deg, #6b7280 0%, #4b5563 100%);
  color: white;
}

.toggle-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 5px 15px rgba(107, 114, 128, 0.4);
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

/* Scrollbar Styling */
.table-wrapper::-webkit-scrollbar {
  width: 8px;
  height: 8px;
}

.table-wrapper::-webkit-scrollbar-track {
  background: #f1f5f9;
  border-radius: 4px;
}

.table-wrapper::-webkit-scrollbar-thumb {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 4px;
}

.table-wrapper::-webkit-scrollbar-thumb:hover {
  background: linear-gradient(135deg, #5a6fd8 0%, #6b4190 100%);
}

/* Responsive */
@media (max-width: 1200px) {
  .filter-row {
    grid-template-columns: 1fr 1fr;
    gap: 1.5rem;
  }
  
  .reviews-table {
    font-size: 0.85rem;
  }
  
  .reviews-table th,
  .reviews-table td {
    padding: 0.8rem 0.6rem;
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
  
  .reviews-table {
    font-size: 0.8rem;
  }
  
  .reviews-table th,
  .reviews-table td {
    padding: 0.6rem 0.4rem;
  }
  
  .col-content {
    width: 200px;
  }
  
  .review-content p {
    -webkit-line-clamp: 2;
  }
  
  .charts-grid {
    grid-template-columns: 1fr;
  }
  
  .user-info {
    flex-direction: column;
    gap: 0.4rem;
    align-items: flex-start;
  }
  
  .user-avatar {
    width: 30px;
    height: 30px;
    font-size: 0.8rem;
  }
  
  .action-buttons {
    flex-direction: column;
    gap: 0.3rem;
  }
  
  .action-btn {
    width: 30px;
    height: 30px;
    font-size: 0.8rem;
  }
}
</style>
