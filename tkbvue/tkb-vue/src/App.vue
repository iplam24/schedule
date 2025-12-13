<template>
  <div class="app-container">
    <header class="global-nav shadow-sm">
      <div class="nav-content">
        <div class="brand">
          <div class="logo-box">🌿</div>
          <span class="brand-text">CALENDAR BY VUXUANLAM</span>
        </div>
        <div class="nav-actions">
          <div v-if="isLoggedIn" class="user-meta">
            <span class="mssv-badge">SV: {{ currentUser }}</span>
            <button @click="logout" class="btn-logout-top">Đăng xuất</button>
          </div>
          <span v-else class="version">vv1.0</span>
        </div>
      </div>
    </header>

    <main class="main-body">
      <div v-if="!isLoggedIn" class="view-login">
        <div class="login-card shadow-animation">
          <div class="login-head">
            <h2>Chào mừng bạn!</h2>
            <p>Vui lòng đăng nhập để đồng bộ lịch học</p>
          </div>
          <transition name="fade">
            <div v-if="errorMessage" class="error-banner"><span>⚠️</span> {{ errorMessage }}</div>
          </transition>
          <div class="form">
            <div class="input-wrap">
              <label>Mã số sinh viên</label>
              <input v-model="loginForm.mssv" type="text" placeholder="67xxxx" @input="errorMessage = ''" />
            </div>
            <div class="input-wrap">
              <label>Mật khẩu Portal</label>
              <input v-model="loginForm.password" type="password" placeholder="••••••••" @input="errorMessage = ''" />
            </div>
            <button @click="handleLogin" :disabled="loading" class="btn-primary-submit">
              <span v-if="loading" class="spinner"></span>
              {{ loading ? 'Đang xử lý...' : 'ĐĂNG NHẬP' }}
            </button>
          </div>
        </div>
      </div>

      <div v-else class="view-dashboard">
        <aside class="dashboard-sidebar">
          <div class="sidebar-inner">
            <div class="sidebar-section">
              <label class="sidebar-label">CHỌN NGÀY XEM LỊCH</label>
              <input type="date" v-model="selectedDate" class="date-picker" />
            </div>
            <div class="sidebar-status desktop-only">
              <p>Trạng thái: <span class="text-green font-bold">Trực tuyến</span></p>
              <p>Đồng bộ: {{ lastUpdateDisplay }}</p>
            </div>
          </div>
        </aside>

        <section class="schedule-area">
          <div class="schedule-header">
            <div class="header-titles">
              <span class="label-mini text-green">LỊCH TRÌNH CHI TIẾT</span>
              <h2>{{ formatDateFull(selectedDate) }}</h2>
            </div>
            <div class="count-pill" v-if="filteredSchedule.length > 0">{{ filteredSchedule.length }} môn học</div>
          </div>

          <div class="schedule-content">
            <div v-if="filteredSchedule.length > 0" class="schedule-grid">
              <div v-for="item in filteredSchedule" :key="item.id" class="lesson-card shadow-animation" @click="openDetail(item)">
                <div class="lesson-stripe" :class="item.tietBatDau >= 6 ? 'pm' : 'am'">
                  <span class="stripe-period">TIẾT</span>
                  <span class="stripe-range">{{ item.tietBatDau }}-{{ item.tietBatDau + item.soTiet - 1 }}</span>
                </div>
                <div class="lesson-body">
                  <div class="body-top">
                    <span class="tag-buoi">{{ item.tietBatDau >= 6 ? 'Chiều' : 'Sáng' }}</span>
                    <span class="tag-ma">{{ item.maMon }}</span>
                  </div>
                  <h3 class="course-name">{{ item.tenMon }}</h3>
                  <div class="body-bot">
                    <span class="room-pill">📍 {{ item.phong }}</span>
                    <span class="class-id">{{ item.lop }}</span>
                  </div>
                </div>
              </div>
            </div>
            <div v-else class="empty-state shadow-animation">
              <div class="empty-icon">🌿</div>
              <h3>Không có lịch học</h3>
              <p>Hãy tận hưởng thời gian nghỉ ngơi nhé!</p>
            </div>
          </div>
        </section>
      </div>
    </main>

    <footer class="global-footer">
      <div class="footer-wrap">
        <p>© 2025 By Vũ Xuân Lâm đẹp trai .</p>
        <div class="footer-links desktop-only"><span>Hướng dẫn</span><span>Bảo mật</span></div>
      </div>
    </footer>

    <transition name="fade">
      <div v-if="selectedLesson" class="modal-overlay" @click="selectedLesson = null">
        <div class="modal-box shadow-animation" @click.stop>
          <div class="modal-header" :class="selectedLesson.tietBatDau >= 6 ? 'pm-bg' : 'am-bg'">
            <div class="header-text">
              <span class="badge-mini">CHI TIẾT HỌC PHẦN</span>
              <h3>THÔNG TIN MÔN HỌC</h3>
            </div>
            <button @click="selectedLesson = null" class="modal-close">&times;</button>
          </div>
          <div class="modal-body">
            <div class="info-grid">
              <div class="info-cell"><strong>Mã môn:</strong> {{ selectedLesson.maMon }}</div>
              <div class="info-cell"><strong>Số tín chỉ:</strong> {{ selectedLesson.soTinChi }} Tín</div>
              <div class="info-cell"><strong>Giảng viên:</strong> {{ selectedLesson.giangVien || 'Chưa cập nhật' }}</div>
              <div class="info-cell"><strong>Phòng học:</strong> <span class="text-green font-bold">📍 {{ selectedLesson.phong }}</span></div>
              <div class="info-cell"><strong>Lớp học:</strong> {{ selectedLesson.lop }}</div>
              <div class="info-cell"><strong>Nhóm/Tổ:</strong> Nhóm {{ selectedLesson.nhom }} - Tổ {{ selectedLesson.toNhom || 'X' }}</div>
              <div class="info-cell"><strong>Thời gian:</strong> Thứ {{ selectedLesson.thu === 8 ? 'CN' : selectedLesson.thu }}</div>
              <div class="info-cell"><strong>Tiết bắt đầu:</strong> Tiết {{ selectedLesson.tietBatDau }} ({{ selectedLesson.soTiet }} tiết)</div>
              <div class="info-cell"><strong>Tuần học:</strong> Tuần thứ {{ selectedLesson.tuanSo }}</div>
              <div class="info-cell"><strong>Ngày học:</strong> {{ selectedLesson.date }}</div>
            </div>
            <div class="info-full">
              <label>Tên môn học:</label>
              <h4>{{ selectedLesson.tenMon }}</h4>
            </div>
          </div>
          <button @click="selectedLesson = null" class="btn-modal-close">Đã hiểu</button>
        </div>
      </div>
    </transition>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted, computed } from 'vue';
import axios from 'axios';

const BASE_URL = 'http://localhost:8080/api';
const loading = ref(false);
const isLoggedIn = ref(false);
const currentUser = ref('');
const timetable = ref([]);
const errorMessage = ref('');
const loginForm = reactive({ mssv: '', password: '' });
const selectedDate = ref(new Date().toISOString().substr(0, 10));
const selectedLesson = ref(null);

const filteredSchedule = computed(() => {
  return timetable.value.filter(item => item.date === selectedDate.value);
});

const lastUpdateDisplay = computed(() => {
  const ts = localStorage.getItem(`last_crawl_${currentUser.value}`);
  return ts ? new Date(parseInt(ts)).toLocaleTimeString('vi-VN') : '--:--';
});

const formatDateFull = (dateStr) => {
  const d = new Date(dateStr);
  return d.toLocaleDateString('vi-VN', { weekday: 'long', day: 'numeric', month: 'long', year: 'numeric' });
};

const handleLogin = async () => {
  if (!loginForm.mssv || !loginForm.password) { errorMessage.value = "Điền đủ MSSV và mật khẩu!"; return; }
  loading.value = true;
  errorMessage.value = '';
  try {
    const lastCrawl = localStorage.getItem(`last_crawl_${loginForm.mssv}`);
    const THREE_DAYS = 3 * 24 * 60 * 60 * 1000;
    if (!lastCrawl || (Date.now() - parseInt(lastCrawl)) > THREE_DAYS) {
      const res = await axios.post(`${BASE_URL}/tkb/refresh`, { mssv: loginForm.mssv, password: loginForm.password });
      if (res.data.status === 401) { errorMessage.value = res.data.message; loading.value = false; return; }
      localStorage.setItem(`last_crawl_${loginForm.mssv}`, Date.now().toString());
    }
    const res = await axios.get(`${BASE_URL}/tkb/${loginForm.mssv}`);
    timetable.value = res.data;
    currentUser.value = loginForm.mssv;
    isLoggedIn.value = true;
    localStorage.setItem('user_session', loginForm.mssv);
  } catch (e) { errorMessage.value = e.response?.data?.message || "Lỗi máy chủ!"; }
  finally { loading.value = false; }
};

const openDetail = (item) => { selectedLesson.value = item; };
const logout = () => { isLoggedIn.value = false; localStorage.removeItem('user_session'); errorMessage.value = ''; };

onMounted(async () => {
  const s = localStorage.getItem('user_session');
  if (s) {
    currentUser.value = s; isLoggedIn.value = true;
    const res = await axios.get(`${BASE_URL}/tkb/${s}`);
    timetable.value = res.data;
  }
});
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@400;600;700;800&display=swap');

/* --- CƠ BẢN --- */
.app-container { font-family: 'Plus Jakarta Sans', sans-serif; background-color: #f8fafc; min-height: 100vh; display: flex; flex-direction: column; width: 100%; }
.main-body { flex: 1; display: flex; flex-direction: column; width: 100%; }
.text-green { color: #10b981 !important; }
.font-bold { font-weight: 700; }

/* --- 1. GLOBAL HEADER --- */
.global-nav { background: white; border-bottom: 1px solid #e2e8f0; height: 60px; display: flex; align-items: center; position: sticky; top: 0; z-index: 1000; width: 100%; }
.nav-content { width: 100%; max-width: 1400px; margin: 0 auto; padding: 0 20px; display: flex; justify-content: space-between; align-items: center; box-sizing: border-box; }
.brand { display: flex; align-items: center; gap: 10px; }
.logo-box { background: #10b981; color: white; width: 32px; height: 32px; border-radius: 8px; display: flex; align-items: center; justify-content: center; font-size: 18px; }
.brand-text { font-weight: 800; color: #064e3b; font-size: 16px; }
.user-meta { display: flex; align-items: center; gap: 12px; }
.mssv-badge { font-size: 12px; font-weight: 700; background: #f0fdf4; color: #16a34a; padding: 4px 10px; border-radius: 6px; }
.btn-logout-top { background: #fee2e2; color: #dc2626; border: none; padding: 5px 12px; border-radius: 6px; font-weight: 700; font-size: 12px; cursor: pointer; }

/* --- 2. LOGIN VIEW --- */
.view-login { flex: 1; display: flex; align-items: center; justify-content: center; padding: 20px; background: #f0fdf4; }
.login-card { background: white; padding: 40px; border-radius: 24px; box-shadow: 0 10px 25px rgba(0,0,0,0.05); width: 100%; max-width: 360px; text-align: center; }
.login-head h2 { margin-bottom: 8px; font-weight: 800; }
.login-head p { color: #64748b; font-size: 13px; margin-bottom: 25px; }
.error-banner { background: #fef2f2; color: #dc2626; padding: 10px; border-radius: 10px; margin-bottom: 15px; font-size: 12px; font-weight: 700; border: 1px solid #fee2e2; }
.input-wrap { text-align: left; margin-bottom: 15px; }
.input-wrap label { display: block; font-size: 11px; font-weight: 800; color: #94a3b8; margin-bottom: 5px; margin-left: 4px; }
.input-wrap input { width: 100%; padding: 12px; border: 1.5px solid #e2e8f0; border-radius: 10px; box-sizing: border-box; font-family: inherit; font-weight: 600; outline: none; }
.btn-primary-submit { width: 100%; padding: 14px; background: #10b981; color: white; border: none; border-radius: 10px; font-weight: 700; cursor: pointer; margin-top: 10px; }

/* --- 3. DASHBOARD VIEW --- */
.view-dashboard { display: flex; width: 100%; max-width: 1400px; margin: 0 auto; flex: 1; }
.dashboard-sidebar { width: 260px; background: white; border-right: 1px solid #e2e8f0; flex-shrink: 0; }
.sidebar-inner { position: sticky; top: 60px; padding: 30px 20px; height: calc(100vh - 60px); box-sizing: border-box; display: flex; flex-direction: column; }
.sidebar-label { font-size: 10px; font-weight: 800; color: #94a3b8; letter-spacing: 1px; display: block; margin-bottom: 10px; }
.date-picker {
  width: 100%;
  padding: 8px 10px;          /* nhỏ lại */
  border: 2px solid #10b981;  /* mảnh hơn chút */
  border-radius: 8px;         /* bo gọn */
  font-family: inherit;
  font-weight: 700;
  font-size: 13px;            /* chữ nhỏ hơn */
  color: #065f46;
  outline: none;
  box-sizing: border-box; 
}

.sidebar-status { margin-top: auto; padding-top: 15px; border-top: 1px solid #f1f5f9; font-size: 11px; color: #64748b; line-height: 1.6; }

.schedule-area { flex: 1; padding: 40px; background: #f8fafc; min-width: 0; }
.schedule-header { display: flex; justify-content: space-between; align-items: flex-end; margin-bottom: 25px; }
.label-mini { font-size: 10px; font-weight: 800; letter-spacing: 1px; }
.schedule-header h2 { margin: 5px 0 0; font-size: 22px; font-weight: 800; }
.count-pill { background: #1e293b; color: white; padding: 4px 12px; border-radius: 20px; font-size: 11px; font-weight: 700; }

.schedule-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(320px, 1fr)); gap: 20px; }
.lesson-card { display: flex; background: white; border-radius: 18px; overflow: hidden; box-shadow: 0 4px 6px rgba(0,0,0,0.03); cursor: pointer; transition: 0.3s; border: 1px solid #f1f5f9; }
.lesson-card:hover { transform: translateY(-5px); border-color: #10b981; box-shadow: 0 10px 20px rgba(0,0,0,0.06); }
.lesson-stripe { width: 70px; display: flex; flex-direction: column; align-items: center; justify-content: center; color: white; padding: 10px; flex-shrink: 0; }
.am { background: linear-gradient(180deg, #f59e0b, #d97706); }
.pm { background: linear-gradient(180deg, #3b82f6, #1d4ed8); }
.stripe-period { font-size: 9px; font-weight: 700; opacity: 0.8; }
.stripe-range { font-size: 16px; font-weight: 800; }
.lesson-body { flex: 1; padding: 15px 20px; display: flex; flex-direction: column; justify-content: center; }
.body-top { display: flex; justify-content: space-between; margin-bottom: 5px; font-size: 10px; font-weight: 800; color: #94a3b8; }
.course-name { margin: 0; font-size: 14px; font-weight: 800; color: #1e293b; line-height: 1.4; height: 38px; overflow: hidden; }
.body-bot { margin-top: 12px; display: flex; justify-content: space-between; align-items: center; }
.room-pill { background: #f1f5f9; padding: 4px 10px; border-radius: 6px; font-size: 11px; color: #475569; }
.class-id { font-size: 10px; font-weight: 600; color: #cbd5e1; }

.empty-state { text-align: center; padding: 60px; background: white; border-radius: 24px; color: #94a3b8; }
.empty-icon { font-size: 40px; margin-bottom: 10px; }

/* --- 4. GLOBAL FOOTER --- */
.global-footer { background: white; border-top: 1px solid #e2e8f0; padding: 20px 0; width: 100%; }
.footer-wrap { max-width: 1400px; margin: 0 auto; padding: 0 20px; display: flex; justify-content: space-between; align-items: center; box-sizing: border-box; }
.footer-wrap p { font-size: 11px; color: #94a3b8; margin: 0; }
.footer-links { display: flex; gap: 20px; }
.footer-links span { font-size: 11px; font-weight: 700; color: #64748b; }

/* --- MODAL (FIX ĐẦY ĐỦ DATA) --- */
.modal-overlay { position: fixed; inset: 0; background: rgba(0,0,0,0.4); backdrop-filter: blur(3px); display: flex; align-items: center; justify-content: center; z-index: 2000; padding: 20px; }
.modal-box { background: white; width: 100%; max-width: 480px; border-radius: 28px; overflow: hidden; box-shadow: 0 25px 50px rgba(0,0,0,0.15); }
.modal-header { padding: 25px 30px; color: white; display: flex; justify-content: space-between; align-items: flex-start; }
.am-bg { background: linear-gradient(135deg, #f59e0b, #d97706); }
.pm-bg { background: linear-gradient(135deg, #3b82f6, #1d4ed8); }
.badge-mini { font-size: 9px; font-weight: 800; background: rgba(255,255,255,0.2); padding: 3px 8px; border-radius: 5px; margin-bottom: 8px; display: inline-block; }
.header-text h3 { margin: 0; font-size: 20px; font-weight: 900; letter-spacing: -0.5px; }
.modal-close { background: rgba(255,255,255,0.2); border: none; color: white; font-size: 24px; cursor: pointer; border-radius: 50%; width: 32px; height: 32px; }

.modal-body { padding: 30px; }
.info-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 15px; margin-bottom: 25px; }
.info-cell { font-size: 13px; font-weight: 600; color: #1e293b; border-bottom: 1.5px dashed #f1f5f9; padding-bottom: 8px; }
.info-cell strong { color: #94a3b8; font-size: 10px; display: block; text-transform: uppercase; margin-bottom: 2px; }
.info-full label { font-size: 10px; color: #94a3b8; font-weight: 800; display: block; margin-bottom: 5px; }
.info-full h4 { margin: 0; color: #1e293b; font-size: 17px; font-weight: 800; line-height: 1.4; border-top: 1px solid #f1f5f9; padding-top: 10px; }
.btn-modal-close { width: 100%; padding: 15px; border: none; background: #10b981; color: white; font-weight: 800; cursor: pointer; transition: 0.2s; }
.btn-modal-close:hover { background: #059669; }

/* --- RESPONSIVE FIX --- */
@media (max-width: 992px) {
  .view-dashboard { flex-direction: column; }
  .dashboard-sidebar { width: 100%; border-right: none; border-bottom: 1px solid #e2e8f0; }
  .sidebar-inner { height: auto; padding: 15px 20px; position: static; flex-direction: row; align-items: center; justify-content: space-between; gap: 20px; }
  .sidebar-section { margin: 0; flex: 1; max-width: 180px; }
  .desktop-only { display: none; }
  .schedule-area { padding: 20px; }
  .schedule-grid { grid-template-columns: 1fr; }
  .footer-wrap { flex-direction: column; text-align: center; gap: 10px; }
  .nav-content { padding: 0 15px; }
}

/* ===== FIX DATE PICKER MOBILE ===== */
@media (max-width: 600px) {
  .dashboard-sidebar {
    padding: 0;
  }

  .sidebar-inner {
    padding: 12px 15px;
    gap: 12px;
  }

  .sidebar-label {
    font-size: 9px;
    margin-bottom: 6px;
  }

  .date-picker {
    padding: 10px 12px;
    font-size: 14px;
    border-width: 2px;
    border-radius: 14px;
    text-align: center;
    background-color: #f0fdf4;
  }
}

/* ===== FIX DATE PICKER DESKTOP (PC) ===== */
@media (min-width: 993px) {
  .dashboard-sidebar {
    width: 230px; /* thu sidebar lại */
  }

  .sidebar-inner {
    padding: 22px 18px;
  }

  .sidebar-label {
    font-size: 9px;
    margin-bottom: 6px;
    letter-spacing: 1.2px;
  }

  .date-picker {
    padding: 8px 10px;        /* nhỏ gọn hơn */
    font-size: 13px;          /* chữ nhỏ lại */
    border-width: 2px;
    border-radius: 12px;
    font-weight: 700;
    background-color: #f8fafc;
  }

  .date-picker::-webkit-calendar-picker-indicator {
    transform: scale(0.85);   /* icon lịch nhỏ hơn */
    cursor: pointer;
  }
}

/* ANIMATIONS */
.spinner { width: 14px; height: 14px; border: 2px solid white; border-top-color: transparent; border-radius: 50%; animation: spin 1s infinite linear; display: inline-block; margin-right: 5px; }
@keyframes spin { to { transform: rotate(360deg); } }
.shadow-animation { animation: slideUp 0.4s ease-out; }
@keyframes slideUp { from { transform: translateY(10px); opacity: 0; } to { transform: translateY(0); opacity: 1; } }
.fade-enter-active, .fade-leave-active { transition: opacity 0.3s; }
.fade-enter-from, .fade-leave-to { opacity: 0; }
</style>