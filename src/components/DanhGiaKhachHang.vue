<template>
  <div class="review-management">
    <!-- Header Section -->
    <div class="page-header">
      <div class="container">
        <div class="row align-items-center">
          <div class="col-md-8">
            <div class="d-flex align-items-center">
              <div class="header-icon me-3">
                <i class="fas fa-star-half-alt"></i>
              </div>
              <div>
                <h1 class="page-title mb-2">Quản lý đánh giá khách hàng</h1>
                <p class="page-subtitle mb-0">Theo dõi và quản lý phản hồi từ khách hàng</p>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div class="container">
      <!-- Advanced Filter Section -->
      <div class="filter-section">
        <div class="card shadow-lg border-0">
          <div class="card-header bg-gradient-primary text-white">
            <h5 class="mb-0"><i class="fas fa-filter me-2"></i>Bộ lọc nâng cao</h5>
          </div>
          <div class="card-body p-4">
            <div class="row g-3">
              <!-- Status Filter -->
              <div class="col-lg-6">
                <label class="form-label fw-semibold">Trạng thái đánh giá</label>
                <div class="btn-group w-100" role="group">
                  <input type="radio" class="btn-check" name="statusFilter" id="all" v-model="filter" value="all">
                  <label class="btn btn-outline-primary" for="all">
                    <i class="fas fa-list me-2"></i>Tất cả
                  </label>
                  <input type="radio" class="btn-check" name="statusFilter" id="pending" v-model="filter" value="chuaduyet">
                  <label class="btn btn-outline-warning" for="pending">
                    <i class="fas fa-clock me-2"></i>Chưa duyệt
                  </label>
                  <input type="radio" class="btn-check" name="statusFilter" id="approved" v-model="filter" value="daduyet">
                  <label class="btn btn-outline-success" for="approved">
                    <i class="fas fa-check me-2"></i>Đã duyệt
                  </label>
                </div>
              </div>

              <!-- Search Filter -->
              <div class="col-lg-6">
                <label class="form-label fw-semibold">Tên người dùng</label>
                <div class="input-group">
                  <span class="input-group-text bg-light border-end-0">
                    <i class="fas fa-search text-muted"></i>
                  </span>
                  <input
                    type="text"
                    class="form-control border-start-0 ps-0"
                    placeholder="Nhập tên người dùng..."
                    v-model="searchName"
                  />
                </div>
              </div>

              <!-- Date Range -->
              <div class="col-lg-3">
                <label class="form-label fw-semibold">Từ ngày</label>
                <input 
                  type="date" 
                  class="form-control" 
                  v-model="startDate" 
                  :max="endDate" 
                />
              </div>

              <div class="col-lg-3">
                <label class="form-label fw-semibold">Đến ngày</label>
                <input 
                  type="date" 
                  class="form-control" 
                  v-model="endDate" 
                  :min="startDate" 
                />
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Service Tabs Section - Horizontal Layout -->
      <div class="service-filter-section">
        <div class="card shadow-sm border-0">
          <div class="card-body p-3">
            <div class="d-flex align-items-center justify-content-between mb-3">
              <h6 class="mb-0 fw-semibold text-primary">
                <i class="fas fa-spa me-2"></i>Lọc theo dịch vụ
              </h6>
              <small class="text-muted">{{ danhSachLoc.length }} kết quả</small>
            </div>
            
            <div class="service-tabs-horizontal">
              <div class="nav nav-pills flex-nowrap overflow-auto" role="tablist">
                <button
                  class="nav-link flex-shrink-0"
                  :class="{ active: selectedDichVu === 'all' }"
                  @click="selectedDichVu = 'all'"
                >
                  <i class="fas fa-th-large me-1"></i>
                  Tất cả dịch vụ
                </button>
                <button
                  v-for="dv in dichVuTabs"
                  :key="dv"
                  class="nav-link flex-shrink-0"
                  :class="{ active: selectedDichVu === dv }"
                  @click="selectedDichVu = dv"
                >
                  <i class="fas fa-spa me-1"></i>
                  {{ dv }}
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Reviews Table -->
      <div class="reviews-section">
        <div v-if="danhSachLoc.length === 0" class="empty-state">
          <div class="card shadow-sm border-0 text-center py-5">
            <div class="card-body">
              <div class="empty-icon mb-3">
                <i class="fas fa-comments text-muted"></i>
              </div>
              <h4 class="text-muted mb-2">Không có đánh giá phù hợp</h4>
              <p class="text-muted mb-0">Thử thay đổi bộ lọc để xem thêm đánh giá</p>
            </div>
          </div>
        </div>

        <div v-else class="table-section">
          <div class="card shadow-lg border-0">
            <div class="card-header bg-gradient-info text-white">
              <h5 class="mb-0">
                <i class="fas fa-table me-2"></i>
                Danh sách đánh giá ({{ danhSachLoc.length }} kết quả)
              </h5>
            </div>
            <div class="card-body p-0">
              <div class="table-responsive" style="max-height: 70vh;">
                <table class="table table-hover mb-0 review-table">
                  <thead class="table-dark sticky-top">
                    <tr>
                      <th scope="col" class="text-center" style="width: 60px;">#</th>
                      <th scope="col" style="width: 200px;">Dịch vụ</th>
                      <th scope="col" style="width: 180px;">Người dùng</th>
                      <th scope="col" style="width: 140px;">Đánh giá</th>
                      <th scope="col" style="width: 300px;">Nội dung</th>
                      <th scope="col" style="width: 160px;">Ngày tạo</th>
                      <th scope="col" style="width: 130px;">Trạng thái</th>
                      <th scope="col" class="text-center" style="width: 120px;">Thao tác</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="(dg, index) in danhSachLocSorted" :key="dg.id" class="review-row">
                      <td class="text-center align-middle">
                        <span class="badge bg-secondary">{{ index + 1 }}</span>
                      </td>
                      
                      <td class="align-middle">
                        <span class="badge bg-info bg-opacity-10 text-info p-2">
                          <i class="fas fa-spa me-1"></i>
                          {{ dg.dichVu?.tenDichVu || "Không rõ" }}
                        </span>
                      </td>
                      
                      <td class="align-middle">
                        <div class="d-flex align-items-center">
                          <div class="user-avatar me-2">
                            <i class="fas fa-user text-white"></i>
                          </div>
                          <div>
                            <span v-if="dg.anDanh" class="text-muted fst-italic small">
                              <i class="fas fa-user-secret me-1"></i>Ẩn danh
                            </span>
                            <span v-else class="fw-medium">{{ dg.user?.name || "Chưa rõ" }}</span>
                          </div>
                        </div>
                      </td>
                      
                      <td class="align-middle">
                        <div class="rating-display">
                          <div class="stars mb-1">
                            <i
                              v-for="n in 5"
                              :key="n"
                              class="fas fa-star"
                              :class="n <= dg.soSao ? 'text-warning' : 'text-muted opacity-25'"
                            ></i>
                          </div>
                          <small class="text-muted">{{ dg.soSao }}/5</small>
                        </div>
                      </td>
                      
                      <td class="align-middle">
                        <div class="review-content">
                          <p class="mb-0 text-truncate-3">{{ dg.noiDung || "(Không có nội dung)" }}</p>
                        </div>
                      </td>
                      
                      <td class="align-middle">
                        <small class="text-muted d-flex align-items-center">
                          <i class="fas fa-calendar-alt me-1"></i>
                          {{ formatDate(dg.ngayTao) }}
                        </small>
                      </td>
                      
                      <td class="align-middle">
                        <span
                          class="badge"
                          :class="dg.daDuyet ? 'bg-success' : 'bg-warning'"
                        >
                          <i
                            :class="dg.daDuyet ? 'fas fa-check-circle' : 'fas fa-hourglass-start'"
                            class="me-1"
                          ></i>
                          {{ dg.daDuyet ? "Đã duyệt" : "Chưa duyệt" }}
                        </span>
                      </td>
                      
                      <td class="text-center align-middle">
                        <div class="btn-group btn-group-sm" role="group">
                          <button
                            v-if="!dg.daDuyet"
                            @click="duyetDanhGia(dg.id)"
                            class="btn btn-success btn-sm"
                            title="Duyệt đánh giá"
                          >
                            <i class="fas fa-check"></i>
                          </button>
                          <button
                            @click="toggleTrangThai(dg.id)"
                            class="btn btn-secondary btn-sm"
                            :title="dg.isActive ? 'Ẩn đánh giá' : 'Hiển thị lại đánh giá'"
                          >
                            <i :class="dg.isActive ? 'fas fa-eye-slash' : 'fas fa-eye'"></i>
                          </button>
                        </div>
                      </td>
                    </tr>
                  </tbody>
                </table>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Charts Section -->
      <div class="charts-section mt-5">
        <div class="row">
          <div class="col-12 mb-4">
            <h2 class="text-white fw-bold">
              <i class="fas fa-chart-bar me-2"></i>Thống kê đánh giá
            </h2>
          </div>
        </div>
        <div class="row g-4">
          <div class="col-lg-6">
            <div class="card shadow-lg border-0 h-100">
              <div class="card-header bg-gradient-primary text-white">
                <h5 class="mb-0">
                  <i class="fas fa-chart-column me-2"></i>Dịch vụ được đánh giá nhiều
                </h5>
              </div>
              <div class="card-body">
                <BarChart :data="chartDataSoLuong" :options="chartOptions" />
              </div>
            </div>
          </div>
          <div class="col-lg-6">
            <div class="card shadow-lg border-0 h-100">
              <div class="card-header bg-gradient-success text-white">
                <h5 class="mb-0">
                  <i class="fas fa-star me-2"></i>Dịch vụ được đánh giá tốt
                </h5>
              </div>
              <div class="card-body">
                <BarChart :data="chartDataTrungBinh" :options="chartOptions" />
              </div>
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

// Sắp xếp để đưa đánh giá chưa duyệt lên đầu
const danhSachLocSorted = computed(() => {
  return [...danhSachLoc.value].sort((a, b) => {
    // Đánh giá chưa duyệt lên đầu
    if (!a.daDuyet && b.daDuyet) return -1;
    if (a.daDuyet && !b.daDuyet) return 1;
    // Sắp xếp theo ngày tạo (mới nhất trước)
    return new Date(b.ngayTao) - new Date(a.ngayTao);
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

/* Header Section */
.page-header {
  padding: 3rem 0;
  color: white;
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

.bg-gradient-primary {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%) !important;
}

.bg-gradient-info {
  background: linear-gradient(135deg, #17a2b8 0%, #138496 100%) !important;
}

.bg-gradient-success {
  background: linear-gradient(135deg, #28a745 0%, #20c997 100%) !important;
}

/* Service Filter Section */
.service-filter-section {
  margin-bottom: 2rem;
}

.service-tabs-horizontal {
  position: relative;
}

.service-tabs-horizontal .nav {
  padding-bottom: 0.5rem;
  scrollbar-width: thin;
  scrollbar-color: #667eea transparent;
}

.service-tabs-horizontal .nav::-webkit-scrollbar {
  height: 4px;
}

.service-tabs-horizontal .nav::-webkit-scrollbar-track {
  background: #f8f9fa;
  border-radius: 2px;
}

.service-tabs-horizontal .nav::-webkit-scrollbar-thumb {
  background: #667eea;
  border-radius: 2px;
}

.service-tabs-horizontal .nav-link {
  border-radius: 20px;
  margin-right: 0.5rem;
  padding: 0.5rem 1rem;
  font-size: 0.9rem;
  white-space: nowrap;
  background: #f8f9fa;
  border: 1px solid #dee2e6;
  color: #6c757d;
  transition: all 0.3s ease;
}

.service-tabs-horizontal .nav-link:hover,
.service-tabs-horizontal .nav-link.active {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-color: #667eea;
  color: white;
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(102, 126, 234, 0.3);
}

/* Table Improvements */
.review-table {
  font-size: 0.9rem;
}

.review-row {
  transition: all 0.2s ease;
  cursor: default;
}

.review-row:hover {
  background: linear-gradient(90deg, rgba(102, 126, 234, 0.05) 0%, rgba(255, 255, 255, 0.8) 100%) !important;
  box-shadow: inset 3px 0 0 #667eea;
}

.user-avatar {
  width: 35px;
  height: 35px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 0.9rem;
}

.text-truncate-3 {
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
  line-height: 1.4;
  max-height: 4.2em;
}

/* Empty State */
.empty-icon {
  font-size: 4rem;
  color: #6c757d;
  opacity: 0.5;
}

/* Charts Section */
.charts-section {
  padding-bottom: 3rem;
}

/* Responsive Adjustments */
@media (max-width: 768px) {
  .page-title {
    font-size: 1.8rem;
  }
  
  .review-table {
    font-size: 0.8rem;
  }
  
  .user-avatar {
    width: 30px;
    height: 30px;
    font-size: 0.8rem;
  }
  
  .text-truncate-3 {
    -webkit-line-clamp: 2;
    max-height: 2.8em;
  }
  
  .service-tabs-horizontal .nav-link {
    font-size: 0.8rem;
    padding: 0.4rem 0.8rem;
  }
}

/* Custom Bootstrap Overrides */
.btn-check:checked + .btn-outline-primary {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-color: #667eea;
}

.btn-check:checked + .btn-outline-warning {
  background: linear-gradient(135deg, #ffc107 0%, #e0a800 100%);
  border-color: #ffc107;
}

.btn-check:checked + .btn-outline-success {
  background: linear-gradient(135deg, #28a745 0%, #20c997 100%);
  border-color: #28a745;
}
</style>