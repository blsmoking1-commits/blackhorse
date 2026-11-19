<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />
  <title>نظام بلاك هورس المتكامل | Black Horse Enterprise ERP</title>
  
  <!-- Tailwind CSS -->
  <script src="https://cdn.tailwindcss.com"></script>
  
  <!-- Google Fonts: Cairo & Tajawal -->
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Cairo:wght@400;600;700;800;900&family=Tajawal:wght@400;500;700&display=swap" rel="stylesheet">
  
  <!-- FontAwesome Icons -->
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css" />
  
  <!-- SheetJS for Real Excel (.xlsx) Export -->
  <script src="https://cdn.sheetjs.com/xlsx-0.20.1/package/dist/xlsx.full.min.js"></script>
  
  <!-- Firebase Compat SDKs (App, Auth, Realtime Database) -->
  <script src="https://www.gstatic.com/firebasejs/10.8.0/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/10.8.0/firebase-database-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/10.8.0/firebase-auth-compat.js"></script>

  <script>
    tailwind.config = {
      theme: {
        extend: {
          fontFamily: {
            sans: ['Cairo', 'Tajawal', 'sans-serif'],
          },
          colors: {
            bh: {
              dark: '#0B1120',
              navy: '#111827',
              card: '#1E293B',
              border: '#334155',
              gold: '#F59E0B',
              goldHover: '#D97706',
              emerald: '#10B981',
              crimson: '#EF4444'
            }
          }
        }
      }
    }
  </script>

  <style>
    * { box-sizing: border-box; }
    body {
      font-family: 'Cairo', sans-serif;
      background-color: #0B1120;
      color: #F8FAFC;
      overflow-x: hidden;
    }
    /* Custom Scrollbar */
    ::-webkit-scrollbar { width: 6px; height: 6px; }
    ::-webkit-scrollbar-track { background: #0B1120; }
    ::-webkit-scrollbar-thumb { background: #334155; border-radius: 9999px; }
    ::-webkit-scrollbar-thumb:hover { background: #F59E0B; }

    /* Glass & Card Styling */
    .bh-panel {
      background: linear-gradient(145deg, #162032 0%, #0F172A 100%);
      border: 1px solid rgba(51, 65, 85, 0.7);
      border-radius: 1rem;
      box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.4);
    }
    .bh-input {
      background-color: #0B1120;
      border: 1px solid #334155;
      color: #F8FAFC;
      border-radius: 0.65rem;
      padding: 0.6rem 0.85rem;
      transition: all 0.2s;
      width: 100%;
    }
    .bh-input:focus {
      outline: none;
      border-color: #F59E0B;
      box-shadow: 0 0 0 2px rgba(245, 158, 11, 0.2);
    }
    .bh-btn {
      border-radius: 0.65rem;
      font-weight: 700;
      transition: all 0.2s;
      display: inline-flex;
      align-items: center;
      justify-content: center;
      gap: 0.45rem;
      cursor: pointer;
    }
    .bh-btn:active { transform: scale(0.97); }

    /* Toggle Switch */
    .switch { position: relative; display: inline-block; width: 46px; height: 24px; }
    .switch input { opacity: 0; width: 0; height: 0; }
    .slider {
      position: absolute; cursor: pointer; top: 0; left: 0; right: 0; bottom: 0;
      background-color: #334155; transition: .3s; border-radius: 24px;
    }
    .slider:before {
      position: absolute; content: ""; height: 18px; width: 18px; left: 3px; bottom: 3px;
      background-color: white; transition: .3s; border-radius: 50%;
    }
    input:checked + .slider { background-color: #10B981; }
    input:checked + .slider:before { transform: translateX(22px); }

    /* Thermal Print Styles (58mm / 80mm) */
    @media print {
      body * { visibility: hidden !important; }
      #thermalPrintArea, #thermalPrintArea * { visibility: visible !important; }
      #thermalPrintArea {
        position: fixed !important;
        left: 0 !important;
        top: 0 !important;
        width: 80mm !important;
        background: #fff !important;
        color: #000 !important;
        padding: 4mm !important;
        font-family: 'Cairo', monospace !important;
        font-size: 12px !important;
        z-index: 99999 !important;
      }
      .no-print { display: none !important; }
    }
  </style>
</head>
<body class="min-h-screen flex flex-col">

  <!-- Toast Notification Container -->
  <div id="toastContainer" class="fixed top-4 left-4 z-50 flex flex-col gap-2 max-w-sm w-full pointer-events-none"></div>

  <!-- Custom Modal Container -->
  <div id="customModal" class="fixed inset-0 bg-black/75 backdrop-blur-sm z-50 hidden items-center justify-center p-4">
    <div class="bh-panel max-w-2xl w-full max-h-[90vh] overflow-y-auto p-6 relative animate-fadeIn">
      <div class="flex items-center justify-between border-b border-slate-700 pb-3 mb-4">
        <h3 id="modalTitle" class="text-xl font-extrabold text-amber-400"></h3>
        <button onclick="closeModal()" class="text-slate-400 hover:text-white p-2 rounded-lg bg-slate-800/80">
          <i class="fa-solid fa-xmark text-lg"></i>
        </button>
      </div>
      <div id="modalBody"></div>
    </div>
  </div>

  <!-- Hidden Thermal Receipt Print Area -->
  <div id="thermalPrintArea" class="hidden"></div>

  <!-- ==================== LOGIN SCREEN ==================== -->
  <div id="loginScreen" class="min-h-screen flex items-center justify-center p-4 bg-[radial-gradient(ellipse_at_top,_var(--tw-gradient-stops))] from-slate-900 via-[#0B1120] to-black">
    <div class="bh-panel max-w-md w-full p-7 border-amber-500/30">
      <div class="text-center mb-6">
        <div class="w-20 h-20 mx-auto rounded-2xl bg-gradient-to-br from-amber-400 to-amber-600 flex items-center justify-center shadow-lg shadow-amber-500/20 mb-4">
          <i class="fa-solid fa-horse-head text-4xl text-slate-950"></i>
        </div>
        <h1 class="text-2xl font-black text-white tracking-wide" data-label="appName">شركة بلاك هورس للتوزيع</h1>
        <p class="text-xs text-amber-400 font-bold mt-1">BLACK HORSE ENTERPRISE ERP • SAP CORE</p>
      </div>

      <form onsubmit="handleLogin(event)" class="space-y-4">
        <div>
          <label class="block text-xs font-bold text-slate-300 mb-1">البريد الإلكتروني أو كود المندوب</label>
          <input type="text" id="loginIdentifier" class="bh-input" placeholder="admin@blackhorse.com أو 101" value="admin@blackhorse.com" required />
        </div>
        <div>
          <label class="block text-xs font-bold text-slate-300 mb-1">كلمة المرور</label>
          <input type="password" id="loginPassword" class="bh-input" placeholder="••••••" value="123456" required />
        </div>
        <button type="submit" class="bh-btn w-full py-3 bg-gradient-to-r from-amber-500 to-amber-600 hover:from-amber-400 hover:to-amber-500 text-slate-950 text-base shadow-lg shadow-amber-500/20">
          <i class="fa-solid fa-right-to-bracket"></i>
          <span>تسجيل الدخول للنظام</span>
        </button>
      </form>

      <div class="mt-6 pt-5 border-t border-slate-800">
        <p class="text-xs text-slate-400 text-center mb-3 font-semibold">دخول سريع للتجربة (تبديل الصلاحيات الفوري):</p>
        <div class="grid grid-cols-3 gap-2">
          <button onclick="quickLogin('admin')" class="bh-btn py-2 px-2 bg-slate-800 hover:bg-amber-500/20 border border-amber-500/40 text-amber-400 text-xs">
            <i class="fa-solid fa-crown"></i> الأدمن العام
          </button>
          <button onclick="quickLogin('rep')" class="bh-btn py-2 px-2 bg-slate-800 hover:bg-emerald-500/20 border border-emerald-500/40 text-emerald-400 text-xs">
            <i class="fa-solid fa-truck-fast"></i> مندوب (101)
          </button>
          <button onclick="quickLogin('auditor')" class="bh-btn py-2 px-2 bg-slate-800 hover:bg-sky-500/20 border border-sky-500/40 text-sky-400 text-xs">
            <i class="fa-solid fa-user-shield"></i> مراجع خارجي
          </button>
        </div>
      </div>
    </div>
  </div>

  <!-- ==================== MAIN APP SHELL ==================== -->
  <div id="appShell" class="hidden flex-1 flex flex-col lg:flex-row min-h-screen">
    
    <!-- Mobile Header -->
    <header class="lg:hidden bg-slate-900/95 border-b border-slate-800 px-4 py-3 flex items-center justify-between sticky top-0 z-40">
      <div class="flex items-center gap-2.5">
        <button onclick="toggleSidebar()" class="p-2 rounded-lg bg-slate-800 text-amber-400">
          <i class="fa-solid fa-bars text-lg"></i>
        </button>
        <div class="flex items-center gap-2">
          <i class="fa-solid fa-horse-head text-amber-400 text-xl"></i>
          <span class="font-extrabold text-sm text-white" data-label="appName">بلاك هورس ERP</span>
        </div>
      </div>
      <div class="flex items-center gap-2">
        <span id="mobileSyncBadge" class="px-2 py-1 rounded-full text-[10px] font-bold bg-emerald-500/20 text-emerald-400 border border-emerald-500/30">
          <i class="fa-solid fa-wifi"></i> متصل
        </span>
        <button onclick="logout()" class="p-2 rounded-lg bg-rose-500/10 text-rose-400 text-xs">
          <i class="fa-solid fa-power-off"></i>
        </button>
      </div>
    </header>

    <!-- Sidebar Navigation -->
    <aside id="sidebar" class="fixed inset-y-0 right-0 z-40 w-72 bg-[#0F172A] border-l border-slate-800 transform translate-x-full lg:translate-x-0 lg:static transition-transform duration-300 flex flex-col">
      <!-- Brand -->
      <div class="p-5 border-b border-slate-800 flex items-center justify-between">
        <div class="flex items-center gap-3">
          <div class="w-11 h-11 rounded-xl bg-gradient-to-br from-amber-400 to-amber-600 flex items-center justify-center shadow-md">
            <i class="fa-solid fa-horse-head text-2xl text-slate-950"></i>
          </div>
          <div>
            <h2 class="font-black text-base text-white leading-tight" data-label="appName">بلاك هورس ERP</h2>
            <span class="text-[11px] text-amber-400 font-bold">SAP CORE + FIELD POS</span>
          </div>
        </div>
        <button onclick="toggleSidebar()" class="lg:hidden text-slate-400 hover:text-white p-1">
          <i class="fa-solid fa-xmark text-xl"></i>
        </button>
      </div>

      <!-- User Profile Card -->
      <div class="p-4 mx-3 mt-3 rounded-xl bg-slate-800/60 border border-slate-700/60">
        <div class="flex items-center gap-3">
          <div class="w-10 h-10 rounded-full bg-amber-500/20 border border-amber-500/40 flex items-center justify-center text-amber-400 font-black" id="userAvatar">A</div>
          <div class="overflow-hidden">
            <p class="font-bold text-sm text-white truncate" id="currentUserName">الأدمن العام</p>
            <p class="text-[11px] text-slate-400 truncate" id="currentUserRole">صلاحية مطلقة (Super Admin)</p>
          </div>
        </div>
        <div class="mt-3 pt-2.5 border-t border-slate-700/60 flex items-center justify-between text-[11px]">
          <span id="syncStatusIndicator" class="flex items-center gap-1.5 text-emerald-400 font-bold">
            <span class="w-2 h-2 rounded-full bg-emerald-400 animate-pulse"></span>
            جاهز (Offline + Cloud)
          </span>
          <button onclick="openFirebaseModal()" class="text-amber-400 hover:underline font-bold">
            <i class="fa-solid fa-database"></i> Firebase
          </button>
        </div>
      </div>

      <!-- Navigation Links -->
      <nav id="sidebarNav" class="flex-1 overflow-y-auto p-3 space-y-1 text-sm font-bold">
        <!-- Populated dynamically based on permissions & feature toggles -->
      </nav>

      <!-- Bottom Actions -->
      <div class="p-3 border-t border-slate-800 space-y-2">
        <button onclick="exportFullSystemBackup()" class="bh-btn w-full py-2.5 bg-slate-800 hover:bg-slate-700 text-emerald-400 border border-emerald-500/30 text-xs">
          <i class="fa-solid fa-file-excel"></i> نسخ احتياطي شامل (Excel)
        </button>
        <button onclick="logout()" class="bh-btn w-full py-2.5 bg-rose-500/10 hover:bg-rose-500/20 text-rose-400 border border-rose-500/30 text-xs">
          <i class="fa-solid fa-right-from-bracket"></i> تسجيل الخروج
        </button>
      </div>
    </aside>

    <!-- Sidebar Backdrop for Mobile -->
    <div id="sidebarBackdrop" onclick="toggleSidebar()" class="fixed inset-0 bg-black/60 z-30 hidden lg:hidden"></div>

    <!-- Main Workspace -->
    <main class="flex-1 flex flex-col min-w-0 bg-[#0B1120]">
      <!-- Top Bar -->
      <div class="hidden lg:flex items-center justify-between px-6 py-4 bg-slate-900/60 border-b border-slate-800/80">
        <div>
          <h1 id="pageTitle" class="text-xl font-black text-white">لوحة القيادة المركزية</h1>
          <p id="pageSubtitle" class="text-xs text-slate-400 mt-0.5">متابعة لحظية للمبيعات، المخازن، الذمم، وحركة المناديب</p>
        </div>
        <div class="flex items-center gap-3">
          <div id="inactiveRadarAlert" class="hidden"></div>
          <div id="fleetMaintAlert" class="hidden"></div>
          <button onclick="navigateTo('pos')" class="bh-btn px-4 py-2 bg-gradient-to-r from-amber-500 to-amber-600 hover:from-amber-400 hover:to-amber-500 text-slate-950 text-xs shadow-md">
            <i class="fa-solid fa-plus"></i> <span data-label="newInvoiceBtn">فاتورة مبيعات جديدة</span>
          </button>
        </div>
      </div>

      <!-- Dynamic View Content -->
      <div id="viewContainer" class="p-4 lg:p-6 flex-1 overflow-y-auto space-y-6">
        <!-- Rendered dynamically -->
      </div>
    </main>
  </div>

<script>
/**
 * ============================================================================
 * BLACK HORSE ENTERPRISE ERP - CORE ENGINE (SAP + FIELD POS + DYNAMIC ADMIN)
 * ============================================================================
 */

// 1. DEFAULT DYNAMIC CONFIGURATION & LABELS
const DEFAULT_LABELS = {
  appName: "شركة بلاك هورس للتوزيع",
  dashboard: "لوحة القيادة المركزية",
  pos: "نقطة البيع والفواتير",
  invoices: "سجل الفواتير (تحكم كامل)",
  inventory: "المخازن وعُهد المناديب",
  debts: "الذمم التراكمية وكشوف الحساب",
  ratios: "حاسبة النسب المرنة والمجمعة",
  supplier: "تسوية الموردين (الشركة الشرقية)",
  clients: "العملاء وخطوط السير (GPS)",
  fleet: "الأسطول وصيانة الـ 1500 كم",
  hr: "الحضور والانصراف والموارد البشرية",
  reports: "التقارير الزمنية وتصفير السجلات",
  superAdmin: "مركز تحكم الأدمن الشامل",
  newInvoiceBtn: "فاتورة مبيعات جديدة",
  repCustodyLabel: "عهدة المندوب",
  debtLabel: "الذمم / المديونية",
  quotaLabel: "الكوتة الشهرية"
};

const DEFAULT_FEATURES = {
  pos: true,
  barcode: true,
  quotaLock: true,
  rawbtPrint: true,
  inventory: true,
  debts: true,
  whatsappStatement: true,
  ratios: true,
  supplier: true,
  clientsGps: true,
  inactiveRadar: true,
  fleet: true,
  hrAttendance: true,
  reports: true,
  excelExport: true
};

const DEFAULT_RULES = {
  inactivityDays: 10,
  oilChangeKm: 1500,
  companyPhone: "01000000000",
  receiptHeader: "شركة بلاك هورس للتجارة والتوزيع\nتوزيع منتجات الشركة الشرقية\nالرقم الضريبي: 300-412-985",
  receiptFooter: "شكراً لتعاملكم مع شركة بلاك هورس\nالبضاعة المباعة ترد وتستبدل خلال 48 ساعة"
};

// 2. INITIAL SEED DATA (Realistic Distribution Data)
const INITIAL_STATE = {
  labels: { ...DEFAULT_LABELS },
  features: { ...DEFAULT_FEATURES },
  rules: { ...DEFAULT_RULES },
  firebaseConfig: null,
  users: [
    { id: "u_admin", name: "حسام (المدير العام)", email: "admin@blackhorse.com", code: "admin", password: "123456", role: "admin", active: true, permissions: { all: true } },
    { id: "u_101", name: "محمد وحيد", email: "rep101@blackhorse.com", code: "101", password: "123456", role: "rep", warehouseId: "wh_rep_101", active: true, permissions: { pos: true, invoices: true, inventory: true, debts: true, clients: true, hr: true } },
    { id: "u_102", name: "هانى حذيفة", email: "rep102@blackhorse.com", code: "102", password: "123456", role: "rep", warehouseId: "wh_rep_102", active: true, permissions: { pos: true, invoices: true, inventory: true, debts: true, clients: true, hr: true } },
    { id: "u_103", name: "مصطفى عبد الرازق", email: "rep103@blackhorse.com", code: "103", password: "123456", role: "rep", warehouseId: "wh_rep_103", active: true, permissions: { pos: true, invoices: true, inventory: true, debts: true, clients: true, hr: true } },
    { id: "u_104", name: "أحمد المرحومى", email: "rep104@blackhorse.com", code: "104", password: "123456", role: "rep", warehouseId: "wh_rep_104", active: true, permissions: { pos: true, invoices: true, inventory: true, debts: true, clients: true, hr: true } },
    { id: "u_105", name: "محمد سلطان", email: "rep105@blackhorse.com", code: "105", password: "123456", role: "rep", warehouseId: "wh_rep_105", active: true, permissions: { pos: true, invoices: true, inventory: true, debts: true, clients: true, hr: true } },
    { id: "u_auditor", name: "مكتب المراجعة الخارجية", email: "auditor@external.com", code: "auditor", password: "123456", role: "external", active: true, permissions: { dashboard: true, invoices: true, debts: true, supplier: true, reports: true, excelExport: true } }
  ],
  warehouses: [
    { id: "wh_main", name: "المخزن الرئيسي", type: "main" },
    { id: "wh_menouf", name: "مخزن فرع منوف", type: "branch" },
    { id: "wh_ashmoun", name: "مخزن فرع أشمون", type: "branch" },
    { id: "wh_rep_101", name: "عهدة المندوب: محمد وحيد (101)", type: "rep", repCode: "101" },
    { id: "wh_rep_102", name: "عهدة المندوب: هانى حذيفة (102)", type: "rep", repCode: "102" },
    { id: "wh_rep_103", name: "عهدة المندوب: مصطفى عبد الرازق (103)", type: "rep", repCode: "103" },
    { id: "wh_rep_104", name: "عهدة المندوب: أحمد المرحومى (104)", type: "rep", repCode: "104" },
    { id: "wh_rep_105", name: "عهدة المندوب: محمد سلطان (105)", type: "rep", repCode: "105" }
  ],
  products: [
    { id: "p_1", barcode: "6221001", name: "كليوباترا كينج سايز (كرتونة)", price: 17500, cost: 17100, defaultQuota: 15, stock: { wh_main: 120, wh_menouf: 45, wh_ashmoun: 40, wh_rep_101: 12, wh_rep_102: 10, wh_rep_103: 8, wh_rep_104: 9, wh_rep_105: 11 } },
    { id: "p_2", barcode: "6221002", name: "كليوباترا سوفت كوين (كرتونة)", price: 17500, cost: 17100, defaultQuota: 15, stock: { wh_main: 100, wh_menouf: 30, wh_ashmoun: 35, wh_rep_101: 10, wh_rep_102: 8, wh_rep_103: 7, wh_rep_104: 6, wh_rep_105: 9 } },
    { id: "p_3", barcode: "6221003", name: "كليوباترا بوكس أبيض (كرتونة)", price: 18200, cost: 17800, defaultQuota: 10, stock: { wh_main: 85, wh_menouf: 25, wh_ashmoun: 20, wh_rep_101: 8, wh_rep_102: 6, wh_rep_103: 5, wh_rep_104: 5, wh_rep_105: 7 } },
    { id: "p_4", barcode: "6221004", name: "بوسطن / لايت (كرتونة)", price: 17800, cost: 17400, defaultQuota: 8, stock: { wh_main: 60, wh_menouf: 15, wh_ashmoun: 18, wh_rep_101: 5, wh_rep_102: 4, wh_rep_103: 4, wh_rep_104: 3, wh_rep_105: 5 } },
    { id: "p_5", barcode: "6221005", name: "مونديال أحمر / أزرق (كرتونة)", price: 18500, cost: 18100, defaultQuota: 10, stock: { wh_main: 90, wh_menouf: 22, wh_ashmoun: 25, wh_rep_101: 7, wh_rep_102: 7, wh_rep_103: 6, wh_rep_104: 6, wh_rep_105: 8 } }
  ],
  clients: [
    { id: "c_1", name: "سوبر ماركت البركة - منوف", phone: "201011112222", route: "خط منوف 1", repCode: "101", lat: 30.4658, lng: 30.9317, carriedOverBalance: -5000, lastOrderDate: new Date(Date.now() - 2 * 86400000).toISOString(), quotas: { p_1: 15, p_2: 15, p_3: 10, p_4: 8, p_5: 10 } },
    { id: "c_2", name: "ماركت أولاد رجب - أشمون", phone: "201022223333", route: "خط أشمون الرئيسي", repCode: "102", lat: 30.2981, lng: 30.9765, carriedOverBalance: 0, lastOrderDate: new Date(Date.now() - 13 * 86400000).toISOString(), quotas: { p_1: 12, p_2: 12, p_3: 8, p_4: 6, p_5: 8 } },
    { id: "c_3", name: "وكالة النور التجارية", phone: "201033334444", route: "خط منوف 2", repCode: "101", lat: 30.4710, lng: 30.9401, carriedOverBalance: -12000, lastOrderDate: new Date(Date.now() - 1 * 86400000).toISOString(), quotas: { p_1: 20, p_2: 20, p_3: 15, p_4: 10, p_5: 15 } },
    { id: "c_4", name: "محلات الأمانة - الباجور", phone: "201044445555", route: "خط الباجور", repCode: "103", lat: 30.4312, lng: 31.0345, carriedOverBalance: 2500, lastOrderDate: new Date(Date.now() - 16 * 86400000).toISOString(), quotas: { p_1: 10, p_2: 10, p_3: 6, p_4: 5, p_5: 6 } }
  ],
  invoices: [
    {
      id: "INV-1001",
      date: new Date(Date.now() - 2 * 86400000).toISOString(),
      clientId: "c_1",
      clientName: "سوبر ماركت البركة - منوف",
      repCode: "101",
      repName: "محمد وحيد",
      sourceWarehouseId: "wh_rep_101",
      items: [
        { productId: "p_1", name: "كليوباترا كينج سايز (كرتونة)", qty: 3, price: 17500, total: 52500 },
        { productId: "p_3", name: "كليوباترا بوكس أبيض (كرتونة)", qty: 2, price: 18200, total: 36400 }
      ],
      totalAmount: 88900,
      paidAmount: 80000,
      status: "active", // active | voided
      notes: "تحصيل نقدي + آجل"
    },
    {
      id: "INV-1002",
      date: new Date().toISOString(),
      clientId: "c_3",
      clientName: "وكالة النور التجارية",
      repCode: "101",
      repName: "محمد وحيد",
      sourceWarehouseId: "wh_rep_101",
      items: [
        { productId: "p_2", name: "كليوباترا سوفت كوين (كرتونة)", qty: 4, price: 17500, total: 70000 }
      ],
      totalAmount: 70000,
      paidAmount: 70000,
      status: "active",
      notes: "خالص نقداً"
    }
  ],
  payments: [
    { id: "PAY-501", date: new Date(Date.now() - 86400000).toISOString(), clientId: "c_1", amount: 5000, method: "نقدي مع المندوب", notes: "سداد جزء من الرصيد المرحل" }
  ],
  ratioRules: [
    { id: "r_1", name: "نسبة تحميل كليوباترا الأساسية", type: "percentage", value: 1.5, targetProductId: "all", description: "1.5% عمولة/نسبة على إجمالي المسحوبات" },
    { id: "r_2", name: "قاعدة تحميل كرتونة بوكس لكل 5 كينج", type: "carton_ratio", baseProductId: "p_1", baseQty: 5, rewardProductId: "p_3", rewardQty: 1, description: "لكل 5 كراتين كينج سايز يفتح حصة 1 كرتونة بوكس أبيض" }
  ],
  supplierSettlements: [
    { id: "SUP-01", date: new Date(Date.now() - 3 * 86400000).toISOString(), supplierName: "الشركة الشرقية (Eastern Company)", type: "deposit", method: "تحويل بنكي - البنك الأهلي", amount: 500000, reference: "TRX-99812", notes: "إيداع حصة الأسبوع" },
    { id: "SUP-02", date: new Date(Date.now() - 2 * 86400000).toISOString(), supplierName: "الشركة الشرقية (Eastern Company)", type: "invoice", method: "فاتورة استلام بضاعة", amount: 485000, reference: "EC-4410", notes: "استلام 28 كرتونة متنوعة" },
    { id: "SUP-03", date: new Date().toISOString(), supplierName: "الشركة الشرقية (Eastern Company)", type: "deposit", method: "إنستا باي (InstaPay)", amount: 150000, reference: "IP-7741", notes: "دفعة تكملة رصيد" }
  ],
  vehicles: [
    { id: "v_1", name: "سيارة الدبابة", plate: "م ن ف 4821", driver: "محمد وحيد", currentKm: 46620, lastOilChangeKm: 45000 },
    { id: "v_2", name: "بايك 1 (توزيع سريع)", plate: "م ن ف 9104", driver: "هانى حذيفة", currentKm: 18950, lastOilChangeKm: 18000 },
    { id: "v_3", name: "بايك 2 (توزيع سريع)", plate: "م ن ف 3319", driver: "مصطفى عبد الرازق", currentKm: 23100, lastOilChangeKm: 21500 }
  ],
  hrEmployees: [
    { id: "emp_1", name: "محمد وحيد", role: "مندوب مبيعات أول", code: "101", salary: 6500, custodyCash: 2500, phone: "01001112233" },
    { id: "emp_2", name: "هانى حذيفة", role: "مندوب مبيعات", code: "102", salary: 6000, custodyCash: 1500, phone: "01002223344" },
    { id: "emp_3", name: "مصطفى عبد الرازق", role: "مندوب مبيعات", code: "103", salary: 6000, custodyCash: 1000, phone: "01003334455" },
    { id: "emp_4", name: "أحمد المرحومى", role: "مندوب مبيعات", code: "104", salary: 5800, custodyCash: 0, phone: "01004445566" },
    { id: "emp_5", name: "محمد سلطان", role: "مندوب مبيعات", code: "105", salary: 5800, custodyCash: 0, phone: "01005556677" },
    { id: "emp_6", name: "محمود السيد", role: "أمين المخزن الرئيسي", code: "W01", salary: 5500, custodyCash: 0, phone: "01006667788" }
  ],
  attendance: [
    { id: "att_1", date: new Date().toISOString().split('T')[0], empName: "محمد وحيد", code: "101", checkIn: "08:05 ص", checkOut: "04:30 م", location: "مقر الشركة - منوف" },
    { id: "att_2", date: new Date().toISOString().split('T')[0], empName: "هانى حذيفة", code: "102", checkIn: "08:15 ص", checkOut: "--", location: "خط أشمون" }
  ],
  auditLogs: [
    { id: "LOG-1", timestamp: new Date().toISOString(), user: "حسام (المدير العام)", action: "تهيئة النظام", details: "تشغيل نظام بلاك هورس ERP الموحد بنجاح" }
  ],
  archivedPeriods: []
};

// Global Application State
let appState = {};
let currentUser = null;
let currentView = "dashboard";
let firebaseApp = null;
let firebaseDb = null;
let isFirebaseConnected = false;
let posCart = [];

// 3. STORAGE & FIREBASE SYNC ENGINE
function loadState() {
  const saved = localStorage.getItem("BH_ERP_STATE_V1");
  if (saved) {
    try {
      const parsed = JSON.parse(saved);
      appState = {
        ...INITIAL_STATE,
        ...parsed,
        labels: { ...DEFAULT_LABELS, ...(parsed.labels || {}) },
        features: { ...DEFAULT_FEATURES, ...(parsed.features || {}) },
        rules: { ...DEFAULT_RULES, ...(parsed.rules || {}) }
      };
    } catch (e) {
      appState = JSON.parse(JSON.stringify(INITIAL_STATE));
    }
  } else {
    appState = JSON.parse(JSON.stringify(INITIAL_STATE));
    saveState(false);
  }
  if (appState.firebaseConfig) {
    initFirebase(appState.firebaseConfig, false);
  }
}

function saveState(syncCloud = true) {
  localStorage.setItem("BH_ERP_STATE_V1", JSON.stringify(appState));
  if (syncCloud && isFirebaseConnected && firebaseDb) {
    firebaseDb.ref("blackHorseERP").set(appState).catch(err => {
      console.warn("Firebase sync queued offline:", err);
    });
  }
}

function initFirebase(config, showAlert = true) {
  try {
    if (!config || !config.databaseURL) throw new Error("رابط databaseURL مطلوب");
    if (!firebase.apps.length) {
      firebaseApp = firebase.initializeApp(config);
    } else {
      firebaseApp = firebase.app();
    }
    firebaseDb = firebase.database();
    
    // Listen to connection status
    firebaseDb.ref(".info/connected").on("value", (snap) => {
      isFirebaseConnected = snap.val() === true;
      updateSyncUI();
    });

    // Real-time two-way listener
    firebaseDb.ref("blackHorseERP").on("value", (snapshot) => {
      const remoteData = snapshot.val();
      if (remoteData) {
        appState = {
          ...INITIAL_STATE,
          ...remoteData,
          labels: { ...DEFAULT_LABELS, ...(remoteData.labels || {}) },
          features: { ...DEFAULT_FEATURES, ...(remoteData.features || {}) },
          rules: { ...DEFAULT_RULES, ...(remoteData.rules || {}) }
        };
        localStorage.setItem("BH_ERP_STATE_V1", JSON.stringify(appState));
        applyDynamicLabels();
        if (currentUser) {
          buildSidebar();
          renderCurrentView();
        }
      } else {
        // Push initial local data to empty Firebase
        firebaseDb.ref("blackHorseERP").set(appState);
      }
    });

    appState.firebaseConfig = config;
    saveState(false);
    if (showAlert) showToast("تم الاتصال بقاعدة بيانات Firebase بنجاح ومزامنة البيانات لحظياً!", "success");
  } catch (err) {
    if (showAlert) showToast("خطأ في إعدادات Firebase: " + err.message, "error");
  }
}

function updateSyncUI() {
  const desktopInd = document.getElementById("syncStatusIndicator");
  const mobileBadge = document.getElementById("mobileSyncBadge");
  if (isFirebaseConnected) {
    if (desktopInd) desktopInd.innerHTML = `<span class="w-2 h-2 rounded-full bg-emerald-400 animate-ping"></span> متصل بـ Firebase لحظياً`;
    if (mobileBadge) mobileBadge.innerHTML = `<i class="fa-solid fa-cloud"></i> Firebase Live`;
  } else {
    if (desktopInd) desktopInd.innerHTML = `<span class="w-2 h-2 rounded-full bg-amber-400"></span> وضع أوفلاين (حفظ محلي)`;
    if (mobileBadge) mobileBadge.innerHTML = `<i class="fa-solid fa-hard-drive"></i> Offline Ready`;
  }
}

function logAudit(action, details) {
  if (!appState.auditLogs) appState.auditLogs = [];
  appState.auditLogs.unshift({
    id: "LOG-" + Date.now().toString().slice(-5),
    timestamp: new Date().toISOString(),
    user: currentUser ? currentUser.name : "النظام",
    action,
    details
  });
}

// 4. UI HELPERS (Labels, Toasts, Modals, Formatting)
function L(key) {
  return (appState.labels && appState.labels[key]) ? appState.labels[key] : (DEFAULT_LABELS[key] || key);
}

function applyDynamicLabels() {
  document.querySelectorAll("[data-label]").forEach(el => {
    const key = el.getAttribute("data-label");
    el.textContent = L(key);
  });
}

function fmtMoney(num) {
  return Number(num || 0).toLocaleString("en-US", { minimumFractionDigits: 0, maximumFractionDigits: 2 }) + " ج.م";
}

function fmtDate(isoStr) {
  if (!isoStr) return "-";
  const d = new Date(isoStr);
  return d.toLocaleDateString("ar-EG", { year: "numeric", month: "short", day: "numeric", hour: "2-digit", minute: "2-digit" });
}

function showToast(msg, type = "info") {
  const container = document.getElementById("toastContainer");
  const colors = {
    success: "bg-emerald-950/95 border-emerald-500 text-emerald-200",
    error: "bg-rose-950/95 border-rose-500 text-rose-200",
    warning: "bg-amber-950/95 border-amber-500 text-amber-200",
    info: "bg-slate-900/95 border-sky-500 text-sky-200"
  };
  const icons = {
    success: "fa-circle-check text-emerald-400",
    error: "fa-circle-exclamation text-rose-400",
    warning: "fa-triangle-exclamation text-amber-400",
    info: "fa-circle-info text-sky-400"
  };
  const el = document.createElement("div");
  el.className = `pointer-events-auto border-r-4 p-3.5 rounded-xl shadow-2xl flex items-center gap-3 text-xs font-bold transition-all ${colors[type] || colors.info}`;
  el.innerHTML = `<i class="fa-solid ${icons[type]} text-base"></i><span class="flex-1">${msg}</span>`;
  container.appendChild(el);
  setTimeout(() => { el.style.opacity = "0"; setTimeout(() => el.remove(), 300); }, 3800);
}

function openModal(title, htmlContent) {
  document.getElementById("modalTitle").textContent = title;
  document.getElementById("modalBody").innerHTML = htmlContent;
  const m = document.getElementById("customModal");
  m.classList.remove("hidden");
  m.classList.add("flex");
}

function closeModal() {
  const m = document.getElementById("customModal");
  m.classList.add("hidden");
  m.classList.remove("flex");
}

function toggleSidebar() {
  const sb = document.getElementById("sidebar");
  const bd = document.getElementById("sidebarBackdrop");
  sb.classList.toggle("translate-x-full");
  bd.classList.toggle("hidden");
}

// 5. AUTHENTICATION & PERMISSIONS
function handleLogin(e) {
  e.preventDefault();
  const idVal = document.getElementById("loginIdentifier").value.trim().toLowerCase();
  const passVal = document.getElementById("loginPassword").value.trim();

  const user = appState.users.find(u =>
    u.active &&
    (u.email.toLowerCase() === idVal || u.code.toLowerCase() === idVal) &&
    u.password === passVal
  );

  if (!user) {
    showToast("بيانات الدخول غير صحيحة أو الحساب موقوف من الإدارة!", "error");
    return;
  }
  loginSuccess(user);
}

function quickLogin(roleType) {
  let user;
  if (roleType === "admin") user = appState.users.find(u => u.role === "admin");
  else if (roleType === "rep") user = appState.users.find(u => u.code === "101");
  else user = appState.users.find(u => u.role === "external");
  if (user) loginSuccess(user);
}

function loginSuccess(user) {
  currentUser = user;
  document.getElementById("loginScreen").classList.add("hidden");
  document.getElementById("appShell").classList.remove("hidden");
  document.getElementById("currentUserName").textContent = user.name;
  document.getElementById("userAvatar").textContent = user.name.charAt(0);
  
  const roleNames = {
    admin: "صلاحية مطلقة (Super Admin)",
    rep: `مندوب مبيعات (كود ${user.code})`,
    external: "مستخدم خارجي (صلاحيات مخصصة)"
  };
  document.getElementById("currentUserRole").textContent = roleNames[user.role] || user.role;

  applyDynamicLabels();
  buildSidebar();
  checkGlobalAlerts();
  navigateTo(hasPerm("dashboard") ? "dashboard" : (hasPerm("pos") ? "pos" : "invoices"));
  showToast(`مرحباً بك يا ${user.name}`, "success");
}

function logout() {
  currentUser = null;
  document.getElementById("appShell").classList.add("hidden");
  document.getElementById("loginScreen").classList.remove("hidden");
}

function hasPerm(permKey) {
  if (!currentUser) return false;
  if (currentUser.role === "admin" || (currentUser.permissions && currentUser.permissions.all)) return true;
  return Boolean(currentUser.permissions && currentUser.permissions[permKey]);
}

function isFeatureEnabled(featKey) {
  return appState.features[featKey] !== false;
}

// 6. SIDEBAR & NAVIGATION
function buildSidebar() {
  const navItems = [
    { id: "dashboard", icon: "fa-chart-pie", label: L("dashboard"), show: true },
    { id: "pos", icon: "fa-cash-register", label: L("pos"), show: isFeatureEnabled("pos") && hasPerm("pos") },
    { id: "invoices", icon: "fa-file-invoice-dollar", label: L("invoices"), show: hasPerm("invoices") },
    { id: "inventory", icon: "fa-warehouse", label: L("inventory"), show: isFeatureEnabled("inventory") && hasPerm("inventory") },
    { id: "debts", icon: "fa-scale-balanced", label: L("debts"), show: isFeatureEnabled("debts") && hasPerm("debts") },
    { id: "ratios", icon: "fa-percent", label: L("ratios"), show: isFeatureEnabled("ratios") && hasPerm("ratios") },
    { id: "supplier", icon: "fa-building-columns", label: L("supplier"), show: isFeatureEnabled("supplier") && hasPerm("supplier") },
    { id: "clients", icon: "fa-map-location-dot", label: L("clients"), show: isFeatureEnabled("clientsGps") && hasPerm("clients") },
    { id: "fleet", icon: "fa-truck-pickup", label: L("fleet"), show: isFeatureEnabled("fleet") && hasPerm("fleet") },
    { id: "hr", icon: "fa-id-card-clip", label: L("hr"), show: isFeatureEnabled("hrAttendance") && hasPerm("hr") },
    { id: "reports", icon: "fa-chart-line", label: L("reports"), show: isFeatureEnabled("reports") && hasPerm("reports") },
    { id: "superAdmin", icon: "fa-sliders", label: L("superAdmin"), show: currentUser.role === "admin", highlight: true }
  ];

  const nav = document.getElementById("sidebarNav");
  nav.innerHTML = navItems.filter(i => i.show).map(item => {
    const active = currentView === item.id;
    const baseClass = active
      ? "bg-gradient-to-l from-amber-500/20 to-amber-500/5 text-amber-400 border-r-4 border-amber-400"
      : (item.highlight ? "text-amber-300 hover:bg-slate-800/80 border border-amber-500/20 mt-3" : "text-slate-300 hover:bg-slate-800/60 hover:text-white");
    return `
      <button onclick="navigateTo('${item.id}')" class="w-full flex items-center gap-3 px-3.5 py-2.5 rounded-xl transition-all text-right ${baseClass}">
        <i class="fa-solid ${item.icon} w-5 text-center ${active ? 'text-amber-400' : 'text-slate-400'}"></i>
        <span class="truncate">${item.label}</span>
      </button>
    `;
  }).join("");
}

function navigateTo(viewId) {
  currentView = viewId;
  buildSidebar();
  if (window.innerWidth < 1024) {
    document.getElementById("sidebar").classList.add("translate-x-full");
    document.getElementById("sidebarBackdrop").classList.add("hidden");
  }
  document.getElementById("pageTitle").textContent = L(viewId) || L("dashboard");
  renderCurrentView();
}

function renderCurrentView() {
  const container = document.getElementById("viewContainer");
  switch (currentView) {
    case "dashboard": renderDashboard(container); break;
    case "pos": renderPOS(container); break;
    case "invoices": renderInvoices(container); break;
    case "inventory": renderInventory(container); break;
    case "debts": renderDebts(container); break;
    case "ratios": renderRatios(container); break;
    case "supplier": renderSupplier(container); break;
    case "clients": renderClients(container); break;
    case "fleet": renderFleet(container); break;
    case "hr": renderHR(container); break;
    case "reports": renderReports(container); break;
    case "superAdmin": renderSuperAdmin(container); break;
    default: renderDashboard(container);
  }
}

// 7. GLOBAL RADAR & ALERTS
function getInactiveClients() {
  const threshold = Number(appState.rules.inactivityDays || 10);
  const now = Date.now();
  return appState.clients.filter(c => {
    const last = new Date(c.lastOrderDate || 0).getTime();
    const diffDays = Math.floor((now - last) / 86400000);
    return diffDays >= threshold;
  });
}

function getMaintenanceAlerts() {
  const threshold = Number(appState.rules.oilChangeKm || 1500);
  return appState.vehicles.filter(v => (v.currentKm - v.lastOilChangeKm) >= threshold);
}

function checkGlobalAlerts() {
  const inactive = isFeatureEnabled("inactiveRadar") ? getInactiveClients() : [];
  const maint = isFeatureEnabled("fleet") ? getMaintenanceAlerts() : [];

  const radarEl = document.getElementById("inactiveRadarAlert");
  if (inactive.length > 0 && radarEl) {
    radarEl.classList.remove("hidden");
    radarEl.innerHTML = `
      <button onclick="navigateTo('clients')" class="px-3 py-1.5 rounded-lg bg-rose-500/15 border border-rose-500/40 text-rose-300 text-xs font-bold flex items-center gap-2 animate-pulse">
        <i class="fa-solid fa-radar"></i> رادار الخمول: ${inactive.length} عملاء متوقفين
      </button>
    `;
  } else if (radarEl) radarEl.classList.add("hidden");

  const maintEl = document.getElementById("fleetMaintAlert");
  if (maint.length > 0 && maintEl) {
    maintEl.classList.remove("hidden");
    maintEl.innerHTML = `
      <button onclick="navigateTo('fleet')" class="px-3 py-1.5 rounded-lg bg-amber-500/15 border border-amber-500/40 text-amber-300 text-xs font-bold flex items-center gap-2">
        <i class="fa-solid fa-oil-can"></i> صيانة زيت مطلوبة (${maint.length})
      </button>
    `;
  } else if (maintEl) maintEl.classList.add("hidden");
}

// 8. FINANCIAL CALCULATIONS (CUMULATIVE DEBT ENGINE)
function getClientFinancials(clientId) {
  const client = appState.clients.find(c => c.id === clientId);
  if (!client) return { totalPurchases: 0, totalDeposits: 0, carriedOver: 0, netBalance: 0 };

  const activeInvoices = appState.invoices.filter(inv => inv.clientId === clientId && inv.status === "active");
  const totalPurchases = activeInvoices.reduce((sum, inv) => sum + Number(inv.totalAmount || 0), 0);
  const invoicePayments = activeInvoices.reduce((sum, inv) => sum + Number(inv.paidAmount || 0), 0);
  const standalonePayments = appState.payments.filter(p => p.clientId === clientId).reduce((sum, p) => sum + Number(p.amount || 0), 0);
  
  const totalDeposits = invoicePayments + standalonePayments;
  const carriedOver = Number(client.carriedOverBalance || 0);
  // Formula: (الإيداع + الرصيد المرحل - قيمة البضاعة) => Negative means client owes money (مدين), Positive means credit (دائن)
  const netBalance = (carriedOver + totalDeposits) - totalPurchases;
  return { totalPurchases, totalDeposits, carriedOver, netBalance };
}

function getClientMonthlyProductQty(clientId, productId) {
  const now = new Date();
  const curMonth = now.getMonth();
  const curYear = now.getFullYear();
  let qty = 0;
  appState.invoices.forEach(inv => {
    if (inv.clientId === clientId && inv.status === "active") {
      const d = new Date(inv.date);
      if (d.getMonth() === curMonth && d.getFullYear() === curYear) {
        (inv.items || []).forEach(it => {
          if (it.productId === productId) qty += Number(it.qty || 0);
        });
      }
    }
  });
  return qty;
}

// ============================================================================
// VIEW 1: DASHBOARD
// ============================================================================
function renderDashboard(container) {
  const activeInvoices = appState.invoices.filter(i => i.status === "active");
  const totalSales = activeInvoices.reduce((s, i) => s + Number(i.totalAmount), 0);
  const totalCollected = activeInvoices.reduce((s, i) => s + Number(i.paidAmount), 0) + appState.payments.reduce((s, p) => s + Number(p.amount), 0);
  
  let totalClientDebts = 0;
  appState.clients.forEach(c => {
    const fin = getClientFinancials(c.id);
    if (fin.netBalance < 0) totalClientDebts += Math.abs(fin.netBalance);
  });

  const inactiveClients = getInactiveClients();
  const maintVehicles = getMaintenanceAlerts();

  container.innerHTML = `
    <!-- KPI Cards -->
    <div class="grid grid-cols-1 sm:grid-cols-2 xl:grid-cols-4 gap-4">
      <div class="bh-panel p-5 border-r-4 border-amber-400">
        <div class="flex items-center justify-between">
          <div>
            <p class="text-xs text-slate-400 font-bold">إجمالي المبيعات النشطة</p>
            <h3 class="text-2xl font-black text-white mt-1">${fmtMoney(totalSales)}</h3>
          </div>
          <div class="w-12 h-12 rounded-xl bg-amber-500/15 text-amber-400 flex items-center justify-center text-xl">
            <i class="fa-solid fa-sack-dollar"></i>
          </div>
        </div>
        <p class="text-[11px] text-slate-400 mt-3">عدد الفواتير النشطة: <span class="text-amber-400 font-bold">${activeInvoices.length}</span></p>
      </div>

      <div class="bh-panel p-5 border-r-4 border-emerald-400">
        <div class="flex items-center justify-between">
          <div>
            <p class="text-xs text-slate-400 font-bold">إجمالي التحصيلات والإيداعات</p>
            <h3 class="text-2xl font-black text-emerald-400 mt-1">${fmtMoney(totalCollected)}</h3>
          </div>
          <div class="w-12 h-12 rounded-xl bg-emerald-500/15 text-emerald-400 flex items-center justify-center text-xl">
            <i class="fa-solid fa-hand-holding-dollar"></i>
          </div>
        </div>
        <p class="text-[11px] text-slate-400 mt-3">نسبة التحصيل: <span class="text-emerald-400 font-bold">${totalSales ? Math.round((totalCollected/totalSales)*100) : 100}%</span></p>
      </div>

      <div class="bh-panel p-5 border-r-4 border-rose-500">
        <div class="flex items-center justify-between">
          <div>
            <p class="text-xs text-slate-400 font-bold">إجمالي ${L("debtLabel")} بالسوق</p>
            <h3 class="text-2xl font-black text-rose-400 mt-1">${fmtMoney(totalClientDebts)}</h3>
          </div>
          <div class="w-12 h-12 rounded-xl bg-rose-500/15 text-rose-400 flex items-center justify-center text-xl">
            <i class="fa-solid fa-file-invoice"></i>
          </div>
        </div>
        <p class="text-[11px] text-slate-400 mt-3">العملاء النشطون: <span class="text-white font-bold">${appState.clients.length} عميل</span></p>
      </div>

      <div class="bh-panel p-5 border-r-4 border-sky-400">
        <div class="flex items-center justify-between">
          <div>
            <p class="text-xs text-slate-400 font-bold">أرصدة المخازن والعُهد</p>
            <h3 class="text-2xl font-black text-sky-400 mt-1">${appState.products.reduce((s, p) => s + Object.values(p.stock || {}).reduce((a, b) => a + Number(b), 0), 0)} كرتونة</h3>
          </div>
          <div class="w-12 h-12 rounded-xl bg-sky-500/15 text-sky-400 flex items-center justify-center text-xl">
            <i class="fa-solid fa-boxes-stacked"></i>
          </div>
        </div>
        <p class="text-[11px] text-slate-400 mt-3">موزعة على <span class="text-sky-400 font-bold">${appState.warehouses.length} مخازن وعهدة</span></p>
      </div>
    </div>

    <!-- Alerts Row -->
    <div class="grid grid-cols-1 lg:grid-cols-2 gap-4">
      <!-- Inactive Clients Radar -->
      <div class="bh-panel p-5">
        <div class="flex items-center justify-between mb-3">
          <h3 class="font-extrabold text-sm text-rose-400 flex items-center gap-2">
            <i class="fa-solid fa-satellite-dish"></i> رادار العملاء الخاملين (أكثر من ${appState.rules.inactivityDays} أيام)
          </h3>
          <span class="px-2.5 py-0.5 rounded-full text-xs font-bold bg-rose-500/20 text-rose-300">${inactiveClients.length}</span>
        </div>
        ${inactiveClients.length === 0 ? `<p class="text-xs text-slate-400">ممتاز! جميع العملاء قاموا بعمليات سحب حديثة.</p>` : `
          <div class="space-y-2 max-h-48 overflow-y-auto">
            ${inactiveClients.map(c => {
              const days = Math.floor((Date.now() - new Date(c.lastOrderDate).getTime()) / 86400000);
              return `
                <div class="flex items-center justify-between p-2.5 rounded-lg bg-slate-900/80 border border-rose-500/20 text-xs">
                  <div>
                    <p class="font-bold text-white">${c.name}</p>
                    <p class="text-slate-400 text-[11px]">${c.route} • مندوب ${c.repCode}</p>
                  </div>
                  <span class="px-2 py-1 rounded bg-rose-500/20 text-rose-300 font-bold">متوقف منذ ${days} يوم</span>
                </div>
              `;
            }).join("")}
          </div>
        `}
      </div>

      <!-- Fleet Oil Maintenance Radar -->
      <div class="bh-panel p-5">
        <div class="flex items-center justify-between mb-3">
          <h3 class="font-extrabold text-sm text-amber-400 flex items-center gap-2">
            <i class="fa-solid fa-wrench"></i> رادار صيانة الأسطول وتغيير الزيت (كل ${appState.rules.oilChangeKm} كم)
          </h3>
          <span class="px-2.5 py-0.5 rounded-full text-xs font-bold bg-amber-500/20 text-amber-300">${maintVehicles.length}</span>
        </div>
        <div class="space-y-2 max-h-48 overflow-y-auto">
          ${appState.vehicles.map(v => {
            const diff = v.currentKm - v.lastOilChangeKm;
            const needsOil = diff >= appState.rules.oilChangeKm;
            return `
              <div class="flex items-center justify-between p-2.5 rounded-lg bg-slate-900/80 border ${needsOil ? 'border-amber-500/50' : 'border-slate-800'} text-xs">
                <div>
                  <p class="font-bold text-white">${v.name} <span class="text-slate-400">(${v.plate})</span></p>
                  <p class="text-slate-400 text-[11px]">العداد الحالي: ${v.currentKm.toLocaleString()} كم • مقطوع منذ الصيانة: ${diff} كم</p>
                </div>
                ${needsOil
                  ? `<span class="px-2 py-1 rounded bg-amber-500/20 text-amber-300 font-bold animate-pulse">تغيير زيت فوراً!</span>`
                  : `<span class="px-2 py-1 rounded bg-emerald-500/20 text-emerald-300 font-bold">سليمة</span>`
                }
              </div>
            `;
          }).join("")}
        </div>
      </div>
    </div>

    <!-- Rep Custody Live Summary -->
    <div class="bh-panel p-5">
      <div class="flex items-center justify-between mb-4">
        <h3 class="font-extrabold text-base text-white flex items-center gap-2">
          <i class="fa-solid fa-truck-ramp-box text-amber-400"></i> موقف ${L("repCustodyLabel")} والمبيعات اللحظية
        </h3>
        <button onclick="exportTableToExcel('repCustodySummaryTable', 'تقرير_عهد_المناديب')" class="bh-btn px-3 py-1.5 bg-emerald-600/20 hover:bg-emerald-600/30 text-emerald-400 border border-emerald-500/30 text-xs">
          <i class="fa-solid fa-file-excel"></i> تصدير Excel
        </button>
      </div>
      <div class="overflow-x-auto">
        <table id="repCustodySummaryTable" class="w-full text-right text-xs">
          <thead>
            <tr class="border-b border-slate-700 text-slate-400">
              <th class="py-3 px-3">كود</th>
              <th class="py-3 px-3">اسم المندوب</th>
              <th class="py-3 px-3">إجمالي مبيعاته</th>
              <th class="py-3 px-3">المحصّل نقداً</th>
              <th class="py-3 px-3">الرصيد المتبقي بالعهدة (كرتونة)</th>
            </tr>
          </thead>
          <tbody class="divide-y divide-slate-800">
            ${appState.users.filter(u => u.role === "rep").map(rep => {
              const repInvs = activeInvoices.filter(i => i.repCode === rep.code);
              const sales = repInvs.reduce((s, i) => s + Number(i.totalAmount), 0);
              const paid = repInvs.reduce((s, i) => s + Number(i.paidAmount), 0);
              const custodyCount = appState.products.reduce((s, p) => s + Number((p.stock && p.stock[rep.warehouseId]) || 0), 0);
              return `
                <tr class="hover:bg-slate-800/40">
                  <td class="py-3 px-3 font-bold text-amber-400">${rep.code}</td>
                  <td class="py-3 px-3 font-bold text-white">${rep.name}</td>
                  <td class="py-3 px-3 font-bold text-sky-400">${fmtMoney(sales)}</td>
                  <td class="py-3 px-3 font-bold text-emerald-400">${fmtMoney(paid)}</td>
                  <td class="py-3 px-3 font-black text-white">${custodyCount} كرتونة</td>
                </tr>
              `;
            }).join("")}
          </tbody>
        </table>
      </div>
    </div>
  `;
}

// ============================================================================
// VIEW 2: POS & QUICK INVOICE CREATOR (SAP ATOMIC + QUOTA + BARCODE + RAWBT)
// ============================================================================
function renderPOS(container) {
  const defaultWh = currentUser.role === "rep" ? currentUser.warehouseId : "wh_rep_101";

  container.innerHTML = `
    <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
      <!-- Left 2 Cols: Invoice Builder -->
      <div class="lg:col-span-2 space-y-4">
        <div class="bh-panel p-5">
          <div class="flex flex-wrap items-center justify-between gap-3 mb-4">
            <h3 class="font-extrabold text-base text-amber-400 flex items-center gap-2">
              <i class="fa-solid fa-bolt"></i> إصدار فاتورة فورية (ربط SAP لحظي)
            </h3>
            ${isFeatureEnabled("barcode") ? `
              <div class="flex items-center gap-2 w-full sm:w-auto">
                <div class="relative flex-1 sm:w-64">
                  <i class="fa-solid fa-barcode absolute right-3 top-3 text-amber-400"></i>
                  <input type="text" id="barcodeScannerInput" onkeyup="handleBarcodeScan(event)" class="bh-input pr-9 text-xs" placeholder="امسح الباركود هنا (مثل 6221001)..." />
                </div>
              </div>
            ` : ""}
          </div>

          <!-- Client & Source Warehouse -->
          <div class="grid grid-cols-1 sm:grid-cols-2 gap-4 mb-4">
            <div>
              <label class="block text-xs font-bold text-slate-300 mb-1">العميل / المحل</label>
              <select id="posClientSelect" onchange="onPosClientChange()" class="bh-input text-xs font-bold">
                <option value="">-- اختر العميل --</option>
                ${appState.clients.map(c => `<option value="${c.id}">${c.name} (${c.route})</option>`).join("")}
              </select>
            </div>
            <div>
              <label class="block text-xs font-bold text-slate-300 mb-1">الصرف من (${L("repCustodyLabel")} / المخزن)</label>
              <select id="posWarehouseSelect" onchange="renderPosProductGrid()" class="bh-input text-xs font-bold" ${currentUser.role === "rep" ? "disabled" : ""}>
                ${appState.warehouses.map(w => `<option value="${w.id}" ${w.id === defaultWh ? "selected" : ""}>${w.name}</option>`).join("")}
              </select>
            </div>
          </div>

          <!-- Client Quota & Debt Status Bar -->
          <div id="posClientInfoBar" class="hidden mb-4 p-3 rounded-xl bg-slate-900/90 border border-slate-700 flex flex-wrap items-center justify-between gap-2 text-xs"></div>

          <!-- Product Quick Add Grid -->
          <h4 class="text-xs font-bold text-slate-400 mb-2">اختر الأصناف للإضافة للفاتورة:</h4>
          <div id="posProductGrid" class="grid grid-cols-1 sm:grid-cols-2 gap-2.5"></div>
        </div>
      </div>

      <!-- Right Col: Cart & Checkout -->
      <div class="bh-panel p-5 flex flex-col justify-between h-fit sticky top-4">
        <div>
          <h3 class="font-extrabold text-base text-white border-b border-slate-700 pb-3 mb-3 flex items-center justify-between">
            <span><i class="fa-solid fa-cart-shopping text-amber-400 ml-1"></i> سلة الفاتورة</span>
            <button onclick="posCart=[]; renderPosCart();" class="text-xs text-rose-400 hover:underline">تفريغ</button>
          </h3>

          <div id="posCartItems" class="space-y-2.5 max-h-72 overflow-y-auto mb-4">
            <p class="text-xs text-slate-500 text-center py-8">لم يتم إضافة أصناف بعد</p>
          </div>
        </div>

        <div class="border-t border-slate-700 pt-4 space-y-3">
          <div class="flex items-center justify-between text-sm">
            <span class="text-slate-400 font-bold">إجمالي الفاتورة:</span>
            <span id="posGrandTotal" class="text-xl font-black text-amber-400">0 ج.م</span>
          </div>

          <div>
            <label class="block text-xs font-bold text-slate-300 mb-1">المبلغ المدفوع / المحصّل الآن (ج.م)</label>
            <div class="flex gap-2">
              <input type="number" id="posPaidAmount" class="bh-input text-sm font-black text-emerald-400" placeholder="0" />
              <button type="button" onclick="fillFullPosAmount()" class="bh-btn px-3 bg-slate-800 hover:bg-slate-700 text-xs text-emerald-400 border border-emerald-500/30 whitespace-nowrap">خالص</button>
            </div>
          </div>

          <div>
            <label class="block text-xs font-bold text-slate-300 mb-1">ملاحظات الفاتورة</label>
            <input type="text" id="posNotes" class="bh-input text-xs" placeholder="اختياري..." />
          </div>

          <button onclick="submitPosInvoice()" class="bh-btn w-full py-3.5 bg-gradient-to-r from-emerald-500 to-emerald-600 hover:from-emerald-400 hover:to-emerald-500 text-slate-950 text-sm font-black shadow-lg shadow-emerald-500/20">
            <i class="fa-solid fa-check-double"></i> حفظ الفاتورة وترحيل SAP
          </button>
        </div>
      </div>
    </div>
  `;

  renderPosProductGrid();
  renderPosCart();
}

function onPosClientChange() {
  const clientId = document.getElementById("posClientSelect").value;
  const bar = document.getElementById("posClientInfoBar");
  if (!clientId) {
    bar.classList.add("hidden");
    renderPosProductGrid();
    return;
  }
  const client = appState.clients.find(c => c.id === clientId);
  const fin = getClientFinancials(clientId);
  const balText = fin.netBalance < 0
    ? `<span class="text-rose-400 font-black">عليه مديونية: ${fmtMoney(Math.abs(fin.netBalance))}</span>`
    : `<span class="text-emerald-400 font-black">له رصيد دائن: ${fmtMoney(fin.netBalance)}</span>`;

  bar.classList.remove("hidden");
  bar.innerHTML = `
    <div><span class="text-slate-400">العميل:</span> <strong class="text-white">${client.name}</strong></div>
    <div>${balText}</div>
    <div><span class="text-slate-400">حالة ${L("quotaLabel")}:</span> <strong class="text-amber-400">${isFeatureEnabled("quotaLock") ? "مفعّل (قفل تلقائي)" : "مفتوح"}</strong></div>
  `;
  renderPosProductGrid();
}

function renderPosProductGrid() {
  const whId = document.getElementById("posWarehouseSelect")?.value || "wh_rep_101";
  const clientId = document.getElementById("posClientSelect")?.value;
  const client = appState.clients.find(c => c.id === clientId);
  const grid = document.getElementById("posProductGrid");
  if (!grid) return;

  grid.innerHTML = appState.products.map(p => {
    const avail = Number((p.stock && p.stock[whId]) || 0);
    let quotaBadge = "";
    if (client && isFeatureEnabled("quotaLock")) {
      const maxQ = Number((client.quotas && client.quotas[p.id]) ?? p.defaultQuota);
      const usedQ = getClientMonthlyProductQty(client.id, p.id);
      const remQ = Math.max(0, maxQ - usedQ);
      quotaBadge = `<span class="text-[10px] px-1.5 py-0.5 rounded bg-amber-500/15 text-amber-300 font-bold">متبقي كوتة: ${remQ}/${maxQ}</span>`;
    }

    return `
      <div onclick="addToPosCart('${p.id}')" class="p-3 rounded-xl bg-slate-900/90 hover:bg-slate-800 border border-slate-700/80 hover:border-amber-500/50 cursor-pointer transition-all flex items-center justify-between">
        <div>
          <p class="font-bold text-xs text-white">${p.name}</p>
          <div class="flex items-center gap-2 mt-1">
            <span class="text-xs font-black text-emerald-400">${fmtMoney(p.price)}</span>
            ${quotaBadge}
          </div>
        </div>
        <div class="text-left">
          <span class="block text-[11px] font-bold ${avail > 0 ? 'text-sky-400' : 'text-rose-400'}">متاح: ${avail}</span>
          <span class="inline-block mt-1 px-2 py-0.5 rounded bg-amber-500 text-slate-950 text-[10px] font-black">+ إضافة</span>
        </div>
      </div>
    `;
  }).join("");
}

function handleBarcodeScan(e) {
  if (e.key === "Enter") {
    const code = e.target.value.trim();
    const prod = appState.products.find(p => p.barcode === code);
    if (prod) {
      addToPosCart(prod.id);
      e.target.value = "";
      showToast(`تمت إضافة: ${prod.name}`, "success");
    } else {
      showToast("الباركود غير مسجل في الأصناف!", "error");
    }
  }
}

function addToPosCart(productId) {
  const whId = document.getElementById("posWarehouseSelect").value;
  const clientId = document.getElementById("posClientSelect").value;
  const prod = appState.products.find(p => p.id === productId);
  if (!prod) return;

  const avail = Number((prod.stock && prod.stock[whId]) || 0);
  const existing = posCart.find(i => i.productId === productId);
  const nextQty = (existing ? existing.qty : 0) + 1;

  if (nextQty > avail) {
    showToast(`الرصيد المتاح في المخزن/العهدة هو (${avail}) كرتونة فقط!`, "error");
    return;
  }

  // Monthly Quota Check
  if (clientId && isFeatureEnabled("quotaLock")) {
    const client = appState.clients.find(c => c.id === clientId);
    const maxQ = Number((client.quotas && client.quotas[productId]) ?? prod.defaultQuota);
    const usedQ = getClientMonthlyProductQty(clientId, productId);
    if ((usedQ + nextQty) > maxQ) {
      if (currentUser.role !== "admin") {
        showToast(`قفل الكوتة الشهرية: العميل استهلك ${usedQ} من ${maxQ} كرتونة لهذا الصنف!`, "error");
        return;
      } else {
        showToast(`تنبيه أدمن: تم تجاوز الكوتة الشهرية (${maxQ}) بصلاحية المدير العام`, "warning");
      }
    }
  }

  if (existing) {
    existing.qty = nextQty;
    existing.total = existing.qty * existing.price;
  } else {
    posCart.push({ productId: prod.id, name: prod.name, qty: 1, price: prod.price, total: prod.price });
  }
  renderPosCart();
}

function updateCartQty(productId, delta) {
  const whId = document.getElementById("posWarehouseSelect").value;
  const prod = appState.products.find(p => p.id === productId);
  const item = posCart.find(i => i.productId === productId);
  if (!item) return;

  const newQty = item.qty + delta;
  if (newQty <= 0) {
    posCart = posCart.filter(i => i.productId !== productId);
  } else {
    const avail = Number((prod.stock && prod.stock[whId]) || 0);
    if (newQty > avail) {
      showToast("تجاوزت الرصيد المتاح!", "error");
      return;
    }
    item.qty = newQty;
    item.total = item.qty * item.price;
  }
  renderPosCart();
}

function renderPosCart() {
  const list = document.getElementById("posCartItems");
  const totalEl = document.getElementById("posGrandTotal");
  if (!list) return;

  if (posCart.length === 0) {
    list.innerHTML = `<p class="text-xs text-slate-500 text-center py-8">لم يتم إضافة أصناف بعد</p>`;
    totalEl.textContent = "0 ج.م";
    return;
  }

  let grand = 0;
  list.innerHTML = posCart.map(item => {
    grand += item.total;
    return `
      <div class="p-2.5 rounded-lg bg-slate-900 border border-slate-800 flex items-center justify-between text-xs">
        <div class="flex-1">
          <p class="font-bold text-white">${item.name}</p>
          <p class="text-amber-400 font-bold">${fmtMoney(item.total)}</p>
        </div>
        <div class="flex items-center gap-1.5">
          <button onclick="updateCartQty('${item.productId}', -1)" class="w-6 h-6 rounded bg-slate-800 text-rose-400 font-black">-</button>
          <span class="w-6 text-center font-black">${item.qty}</span>
          <button onclick="updateCartQty('${item.productId}', 1)" class="w-6 h-6 rounded bg-slate-800 text-emerald-400 font-black">+</button>
        </div>
      </div>
    `;
  }).join("");

  totalEl.textContent = fmtMoney(grand);
}

function fillFullPosAmount() {
  const grand = posCart.reduce((s, i) => s + i.total, 0);
  document.getElementById("posPaidAmount").value = grand;
}

function submitPosInvoice() {
  const clientId = document.getElementById("posClientSelect").value;
  const whId = document.getElementById("posWarehouseSelect").value;
  const paidAmount = Number(document.getElementById("posPaidAmount").value || 0);
  const notes = document.getElementById("posNotes").value.trim();

  if (!clientId) { showToast("يرجى اختيار العميل أولاً!", "error"); return; }
  if (posCart.length === 0) { showToast("الفاتورة فارغة! أضف صنفاً واحداً على الأقل.", "error"); return; }

  const client = appState.clients.find(c => c.id === clientId);
  const wh = appState.warehouses.find(w => w.id === whId);
  const totalAmount = posCart.reduce((s, i) => s + i.total, 0);

  // SAP Atomic Deduction from Warehouse / Rep Custody
  posCart.forEach(item => {
    const prod = appState.products.find(p => p.id === item.productId);
    if (prod && prod.stock) {
      prod.stock[whId] = Math.max(0, Number(prod.stock[whId] || 0) - item.qty);
    }
  });

  // Update Client Last Order Date
  client.lastOrderDate = new Date().toISOString();

  const newInv = {
    id: "INV-" + (1000 + appState.invoices.length + 1),
    date: new Date().toISOString(),
    clientId: client.id,
    clientName: client.name,
    repCode: wh.repCode || currentUser.code || "admin",
    repName: currentUser.name,
    sourceWarehouseId: whId,
    items: JSON.parse(JSON.stringify(posCart)),
    totalAmount,
    paidAmount,
    status: "active",
    notes
  };

  appState.invoices.unshift(newInv);
  logAudit("إصدار فاتورة SAP", `فاتورة ${newInv.id} للعميل ${client.name} بقيمة ${fmtMoney(totalAmount)}`);
  saveState();

  posCart = [];
  showToast(`تم حفظ الفاتورة ${newInv.id} وتحديث المخزن والذمم لحظياً!`, "success");
  openThermalReceiptModal(newInv.id);
  renderPOS(document.getElementById("viewContainer"));
}

// ============================================================================
// THERMAL RECEIPT & RAWBT BLUETOOTH PRINTING (XP-P323B / XP-80C)
// ============================================================================
function generateReceiptPlainText(inv) {
  const fin = getClientFinancials(inv.clientId);
  let txt = `${appState.rules.receiptHeader}\n`;
  txt += `================================\n`;
  txt += `فاتورة رقم: ${inv.id}\n`;
  txt += `التاريخ: ${fmtDate(inv.date)}\n`;
  txt += `العميل: ${inv.clientName}\n`;
  txt += `المندوب: ${inv.repName}\n`;
  txt += `--------------------------------\n`;
  inv.items.forEach(it => {
    txt += `${it.name}\n`;
    txt += `  ${it.qty} × ${it.price} = ${it.total} ج.م\n`;
  });
  txt += `--------------------------------\n`;
  txt += `إجمالي الفاتورة: ${inv.totalAmount} ج.م\n`;
  txt += `المدفوع نقداً: ${inv.paidAmount} ج.م\n`;
  txt += `الرصيد التراكمي للعميل: ${fin.netBalance} ج.م\n`;
  txt += `================================\n`;
  txt += `${appState.rules.receiptFooter}\n\n\n`;
  return txt;
}

function openThermalReceiptModal(invId) {
  const inv = appState.invoices.find(i => i.id === invId);
  if (!inv) return;
  const fin = getClientFinancials(inv.clientId);
  const plainText = generateReceiptPlainText(inv);
  // Base64 UTF-8 safe encoding for RawBT Android App
  const rawbtBase64 = btoa(unescape(encodeURIComponent(plainText)));
  const rawbtUrl = `rawbt:base64,${rawbtBase64}`;

  const receiptHtml = `
    <div id="receiptPreviewBox" class="bg-white text-slate-950 p-5 rounded-xl max-w-xs mx-auto font-mono text-xs shadow-xl border-2 border-dashed border-slate-400">
      <div class="text-center border-b-2 border-dashed border-slate-400 pb-2 mb-2 whitespace-pre-line font-bold">
        ${appState.rules.receiptHeader}
      </div>
      <div class="space-y-1 text-[11px] border-b border-slate-300 pb-2 mb-2">
        <div class="flex justify-between"><span>رقم الفاتورة:</span><strong>${inv.id}</strong></div>
        <div class="flex justify-between"><span>التاريخ:</span><span>${fmtDate(inv.date)}</span></div>
        <div class="flex justify-between"><span>العميل:</span><strong>${inv.clientName}</strong></div>
        <div class="flex justify-between"><span>المندوب:</span><span>${inv.repName}</span></div>
        ${inv.status === 'voided' ? `<div class="text-center bg-red-600 text-white font-bold py-1 mt-1">*** فاتورة ملغاة ***</div>` : ''}
      </div>
      <table class="w-full text-right text-[11px] border-b border-slate-300 pb-2 mb-2">
        <thead>
          <tr class="border-b border-slate-400">
            <th class="py-1">الصنف</th>
            <th class="py-1">ك</th>
            <th class="py-1">الإجمالي</th>
          </tr>
        </thead>
        <tbody>
          ${inv.items.map(it => `
            <tr>
              <td class="py-1">${it.name}</td>
              <td class="py-1 font-bold">${it.qty}</td>
              <td class="py-1 font-bold">${it.total}</td>
            </tr>
          `).join("")}
        </tbody>
      </table>
      <div class="space-y-1 text-xs font-bold">
        <div class="flex justify-between"><span>الإجمالي:</span><span>${fmtMoney(inv.totalAmount)}</span></div>
        <div class="flex justify-between"><span>المدفوع:</span><span>${fmtMoney(inv.paidAmount)}</span></div>
        <div class="flex justify-between border-t border-slate-400 pt-1">
          <span>صافي رصيد العميل:</span>
          <span>${fmtMoney(fin.netBalance)}</span>
        </div>
      </div>
      <div class="text-center border-t-2 border-dashed border-slate-400 pt-2 mt-3 text-[10px] whitespace-pre-line">
        ${appState.rules.receiptFooter}
      </div>
    </div>

    <div class="grid grid-cols-1 sm:grid-cols-2 gap-3 mt-5">
      <button onclick="printThermalBrowser()" class="bh-btn py-2.5 bg-amber-500 hover:bg-amber-400 text-slate-950 text-xs">
        <i class="fa-solid fa-print"></i> طباعة مباشرة (58mm/80mm)
      </button>
      ${isFeatureEnabled("rawbtPrint") ? `
        <a href="${rawbtUrl}" class="bh-btn py-2.5 bg-sky-600 hover:bg-sky-500 text-white text-xs">
          <i class="fa-brands fa-bluetooth-b"></i> طباعة بلوتوث RawBT
        </a>
      ` : ""}
    </div>
  `;

  openModal(`معاينة وطباعة الفاتورة (${inv.id})`, receiptHtml);
}

function printThermalBrowser() {
  const content = document.getElementById("receiptPreviewBox").innerHTML;
  const printArea = document.getElementById("thermalPrintArea");
  printArea.innerHTML = content;
  printArea.classList.remove("hidden");
  window.print();
  printArea.classList.add("hidden");
}

// ============================================================================
// VIEW 3: INVOICES LOG & FULL ADMIN CRUD (EDIT / VOID / HARD DELETE WITH SAP REVERSAL)
// ============================================================================
function renderInvoices(container) {
  const isAdmin = currentUser.role === "admin";
  const visibleInvoices = currentUser.role === "rep"
    ? appState.invoices.filter(i => i.repCode === currentUser.code)
    : appState.invoices;

  container.innerHTML = `
    <div class="bh-panel p-5">
      <div class="flex flex-wrap items-center justify-between gap-3 mb-4">
        <div>
          <h3 class="font-extrabold text-base text-white">${L("invoices")}</h3>
          <p class="text-xs text-slate-400">صلاحية الأدمن المطلقة: تعديل الفواتير، إلغاؤها، أو حذفها مع العكس التلقائي للأرصدة والمخازن</p>
        </div>
        <div class="flex items-center gap-2">
          <input type="text" id="invSearchInput" onkeyup="filterInvoicesTable()" class="bh-input text-xs w-48" placeholder="بحث برقم الفاتورة أو العميل..." />
          <button onclick="exportTableToExcel('invoicesMainTable', 'سجل_الفواتير')" class="bh-btn px-3 py-2 bg-emerald-600/20 text-emerald-400 border border-emerald-500/30 text-xs">
            <i class="fa-solid fa-file-excel"></i> Excel
          </button>
        </div>
      </div>

      <div class="overflow-x-auto">
        <table id="invoicesMainTable" class="w-full text-right text-xs">
          <thead>
            <tr class="border-b border-slate-700 text-slate-400">
              <th class="py-3 px-2">رقم الفاتورة</th>
              <th class="py-3 px-2">التاريخ</th>
              <th class="py-3 px-2">العميل</th>
              <th class="py-3 px-2">المندوب</th>
              <th class="py-3 px-2">الإجمالي</th>
              <th class="py-3 px-2">المدفوع</th>
              <th class="py-3 px-2">الحالة</th>
              <th class="py-3 px-2 text-center">إجراءات التحكم</th>
            </tr>
          </thead>
          <tbody id="invoicesTableBody" class="divide-y divide-slate-800">
            ${visibleInvoices.map(inv => `
              <tr class="hover:bg-slate-800/40 ${inv.status === 'voided' ? 'opacity-50 bg-rose-950/10' : ''}">
                <td class="py-3 px-2 font-black text-amber-400">${inv.id}</td>
                <td class="py-3 px-2 text-slate-300">${fmtDate(inv.date)}</td>
                <td class="py-3 px-2 font-bold text-white">${inv.clientName}</td>
                <td class="py-3 px-2 text-slate-300">${inv.repName}</td>
                <td class="py-3 px-2 font-bold text-sky-400">${fmtMoney(inv.totalAmount)}</td>
                <td class="py-3 px-2 font-bold text-emerald-400">${fmtMoney(inv.paidAmount)}</td>
                <td class="py-3 px-2">
                  ${inv.status === 'active'
                    ? `<span class="px-2 py-0.5 rounded bg-emerald-500/20 text-emerald-400 font-bold">سارية</span>`
                    : `<span class="px-2 py-0.5 rounded bg-rose-500/20 text-rose-400 font-bold">ملغاة (Void)</span>`
                  }
                </td>
                <td class="py-3 px-2">
                  <div class="flex items-center justify-center gap-1.5">
                    <button onclick="openThermalReceiptModal('${inv.id}')" title="طباعة ومعاينة" class="p-1.5 rounded bg-slate-800 hover:bg-sky-500/20 text-sky-400">
                      <i class="fa-solid fa-print"></i>
                    </button>
                    ${isAdmin ? `
                      <button onclick="openEditInvoiceModal('${inv.id}')" title="تعديل الفاتورة (SAP Recalc)" class="p-1.5 rounded bg-slate-800 hover:bg-amber-500/20 text-amber-400">
                        <i class="fa-solid fa-pen-to-square"></i>
                      </button>
                      ${inv.status === 'active' ? `
                        <button onclick="voidInvoice('${inv.id}')" title="إلغاء الفاتورة وعكس المخزن" class="p-1.5 rounded bg-slate-800 hover:bg-orange-500/20 text-orange-400">
                          <i class="fa-solid fa-ban"></i>
                        </button>
                      ` : ''}
                      <button onclick="hardDeleteInvoice('${inv.id}')" title="حذف نهائي وعكس الأرصدة" class="p-1.5 rounded bg-slate-800 hover:bg-rose-500/20 text-rose-400">
                        <i class="fa-solid fa-trash-can"></i>
                      </button>
                    ` : ''}
                  </div>
                </td>
              </tr>
            `).join("")}
          </tbody>
        </table>
      </div>
    </div>
  `;
}

function filterInvoicesTable() {
  const q = document.getElementById("invSearchInput").value.toLowerCase();
  document.querySelectorAll("#invoicesTableBody tr").forEach(tr => {
    tr.style.display = tr.textContent.toLowerCase().includes(q) ? "" : "none";
  });
}

// SAP Auto-Reversal Helper (Returns items back to warehouse/custody)
function reverseInvoiceStock(inv) {
  if (!inv || inv.status !== "active") return;
  const whId = inv.sourceWarehouseId;
  (inv.items || []).forEach(it => {
    const prod = appState.products.find(p => p.id === it.productId);
    if (prod && prod.stock) {
      prod.stock[whId] = Number(prod.stock[whId] || 0) + Number(it.qty || 0);
    }
  });
}

function voidInvoice(invId) {
  const inv = appState.invoices.find(i => i.id === invId);
  if (!inv || inv.status === "voided") return;

  reverseInvoiceStock(inv);
  inv.status = "voided";
  logAudit("إلغاء فاتورة (Void)", `تم إلغاء الفاتورة ${inv.id} للعميل ${inv.clientName} وإرجاع البضاعة للمخزن وعكس المديونية`);
  saveState();
  showToast(`تم إلغاء الفاتورة ${inv.id} وعكس المخزون والمديونية تلقائياً!`, "warning");
  renderCurrentView();
}

function hardDeleteInvoice(invId) {
  const idx = appState.invoices.findIndex(i => i.id === invId);
  if (idx === -1) return;
  const inv = appState.invoices[idx];

  if (inv.status === "active") {
    reverseInvoiceStock(inv);
  }
  appState.invoices.splice(idx, 1);
  logAudit("حذف فاتورة نهائياً", `تم حذف الفاتورة ${inv.id} (${inv.clientName}) بقيمة ${fmtMoney(inv.totalAmount)} وعكس تأثيرها`);
  saveState();
  showToast(`تم حذف الفاتورة ${inv.id} نهائياً وعكس كافة أرصدتها!`, "error");
  renderCurrentView();
}

function openEditInvoiceModal(invId) {
  const inv = appState.invoices.find(i => i.id === invId);
  if (!inv) return;

  const html = `
    <div class="space-y-4 text-xs">
      <div class="p-3 rounded-lg bg-amber-500/10 border border-amber-500/30 text-amber-300">
        <i class="fa-solid fa-circle-info"></i> عند حفظ التعديل، سيقوم محرك SAP بإرجاع الكميات القديمة للمخزن ثم خصم الكميات الجديدة وإعادة حساب مديونية العميل لحظياً.
      </div>
      <div>
        <label class="block font-bold text-slate-300 mb-1">العميل</label>
        <select id="editInvClient" class="bh-input">
          ${appState.clients.map(c => `<option value="${c.id}" ${c.id === inv.clientId ? "selected" : ""}>${c.name}</option>`).join("")}
        </select>
      </div>
      <div class="space-y-2">
        <label class="block font-bold text-slate-300">أصناف الفاتورة والكميات:</label>
        ${inv.items.map((it, idx) => `
          <div class="grid grid-cols-3 gap-2 items-center bg-slate-900 p-2 rounded-lg border border-slate-800">
            <span class="font-bold text-white col-span-1">${it.name}</span>
            <div>
              <span class="text-[10px] text-slate-400">الكمية:</span>
              <input type="number" id="editItemQty_${idx}" value="${it.qty}" min="0" class="bh-input py-1" />
            </div>
            <div>
              <span class="text-[10px] text-slate-400">سعر الكرتونة:</span>
              <input type="number" id="editItemPrice_${idx}" value="${it.price}" min="0" class="bh-input py-1" />
            </div>
          </div>
        `).join("")}
      </div>
      <div>
        <label class="block font-bold text-slate-300 mb-1">المبلغ المدفوع (ج.م)</label>
        <input type="number" id="editInvPaid" value="${inv.paidAmount}" class="bh-input font-bold text-emerald-400" />
      </div>
      <button onclick="saveEditedInvoice('${inv.id}')" class="bh-btn w-full py-3 bg-amber-500 hover:bg-amber-400 text-slate-950 font-black">
        <i class="fa-solid fa-floppy-disk"></i> حفظ التعديلات وإعادة الاحتساب
      </button>
    </div>
  `;
  openModal(`تعديل الفاتورة ${inv.id} (صلاحية الأدمن)`, html);
}

function saveEditedInvoice(invId) {
  const inv = appState.invoices.find(i => i.id === invId);
  if (!inv) return;

  // 1. Reverse old stock if invoice was active
  if (inv.status === "active") {
    reverseInvoiceStock(inv);
  }

  // 2. Build updated items
  const updatedItems = [];
  let newTotal = 0;
  inv.items.forEach((it, idx) => {
    const qty = Number(document.getElementById(`editItemQty_${idx}`).value || 0);
    const price = Number(document.getElementById(`editItemPrice_${idx}`).value || 0);
    if (qty > 0) {
      const total = qty * price;
      newTotal += total;
      updatedItems.push({ ...it, qty, price, total });
    }
  });

  // 3. Deduct new stock
  if (inv.status === "active") {
    updatedItems.forEach(it => {
      const prod = appState.products.find(p => p.id === it.productId);
      if (prod && prod.stock) {
        prod.stock[inv.sourceWarehouseId] = Math.max(0, Number(prod.stock[inv.sourceWarehouseId] || 0) - it.qty);
      }
    });
  }

  const newClientId = document.getElementById("editInvClient").value;
  const newClient = appState.clients.find(c => c.id === newClientId);
  const oldTotal = inv.totalAmount;

  inv.clientId = newClient.id;
  inv.clientName = newClient.name;
  inv.items = updatedItems;
  inv.totalAmount = newTotal;
  inv.paidAmount = Number(document.getElementById("editInvPaid").value || 0);

  logAudit("تعديل فاتورة", `تعديل فاتورة ${inv.id}: الإجمالي تغير من ${fmtMoney(oldTotal)} إلى ${fmtMoney(newTotal)}`);
  saveState();
  closeModal();
  showToast(`تم تعديل الفاتورة ${inv.id} وتحديث المخازن والذمم بنجاح!`, "success");
  renderCurrentView();
}

// ============================================================================
// VIEW 4: INVENTORY, MULTI-WAREHOUSE & REP CUSTODY TRANSFER
// ============================================================================
function renderInventory(container) {
  const isAdmin = currentUser.role === "admin";

  container.innerHTML = `
    <div class="space-y-6">
      ${isAdmin ? `
        <div class="grid grid-cols-1 lg:grid-cols-2 gap-4">
          <!-- Stock Transfer / Load Rep Custody -->
          <div class="bh-panel p-5">
            <h3 class="font-extrabold text-sm text-amber-400 mb-3 flex items-center gap-2">
              <i class="fa-solid fa-right-left"></i> تحويل بضاعة بين المخازن / تحميل ${L("repCustodyLabel")}
            </h3>
            <div class="grid grid-cols-1 sm:grid-cols-2 gap-3 text-xs">
              <div>
                <label class="block text-slate-400 mb-1">من مخزن:</label>
                <select id="trFromWh" class="bh-input">
                  ${appState.warehouses.map(w => `<option value="${w.id}">${w.name}</option>`).join("")}
                </select>
              </div>
              <div>
                <label class="block text-slate-400 mb-1">إلى مخزن / عهدة:</label>
                <select id="trToWh" class="bh-input">
                  ${appState.warehouses.map((w, i) => `<option value="${w.id}" ${i === 3 ? "selected" : ""}>${w.name}</option>`).join("")}
                </select>
              </div>
              <div>
                <label class="block text-slate-400 mb-1">الصنف:</label>
                <select id="trProduct" class="bh-input">
                  ${appState.products.map(p => `<option value="${p.id}">${p.name}</option>`).join("")}
                </select>
              </div>
              <div>
                <label class="block text-slate-400 mb-1">الكمية (كرتونة):</label>
                <input type="number" id="trQty" min="1" value="5" class="bh-input" />
              </div>
            </div>
            <button onclick="executeStockTransfer()" class="bh-btn w-full mt-3 py-2.5 bg-amber-500 hover:bg-amber-400 text-slate-950 text-xs">
              <i class="fa-solid fa-truck-loading"></i> تنفيذ التحويل اللحظي
            </button>
          </div>

          <!-- Add New Product / Add Incoming Stock -->
          <div class="bh-panel p-5">
            <h3 class="font-extrabold text-sm text-emerald-400 mb-3 flex items-center gap-2">
              <i class="fa-solid fa-box-open"></i> إضافة وارد للمخزن الرئيسي / صنف جديد
            </h3>
            <div class="grid grid-cols-1 sm:grid-cols-2 gap-3 text-xs">
              <div>
                <label class="block text-slate-400 mb-1">اسم الصنف:</label>
                <input type="text" id="newProdName" class="bh-input" placeholder="مثال: كليوباترا سوبر (كرتونة)" />
              </div>
              <div>
                <label class="block text-slate-400 mb-1">الباركود:</label>
                <input type="text" id="newProdBarcode" class="bh-input" placeholder="6221009" />
              </div>
              <div>
                <label class="block text-slate-400 mb-1">سعر البيع للكرتونة:</label>
                <input type="number" id="newProdPrice" class="bh-input" placeholder="18000" />
              </div>
              <div>
                <label class="block text-slate-400 mb-1">الكمية الواردة (المخزن الرئيسي):</label>
                <input type="number" id="newProdQty" class="bh-input" placeholder="50" />
              </div>
            </div>
            <button onclick="addNewProductOrStock()" class="bh-btn w-full mt-3 py-2.5 bg-emerald-500 hover:bg-emerald-400 text-slate-950 text-xs">
              <i class="fa-solid fa-plus"></i> إضافة للمخزون
            </button>
          </div>
        </div>
      ` : ""}

      <!-- Full Stock Matrix Table -->
      <div class="bh-panel p-5">
        <div class="flex items-center justify-between mb-4">
          <h3 class="font-extrabold text-base text-white">جرد الأرصدة اللحظي عبر جميع المخازن وعُهد المناديب</h3>
          <button onclick="exportTableToExcel('inventoryMatrixTable', 'أرصدة_المخازن_والعهد')" class="bh-btn px-3 py-1.5 bg-emerald-600/20 text-emerald-400 border border-emerald-500/30 text-xs">
            <i class="fa-solid fa-file-excel"></i> تصدير Excel
          </button>
        </div>
        <div class="overflow-x-auto">
          <table id="inventoryMatrixTable" class="w-full text-right text-xs">
            <thead>
              <tr class="border-b border-slate-700 text-slate-400">
                <th class="py-3 px-2">الباركود</th>
                <th class="py-3 px-2">الصنف</th>
                <th class="py-3 px-2">السعر</th>
                ${appState.warehouses.map(w => `<th class="py-3 px-2">${w.name}</th>`).join("")}
                <th class="py-3 px-2 text-amber-400">الإجمالي</th>
              </tr>
            </thead>
            <tbody class="divide-y divide-slate-800">
              ${appState.products.map(p => {
                let rowTotal = 0;
                const cells = appState.warehouses.map(w => {
                  const q = Number((p.stock && p.stock[w.id]) || 0);
                  rowTotal += q;
                  return `<td class="py-3 px-2 font-bold ${q === 0 ? 'text-rose-500' : 'text-slate-200'}">${q}</td>`;
                }).join("");
                return `
                  <tr class="hover:bg-slate-800/40">
                    <td class="py-3 px-2 text-slate-400 font-mono">${p.barcode}</td>
                    <td class="py-3 px-2 font-bold text-white">${p.name}</td>
                    <td class="py-3 px-2 text-emerald-400 font-bold">${fmtMoney(p.price)}</td>
                    ${cells}
                    <td class="py-3 px-2 font-black text-amber-400">${rowTotal}</td>
                  </tr>
                `;
              }).join("")}
            </tbody>
          </table>
        </div>
      </div>
    </div>
  `;
}

function executeStockTransfer() {
  const fromId = document.getElementById("trFromWh").value;
  const toId = document.getElementById("trToWh").value;
  const prodId = document.getElementById("trProduct").value;
  const qty = Number(document.getElementById("trQty").value || 0);

  if (fromId === toId) { showToast("لا يمكن التحويل لنفس المخزن!", "error"); return; }
  if (qty <= 0) { showToast("أدخل كمية صحيحة!", "error"); return; }

  const prod = appState.products.find(p => p.id === prodId);
  const avail = Number((prod.stock && prod.stock[fromId]) || 0);
  if (qty > avail) { showToast(`الرصيد المتاح في المخزن المصدر هو ${avail} فقط!`, "error"); return; }

  prod.stock[fromId] = avail - qty;
  prod.stock[toId] = Number(prod.stock[toId] || 0) + qty;

  const fromName = appState.warehouses.find(w => w.id === fromId)?.name;
  const toName = appState.warehouses.find(w => w.id === toId)?.name;
  logAudit("تحويل مخزني", `تحويل ${qty} كرتونة من (${prod.name}) من ${fromName} إلى ${toName}`);
  saveState();
  showToast("تم التحويل المخزني وتحديث العهدة بنجاح!", "success");
  renderCurrentView();
}

function addNewProductOrStock() {
  const name = document.getElementById("newProdName").value.trim();
  const barcode = document.getElementById("newProdBarcode").value.trim() || ("622" + Date.now().toString().slice(-4));
  const price = Number(document.getElementById("newProdPrice").value || 0);
  const qty = Number(document.getElementById("newProdQty").value || 0);

  if (!name || price <= 0) { showToast("أدخل اسم الصنف والسعر!", "error"); return; }

  const existing = appState.products.find(p => p.name === name || p.barcode === barcode);
  if (existing) {
    existing.price = price;
    existing.stock.wh_main = Number(existing.stock.wh_main || 0) + qty;
  } else {
    appState.products.push({
      id: "p_" + Date.now(),
      barcode,
      name,
      price,
      cost: Math.round(price * 0.97),
      defaultQuota: 10,
      stock: { wh_main: qty }
    });
  }
  logAudit("إضافة مخزون", `إضافة ${qty} كرتونة من (${name}) للمخزن الرئيسي`);
  saveState();
  showToast("تم تحديث المخزن الرئيسي بنجاح!", "success");
  renderCurrentView();
}

// ============================================================================
// VIEW 5: CUMULATIVE DEBTS, ROLLOVER & WHATSAPP STATEMENTS
// ============================================================================
function renderDebts(container) {
  const isAdmin = currentUser.role === "admin";

  container.innerHTML = `
    <div class="space-y-6">
      <!-- Record Payment / Deposit -->
      <div class="bh-panel p-5">
        <div class="flex flex-wrap items-center justify-between gap-3 mb-4">
          <h3 class="font-extrabold text-base text-emerald-400">
            <i class="fa-solid fa-money-bill-transfer"></i> تسجيل إيداع / تحصيل نقدي من عميل
          </h3>
          ${isAdmin ? `
            <button onclick="rolloverAllBalances()" class="bh-btn px-3 py-2 bg-amber-500/20 hover:bg-amber-500/30 text-amber-300 border border-amber-500/40 text-xs">
              <i class="fa-solid fa-calendar-check"></i> ترحيل الأرصدة للشهر الجديد
            </button>
          ` : ""}
        </div>
        <div class="grid grid-cols-1 sm:grid-cols-4 gap-3 text-xs">
          <select id="payClientSelect" class="bh-input">
            <option value="">-- اختر العميل --</option>
            ${appState.clients.map(c => `<option value="${c.id}">${c.name}</option>`).join("")}
          </select>
          <input type="number" id="payAmountInput" class="bh-input" placeholder="المبلغ المحصّل (ج.م)" />
          <input type="text" id="payMethodInput" class="bh-input" placeholder="طريقة الدفع (نقدي / تحويل / إنستاباي)" value="نقدي مع المندوب" />
          <button onclick="recordClientPayment()" class="bh-btn bg-emerald-500 hover:bg-emerald-400 text-slate-950 font-black py-2.5">
            <i class="fa-solid fa-plus-circle"></i> تسجيل الإيداع
          </button>
        </div>
      </div>

      <!-- Cumulative Debts Table -->
      <div class="bh-panel p-5">
        <div class="flex items-center justify-between mb-4">
          <div>
            <h3 class="font-extrabold text-base text-white">كشف ${L("debtLabel")} التراكمي للعملاء</h3>
            <p class="text-xs text-slate-400">المعادلة: (الرصيد المرحل + إجمالي الإيداعات - قيمة البضاعة المسحوبة)</p>
          </div>
          <button onclick="exportTableToExcel('debtsSummaryTable', 'كشف_الذمم_التراكمية')" class="bh-btn px-3 py-1.5 bg-emerald-600/20 text-emerald-400 border border-emerald-500/30 text-xs">
            <i class="fa-solid fa-file-excel"></i> تصدير Excel
          </button>
        </div>

        <div class="overflow-x-auto">
          <table id="debtsSummaryTable" class="w-full text-right text-xs">
            <thead>
              <tr class="border-b border-slate-700 text-slate-400">
                <th class="py-3 px-2">العميل</th>
                <th class="py-3 px-2">الرصيد المرحل</th>
                <th class="py-3 px-2">إجمالي المسحوبات</th>
                <th class="py-3 px-2">إجمالي الإيداعات</th>
                <th class="py-3 px-2">الرصيد الصافي الحالي</th>
                <th class="py-3 px-2 text-center">كشف حساب واتساب</th>
              </tr>
            </thead>
            <tbody class="divide-y divide-slate-800">
              ${appState.clients.map(c => {
                const fin = getClientFinancials(c.id);
                const statusBadge = fin.netBalance < 0
                  ? `<span class="text-rose-400 font-black">مدين بـ ${fmtMoney(Math.abs(fin.netBalance))}</span>`
                  : (fin.netBalance > 0
                    ? `<span class="text-emerald-400 font-black">دائن بـ ${fmtMoney(fin.netBalance)}</span>`
                    : `<span class="text-slate-400 font-bold">خالص (0 ج.م)</span>`);
                return `
                  <tr class="hover:bg-slate-800/40">
                    <td class="py-3 px-2 font-bold text-white">${c.name}</td>
                    <td class="py-3 px-2 text-slate-300">${fmtMoney(fin.carriedOver)}</td>
                    <td class="py-3 px-2 text-sky-400 font-bold">${fmtMoney(fin.totalPurchases)}</td>
                    <td class="py-3 px-2 text-emerald-400 font-bold">${fmtMoney(fin.totalDeposits)}</td>
                    <td class="py-3 px-2">${statusBadge}</td>
                    <td class="py-3 px-2 text-center">
                      ${isFeatureEnabled("whatsappStatement") ? `
                        <button onclick="sendWhatsAppStatement('${c.id}')" class="bh-btn px-3 py-1.5 bg-emerald-500/20 hover:bg-emerald-500/30 text-emerald-400 border border-emerald-500/40 text-xs">
                          <i class="fa-brands fa-whatsapp text-sm"></i> إرسال كشف حساب
                        </button>
                      ` : "-"}
                    </td>
                  </tr>
                `;
              }).join("")}
            </tbody>
          </table>
        </div>
      </div>
    </div>
  `;
}

function recordClientPayment() {
  const clientId = document.getElementById("payClientSelect").value;
  const amount = Number(document.getElementById("payAmountInput").value || 0);
  const method = document.getElementById("payMethodInput").value.trim() || "نقدي";

  if (!clientId || amount <= 0) { showToast("اختر العميل وأدخل مبلغ الإيداع!", "error"); return; }

  const client = appState.clients.find(c => c.id === clientId);
  appState.payments.unshift({
    id: "PAY-" + Date.now().toString().slice(-5),
    date: new Date().toISOString(),
    clientId,
    amount,
    method,
    notes: "تحصيل مسجل بالنظام"
  });

  logAudit("تسجيل إيداع عميل", `إيداع مبلغ ${fmtMoney(amount)} للعميل ${client.name} (${method})`);
  saveState();
  showToast("تم تسجيل الإيداع وتحديث رصيد العميل فوراً!", "success");
  renderCurrentView();
}

function rolloverAllBalances() {
  appState.clients.forEach(c => {
    const fin = getClientFinancials(c.id);
    c.carriedOverBalance = fin.netBalance;
  });
  // Archive current invoices/payments snapshot and clear active cycle if needed
  appState.archivedPeriods.push({
    date: new Date().toISOString(),
    invoicesCount: appState.invoices.length,
    invoices: JSON.parse(JSON.stringify(appState.invoices)),
    payments: JSON.parse(JSON.stringify(appState.payments))
  });
  appState.invoices = [];
  appState.payments = [];
  logAudit("ترحيل أرصدة شهري", "تم ترحيل صافي أرصدة جميع العملاء للشهر الجديد وأرشفة الفواتير السابقة");
  saveState();
  showToast("تم ترحيل جميع أرصدة العملاء للشهر الجديد بنجاح!", "success");
  renderCurrentView();
}

function sendWhatsAppStatement(clientId) {
  const client = appState.clients.find(c => c.id === clientId);
  if (!client) return;
  const fin = getClientFinancials(clientId);
  const clientInvoices = appState.invoices.filter(i => i.clientId === clientId && i.status === "active");

  let msg = `*${L("appName")}*\n`;
  msg += `📋 *كشف حساب تفصيلي للعميل*\n`;
  msg += `العميل: *${client.name}*\n`;
  msg += `التاريخ: ${new Date().toLocaleDateString("ar-EG")}\n`;
  msg += `----------------------------\n`;
  msg += `• الرصيد المرحل السابق: ${fmtMoney(fin.carriedOver)}\n`;
  msg += `• إجمالي المسحوبات: ${fmtMoney(fin.totalPurchases)}\n`;
  msg += `• إجمالي الإيداعات: ${fmtMoney(fin.totalDeposits)}\n`;
  msg += `----------------------------\n`;
  if (clientInvoices.length > 0) {
    msg += `*تفاصيل الفواتير:*\n`;
    clientInvoices.slice(0, 8).forEach(inv => {
      const itemsStr = inv.items.map(it => `${it.name} (${it.qty})`).join(" + ");
      msg += `▪ ${inv.id}: ${itemsStr} = *${fmtMoney(inv.totalAmount)}*\n`;
    });
    msg += `----------------------------\n`;
  }
  msg += fin.netBalance < 0
    ? `🔴 *صافي المديونية المستحقة: ${fmtMoney(Math.abs(fin.netBalance))}*`
    : `🟢 *صافي الرصيد الدائن لكم: ${fmtMoney(fin.netBalance)}*`;

  const phone = client.phone.replace(/[^0-9]/g, "");
  const url = `https://wa.me/${phone}?text=${encodeURIComponent(msg)}`;
  window.open(url, "_blank");
}

// ============================================================================
// VIEW 6: FLEXIBLE & BULK RATIO / COMMISSION CALCULATOR
// ============================================================================
function renderRatios(container) {
  const isAdmin = currentUser.role === "admin";

  container.innerHTML = `
    <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
      <!-- Bulk Ratio Calculator -->
      <div class="bh-panel p-5">
        <h3 class="font-extrabold text-base text-amber-400 mb-3 flex items-center gap-2">
          <i class="fa-solid fa-calculator"></i> حاسبة النسب المجمعة (للمناديب وكبار العملاء)
        </h3>
        <div class="space-y-3 text-xs">
          <div>
            <label class="block text-slate-300 font-bold mb-1">اختر قاعدة الاحتساب:</label>
            <select id="calcRuleSelect" class="bh-input">
              ${appState.ratioRules.map(r => `<option value="${r.id}">${r.name} (${r.description})</option>`).join("")}
            </select>
          </div>
          <div>
            <label class="block text-slate-300 font-bold mb-1">اختر المندوب أو العميل لحساب إجمالي تحميلاته:</label>
            <select id="calcTargetSelect" class="bh-input">
              <optgroup label="المناديب">
                ${appState.users.filter(u => u.role === "rep").map(r => `<option value="rep:${r.code}">مندوب: ${r.name} (${r.code})</option>`).join("")}
              </optgroup>
              <optgroup label="العملاء">
                ${appState.clients.map(c => `<option value="client:${c.id}">عميل: ${c.name}</option>`).join("")}
              </optgroup>
            </select>
          </div>
          <button onclick="runBulkRatioCalculation()" class="bh-btn w-full py-3 bg-gradient-to-r from-amber-500 to-amber-600 text-slate-950 font-black">
            <i class="fa-solid fa-bolt"></i> احسب النسبة / التحميل المجمع بضغطة واحدة
          </button>

          <div id="ratioCalcResultBox" class="hidden p-4 rounded-xl bg-slate-900 border border-emerald-500/40 mt-4"></div>
        </div>
      </div>

      <!-- Dynamic Ratio Rules Editor -->
      <div class="bh-panel p-5">
        <h3 class="font-extrabold text-base text-white mb-3 flex items-center gap-2">
          <i class="fa-solid fa-sliders text-sky-400"></i> قواعد النسب والتحميل المبرمجة
        </h3>
        <div class="space-y-2.5 mb-4">
          ${appState.ratioRules.map(r => `
            <div class="p-3 rounded-xl bg-slate-900 border border-slate-800 flex items-center justify-between text-xs">
              <div>
                <p class="font-bold text-white">${r.name}</p>
                <p class="text-slate-400 text-[11px] mt-0.5">${r.description}</p>
              </div>
              ${isAdmin ? `
                <button onclick="deleteRatioRule('${r.id}')" class="text-rose-400 hover:bg-rose-500/20 p-2 rounded-lg">
                  <i class="fa-solid fa-trash"></i>
                </button>
              ` : ""}
            </div>
          `).join("")}
        </div>

        ${isAdmin ? `
          <div class="border-t border-slate-800 pt-4 space-y-2.5 text-xs">
            <p class="font-bold text-amber-400">+ إضافة قاعدة نسبة مئوية جديدة من لوحة الإدارة:</p>
            <input type="text" id="newRuleName" class="bh-input" placeholder="اسم القاعدة (مثلاً: حافز مبيعات السوبر)" />
            <div class="grid grid-cols-2 gap-2">
              <input type="number" step="0.1" id="newRulePercent" class="bh-input" placeholder="النسبة % (مثلاً 2.5)" />
              <button onclick="addPercentageRule()" class="bh-btn bg-sky-600 hover:bg-sky-500 text-white">حفظ القاعدة</button>
            </div>
          </div>
        ` : ""}
      </div>
    </div>
  `;
}

function runBulkRatioCalculation() {
  const ruleId = document.getElementById("calcRuleSelect").value;
  const target = document.getElementById("calcTargetSelect").value;
  const rule = appState.ratioRules.find(r => r.id === ruleId);
  const [type, id] = target.split(":");

  const invs = appState.invoices.filter(i => i.status === "active" && (type === "rep" ? i.repCode === id : i.clientId === id));
  const totalSales = invs.reduce((s, i) => s + Number(i.totalAmount), 0);

  const resBox = document.getElementById("ratioCalcResultBox");
  resBox.classList.remove("hidden");

  if (rule.type === "percentage") {
    const commission = (totalSales * Number(rule.value)) / 100;
    resBox.innerHTML = `
      <p class="text-slate-400">إجمالي المبيعات الخاضعة للنسبة: <strong class="text-white">${fmtMoney(totalSales)}</strong></p>
      <p class="text-slate-400 mt-1">النسبة المطبقة: <strong class="text-amber-400">${rule.value}%</strong></p>
      <p class="text-base font-black text-emerald-400 mt-2">صافي النسبة المستحقة: ${fmtMoney(commission)}</p>
    `;
  } else {
    let baseCount = 0;
    invs.forEach(inv => inv.items.forEach(it => { if (it.productId === rule.baseProductId) baseCount += Number(it.qty); }));
    const unlockedCartons = Math.floor(baseCount / rule.baseQty) * rule.rewardQty;
    resBox.innerHTML = `
      <p class="text-slate-400">إجمالي مسحوبات الصنف الأساسي: <strong class="text-white">${baseCount} كرتونة</strong></p>
      <p class="text-base font-black text-emerald-400 mt-2">حصة التحميل المستحقة: ${unlockedCartons} كرتونة إضافية</p>
    `;
  }
}

function addPercentageRule() {
  const name = document.getElementById("newRuleName").value.trim();
  const val = Number(document.getElementById("newRulePercent").value || 0);
  if (!name || val <= 0) { showToast("أدخل اسم القاعدة والنسبة!", "error"); return; }
  appState.ratioRules.push({
    id: "r_" + Date.now(),
    name,
    type: "percentage",
    value: val,
    description: `نسبة ${val}% على إجمالي المبيعات`
  });
  saveState();
  showToast("تمت إضافة قاعدة النسبة بنجاح!", "success");
  renderCurrentView();
}

function deleteRatioRule(id) {
  appState.ratioRules = appState.ratioRules.filter(r => r.id !== id);
  saveState();
  renderCurrentView();
}

// ============================================================================
// VIEW 7: SUPPLIER RECONCILIATION (الشركة الشرقية)
// ============================================================================
function renderSupplier(container) {
  const deposits = appState.supplierSettlements.filter(s => s.type === "deposit").reduce((a, b) => a + Number(b.amount), 0);
  const invoices = appState.supplierSettlements.filter(s => s.type === "invoice").reduce((a, b) => a + Number(b.amount), 0);
  const diff = deposits - invoices;

  container.innerHTML = `
    <div class="space-y-6">
      <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
        <div class="bh-panel p-4 border-r-4 border-emerald-400">
          <p class="text-xs text-slate-400">إجمالي الإيداعات والتحويلات للمورد</p>
          <h3 class="text-xl font-black text-emerald-400 mt-1">${fmtMoney(deposits)}</h3>
        </div>
        <div class="bh-panel p-4 border-r-4 border-sky-400">
          <p class="text-xs text-slate-400">إجمالي فواتير التحميلات الواردة</p>
          <h3 class="text-xl font-black text-sky-400 mt-1">${fmtMoney(invoices)}</h3>
        </div>
        <div class="bh-panel p-4 border-r-4 ${diff >= 0 ? 'border-amber-400' : 'border-rose-500'}">
          <p class="text-xs text-slate-400">صافي التسوية / الفرق لدى الشركة الشرقية</p>
          <h3 class="text-xl font-black ${diff >= 0 ? 'text-amber-400' : 'text-rose-400'} mt-1">${fmtMoney(diff)}</h3>
        </div>
      </div>

      <!-- Add Supplier Transaction -->
      <div class="bh-panel p-5">
        <h3 class="font-extrabold text-sm text-amber-400 mb-3">تسجيل حركة تسوية جديدة مع الشركة الشرقية</h3>
        <div class="grid grid-cols-1 sm:grid-cols-5 gap-3 text-xs">
          <select id="supType" class="bh-input">
            <option value="deposit">إيداع / تحويل بنكي للمورد (+)</option>
            <option value="invoice">فاتورة تحميل بضاعة واردة (-)</option>
          </select>
          <select id="supMethod" class="bh-input">
            <option value="تحويل بنكي - البنك الأهلي">تحويل بنكي - البنك الأهلي</option>
            <option value="تحويل بنكي - بنك مصر">تحويل بنكي - بنك مصر</option>
            <option value="إنستا باي (InstaPay)">إنستا باي (InstaPay)</option>
            <option value="فودافون كاش / محافظ إلكترونية">فودافون كاش / محافظ إلكترونية</option>
            <option value="فاتورة استلام بضاعة">فاتورة استلام بضاعة</option>
          </select>
          <input type="number" id="supAmount" class="bh-input" placeholder="المبلغ (ج.م)" />
          <input type="text" id="supRef" class="bh-input" placeholder="رقم الإيصال / المرجع" />
          <button onclick="addSupplierRecord()" class="bh-btn bg-amber-500 hover:bg-amber-400 text-slate-950 font-black">إضافة الحركة</button>
        </div>
      </div>

      <!-- Table -->
      <div class="bh-panel p-5">
        <div class="flex items-center justify-between mb-4">
          <h3 class="font-extrabold text-base text-white">سجل مطابقة حسابات الشركة الشرقية</h3>
          <button onclick="exportTableToExcel('supplierTable', 'تسوية_الشركة_الشرقية')" class="bh-btn px-3 py-1.5 bg-emerald-600/20 text-emerald-400 border border-emerald-500/30 text-xs">
            <i class="fa-solid fa-file-excel"></i> تصدير Excel
          </button>
        </div>
        <div class="overflow-x-auto">
          <table id="supplierTable" class="w-full text-right text-xs">
            <thead>
              <tr class="border-b border-slate-700 text-slate-400">
                <th class="py-3 px-2">التاريخ</th>
                <th class="py-3 px-2">نوع الحركة</th>
                <th class="py-3 px-2">الوسيلة / البنك</th>
                <th class="py-3 px-2">رقم المرجع</th>
                <th class="py-3 px-2">المبلغ</th>
              </tr>
            </thead>
            <tbody class="divide-y divide-slate-800">
              ${appState.supplierSettlements.map(s => `
                <tr class="hover:bg-slate-800/40">
                  <td class="py-3 px-2 text-slate-300">${fmtDate(s.date)}</td>
                  <td class="py-3 px-2">
                    ${s.type === 'deposit'
                      ? `<span class="px-2 py-0.5 rounded bg-emerald-500/20 text-emerald-400 font-bold">إيداع / تحويل</span>`
                      : `<span class="px-2 py-0.5 rounded bg-sky-500/20 text-sky-400 font-bold">فاتورة تحميل</span>`
                    }
                  </td>
                  <td class="py-3 px-2 font-bold text-white">${s.method}</td>
                  <td class="py-3 px-2 font-mono text-amber-400">${s.reference}</td>
                  <td class="py-3 px-2 font-black ${s.type === 'deposit' ? 'text-emerald-400' : 'text-sky-400'}">${fmtMoney(s.amount)}</td>
                </tr>
              `).join("")}
            </tbody>
          </table>
        </div>
      </div>
    </div>
  `;
}

function addSupplierRecord() {
  const type = document.getElementById("supType").value;
  const method = document.getElementById("supMethod").value;
  const amount = Number(document.getElementById("supAmount").value || 0);
  const reference = document.getElementById("supRef").value.trim() || ("REF-" + Date.now().toString().slice(-4));

  if (amount <= 0) { showToast("أدخل مبلغاً صحيحاً!", "error"); return; }

  appState.supplierSettlements.unshift({
    id: "SUP-" + Date.now().toString().slice(-4),
    date: new Date().toISOString(),
    supplierName: "الشركة الشرقية",
    type,
    method,
    amount,
    reference
  });
  logAudit("تسوية مورد", `تسجيل ${type === 'deposit' ? 'إيداع' : 'فاتورة'} بقيمة ${fmtMoney(amount)} (${method})`);
  saveState();
  showToast("تم تسجيل حركة التسوية بنجاح!", "success");
  renderCurrentView();
}

// ============================================================================
// VIEW 8: CLIENTS, GPS ROUTES & INACTIVE RADAR
// ============================================================================
function renderClients(container) {
  const threshold = Number(appState.rules.inactivityDays || 10);

  container.innerHTML = `
    <div class="space-y-6">
      <!-- Register Client with GPS -->
      <div class="bh-panel p-5">
        <h3 class="font-extrabold text-sm text-amber-400 mb-3 flex items-center gap-2">
          <i class="fa-solid fa-location-crosshairs"></i> تسجيل عميل جديد مع إحداثيات الـ GPS وخط السير
        </h3>
        <div class="grid grid-cols-1 sm:grid-cols-3 lg:grid-cols-6 gap-3 text-xs">
          <input type="text" id="clName" class="bh-input" placeholder="اسم المحل / العميل" />
          <input type="text" id="clPhone" class="bh-input" placeholder="رقم الواتساب (2010...)" />
          <input type="text" id="clRoute" class="bh-input" placeholder="خط السير (مثلاً: خط منوف 1)" />
          <select id="clRep" class="bh-input">
            ${appState.users.filter(u => u.role === "rep").map(r => `<option value="${r.code}">${r.name} (${r.code})</option>`).join("")}
          </select>
          <div class="flex gap-1">
            <input type="text" id="clGps" class="bh-input" placeholder="Lat, Lng" readonly />
            <button type="button" onclick="captureClientGPS()" class="bh-btn px-3 bg-sky-600 text-white" title="التقاط الموقع الحالي">
              <i class="fa-solid fa-location-dot"></i>
            </button>
          </div>
          <button onclick="addNewClient()" class="bh-btn bg-emerald-500 hover:bg-emerald-400 text-slate-950 font-black">حفظ العميل</button>
        </div>
      </div>

      <!-- Clients & Routes Table -->
      <div class="bh-panel p-5">
        <div class="flex items-center justify-between mb-4">
          <h3 class="font-extrabold text-base text-white">دليل العملاء وخطوط السير ورادار النشاط</h3>
          <button onclick="exportTableToExcel('clientsRoutesTable', 'العملاء_وخطوط_السير')" class="bh-btn px-3 py-1.5 bg-emerald-600/20 text-emerald-400 border border-emerald-500/30 text-xs">
            <i class="fa-solid fa-file-excel"></i> تصدير Excel
          </button>
        </div>
        <div class="overflow-x-auto">
          <table id="clientsRoutesTable" class="w-full text-right text-xs">
            <thead>
              <tr class="border-b border-slate-700 text-slate-400">
                <th class="py-3 px-2">اسم العميل</th>
                <th class="py-3 px-2">خط السير</th>
                <th class="py-3 px-2">المندوب</th>
                <th class="py-3 px-2">حالة النشاط (رادار ${threshold} أيام)</th>
                <th class="py-3 px-2">الموقع GPS</th>
                <th class="py-3 px-2">تعديل الكوتة</th>
              </tr>
            </thead>
            <tbody class="divide-y divide-slate-800">
              ${appState.clients.map(c => {
                const days = Math.floor((Date.now() - new Date(c.lastOrderDate || 0).getTime()) / 86400000);
                const inactive = days >= threshold;
                return `
                  <tr class="hover:bg-slate-800/40">
                    <td class="py-3 px-2 font-bold text-white">${c.name}<br><span class="text-[10px] text-slate-400">${c.phone}</span></td>
                    <td class="py-3 px-2 text-amber-400 font-bold">${c.route}</td>
                    <td class="py-3 px-2 text-slate-300">${c.repCode}</td>
                    <td class="py-3 px-2">
                      ${inactive
                        ? `<span class="px-2 py-0.5 rounded bg-rose-500/20 text-rose-400 font-bold">خامل (${days} يوم)</span>`
                        : `<span class="px-2 py-0.5 rounded bg-emerald-500/20 text-emerald-400 font-bold">نشط (منذ ${days} يوم)</span>`
                      }
                    </td>
                    <td class="py-3 px-2">
                      ${c.lat ? `<a href="https://maps.google.com/?q=${c.lat},${c.lng}" target="_blank" class="text-sky-400 hover:underline font-bold"><i class="fa-solid fa-map-pin"></i> فتح الخريطة</a>` : "غير مسجل"}
                    </td>
                    <td class="py-3 px-2">
                      ${currentUser.role === "admin" ? `
                        <button onclick="openClientQuotaModal('${c.id}')" class="bh-btn px-2.5 py-1 bg-slate-800 hover:bg-amber-500/20 text-amber-400 text-[11px]">
                          <i class="fa-solid fa-lock"></i> ${L("quotaLabel")}
                        </button>
                      ` : "-"}
                    </td>
                  </tr>
                `;
              }).join("")}
            </tbody>
          </table>
        </div>
      </div>
    </div>
  `;
}

function captureClientGPS() {
  if (!navigator.geolocation) {
    document.getElementById("clGps").value = "30.4658, 30.9317";
    showToast("تم تعيين إحداثيات افتراضية للمتصفح", "info");
    return;
  }
  navigator.geolocation.getCurrentPosition(
    pos => {
      document.getElementById("clGps").value = `${pos.coords.latitude.toFixed(5)}, ${pos.coords.longitude.toFixed(5)}`;
      showToast("تم التقاط إحداثيات الـ GPS بدقة!", "success");
    },
    () => {
      document.getElementById("clGps").value = "30.4658, 30.9317";
      showToast("تعذر الوصول للـ GPS، تم وضع إحداثيات المنطقة الافتراضية", "warning");
    }
  );
}

function addNewClient() {
  const name = document.getElementById("clName").value.trim();
  const phone = document.getElementById("clPhone").value.trim() || "201000000000";
  const route = document.getElementById("clRoute").value.trim() || "خط عام";
  const repCode = document.getElementById("clRep").value;
  const gps = document.getElementById("clGps").value.split(",");

  if (!name) { showToast("أدخل اسم العميل!", "error"); return; }

  appState.clients.push({
    id: "c_" + Date.now(),
    name,
    phone,
    route,
    repCode,
    lat: gps[0] ? Number(gps[0].trim()) : 30.4658,
    lng: gps[1] ? Number(gps[1].trim()) : 30.9317,
    carriedOverBalance: 0,
    lastOrderDate: new Date().toISOString(),
    quotas: {}
  });
  saveState();
  showToast("تم تسجيل العميل وربطه بخط السير والـ GPS بنجاح!", "success");
  renderCurrentView();
}

function openClientQuotaModal(clientId) {
  const client = appState.clients.find(c => c.id === clientId);
  if (!client) return;
  const html = `
    <div class="space-y-3 text-xs">
      <p class="text-slate-300">تحديد الحد الأقصى الشهري (بالكرتونة) للعميل: <strong class="text-amber-400">${client.name}</strong></p>
      ${appState.products.map(p => {
        const val = (client.quotas && client.quotas[p.id]) ?? p.defaultQuota;
        return `
          <div class="flex items-center justify-between bg-slate-900 p-2.5 rounded-lg border border-slate-800">
            <span class="font-bold text-white">${p.name}</span>
            <input type="number" id="qInput_${p.id}" value="${val}" class="bh-input w-24 text-center font-bold" />
          </div>
        `;
      }).join("")}
      <button onclick="saveClientQuotas('${client.id}')" class="bh-btn w-full py-2.5 bg-amber-500 text-slate-950 font-black">حفظ الكوتة الشهرية</button>
    </div>
  `;
  openModal("إدارة الكوتة الشهرية للعميل", html);
}

function saveClientQuotas(clientId) {
  const client = appState.clients.find(c => c.id === clientId);
  if (!client.quotas) client.quotas = {};
  appState.products.forEach(p => {
    client.quotas[p.id] = Number(document.getElementById(`qInput_${p.id}`).value || 0);
  });
  saveState();
  closeModal();
  showToast("تم تحديث الكوتة الشهرية للعميل!", "success");
}

// ============================================================================
// VIEW 9: FLEET & 1,500 KM OIL CHANGE MAINTENANCE
// ============================================================================
function renderFleet(container) {
  const threshold = Number(appState.rules.oilChangeKm || 1500);

  container.innerHTML = `
    <div class="bh-panel p-5">
      <div class="flex items-center justify-between mb-4">
        <div>
          <h3 class="font-extrabold text-base text-white">إدارة أسطول النقل وعدّاد الصيانة (كل ${threshold} كم)</h3>
          <p class="text-xs text-slate-400">تسجيل قراءة العداد اليومية وإطلاق تنبيه تغيير الزيت تلقائياً</p>
        </div>
        <button onclick="exportTableToExcel('fleetTable', 'سجل_الأسطول_والصيانة')" class="bh-btn px-3 py-1.5 bg-emerald-600/20 text-emerald-400 border border-emerald-500/30 text-xs">
          <i class="fa-solid fa-file-excel"></i> تصدير Excel
        </button>
      </div>

      <div class="overflow-x-auto">
        <table id="fleetTable" class="w-full text-right text-xs">
          <thead>
            <tr class="border-b border-slate-700 text-slate-400">
              <th class="py-3 px-2">المركبة</th>
              <th class="py-3 px-2">رقم اللوحة</th>
              <th class="py-3 px-2">السائق / المندوب</th>
              <th class="py-3 px-2">العداد الحالي (كم)</th>
              <th class="py-3 px-2">آخر تغيير زيت (كم)</th>
              <th class="py-3 px-2">المقطوع منذ الصيانة</th>
              <th class="py-3 px-2">تحديث العداد / الصيانة</th>
            </tr>
          </thead>
          <tbody class="divide-y divide-slate-800">
            ${appState.vehicles.map(v => {
              const diff = v.currentKm - v.lastOilChangeKm;
              const alert = diff >= threshold;
              return `
                <tr class="hover:bg-slate-800/40">
                  <td class="py-3 px-2 font-bold text-white">${v.name}</td>
                  <td class="py-3 px-2 font-mono text-amber-400">${v.plate}</td>
                  <td class="py-3 px-2 text-slate-300">${v.driver}</td>
                  <td class="py-3 px-2 font-black text-sky-400">${v.currentKm.toLocaleString()} كم</td>
                  <td class="py-3 px-2 text-slate-400">${v.lastOilChangeKm.toLocaleString()} كم</td>
                  <td class="py-3 px-2">
                    <span class="px-2 py-0.5 rounded font-bold ${alert ? 'bg-rose-500/20 text-rose-400 animate-pulse' : 'bg-emerald-500/20 text-emerald-400'}">
                      ${diff} كم ${alert ? '(تغيير زيت!)' : ''}
                    </span>
                  </td>
                  <td class="py-3 px-2">
                    <div class="flex items-center gap-1.5">
                      <input type="number" id="kmInput_${v.id}" value="${v.currentKm}" class="bh-input py-1 w-24 text-xs" />
                      <button onclick="updateVehicleKm('${v.id}')" class="bh-btn px-2.5 py-1 bg-sky-600 text-white text-[11px]">تحديث</button>
                      <button onclick="resetVehicleOil('${v.id}')" class="bh-btn px-2.5 py-1 bg-emerald-600 text-white text-[11px]" title="تصفير عداد الزيت بعد الصيانة">تم تغيير الزيت</button>
                    </div>
                  </td>
                </tr>
              `;
            }).join("")}
          </tbody>
        </table>
      </div>
    </div>
  `;
}

function updateVehicleKm(vId) {
  const v = appState.vehicles.find(x => x.id === vId);
  const newKm = Number(document.getElementById(`kmInput_${vId}`).value || 0);
  if (newKm < v.currentKm) { showToast("لا يمكن إدخال قراءة عداد أقل من القراءة السابقة!", "error"); return; }
  v.currentKm = newKm;
  saveState();
  checkGlobalAlerts();
  showToast("تم تحديث عداد الكيلومترات!", "success");
  renderCurrentView();
}

function resetVehicleOil(vId) {
  const v = appState.vehicles.find(x => x.id === vId);
  v.lastOilChangeKm = v.currentKm;
  logAudit("صيانة أسطول", `تسجيل تغيير زيت للمركبة ${v.name} عند عداد ${v.currentKm} كم`);
  saveState();
  checkGlobalAlerts();
  showToast("تم تسجيل الصيانة وتصفير عداد الزيت بنجاح!", "success");
  renderCurrentView();
}

// ============================================================================
// VIEW 10: ATTENDANCE & HR
// ============================================================================
function renderHR(container) {
  const today = new Date().toISOString().split('T')[0];

  container.innerHTML = `
    <div class="space-y-6">
      <!-- Quick Check-in / Check-out -->
      <div class="bh-panel p-5 flex flex-wrap items-center justify-between gap-4">
        <div>
          <h3 class="font-extrabold text-base text-white">تسجيل الحضور والانصراف الفوري</h3>
          <p class="text-xs text-slate-400">تاريخ اليوم: ${today} • الموظف الحالي: <strong class="text-amber-400">${currentUser.name}</strong></p>
        </div>
        <div class="flex gap-3">
          <button onclick="recordAttendance('in')" class="bh-btn px-5 py-2.5 bg-emerald-500 hover:bg-emerald-400 text-slate-950 text-xs">
            <i class="fa-solid fa-fingerprint"></i> تسجيل حضور الآن
          </button>
          <button onclick="recordAttendance('out')" class="bh-btn px-5 py-2.5 bg-amber-500 hover:bg-amber-400 text-slate-950 text-xs">
            <i class="fa-solid fa-door-open"></i> تسجيل انصراف
          </button>
        </div>
      </div>

      <!-- Attendance Log -->
      <div class="bh-panel p-5">
        <div class="flex items-center justify-between mb-3">
          <h3 class="font-extrabold text-sm text-amber-400">سجل الحضور والانصراف</h3>
          <button onclick="exportTableToExcel('attendanceTable', 'سجل_الحضور_والانصراف')" class="bh-btn px-3 py-1 bg-emerald-600/20 text-emerald-400 border border-emerald-500/30 text-xs">
            <i class="fa-solid fa-file-excel"></i> Excel
          </button>
        </div>
        <div class="overflow-x-auto">
          <table id="attendanceTable" class="w-full text-right text-xs">
            <thead>
              <tr class="border-b border-slate-700 text-slate-400">
                <th class="py-2.5 px-2">التاريخ</th>
                <th class="py-2.5 px-2">الاسم</th>
                <th class="py-2.5 px-2">وقت الحضور</th>
                <th class="py-2.5 px-2">وقت الانصراف</th>
                <th class="py-2.5 px-2">الموقع</th>
              </tr>
            </thead>
            <tbody class="divide-y divide-slate-800">
              ${appState.attendance.map(a => `
                <tr>
                  <td class="py-2.5 px-2 text-slate-300">${a.date}</td>
                  <td class="py-2.5 px-2 font-bold text-white">${a.empName}</td>
                  <td class="py-2.5 px-2 text-emerald-400 font-bold">${a.checkIn}</td>
                  <td class="py-2.5 px-2 text-amber-400 font-bold">${a.checkOut}</td>
                  <td class="py-2.5 px-2 text-slate-400">${a.location}</td>
                </tr>
              `).join("")}
            </tbody>
          </table>
        </div>
      </div>

      <!-- HR Employees & Salaries -->
      <div class="bh-panel p-5">
        <div class="flex items-center justify-between mb-3">
          <h3 class="font-extrabold text-sm text-sky-400">بيانات العاملين والرواتب والعُهد النقدية (HR)</h3>
          <button onclick="exportTableToExcel('hrEmployeesTable', 'بيانات_الموظفين_والرواتب')" class="bh-btn px-3 py-1 bg-emerald-600/20 text-emerald-400 border border-emerald-500/30 text-xs">
            <i class="fa-solid fa-file-excel"></i> Excel
          </button>
        </div>
        <div class="overflow-x-auto">
          <table id="hrEmployeesTable" class="w-full text-right text-xs">
            <thead>
              <tr class="border-b border-slate-700 text-slate-400">
                <th class="py-2.5 px-2">الكود</th>
                <th class="py-2.5 px-2">الاسم</th>
                <th class="py-2.5 px-2">الوظيفة</th>
                <th class="py-2.5 px-2">الهاتف</th>
                <th class="py-2.5 px-2">الراتب الأساسي</th>
                <th class="py-2.5 px-2">العهدة النقدية</th>
              </tr>
            </thead>
            <tbody class="divide-y divide-slate-800">
              ${appState.hrEmployees.map(e => `
                <tr>
                  <td class="py-2.5 px-2 font-mono text-amber-400">${e.code}</td>
                  <td class="py-2.5 px-2 font-bold text-white">${e.name}</td>
                  <td class="py-2.5 px-2 text-slate-300">${e.role}</td>
                  <td class="py-2.5 px-2 text-slate-400">${e.phone}</td>
                  <td class="py-2.5 px-2 text-emerald-400 font-bold">${fmtMoney(e.salary)}</td>
                  <td class="py-2.5 px-2 text-sky-400 font-bold">${fmtMoney(e.custodyCash)}</td>
                </tr>
              `).join("")}
            </tbody>
          </table>
        </div>
      </div>
    </div>
  `;
}

function recordAttendance(type) {
  const today = new Date().toISOString().split('T')[0];
  const timeStr = new Date().toLocaleTimeString("ar-EG", { hour: "2-digit", minute: "2-digit" });
  const existing = appState.attendance.find(a => a.date === today && a.empName === currentUser.name);

  if (type === "in") {
    if (existing) { showToast("تم تسجيل حضورك اليوم مسبقاً!", "warning"); return; }
    appState.attendance.unshift({
      id: "att_" + Date.now(),
      date: today,
      empName: currentUser.name,
      code: currentUser.code || "ADM",
      checkIn: timeStr,
      checkOut: "--",
      location: "تم التسجيل عبر التطبيق"
    });
    showToast(`تم تسجيل حضورك الساعة ${timeStr}`, "success");
  } else {
    if (!existing) { showToast("سجّل الحضور أولاً!", "error"); return; }
    existing.checkOut = timeStr;
    showToast(`تم تسجيل انصرافك الساعة ${timeStr}`, "success");
  }
  saveState();
  renderCurrentView();
}

// ============================================================================
// VIEW 11: TIME-FILTERED REPORTS & PERIOD RESET
// ============================================================================
function renderReports(container) {
  container.innerHTML = `
    <div class="bh-panel p-5 space-y-5">
      <div class="flex flex-wrap items-center justify-between gap-3">
        <div>
          <h3 class="font-extrabold text-base text-white">${L("reports")}</h3>
          <p class="text-xs text-slate-400">فلترة المبيعات (اليوم، آخر شهر، آخر 3 أشهر) وتصدير Excel وأرشفة السجلات</p>
        </div>
        <div class="flex flex-wrap gap-2">
          <button onclick="filterReportRange('today')" class="bh-btn px-3 py-1.5 bg-slate-800 hover:bg-amber-500 hover:text-slate-950 text-xs">مبيعات اليوم</button>
          <button onclick="filterReportRange('month')" class="bh-btn px-3 py-1.5 bg-slate-800 hover:bg-amber-500 hover:text-slate-950 text-xs">آخر 30 يوم</button>
          <button onclick="filterReportRange('quarter')" class="bh-btn px-3 py-1.5 bg-slate-800 hover:bg-amber-500 hover:text-slate-950 text-xs">آخر 3 أشهر</button>
          <button onclick="filterReportRange('all')" class="bh-btn px-3 py-1.5 bg-amber-500 text-slate-950 text-xs">الكل</button>
          <button onclick="exportTableToExcel('timeReportTable', 'تقرير_المبيعات_الزمني')" class="bh-btn px-3 py-1.5 bg-emerald-600 text-white text-xs">
            <i class="fa-solid fa-file-excel"></i> تصدير Excel
          </button>
        </div>
      </div>

      <div id="reportSummaryMetrics" class="grid grid-cols-1 sm:grid-cols-3 gap-4"></div>

      <div class="overflow-x-auto">
        <table id="timeReportTable" class="w-full text-right text-xs">
          <thead>
            <tr class="border-b border-slate-700 text-slate-400">
              <th class="py-2.5 px-2">رقم الفاتورة</th>
              <th class="py-2.5 px-2">التاريخ</th>
              <th class="py-2.5 px-2">العميل</th>
              <th class="py-2.5 px-2">المندوب</th>
              <th class="py-2.5 px-2">الإجمالي</th>
              <th class="py-2.5 px-2">المحصّل</th>
            </tr>
          </thead>
          <tbody id="timeReportBody" class="divide-y divide-slate-800"></tbody>
        </table>
      </div>
    </div>
  `;
  filterReportRange("all");
}

function filterReportRange(range) {
  const now = Date.now();
  const cutoffs = {
    today: now - 86400000,
    month: now - 30 * 86400000,
    quarter: now - 90 * 86400000,
    all: 0
  };
  const minTime = cutoffs[range] || 0;
  const filtered = appState.invoices.filter(i => i.status === "active" && new Date(i.date).getTime() >= minTime);

  const total = filtered.reduce((s, i) => s + Number(i.totalAmount), 0);
  const paid = filtered.reduce((s, i) => s + Number(i.paidAmount), 0);

  document.getElementById("reportSummaryMetrics").innerHTML = `
    <div class="p-3.5 rounded-xl bg-slate-900 border border-slate-800">
      <span class="text-xs text-slate-400">عدد الفواتير بالفترة:</span>
      <p class="text-lg font-black text-white mt-0.5">${filtered.length} فاتورة</p>
    </div>
    <div class="p-3.5 rounded-xl bg-slate-900 border border-slate-800">
      <span class="text-xs text-slate-400">إجمالي مبيعات الفترة:</span>
      <p class="text-lg font-black text-amber-400 mt-0.5">${fmtMoney(total)}</p>
    </div>
    <div class="p-3.5 rounded-xl bg-slate-900 border border-slate-800">
      <span class="text-xs text-slate-400">إجمالي المحصّل بالفترة:</span>
      <p class="text-lg font-black text-emerald-400 mt-0.5">${fmtMoney(paid)}</p>
    </div>
  `;

  document.getElementById("timeReportBody").innerHTML = filtered.map(i => `
    <tr>
      <td class="py-2.5 px-2 font-bold text-amber-400">${i.id}</td>
      <td class="py-2.5 px-2 text-slate-300">${fmtDate(i.date)}</td>
      <td class="py-2.5 px-2 font-bold text-white">${i.clientName}</td>
      <td class="py-2.5 px-2 text-slate-300">${i.repName}</td>
      <td class="py-2.5 px-2 font-bold text-sky-400">${fmtMoney(i.totalAmount)}</td>
      <td class="py-2.5 px-2 font-bold text-emerald-400">${fmtMoney(i.paidAmount)}</td>
    </tr>
  `).join("");
}

// ============================================================================
// VIEW 12: SUPER ADMIN CONTROL CENTER (ON/OFF SWITCHES, LABELS, EXTERNAL EMAILS, AUDIT LOG)
// ============================================================================
function renderSuperAdmin(container) {
  const featureList = [
    { key: "pos", name: "نقطة البيع وإصدار الفواتير السريعة" },
    { key: "barcode", name: "قارئ الباركود التلقائي" },
    { key: "quotaLock", name: "قفل الكوتة الشهرية التلقائي للعملاء" },
    { key: "rawbtPrint", name: "الطباعة الحرارية بلوتوث (RawBT)" },
    { key: "inventory", name: "إدارة المخازن وعُهد المناديب" },
    { key: "debts", name: "محرك الذمم التراكمية والترحيل" },
    { key: "whatsappStatement", name: "إرسال كشوف الحساب عبر واتساب" },
    { key: "ratios", name: "حاسبة النسب المرنة والمجمعة" },
    { key: "supplier", name: "تسوية حسابات الموردين (الشركة الشرقية)" },
    { key: "clientsGps", name: "تسجيل العملاء بالـ GPS وخطوط السير" },
    { key: "inactiveRadar", name: "رادار تنبيه العملاء الخاملين" },
    { key: "fleet", name: "إدارة الأسطول وصيانة تغيير الزيت" },
    { key: "hrAttendance", name: "سجل الحضور والانصراف والموارد البشرية" },
    { key: "reports", name: "التقارير الزمنية والأرشفة" },
    { key: "excelExport", name: "أزرار التصدير إلى Excel" }
  ];

  container.innerHTML = `
    <div class="space-y-6">
      <!-- 1. Feature Toggles (ON/OFF Switches) -->
      <div class="bh-panel p-5">
        <div class="flex items-center justify-between mb-4">
          <div>
            <h3 class="font-extrabold text-base text-amber-400 flex items-center gap-2">
              <i class="fa-solid fa-toggle-on"></i> مفاتيح تشغيل وإيقاف أجزاء النظام (ON / OFF)
            </h3>
            <p class="text-xs text-slate-400">أي ميزة يتم إيقافها هنا تختفي فوراً من عند المناديب والمستخدمين لحظياً</p>
          </div>
          <button onclick="openFirebaseModal()" class="bh-btn px-3 py-2 bg-sky-600/20 text-sky-400 border border-sky-500/40 text-xs">
            <i class="fa-solid fa-fire"></i> إعدادات ربط Firebase
          </button>
        </div>

        <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-3">
          ${featureList.map(f => `
            <div class="p-3 rounded-xl bg-slate-900/90 border border-slate-800 flex items-center justify-between">
              <span class="text-xs font-bold text-slate-200">${f.name}</span>
              <label class="switch">
                <input type="checkbox" ${appState.features[f.key] !== false ? "checked" : ""} onchange="toggleFeature('${f.key}', this.checked)">
                <span class="slider"></span>
              </label>
            </div>
          `).join("")}
        </div>
      </div>

      <!-- 2. Dynamic Labels Editor -->
      <div class="bh-panel p-5">
        <div class="flex items-center justify-between mb-4">
          <div>
            <h3 class="font-extrabold text-base text-emerald-400 flex items-center gap-2">
              <i class="fa-solid fa-pen-nib"></i> محرر المسميات والليبلز الديناميكي (Dynamic Labels)
            </h3>
            <p class="text-xs text-slate-400">غيّر أي مسمى في النظام ليظهر فوراً في الشاشات وتقارير الإكسيل</p>
          </div>
          <button onclick="saveDynamicLabels()" class="bh-btn px-4 py-2 bg-emerald-500 hover:bg-emerald-400 text-slate-950 text-xs">
            <i class="fa-solid fa-check"></i> حفظ وتطبيق المسميات
          </button>
        </div>

        <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-3 text-xs">
          ${Object.keys(DEFAULT_LABELS).map(k => `
            <div class="bg-slate-900 p-2.5 rounded-xl border border-slate-800">
              <label class="block text-[10px] text-slate-400 mb-1 font-mono">${k}</label>
              <input type="text" id="lblInput_${k}" value="${L(k)}" class="bh-input py-1.5 text-xs font-bold" />
            </div>
          `).join("")}
        </div>
      </div>

      <!-- 3. Dynamic Business Rules -->
      <div class="bh-panel p-5">
        <h3 class="font-extrabold text-base text-sky-400 mb-4 flex items-center gap-2">
          <i class="fa-solid fa-gear"></i> قواعد العمل والرادار وترويسة الفواتير
        </h3>
        <div class="grid grid-cols-1 sm:grid-cols-2 gap-4 text-xs">
          <div>
            <label class="block text-slate-300 font-bold mb-1">تنبيه خمول العميل بعد (عدد الأيام):</label>
            <input type="number" id="ruleInactivityDays" value="${appState.rules.inactivityDays}" class="bh-input" />
          </div>
          <div>
            <label class="block text-slate-300 font-bold mb-1">تنبيه تغيير زيت الأسطول كل (كم):</label>
            <input type="number" id="ruleOilKm" value="${appState.rules.oilChangeKm}" class="bh-input" />
          </div>
          <div>
            <label class="block text-slate-300 font-bold mb-1">ترويسة الفاتورة الحرارية (Header):</label>
            <textarea id="ruleReceiptHeader" rows="3" class="bh-input">${appState.rules.receiptHeader}</textarea>
          </div>
          <div>
            <label class="block text-slate-300 font-bold mb-1">تذييل الفاتورة الحرارية (Footer):</label>
            <textarea id="ruleReceiptFooter" rows="3" class="bh-input">${appState.rules.receiptFooter}</textarea>
          </div>
        </div>
        <button onclick="saveDynamicRules()" class="bh-btn mt-4 px-5 py-2.5 bg-sky-500 hover:bg-sky-400 text-slate-950 text-xs">
          حفظ قواعد العمل
        </button>
      </div>

      <!-- 4. Users & External Emails Access Manager -->
      <div class="bh-panel p-5">
        <h3 class="font-extrabold text-base text-white mb-3 flex items-center gap-2">
          <i class="fa-solid fa-user-plus text-amber-400"></i> إدارة المستخدمين وإضافة إيميلات خارجية بصلاحيات مخصصة
        </h3>
        <div class="grid grid-cols-1 sm:grid-cols-5 gap-2.5 text-xs mb-4 bg-slate-900/90 p-3.5 rounded-xl border border-slate-800">
          <input type="text" id="newUsrName" class="bh-input" placeholder="الاسم (مثلاً: محاسب خارجي)" />
          <input type="email" id="newUsrEmail" class="bh-input" placeholder="الإيميل الخارجي" />
          <input type="text" id="newUsrCode" class="bh-input" placeholder="الكود (مثل 106)" />
          <input type="text" id="newUsrPass" class="bh-input" placeholder="كلمة المرور" value="123456" />
          <select id="newUsrRole" class="bh-input">
            <option value="external">إيميل خارجي (مراجعة وتقارير)</option>
            <option value="rep">مندوب مبيعات جديد</option>
            <option value="admin">مدير نظام (Admin)</option>
          </select>
          <button onclick="addSystemUser()" class="bh-btn sm:col-span-5 py-2.5 bg-amber-500 hover:bg-amber-400 text-slate-950 font-black">
            + إضافة المستخدم / الإيميل الخارجي للنظام
          </button>
        </div>

        <div class="overflow-x-auto">
          <table class="w-full text-right text-xs">
            <thead>
              <tr class="border-b border-slate-700 text-slate-400">
                <th class="py-2.5 px-2">الاسم</th>
                <th class="py-2.5 px-2">الإيميل / الكود</th>
                <th class="py-2.5 px-2">الدور</th>
                <th class="py-2.5 px-2">كلمة المرور</th>
                <th class="py-2.5 px-2">الحالة</th>
                <th class="py-2.5 px-2">إجراءات</th>
              </tr>
            </thead>
            <tbody class="divide-y divide-slate-800">
              ${appState.users.map(u => `
                <tr>
                  <td class="py-2.5 px-2 font-bold text-white">${u.name}</td>
                  <td class="py-2.5 px-2 text-slate-300 font-mono">${u.email} (${u.code})</td>
                  <td class="py-2.5 px-2 text-amber-400 font-bold">${u.role}</td>
                  <td class="py-2.5 px-2 font-mono text-slate-400">${u.password}</td>
                  <td class="py-2.5 px-2">
                    <span class="px-2 py-0.5 rounded ${u.active ? 'bg-emerald-500/20 text-emerald-400' : 'bg-rose-500/20 text-rose-400'} font-bold">
                      ${u.active ? 'نشط' : 'موقوف'}
                    </span>
                  </td>
                  <td class="py-2.5 px-2 flex gap-1.5">
                    ${u.id !== 'u_admin' ? `
                      <button onclick="toggleUserActive('${u.id}')" class="bh-btn px-2 py-1 bg-slate-800 text-amber-400 text-[11px]">
                        ${u.active ? 'إيقاف' : 'تفعيل'}
                      </button>
                      <button onclick="deleteSystemUser('${u.id}')" class="bh-btn px-2 py-1 bg-rose-500/20 text-rose-400 text-[11px]">حذف</button>
                    ` : '<span class="text-slate-500">الحساب الرئيسي</span>'}
                  </td>
                </tr>
              `).join("")}
            </tbody>
          </table>
        </div>
      </div>

      <!-- 5. Immutable Admin Audit Log -->
      <div class="bh-panel p-5">
        <div class="flex items-center justify-between mb-3">
          <h3 class="font-extrabold text-base text-rose-400 flex items-center gap-2">
            <i class="fa-solid fa-shield-halved"></i> سجل الرقابة والتدقيق (Audit Log)
          </h3>
          <button onclick="exportTableToExcel('auditLogTable', 'سجل_الرقابة_والتدقيق')" class="bh-btn px-3 py-1 bg-emerald-600/20 text-emerald-400 border border-emerald-500/30 text-xs">
            <i class="fa-solid fa-file-excel"></i> تصدير Excel
          </button>
        </div>
        <div class="overflow-x-auto max-h-64">
          <table id="auditLogTable" class="w-full text-right text-xs">
            <thead>
              <tr class="border-b border-slate-700 text-slate-400">
                <th class="py-2 px-2">الوقت</th>
                <th class="py-2 px-2">المستخدم</th>
                <th class="py-2 px-2">الإجراء</th>
                <th class="py-2 px-2">التفاصيل</th>
              </tr>
            </thead>
            <tbody class="divide-y divide-slate-800">
              ${(appState.auditLogs || []).map(l => `
                <tr>
                  <td class="py-2 px-2 text-slate-400">${fmtDate(l.timestamp)}</td>
                  <td class="py-2 px-2 font-bold text-amber-400">${l.user}</td>
                  <td class="py-2 px-2 font-bold text-white">${l.action}</td>
                  <td class="py-2 px-2 text-slate-300">${l.details}</td>
                </tr>
              `).join("")}
            </tbody>
          </table>
        </div>
      </div>
    </div>
  `;
}

function toggleFeature(key, enabled) {
  appState.features[key] = enabled;
  logAudit("تعديل خصائص النظام", `${enabled ? 'تشغيل' : 'إيقاف'} ميزة (${key})`);
  saveState();
  buildSidebar();
  checkGlobalAlerts();
  showToast(`تم ${enabled ? 'تفعيل' : 'إيقاف'} الميزة بنجاح!`, enabled ? "success" : "warning");
}

function saveDynamicLabels() {
  Object.keys(DEFAULT_LABELS).forEach(k => {
    const val = document.getElementById(`lblInput_${k}`)?.value.trim();
    if (val) appState.labels[k] = val;
  });
  logAudit("تعديل المسميات", "تحديث قاموس المسميات والليبلز في النظام");
  saveState();
  applyDynamicLabels();
  buildSidebar();
  showToast("تم تطبيق المسميات الجديدة على النظام بالكامل!", "success");
}

function saveDynamicRules() {
  appState.rules.inactivityDays = Number(document.getElementById("ruleInactivityDays").value || 10);
  appState.rules.oilChangeKm = Number(document.getElementById("ruleOilKm").value || 1500);
  appState.rules.receiptHeader = document.getElementById("ruleReceiptHeader").value;
  appState.rules.receiptFooter = document.getElementById("ruleReceiptFooter").value;
  saveState();
  checkGlobalAlerts();
  showToast("تم حفظ قواعد العمل بنجاح!", "success");
}

function addSystemUser() {
  const name = document.getElementById("newUsrName").value.trim();
  const email = document.getElementById("newUsrEmail").value.trim();
  const code = document.getElementById("newUsrCode").value.trim() || ("U" + Date.now().toString().slice(-3));
  const password = document.getElementById("newUsrPass").value.trim() || "123456";
  const role = document.getElementById("newUsrRole").value;

  if (!name || !email) { showToast("أدخل الاسم والإيميل!", "error"); return; }

  const newUser = {
    id: "u_" + Date.now(),
    name,
    email,
    code,
    password,
    role,
    active: true,
    permissions: role === "admin"
      ? { all: true }
      : (role === "rep"
        ? { pos: true, invoices: true, inventory: true, debts: true, clients: true, hr: true }
        : { dashboard: true, invoices: true, debts: true, supplier: true, reports: true, excelExport: true })
  };

  if (role === "rep") {
    const whId = "wh_rep_" + code;
    newUser.warehouseId = whId;
    appState.warehouses.push({ id: whId, name: `عهدة المندوب: ${name} (${code})`, type: "rep", repCode: code });
  }

  appState.users.push(newUser);
  logAudit("إضافة مستخدم", `إضافة ${name} (${email}) بصلاحية ${role}`);
  saveState();
  showToast("تمت إضافة المستخدم/الإيميل الخارجي بنجاح!", "success");
  renderCurrentView();
}

function toggleUserActive(id) {
  const u = appState.users.find(x => x.id === id);
  if (u) {
    u.active = !u.active;
    saveState();
    renderCurrentView();
  }
}

function deleteSystemUser(id) {
  appState.users = appState.users.filter(x => x.id !== id);
  saveState();
  renderCurrentView();
}

// ============================================================================
// FIREBASE CONFIGURATION MODAL
// ============================================================================
function openFirebaseModal() {
  const curStr = appState.firebaseConfig ? JSON.stringify(appState.firebaseConfig, null, 2) : "";
  const html = `
    <div class="space-y-4 text-xs">
      <p class="text-slate-300 leading-relaxed">
        الصق كائن إعدادات <strong>Firebase Config (JSON)</strong> الخاص بمشروعك لربط جميع أجهزة المناديب والأدمن لحظياً في السحابة، مع استمرار العمل أوفلاين عند انقطاع الإنترنت:
      </p>
      <textarea id="firebaseJsonInput" rows="8" class="bh-input font-mono text-left dir-ltr" placeholder='{
  "apiKey": "AIzaSy...",
  "authDomain": "black-horse-erp.firebaseapp.com",
  "databaseURL": "https://black-horse-erp-default-rtdb.firebaseio.com",
  "projectId": "black-horse-erp",
  "storageBucket": "black-horse-erp.appspot.com",
  "messagingSenderId": "123456789",
  "appId": "1:123456789:web:abcdef"
}'>${curStr}</textarea>
      <div class="flex gap-3">
        <button onclick="connectFirebaseFromModal()" class="bh-btn flex-1 py-2.5 bg-amber-500 hover:bg-amber-400 text-slate-950 font-black">
          <i class="fa-solid fa-plug"></i> حفظ والاتصال بـ Firebase
        </button>
        <button onclick="resetDemoData()" class="bh-btn px-4 py-2.5 bg-rose-500/20 text-rose-400 border border-rose-500/40">
          إعادة ضبط البيانات التجريبية
        </button>
      </div>
    </div>
  `;
  openModal("إعدادات الربط السحابي (Firebase Realtime DB)", html);
}

function connectFirebaseFromModal() {
  try {
    const raw = document.getElementById("firebaseJsonInput").value.trim();
    const cfg = JSON.parse(raw);
    initFirebase(cfg, true);
    closeModal();
  } catch (e) {
    showToast("صيغة JSON غير صحيحة! تأكد من نسخ الكائن كاملاً بين { }", "error");
  }
}

function resetDemoData() {
  localStorage.removeItem("BH_ERP_STATE_V1");
  location.reload();
}

// ============================================================================
// UNIVERSAL EXCEL EXPORT ENGINE (SheetJS XLSX)
// ============================================================================
function exportTableToExcel(tableId, fileName) {
  if (!isFeatureEnabled("excelExport")) {
    showToast("ميزة تصدير الإكسيل موقوفة حالياً من لوحة الأدمن!", "warning");
    return;
  }
  const table = document.getElementById(tableId);
  if (!table) return;
  const wb = XLSX.utils.table_to_book(table, { sheet: "البيانات" });
  XLSX.writeFile(wb, `${fileName}_${new Date().toISOString().split('T')[0]}.xlsx`);
  showToast(`تم تصدير ملف (${fileName}.xlsx) بنجاح!`, "success");
}

function exportFullSystemBackup() {
  const wb = XLSX.utils.book_new();
  
  const invSheet = XLSX.utils.json_to_sheet(appState.invoices.map(i => ({
    رقم_الفاتورة: i.id,
    التاريخ: fmtDate(i.date),
    العميل: i.clientName,
    المندوب: i.repName,
    الإجمالي: i.totalAmount,
    المدفوع: i.paidAmount,
    الحالة: i.status
  })));
  XLSX.utils.book_append_sheet(wb, invSheet, "الفواتير");

  const clientsSheet = XLSX.utils.json_to_sheet(appState.clients.map(c => {
    const fin = getClientFinancials(c.id);
    return {
      العميل: c.name,
      الهاتف: c.phone,
      خط_السير: c.route,
      الرصيد_المرحل: fin.carriedOver,
      المسحوبات: fin.totalPurchases,
      الإيداعات: fin.totalDeposits,
      الرصيد_الصافي: fin.netBalance
    };
  }));
  XLSX.utils.book_append_sheet(wb, clientsSheet, L("debts").slice(0, 30));

  const prodSheet = XLSX.utils.json_to_sheet(appState.products.map(p => ({
    الباركود: p.barcode,
    الصنف: p.name,
    السعر: p.price,
    المخزن_الرئيسي: p.stock?.wh_main || 0
  })));
  XLSX.utils.book_append_sheet(wb, prodSheet, "الأصناف_والمخزن");

  const supSheet = XLSX.utils.json_to_sheet(appState.supplierSettlements);
  XLSX.utils.book_append_sheet(wb, supSheet, "الشركة_الشرقية");

  XLSX.writeFile(wb, `BlackHorse_ERP_FullBackup_${new Date().toISOString().split('T')[0]}.xlsx`);
  showToast("تم تصدير النسخة الاحتياطية الكاملة للنظام بصيغة Excel!", "success");
}

// Initialize App on Load
window.addEventListener("DOMContentLoaded", () => {
  loadState();
  applyDynamicLabels();
});
</script>
</body>
</html>
