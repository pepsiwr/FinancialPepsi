<template>
  <div
    class="container-fluid py-3"
    style="background-color: #f4f7f6; min-height: 100vh"
  >
    <div id="qr-reader-hidden" style="display: none"></div>

    <div class="container px-md-4">
      <div class="d-flex justify-content-between align-items-center mb-4">
        <h3 class="fw-bold text-primary mb-0">Wallet Me</h3>
        <button @click="logout" class="btn btn-sm btn-outline-danger px-3">
          ออกจากระบบ
        </button>
      </div>

      <div class="row g-3 mb-4 align-items-end">
        <div class="col-12 col-md-6 text-center text-md-start">
          <div
            class="p-2 bg-white rounded shadow-sm border d-inline-block px-4"
          >
            <span class="text-muted small"
              >สรุปปี {{ filterYear + 543 }}:
            </span>
            <span class="text-success fw-bold">
              +{{ yearlyIncome.toLocaleString() }}</span
            >
            <span class="text-danger fw-bold ms-2">
              -{{ yearlyExpense.toLocaleString() }}</span
            >
            <div class="">
              <span class="text-muted small">คงเหลือ:</span>
              <span class="ms-1 text-primary fw-bold">{{
                total.toLocaleString()
              }}</span>
            </div>
          </div>
        </div>
        <div class="col-12 col-md-6">
          <div class="row g-2 justify-content-md-end">
            <div class="col-7 col-md-4">
              <select
                v-model="filterMonth"
                class="form-select border-0 shadow-sm"
              >
                <option
                  v-for="(m, index) in months"
                  :key="index"
                  :value="index"
                >
                  {{ m }}
                </option>
              </select>
            </div>
            <div class="col-5 col-md-3">
              <select
                v-model="filterYear"
                class="form-select border-0 shadow-sm"
              >
                <option v-for="y in years" :key="y" :value="y">
                  {{ y + 543 }}
                </option>
              </select>
            </div>
          </div>
        </div>
      </div>

      <div class="row g-4">
        <div class="col-12 col-lg-4">
          <div class="row g-2 mb-4">
            <div class="col-12">
              <div class="card border-0 shadow-sm bg-primary text-white p-3">
                <div class="small opacity-75">
                  คงเหลือ ({{ months[filterMonth] }})
                </div>
                <h2 class="fw-bold mb-0">
                  ฿ {{ monthlyBalance.toLocaleString() }}
                </h2>
              </div>
            </div>
            <div class="col-6">
              <div
                class="card border-0 shadow-sm bg-success text-white p-2 text-center"
              >
                <div class="small opacity-75">รายรับ</div>
                <div class="fw-bold">฿{{ monthlyIncome.toLocaleString() }}</div>
              </div>
            </div>
            <div class="col-6">
              <div
                class="card border-0 shadow-sm bg-danger text-white p-2 text-center"
              >
                <div class="small opacity-75">รายจ่าย</div>
                <div class="fw-bold">
                  ฿{{ monthlyExpense.toLocaleString() }}
                </div>
              </div>
            </div>
          </div>

          <div
            class="card border-0 shadow-sm p-4 sticky-lg-top"
            style="top: 20px"
          >
            <h6 class="fw-bold mb-3">บันทึกรายการใหม่</h6>
            <div class="mb-3 text-center">
              <label
                class="btn btn-outline-primary w-100 py-2 border-dashed shadow-sm"
              >
                <i class="bi bi-camera-fill me-2"></i> สแกนสลิป
                <input
                  type="file"
                  @change="handleFileUpload"
                  class="d-none"
                  accept="image/*"
                />
              </label>
            </div>
            <div class="row g-2">
              <div class="col-12 mb-2">
                <div class="input-group">
                  <span class="input-group-text bg-white border-end-0">฿</span>
                  <input
                    type="number"
                    v-model="form.amount"
                    class="form-control border-start-0 ps-0 shadow-none bg-light"
                    placeholder="0.00"
                  />
                </div>
              </div>
              <div class="col-6">
                <select
                  v-model="form.type"
                  class="form-select bg-light border-0 shadow-none"
                >
                  <option value="expense">📉รายจ่าย</option>
                  <option value="income">📈รายรับ</option>
                </select>
              </div>
              <div class="col-6">
                <select
                  v-model="form.category"
                  class="form-select bg-light border-0 shadow-none"
                >
                  <option v-for="cat in categories" :key="cat" :value="cat">
                    {{ cat }}
                  </option>
                </select>
              </div>
              <div class="col-12 mt-3">
                <textarea
                  class="form-control"
                  aria-label="With textarea"
                  v-model="form.note"
                ></textarea>
              </div>
              <div class="col-12 mt-3">
                <button
                  @click="saveTransaction"
                  class="btn btn-primary w-100 py-2 fw-bold shadow-sm"
                >
                  บันทึกรายการ
                </button>
              </div>
            </div>
          </div>
        </div>

        <div class="col-12 col-lg-8">
          <div class="card border-0 shadow-sm p-3 p-md-4">
            <h5 class="fw-bold mb-4">รายการประจำเดือน</h5>

            <div class="table-responsive d-none d-md-block">
              <table class="table table-hover align-middle">
                <thead class="table-light">
                  <tr>
                    <th>วันที่</th>
                    <th>หมวดหมู่</th>
                    <th>ประเภท</th>
                    <th class="text-end">จำนวนเงิน</th>
                    <th class="text-center">จัดการ</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="item in filteredTransactions" :key="item._id">
                    <td class="small">
                      <div class="fw-bold">{{ formatDate(item.date) }}</div>
                      <small class="text-muted">{{
                        formatTime(item.date)
                      }}</small>
                    </td>
                    <td>
                      <div class="fw-bold">{{ item.category }}</div>
                      <div class="text-muted small">{{ item.note || "-" }}</div>
                    </td>
                    <td>
                      <span
                        :class="
                          item.type === 'income'
                            ? 'badge bg-success-subtle text-success'
                            : 'badge bg-danger-subtle text-danger'
                        "
                      >
                        {{ item.type === "income" ? "รายรับ" : "รายจ่าย" }}
                      </span>
                    </td>
                    <td
                      class="text-end fw-bold"
                      :class="
                        item.type === 'income' ? 'text-success' : 'text-danger'
                      "
                    >
                      {{ item.type === "income" ? "+" : "-"
                      }}{{ item.amount.toLocaleString() }}
                    </td>
                    <td class="text-center">
                      <button
                        @click="deleteTransaction(item._id)"
                        class="btn btn-sm btn-outline-danger border-0"
                      >
                        <i class="bi bi-trash"></i> ลบ
                      </button>
                    </td>
                  </tr>
                </tbody>
              </table>
            </div>
            <!-- mobile -->
            <div class="d-md-none">
              <div
                v-for="item in filteredTransactions"
                :key="item._id"
                class="border-bottom py-3"
              >
                <div class="d-flex justify-content-between align-items-start">
                  <div>
                    <div class="fw-bold">{{ item.category }}</div>
                    <div class="text-muted" style="font-size: 0.8rem">
                      {{ formatDate(item.date) }} | {{ formatTime(item.date) }}
                      <div
                        v-if="item.note"
                        class="mt-1 text-secondary small pe-2"
                      >
                        {{ item.note }}
                      </div>
                    </div>
                  </div>
                  <div class="text-end">
                    <div
                      :class="
                        item.type === 'income' ? 'text-success' : 'text-danger'
                      "
                      class="fw-bold"
                    >
                      {{ item.type === "income" ? "+" : "-"
                      }}{{ item.amount.toLocaleString() }}
                    </div>
                    <button
                      @click="deleteTransaction(item._id)"
                      class="btn btn-link text-danger text-nowrap p-0"
                      style="text-decoration: none"
                    >
                      <small>ลบรายการ</small>
                    </button>
                  </div>
                </div>
              </div>
            </div>

            <div
              v-if="filteredTransactions.length === 0"
              class="text-center py-5 text-muted"
            >
              ไม่มีข้อมูลในเดือนนี้
            </div>
          </div>
        </div>
      </div>
    </div>

    <div v-if="loading" class="loading-overlay">
      <div class="text-center text-white">
        <div
          class="spinner-border text-primary mb-3"
          style="width: 3rem; height: 3rem"
        ></div>
        <h5 class="fw-bold">กำลังประมวลผล...</h5>
        <p class="opacity-75 small">
          กรุณารอสักครู่ ระบบกำลังจัดการข้อมูลให้คุณ
        </p>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import urlapi from "@/typeScript/urlapi";
import { Html5Qrcode } from "html5-qrcode";
import Crypto from "@/typeScript/CryptoJS.ts";
import { nextTick } from "vue";

export default {
  data() {
    return {
      user: null,
      transactions: [],
      loading: false,
      categories: [
        "🍔อาหาร",
        "💵เงินเดือน",
        "🚗เดินทาง",
        "🛍️ช้อปปิ้ง",
        "🏠ที่อยู่อาศัย",
        "📦ทั่วไป",
      ],
      form: {
        userId: "",
        amount: 0,
        category: "🍔อาหาร",
        type: "expense",
        note: "",
        date: "",
        transRef: null,
      },
      filterMonth: new Date().getMonth(),
      filterYear: new Date().getFullYear(),
      months: [
        "มกราคม",
        "กุมภาพันธ์",
        "มีนาคม",
        "เมษายน",
        "พฤษภาคม",
        "มิถุนายน",
        "กรกฎาคม",
        "สิงหาคม",
        "กันยายน",
        "ตุลาคม",
        "พฤศจิกายน",
        "ธันวาคม",
      ],
      years: [2024, 2025, 2026],
    };
  },
  computed: {
    yearlyTransactions() {
      return this.transactions.filter(
        (t) => new Date(t.date).getFullYear() === this.filterYear
      );
    },
    yearlyIncome() {
      return this.yearlyTransactions
        .filter((t) => t.type === "income")
        .reduce((s, t) => s + t.amount, 0);
    },
    yearlyExpense() {
      return this.yearlyTransactions
        .filter((t) => t.type === "expense")
        .reduce((s, t) => s + t.amount, 0);
    },
    total() {
      return this.yearlyIncome - this.yearlyExpense;
    },
    filteredTransactions() {
      return this.transactions
        .filter((t) => {
          const d = new Date(t.date);
          return (
            d.getMonth() === this.filterMonth &&
            d.getFullYear() === this.filterYear
          );
        })
        .sort((a, b) => new Date(b.date) - new Date(a.date));
    },
    monthlyIncome() {
      return this.filteredTransactions
        .filter((t) => t.type === "income")
        .reduce((s, t) => s + t.amount, 0);
    },
    monthlyExpense() {
      return this.filteredTransactions
        .filter((t) => t.type === "expense")
        .reduce((s, t) => s + t.amount, 0);
    },
    monthlyBalance() {
      return this.monthlyIncome - this.monthlyExpense;
    },
  },
  async mounted() {
    await this.PerformanceEntry();
    this.fetchTransactions();
  },
  methods: {
    async PerformanceEntry() {
      let userinfo = sessionStorage.getItem("userInfo");
      if (userinfo) {
        const decryptText = await Crypto.decryptText(userinfo);
        this.user = JSON.parse(decryptText);
        this.form.userId = this.user.id;
      } else {
        this.logout();
      }
    },
    async fetchTransactions() {
      const res = await axios.get(
        `${urlapi}/api/my-transactions/${this.user.id}`
      );
      this.transactions = res.data;
    },
    async handleFileUpload(event) {
      const file = event.target.files[0];
      if (!file) return;

      this.loading = true;
      // ⚠️ สำคัญ: รอให้ Vue มั่นใจว่า DOM พร้อม
      await nextTick();

      try {
        const html5QrCode = new Html5Qrcode("qr-reader-hidden");
        // 1. แกะข้อความ QR จากรูปภาพ (ฝั่ง Client)
        const qrData = await html5QrCode.scanFile(file, true);
        console.log("สแกนสำเร็จ:", qrData);
        // 2. ส่งข้อความ QR ไปให้ Backend (Vercel) ของเรา
        // เปลี่ยน URL เป็นของ Vercel คุณ
        const res = await axios.post(`${urlapi}/api/verify-slip-qr`, {
          qrData,
        });
        if (res.data.status === 200) {
          const slip = res.data.data;
          console.log(slip);
          // จำลองข้อมูลจากสลิป (ตามตัวอย่างที่คุณส่งมา)
          // const slip = {
          //   transRef: "C20260106600618131929",
          //   date: new Date().toISOString(),
          //   amount: { amount: 23 },
          //   sender: { account: { name: { th: "วิวัฒน์" } } },
          //   receiver: { account: { name: { th: "ร้านค้ามหาเฮง" } } },
          // };

          this.form.amount = slip.amount.amount;
          this.form.date = slip.date;
          this.form.note = `จาก ${slip.sender.account.name.th} ไปยัง ${slip.receiver.account.name.th}`;
          this.form.transRef = slip.transRef;
        }
      } catch (err) {
        console.error(err);
        this.form.transRef = null;
        alert("ไม่พบ QR Code หรือไฟล์รูปภาพไม่ถูกต้อง");
      } finally {
        this.loading = false;
        event.target.value = "";
      }
    },
    async saveTransaction() {
      if (this.form.amount <= 0) return alert("กรุณาระบุจำนวนเงิน");
      this.loading = true;
      try {
        // กำหนดวันที่ปัจจุบันถ้าไม่ได้มาจากสลิป
        if (!this.form.date) this.form.date = new Date().toISOString();

        await axios.post(`${urlapi}/api/add-transaction`, this.form);
        this.form.amount = 0;
        this.form.note = "";
        this.form.transRef = null;
        await this.fetchTransactions();
      } catch (err) {
        alert("เกิดข้อผิดพลาดในการบันทึก");
        this.form.amount = 0;
        this.form.note = "";
        this.form.transRef = null;
      } finally {
        this.loading = false;
      }
    },
    async deleteTransaction(id) {
      if (confirm("คุณแน่ใจหรือไม่ว่าต้องการลบรายการนี้?")) {
        this.loading = true;
        try {
          await axios.delete(`${urlapi}/api/delete-transaction/${id}`);
          await this.fetchTransactions();
        } catch (err) {
          alert("ไม่สามารถลบรายการได้");
        } finally {
          this.loading = false;
        }
      }
    },
    formatDate(date) {
      return new Date(date).toLocaleDateString("th-TH", {
        day: "2-digit",
        month: "2-digit",
        year: "2-digit",
      });
    },
    formatTime(date) {
      return (
        new Date(date).toLocaleTimeString("th-TH", {
          hour: "2-digit",
          minute: "2-digit",
          hour12: false,
        }) + " น."
      );
    },
    logout() {
      sessionStorage.clear();
      this.$router.push("/login");
    },
  },
};
</script>

<style scoped>
.card {
  border-radius: 15px;
}
.border-dashed {
  border-style: dashed !important;
}
.loading-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.7);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 9999;
  backdrop-filter: blur(4px);
}
.badge {
  padding: 0.5em 0.8em;
}
</style>