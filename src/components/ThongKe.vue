<template>
  <div>
    <!-- Danh sách hóa đơn -->
    <h4 class="mb-3">📋 Danh sách hóa đơn</h4>
    <div class="table-responsive">
      <table class="table table-bordered table-hover align-middle">
        <thead class="table-primary">
          <tr>
            <th>#</th>
            <th>Ngày tạo</th>
            <th>Tổng tiền</th>
            <th>Khách đưa</th>
            <th>Thối lại</th>
            <th>Hình thức</th>
            <th>Trạng thái</th>
            <th>Dịch vụ</th>
            <th>Mã giảm giá</th>
            <th>Giá trị giảm</th>
            <th>Tải hóa đơn</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(hd, index) in danhSachHoaDon" :key="hd.hoaDonID">
            <td>{{ index + 1 }}</td>
            <td>{{ formatDateTime(hd.ngayTao) }}</td>
            <td>{{ hd.tongTien.toLocaleString() }}₫</td>
            <td>{{ hd.tienKhachDua?.toLocaleString() ?? "—" }}₫</td>
            <td>{{ hd.tienThoiLai?.toLocaleString() ?? "—" }}₫</td>
            <td>{{ hd.hinhThucThanhToan }}</td>
            <td>
              <span
                :class="hd.trangThai === 1 ? 'text-success' : 'text-danger'"
              >
                {{ hd.trangThai === 1 ? "✔ Hoàn tất" : "⏳ Chờ xử lý" }}
              </span>
            </td>
            <td>
              <ul class="mb-0 ps-3">
                <li v-for="ct in hd.chiTietHoaDons" :key="ct.chiTietHoaDonID">
                  {{ ct.soLuongSP }} x {{ ct.dichVu?.tenDichVu ?? "—" }} -
                  {{ ct.dichVu?.thoiGian ?? 0 }}p -
                  {{ ct.thanhTien?.toLocaleString() }}₫
                </li>
              </ul>
            </td>
            <td>{{ hd.voucher?.maCode ?? "Không có" }}</td>
            <td>{{ hd.giaTriGiam }}</td>
            <td>
              <button
                class="btn btn-sm btn-outline-primary"
                @click="taiHoaDon(hd.hoaDonID)"
              >
                ⬇️ Tải
              </button>
            </td>
          </tr>
          <tr v-if="danhSachHoaDon.length === 0">
            <td colspan="11" class="text-center text-muted">Không có dữ liệu</td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Thống kê Thu Chi -->
    <h4 class="mt-5 mb-3">💰 Thống kê Thu - Chi</h4>
    <div class="row">
      <div class="col-md-6">
        <h5>Thu</h5>
        <table class="table table-bordered">
          <thead class="table-success">
            <tr>
              <th>Loại thu</th>
              <th>Số tiền</th>
              <th>Phần trăm</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="thu in thuChiData.thu" :key="thu.loaiThu">
              <td>{{ thu.loaiThu }}</td>
              <td>{{ thu.soTien?.toLocaleString() ?? 0 }}₫</td>
              <td>{{ thu.phanTram ?? 0 }}%</td>
            </tr>
            <tr v-if="thuChiData.thu.length === 0">
              <td colspan="3" class="text-center text-muted">Không có dữ liệu</td>
            </tr>
            <tr class="table-primary">
              <td><strong>Tổng thu</strong></td>
              <td><strong>{{ thuChiData.tongThu?.toLocaleString() ?? 0 }}₫</strong></td>
              <td></td>
            </tr>
          </tbody>
        </table>
      </div>
      <div class="col-md-6">
        <h5>Chi</h5>
        <table class="table table-bordered">
          <thead class="table-warning">
            <tr>
              <th>Loại chi</th>
              <th>Số tiền</th>
              <th>Phần trăm</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="chi in thuChiData.chi" :key="chi.loaiChi">
              <td>{{ chi.loaiChi }}</td>
              <td>{{ chi.soTien?.toLocaleString() ?? 0 }}₫</td>
              <td>{{ chi.phanTram ?? 0 }}%</td>
            </tr>
            <tr v-if="thuChiData.chi.length === 0">
              <td colspan="3" class="text-center text-muted">Không có dữ liệu</td>
            </tr>
            <tr class="table-primary">
              <td><strong>Tổng chi</strong></td>
              <td><strong>{{ thuChiData.tongChi?.toLocaleString() ?? 0 }}₫</strong></td>
              <td></td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- Danh sách chi phí -->
    <h4 class="mt-5 mb-3">📊 Danh sách chi phí</h4>
    <div class="mb-3">
      <button
        class="btn btn-primary"
        data-bs-toggle="modal"
        data-bs-target="#addExpenseModal"
      >
        Thêm chi phí mới
      </button>
    </div>
    <div class="table-responsive">
      <table class="table table-bordered table-hover align-middle">
        <thead class="table-primary">
          <tr>
            <th>#</th>
            <th>Loại chi phí</th>
            <th>Số tiền</th>
            <th>Ngày</th>
            <th>Ghi chú</th>
            <th>Hành động</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(expense, index) in expenses" :key="expense.expenseId">
            <td>{{ index + 1 }}</td>
            <td>{{ expense.expenseType }}</td>
            <td>{{ expense.amount.toLocaleString() }}₫</td>
            <td>{{ formatDate(expense.date) }}</td>
            <td>{{ expense.note ?? "—" }}</td>
            <td>
              <button
                class="btn btn-sm btn-outline-primary me-2"
                @click="openEditExpenseModal(expense)"
              >
                Sửa
              </button>
              <button
                class="btn btn-sm btn-outline-danger"
                @click="deleteExpense(expense.expenseId)"
              >
                Xóa
              </button>
            </td>
          </tr>
          <tr v-if="expenses.length === 0">
            <td colspan="6" class="text-center text-muted">Không có dữ liệu</td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Modal thêm chi phí -->
    <div
      class="modal fade"
      id="addExpenseModal"
      tabindex="-1"
      aria-labelledby="addExpenseModalLabel"
      aria-hidden="true"
    >
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title" id="addExpenseModalLabel">Thêm chi phí mới</h5>
            <button
              type="button"
              class="btn-close"
              data-bs-dismiss="modal"
              aria-label="Close"
            ></button>
          </div>
          <div class="modal-body">
            <form @submit.prevent="createExpense">
              <div class="mb-3">
                <label for="expenseType" class="form-label">Loại chi phí</label>
                <input
                  type="text"
                  class="form-control"
                  id="expenseType"
                  v-model="newExpense.expenseType"
                  required
                />
              </div>
              <div class="mb-3">
                <label for="amount" class="form-label">Số tiền</label>
                <input
                  type="number"
                  class="form-control"
                  id="amount"
                  v-model.number="newExpense.amount"
                  required
                />
              </div>
              <div class="mb-3">
                <label for="date" class="form-label">Ngày</label>
                <input
                  type="date"
                  class="form-control"
                  id="date"
                  v-model="newExpense.date"
                  required
                />
              </div>
              <div class="mb-3">
                <label for="note" class="form-label">Ghi chú</label>
                <textarea
                  class="form-control"
                  id="note"
                  v-model="newExpense.note"
                ></textarea>
              </div>
              <button type="submit" class="btn btn-primary">Thêm</button>
            </form>
          </div>
        </div>
      </div>
    </div>

    <!-- Modal sửa chi phí -->
    <div
      class="modal fade"
      id="editExpenseModal"
      tabindex="-1"
      aria-labelledby="editExpenseModalLabel"
      aria-hidden="true"
    >
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title" id="editExpenseModalLabel">Sửa chi phí</h5>
            <button
              type="button"
              class="btn-close"
              data-bs-dismiss="modal"
              aria-label="Close"
            ></button>
          </div>
          <div class="modal-body">
            <form @submit.prevent="updateExpense">
              <input type="hidden" v-model="editExpense.expenseId" />
              <div class="mb-3">
                <label for="editExpenseType" class="form-label">Loại chi phí</label>
                <input
                  type="text"
                  class="form-control"
                  id="editExpenseType"
                  v-model="editExpense.expenseType"
                  required
                />
              </div>
              <div class="mb-3">
                <label for="editAmount" class="form-label">Số tiền</label>
                <input
                  type="number"
                  class="form-control"
                  id="editAmount"
                  v-model.number="editExpense.amount"
                  required
                />
              </div>
              <div class="mb-3">
                <label for="editDate" class="form-label">Ngày</label>
                <input
                  type="date"
                  class="form-control"
                  id="editDate"
                  v-model="editExpense.date"
                  required
                />
              </div>
              <div class="mb-3">
                <label for="editNote" class="form-label">Ghi chú</label>
                <textarea
                  class="form-control"
                  id="editNote"
                  v-model="editExpense.note"
                ></textarea>
              </div>
              <button type="submit" class="btn btn-primary">Cập nhật</button>
            </form>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import axios from "axios";
import apiClient from "../utils/axiosClient";

const danhSachHoaDon = ref([]);
const thuChiData = ref({ thu: [], chi: [], tongThu: 0, tongChi: 0 });
const expenses = ref([]);
const newExpense = ref({
  expenseType: "",
  amount: 0,
  date: new Date().toISOString().split("T")[0],
  note: "",
});
const editExpense = ref({
  expenseId: 0,
  expenseType: "",
  amount: 0,
  date: "",
  note: "",
});

// Lấy danh sách hóa đơn
const layDanhSach = async () => {
  try {
    const res = await apiClient.get("/ThongKe/thongKeHoaDon");
    danhSachHoaDon.value = res;
  } catch (err) {
    console.error("Lỗi lấy danh sách hóa đơn:", err);
  }
};

// Lấy dữ liệu thu chi
const layThuChi = async () => {
  try {
    const res = await apiClient.get("/ThongKe/ThuChi");
    console.log("Dữ liệu ThuChi:", res); // Debug log
    thuChiData.value = res;
  } catch (err) {
    console.error("Lỗi lấy dữ liệu thu chi:", err);
  }
};

// Lấy danh sách chi phí
const layDanhSachChiPhi = async () => {
  try {
    const res = await apiClient.get("/ThongKe/Expense");
    expenses.value = res;
  } catch (err) {
    console.error("Lỗi lấy danh sách chi phí:", err);
  }
};

// Thêm chi phí mới
const createExpense = async () => {
  try {
    await apiClient.post("/ThongKe/Expense", newExpense.value);
    alert("Thêm chi phí thành công");
    newExpense.value = {
      expenseType: "",
      amount: 0,
      date: new Date().toISOString().split("T")[0],
      note: "",
    };
    await layDanhSachChiPhi();
    await layThuChi(); // Cập nhật lại thu chi
    document.querySelector("#addExpenseModal .btn-close").click();
  } catch (err) {
    console.error("Lỗi khi thêm chi phí:", err);
    alert("Không thể thêm chi phí.");
  }
};

// Mở modal sửa chi phí
const openEditExpenseModal = (expense) => {
  editExpense.value = {
    expenseId: expense.expenseId,
    expenseType: expense.expenseType,
    amount: expense.amount,
    date: new Date(expense.date).toISOString().split("T")[0],
    note: expense.note,
  };
  const modal = new bootstrap.Modal(document.getElementById("editExpenseModal"));
  modal.show();
};

// Cập nhật chi phí
const updateExpense = async () => {
  try {
    await apiClient.put(`/ThongKe/Expense/${editExpense.value.expenseId}`, editExpense.value);
    alert("Cập nhật chi phí thành công");
    await layDanhSachChiPhi();
    await layThuChi(); // Cập nhật lại thu chi
    document.querySelector("#editExpenseModal .btn-close").click();
  } catch (err) {
    console.error("Lỗi khi cập nhật chi phí:", err);
    alert("Không thể cập nhật chi phí.");
  }
};

// Xóa chi phí
const deleteExpense = async (expenseId) => {
  if (confirm("Bạn có chắc muốn xóa chi phí này?")) {
    try {
      await apiClient.delete(`/ThongKe/Expense/${expenseId}`);
      alert("Xóa chi phí thành công");
      await layDanhSachChiPhi();
      await layThuChi(); // Cập nhật lại thu chi
    } catch (err) {
      console.error("Lỗi khi xóa chi phí:", err);
      alert("Không thể xóa chi phí.");
    }
  }
};

// Tải hóa đơn
const taiHoaDon = async (hoaDonID) => {
  try {
    const response = await apiClient.get(`/ThanhToan/xuat-hoadon/${hoaDonID}`, {
      responseType: "blob",
    });
    const url = window.URL.createObjectURL(new Blob([response]));
    const link = document.createElement("a");
    link.href = url;
    link.setAttribute("download", `HoaDon_${hoaDonID}.pdf`);
    document.body.appendChild(link);
    link.click();
    link.remove();
  } catch (error) {
    console.error("Lỗi khi tải hóa đơn:", error);
    alert("Không thể tải hóa đơn.");
  }
};

// Format ngày giờ
const formatDateTime = (dateStr) => {
  const date = new Date(dateStr);
  return `${date.toLocaleTimeString([], {
    hour: "2-digit",
    minute: "2-digit",
  })} - ${date.toLocaleDateString("vi-VN")}`;
};

// Format ngày
const formatDate = (dateStr) => {
  const date = new Date(dateStr);
  return date.toLocaleDateString("vi-VN");
};

// Khởi tạo dữ liệu khi component được mount
onMounted(() => {
  layDanhSach();
  layThuChi();
  layDanhSachChiPhi();
});
</script>