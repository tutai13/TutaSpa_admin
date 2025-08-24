<template>
  <div>
    <!-- Tiêu đề + nút chuyển tab -->
    <div class="d-flex justify-content-between align-items-center mb-3">
      <h4 class="mb-0">📊 Quản lý</h4>
      <div class="btn-group">
        <button
          class="btn"
          :class="
            activeTab === 'lichHen' ? 'btn-primary' : 'btn-outline-primary'
          "
          @click="
            activeTab = 'lichHen';
            fetchDatLich();
          "
        >
          <i class="fas fa-calendar-alt"></i> Lịch hẹn
        </button>
        <button
          class="btn"
          :class="
            activeTab === 'hoaDon' ? 'btn-primary' : 'btn-outline-primary'
          "
          @click="
            activeTab = 'hoaDon';
            fetchHoaDon();
          "
        >
          <i class="fas fa-file-invoice"></i> Hóa đơn
        </button>
      </div>
    </div>

    <!-- TAB LỊCH HẸN -->
    <div v-if="activeTab === 'lichHen'">
      <div>
        <h4 class="mb-3">📅 Danh sách Đặt Lịch</h4>

        <!-- Bộ lọc -->
        <div class="d-flex mb-3 gap-2">
          <input
            type="date"
            v-model="fromDate"
            class="form-control w-auto"
            @change="fetchDatLich"
          />
          <input
            type="date"
            v-model="toDate"
            class="form-control w-auto"
            @change="fetchDatLich"
          />
          <input
            type="text"
            v-model="phoneFilter"
            class="form-control w-auto"
            placeholder="Nhập số điện thoại"
            @input="fetchDatLich"
          />
        </div>

        <!-- Thông báo lỗi -->
        <div v-if="error.lichHen" class="alert alert-danger">
          {{ error.lichHen }}
        </div>

        <!-- Trạng thái tải -->
        <div v-if="isLoading.lichHen" class="text-center">Đang tải...</div>
        <div v-else-if="datLichs.length === 0" class="text-center text-muted">
          Không có dữ liệu
        </div>
        <div v-else class="table-responsive">
          <table class="table table-bordered table-hover align-middle">
            <thead class="table-success">
              <tr>
                <th>#</th>
                <th>Thời gian</th>
                <th>SĐT</th>
                <th>Trạng thái</th>
                <th>Thanh toán</th>
                <th>Dịch vụ</th>
                <th>Ghi chú</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(dl, index) in datLichs" :key="dl.datLichID">
                <td>{{ index + 1 }}</td>
                <td>{{ formatDateTime(dl.thoiGian) }} ({{ dl.thoiLuong }}p)</td>
                <td>{{ dl.soDienThoai }}</td>
                <td>
                  <span
                    :class="
                      dl.trangThai === 'Đã đến'
                        ? 'text-success fw-bold'
                        : 'text-warning fw-bold'
                    "
                  >
                    {{ dl.trangThai }}
                  </span>
                </td>
                <td>
                  <span
                    :class="dl.daThanhToan ? 'text-success' : 'text-danger'"
                  >
                    {{ dl.daThanhToan ? "Đã thanh toán" : "Chưa thanh toán" }}
                  </span>
                </td>
                <td>
                  <ul class="mb-0 ps-3">
                    <li
                      v-for="ct in dl.chiTietDatLichs || []"
                      :key="ct.chiTietDatLichID"
                    >
                      {{ ct.soLuongDV }} x {{ ct.dichVu?.tenDichVu }} -
                      {{ formatCurrency(ct.dichVu?.gia) }}
                    </li>
                  </ul>
                </td>
                <td>{{ dl.ghiChu || "—" }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>

    <!-- TAB HÓA ĐƠN -->
    <div v-if="activeTab === 'hoaDon'">
      <h4 class="mb-3">📋 Danh sách hóa đơn</h4>

      <!-- Bộ lọc -->
      <div class="d-flex mb-3 gap-2">
        <input
          type="date"
          v-model="fromDate"
          class="form-control w-auto"
          @change="fetchHoaDon"
        />
        <input
          type="date"
          v-model="toDate"
          class="form-control w-auto"
          @change="fetchHoaDon"
        />
        <input
          type="text"
          v-model="phoneFilter"
          class="form-control w-auto"
          placeholder="Nhập số điện thoại"
          @input="fetchHoaDon"
        />
      </div>

      <!-- Thông báo lỗi -->
      <div v-if="error.hoaDon" class="alert alert-danger">
        {{ error.hoaDon }}
      </div>

      <!-- Trạng thái tải -->
      <div v-if="isLoading.hoaDon" class="text-center">Đang tải...</div>
      <div v-else-if="hoaDons.length === 0" class="text-center text-muted">
        Không có dữ liệu
      </div>
      <div v-else class="table-responsive">
        <table class="table table-bordered table-hover align-middle">
          <thead class="table-primary">
            <tr>
              <th>#</th>
              <th>Ngày tạo</th>
              <th>Tổng tiền</th>
              <th>Khách đưa</th>
              <th>Thối lại</th>
              <th>Hình thức</th>
              <th>Số điện thoại</th>
              <th>Dịch vụ</th>
              <th>Mã giảm giá</th>
              <th>Giá trị giảm</th>
              <th>Tải hóa đơn</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(hd, index) in hoaDons" :key="hd.hoaDonID">
              <td>{{ index + 1 }}</td>
              <td>{{ formatDateTime(hd.ngayTao) }}</td>
              <td>{{ formatCurrency(hd.tongTien) }}</td>
              <td>{{ formatCurrency(hd.tienKhachDua) }}</td>
              <td>{{ formatCurrency(hd.tienThoiLai) }}</td>
              <td>{{ hd.hinhThucThanhToan ?? "—" }}</td>
              <td>{{ hd.userID }}</td>
              <td>
                <ul class="mb-0 ps-3">
                  <li
                    v-for="ct in hd.chiTietHoaDons || []"
                    :key="ct.chiTietHoaDonID"
                  >
                    {{ ct.soLuongSP ?? "—" }} x
                    <span v-if="ct.dichVu">
                      {{ ct.dichVu.tenDichVu }} - {{ ct.dichVu.thoiGian ?? 0 }}p
                    </span>
                    <span v-else-if="ct.sanPham">
                      {{ ct.sanPham.productName }}
                    </span>
                    - {{ formatCurrency(ct.thanhTien) }}
                  </li>
                </ul>
              </td>
              <td>{{ hd.voucher?.maCode ?? "Không có" }}</td>
              <td>{{ formatCurrency(hd.giaTriGiam) }}</td>
              <td>
                <button
                  class="btn btn-sm btn-outline-primary"
                  :disabled="isDownloading[hd.hoaDonID]"
                  @click="taiHoaDon(hd.hoaDonID)"
                >
                  <span v-if="isDownloading[hd.hoaDonID]">Đang tải...</span>
                  <span v-else>⬇️ Tải</span>
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import apiClient from "../utils/axiosClient";
import dayjs from "dayjs";

// Khai báo các biến phản ứng
const activeTab = ref("hoaDon");
const datLichs = ref([]);
const hoaDons = ref([]);
const fromDate = ref(dayjs().startOf("month").format("YYYY-MM-DD"));
const toDate = ref(dayjs().add(1, "day").format("YYYY-MM-DD"));
const phoneFilter = ref("");
const isLoading = ref({ lichHen: false, hoaDon: false });
const error = ref({ lichHen: null, hoaDon: null });
const isDownloading = ref({});

// Định dạng ngày giờ
const formatDateTime = (dateTime) => {
  if (!dateTime) return "—";
  return dayjs(dateTime).format("HH:mm - DD/MM");
};

// Định dạng tiền tệ
const formatCurrency = (value) => {
  if (value == null) return "—";
  return value.toLocaleString("vi-VN", { style: "currency", currency: "VND" });
};

// Kiểm tra ngày hợp lệ
const isValidDateRange = () => {
  if (!fromDate.value || !toDate.value) return true;
  return dayjs(fromDate.value).isBefore(dayjs(toDate.value));
};

// Lấy danh sách lịch hẹn
const fetchDatLich = async () => {
  if (!isValidDateRange()) {
    error.value.lichHen = "Ngày bắt đầu phải trước ngày kết thúc";
    return;
  }
  try {
    isLoading.value.lichHen = true;
    error.value.lichHen = null;
    const params = {};
    if (fromDate.value) params.startDate = fromDate.value;
    if (toDate.value) params.endDate = toDate.value;
    if (phoneFilter.value) params.sodienthoai = phoneFilter.value.trim();
    const res = await apiClient.get("/thongke/thongKeDatLich", { params });
    datLichs.value = Array.isArray(res) ? res : [];
  } catch (err) {
    error.value.lichHen = "Lỗi khi lấy danh sách lịch hẹn";
    console.error("Lỗi lấy danh sách lịch hẹn:", err.response || err);
    datLichs.value = [];
  } finally {
    isLoading.value.lichHen = false;
  }
};

// Lấy danh sách hóa đơn
const fetchHoaDon = async () => {
  if (!isValidDateRange()) {
    error.value.hoaDon = "Ngày bắt đầu phải trước ngày kết thúc";
    return;
  }
  try {
    isLoading.value.hoaDon = true;
    error.value.hoaDon = null;
    const params = {};
    if (fromDate.value) params.startDate = fromDate.value;
    if (toDate.value) params.endDate = toDate.value;
    if (phoneFilter.value) params.sodienthoai = phoneFilter.value.trim();
    const res = await apiClient.get("/thongke/thongKeHoaDon", { params });
    hoaDons.value = Array.isArray(res) ? res : [];
  } catch (err) {
    error.value.hoaDon = "Lỗi khi lấy danh sách hóa đơn";
    console.error("Lỗi lấy danh sách hóa đơn:", err.response || err);
    hoaDons.value = [];
  } finally {
    isLoading.value.hoaDon = false;
  }
};

// Tải hóa đơn
const taiHoaDon = async (hoaDonID) => {
  try {
    isDownloading.value[hoaDonID] = true;
    error.value.hoaDon = null;

    const response = await apiClient.get(`/ThanhToan/xuat-hoadon/${hoaDonID}`, {
      responseType: "blob", // Đảm bảo nhận dữ liệu dưới dạng blob
    });

    // Kiểm tra headers và dữ liệu response
    console.log("Response headers:", response.headers);
    console.log("Response data size:", response.size);

    if (!response || response.size === 0) {
      throw new Error("Dữ liệu PDF trống từ server");
    }

    const url = window.URL.createObjectURL(
      new Blob([response], { type: "application/pdf" })
    );
    const link = document.createElement("a");
    link.href = url;
    link.setAttribute("download", `HoaDon_${hoaDonID}.pdf`);
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
    window.URL.revokeObjectURL(url);
  } catch (err) {
    console.error("Lỗi khi tải hóa đơn:", err);
    if (err.response?.status === 404) {
      error.value.hoaDon = "Không tìm thấy hóa đơn";
    } else if (err.message.includes("PDF trống")) {
      error.value.hoaDon = "Dữ liệu PDF trống từ server";
    } else {
      error.value.hoaDon = "Lỗi khi tải hóa đơn, vui lòng thử lại";
    }
    console.log("Chi tiết lỗi:", err.response?.value || err.message);
  } finally {
    isDownloading.value[hoaDonID] = false;
  }
};

// Tải dữ liệu ban đầu
onMounted(() => {
  if (activeTab.value === "lichHen") {
    fetchDatLich();
  } else {
    fetchHoaDon();
  }
});
</script>
