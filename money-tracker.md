<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover, maximum-scale=1">
  <meta name="color-scheme" content="light">
  <meta name="theme-color" content="#EDE6D6">
  <meta name="apple-mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-status-bar-style" content="default">
  <meta name="apple-mobile-web-app-title" content="Sổ Chi Tiêu">
  <meta name="mobile-web-app-capable" content="yes">
  <meta name="description" content="Sổ chi tiêu cá nhân — quản lý thu chi, khoản vay, ngân sách, mục tiêu tiết kiệm. Chạy hoàn toàn trên máy, không gửi dữ liệu lên máy chủ.">
  <title>Sổ Chi Tiêu</title>
  <link rel="manifest" href="manifest.json">
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Be+Vietnam+Pro:wght@400;500;600;700&family=IBM+Plex+Mono:wght@400;500;600;700&family=Noto+Serif:ital,wght@0,600;0,700;1,500&display=swap" rel="stylesheet">
  <script src="https://cdn.tailwindcss.com"></script>
  <script>
    tailwind.config = {
      theme: {
        extend: {
          colors: {
            paper: '#EDE6D6',
            cream: '#F5EFE3',
            ink: '#2B2420',
            muted: '#8B8175',
            indigo: { DEFAULT: '#34456B', dark: '#28354F' },
            amber: { DEFAULT: '#B8863B', light: '#F3E8D2' },
            ledger: { green: '#4B7A5E', red: '#AE4030' }
          },
          fontFamily: {
            sans: ['"Be Vietnam Pro"', 'sans-serif'],
            mono: ['"IBM Plex Mono"', 'monospace'],
            serif: ['"Noto Serif"', 'serif']
          }
        }
      }
    };
  </script>
  <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.1/dist/chart.umd.min.js" onerror="window.__chartLoadFailed=true"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js" onerror="window.__xlsxLoadFailed=true"></script>
  <script src="https://cdn.jsdelivr.net/npm/appwrite@26.2.0" onerror="window.__appwriteLoadFailed=true"></script>
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.6.0/css/all.min.css">
  <style>
    :root {
      --safe-t: env(safe-area-inset-top, 0px);
      --safe-b: env(safe-area-inset-bottom, 0px);
      --safe-l: env(safe-area-inset-left, 0px);
      --safe-r: env(safe-area-inset-right, 0px);
    }
    * { -webkit-tap-highlight-color: transparent; }
    html, body { overscroll-behavior-y: none; }
    body {
      background:
        radial-gradient(circle at 10% 15%, rgba(52,69,107,0.07) 0%, transparent 40%),
        radial-gradient(circle at 90% 85%, rgba(184,134,59,0.09) 0%, transparent 40%),
        #EDE6D6;
      padding-left: var(--safe-l); padding-right: var(--safe-r);
    }
    .nav-tab { color: #8B8175; border-bottom: 2px solid transparent; transition: color 0.2s, border-color 0.2s; cursor: pointer; }
    .nav-tab.active { color: #34456B; border-bottom-color: #34456B; }
    .tab-content { animation: fadeIn 0.2s ease; }
    @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
    input:focus, select:focus, textarea:focus { outline: none; }
    select { appearance: none; }
    button, [role="button"], a { min-height: 0; }
    button:not(.chip-btn):not(.icon-btn) { min-height: 44px; }
    .icon-btn { min-width: 40px; min-height: 40px; display: inline-flex; align-items: center; justify-content: center; }
    .quick-amount-btn { transition: transform 0.15s; }
    .quick-amount-btn:active { transform: scale(0.95); }

    /* Focus visibility for keyboard + a11y */
    a:focus-visible, button:focus-visible, input:focus-visible, select:focus-visible,
    textarea:focus-visible, [tabindex]:focus-visible {
      outline: 2px solid #34456B; outline-offset: 2px; border-radius: 2px;
    }

    /* Skeleton loading */
    .skel { position: relative; overflow: hidden; background: rgba(43,36,32,0.08); border-radius: 6px; }
    .skel::after {
      content: ''; position: absolute; inset: 0; transform: translateX(-100%);
      background: linear-gradient(90deg, transparent, rgba(255,255,255,0.55), transparent);
      animation: shimmer 1.4s infinite;
    }
    @keyframes shimmer { 100% { transform: translateX(100%); } }

    /* Toast */
    #toastRoot { position: fixed; left: 0; right: 0; bottom: calc(16px + var(--safe-b)); z-index: 200; display: flex; flex-direction: column; align-items: center; gap: 8px; pointer-events: none; padding: 0 16px; }
    .toast {
      pointer-events: auto; max-width: 360px; width: 100%; background: #2B2420; color: #F5EFE3;
      border-radius: 10px; padding: 12px 14px; font-size: 13px; display: flex; align-items: center; gap: 10px;
      box-shadow: 0 8px 24px rgba(0,0,0,0.25); animation: toastIn 0.22s ease;
    }
    .toast.err { background: #6B2A22; }
    .toast.ok { background: #2B4536; }
    @keyframes toastIn { from { opacity: 0; transform: translateY(8px); } to { opacity: 1; transform: translateY(0); } }
    .toast.leaving { animation: toastOut 0.18s ease forwards; }
    @keyframes toastOut { to { opacity: 0; transform: translateY(6px); } }
    .toast-action { color: #F3E8D2; text-decoration: underline; font-weight: 600; white-space: nowrap; }

    /* Modal */
    .modal-overlay {
      position: fixed; inset: 0; background: rgba(43,36,32,0.45); z-index: 150;
      display: flex; align-items: flex-end; justify-content: center; padding: 0;
      animation: overlayIn 0.18s ease;
    }
    @keyframes overlayIn { from { opacity: 0; } to { opacity: 1; } }
    @media (min-width: 640px) { .modal-overlay { align-items: center; padding: 16px; } }
    .modal-card {
      background: #F5EFE3; width: 100%; max-width: 440px; max-height: 88vh; overflow-y: auto;
      border-radius: 18px 18px 0 0; padding: 20px 20px calc(20px + var(--safe-b));
      animation: sheetIn 0.22s cubic-bezier(.2,.9,.3,1);
    }
    @media (min-width: 640px) { .modal-card { border-radius: 16px; padding: 22px; } }
    @keyframes sheetIn { from { transform: translateY(24px); opacity: 0.4; } to { transform: translateY(0); opacity: 1; } }
    .modal-handle { width: 36px; height: 4px; background: rgba(43,36,32,0.15); border-radius: 999px; margin: 0 auto 14px; }
    @media (min-width: 640px) { .modal-handle { display: none; } }

    #printArea, #reportPrintArea { display: none; }
    @media print {
      #lockScreen, #appRoot, #sigModal, #modalRoot, #toastRoot { display: none !important; }
      #printArea.printing, #reportPrintArea.printing { display: block !important; }
    }

    @media (prefers-reduced-motion: reduce) {
      *, *::before, *::after { animation-duration: 0.001ms !important; animation-iteration-count: 1 !important; transition-duration: 0.001ms !important; }
    }

    /* Category swatch */
    .cat-dot { width: 10px; height: 10px; border-radius: 999px; display: inline-block; flex-shrink: 0; }
    .cat-chip { display: inline-flex; align-items: center; gap: 6px; padding: 6px 12px; border-radius: 999px; border: 1px solid rgba(43,36,32,0.15); font-size: 12px; transition: border-color .15s, background .15s; }
    .cat-chip.selected { border-color: #34456B; background: #F3E8D2; }

    /* Progress bars */
    .progress-track { height: 6px; background: rgba(43,36,32,0.1); border-radius: 999px; overflow: hidden; }
    .progress-fill { height: 100%; border-radius: 999px; transition: width .3s ease; }

    ::selection { background: #B8863B; color: #F5EFE3; }

    input[type="date"]::-webkit-calendar-picker-indicator { opacity: 0.6; }
    /* type="tel" cho bàn phím số đáng tin cậy hơn type="password"+inputmode; che số bằng CSS thay vì mask của trình duyệt */
    .pin-mask { -webkit-text-security: disc; text-security: disc; }

    .swipe-row { position: relative; touch-action: pan-y; }

    @media (orientation: landscape) and (max-height: 480px) {
      #appRoot .max-w-md { max-width: 100%; }
      .landscape-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 0; }
    }
  </style>
</head>
<body class="paper-bg min-h-screen pb-16 font-sans text-ink">

  <!-- Lock screen -->
  <div id="lockScreen" class="hidden fixed inset-0 bg-cream z-[100] flex flex-col items-center justify-center px-8 text-center" style="padding-top:var(--safe-t);padding-bottom:var(--safe-b)">
    <p class="text-[10px] tracking-[0.25em] text-indigo font-semibold uppercase">Sổ Chi Tiêu</p>
    <h1 class="font-serif text-2xl text-ink mt-1 mb-2" id="lockTitle">Đang kiểm tra...</h1>
    <p id="lockSubtitle" class="text-xs text-muted mb-6 max-w-[260px] leading-relaxed"></p>
    <input id="pinInput" type="tel" inputmode="numeric" pattern="[0-9]*" autocomplete="off" maxlength="8" placeholder="••••" aria-label="Mã PIN" class="pin-mask w-40 text-center text-3xl font-mono border-b-2 border-ink/20 focus:border-indigo bg-transparent py-2 tracking-[0.5em] text-ink">
    <p id="lockError" class="text-ledger-red text-xs mt-3 h-8" role="alert" aria-live="assertive"></p>
    <button onclick="Security.submitPin()" id="pinSubmitBtn" class="mt-2 bg-indigo text-cream px-10 py-3 rounded font-semibold disabled:opacity-40">Xác nhận</button>
    <button onclick="Security.openRecoverySheet()" id="forgotPinBtn" class="hidden mt-6 text-muted text-xs underline">Quên mã PIN?</button>
    <p id="lockFooterNote" class="text-[11px] text-muted/70 mt-10 max-w-[240px] leading-relaxed">Dữ liệu được mã hoá ngay trên máy bằng mã PIN của bạn.</p>
  </div>

  <!-- Auth screen: đăng nhập / đăng ký / quên mật khẩu — chỉ hiện khi APPWRITE_CONFIG đã được điền -->
  <div id="authScreen" class="hidden fixed inset-0 bg-cream z-[110] flex flex-col items-center justify-center px-8 text-center overflow-y-auto" style="padding-top:var(--safe-t);padding-bottom:var(--safe-b)">
    <p class="text-[10px] tracking-[0.25em] text-indigo font-semibold uppercase mb-1">Sổ Chi Tiêu</p>

    <p id="authLoading" class="hidden text-sm text-muted py-10">Đang kiểm tra đăng nhập...</p>

    <div id="authView_login" class="hidden w-full max-w-[280px]">
      <h1 class="font-serif text-2xl text-ink mb-6">Đăng nhập</h1>
      <input id="authLoginEmail" type="email" autocomplete="username" placeholder="Email" class="w-full border-b-2 border-ink/15 focus:border-indigo bg-transparent py-3 text-ink placeholder:text-muted/60 mb-4">
      <div class="relative mb-2">
        <input id="authLoginPassword" type="password" autocomplete="current-password" placeholder="Mật khẩu" class="w-full border-b-2 border-ink/15 focus:border-indigo bg-transparent py-3 pr-9 text-ink placeholder:text-muted/60">
        <button type="button" class="absolute right-0 bottom-3 icon-btn text-ink/40" aria-label="Hiện/ẩn mật khẩu" onclick="Session.togglePasswordVisibility('authLoginPassword', this)"><i class="fas fa-eye text-xs" aria-hidden="true"></i></button>
      </div>
      <button onclick="Session.showAuthView('forgot')" class="text-[11px] text-muted underline mb-5 block ml-auto">Quên mật khẩu?</button>
      <p id="authError_login" class="text-ledger-red text-xs mb-3 min-h-[1.5em]" role="alert" aria-live="assertive"></p>
      <button id="authLoginBtn" onclick="Session.submitLogin()" class="w-full bg-indigo text-cream py-3.5 rounded font-semibold disabled:opacity-50">Đăng nhập</button>
      <button onclick="Session.showAuthView('register')" class="w-full border border-ink/20 text-ink py-3.5 rounded font-semibold mt-2.5">Đăng ký</button>
    </div>

    <div id="authView_register" class="hidden w-full max-w-[280px]">
      <h1 class="font-serif text-2xl text-ink mb-6">Tạo tài khoản</h1>
      <input id="authRegisterName" maxlength="80" placeholder="Tên hiển thị" class="w-full border-b-2 border-ink/15 focus:border-indigo bg-transparent py-3 text-ink placeholder:text-muted/60 mb-4">
      <input id="authRegisterEmail" type="email" autocomplete="username" placeholder="Email" class="w-full border-b-2 border-ink/15 focus:border-indigo bg-transparent py-3 text-ink placeholder:text-muted/60 mb-4">
      <div class="relative mb-4">
        <input id="authRegisterPassword" type="password" autocomplete="new-password" placeholder="Mật khẩu (ít nhất 8 ký tự)" class="w-full border-b-2 border-ink/15 focus:border-indigo bg-transparent py-3 pr-9 text-ink placeholder:text-muted/60">
        <button type="button" class="absolute right-0 bottom-3 icon-btn text-ink/40" aria-label="Hiện/ẩn mật khẩu" onclick="Session.togglePasswordVisibility('authRegisterPassword', this)"><i class="fas fa-eye text-xs" aria-hidden="true"></i></button>
      </div>
      <input id="authRegisterPassword2" type="password" autocomplete="new-password" placeholder="Nhập lại mật khẩu" class="w-full border-b-2 border-ink/15 focus:border-indigo bg-transparent py-3 text-ink placeholder:text-muted/60 mb-4">
      <label class="flex items-start gap-2 text-[11px] text-muted text-left mb-5">
        <input id="authRegisterAgree" type="checkbox" class="mt-0.5">
        <span>Tôi đồng ý dữ liệu giao dịch của tôi sẽ được lưu trên tài khoản này để đồng bộ nhiều thiết bị.</span>
      </label>
      <p id="authError_register" class="text-ledger-red text-xs mb-3 min-h-[1.5em]" role="alert" aria-live="assertive"></p>
      <button id="authRegisterBtn" onclick="Session.submitRegister()" class="w-full bg-indigo text-cream py-3.5 rounded font-semibold disabled:opacity-50">Đăng ký</button>
      <button onclick="Session.showAuthView('login')" class="w-full border border-ink/20 text-ink py-3.5 rounded font-semibold mt-2.5">Đã có tài khoản — Đăng nhập</button>
    </div>

    <div id="authView_forgot" class="hidden w-full max-w-[280px]">
      <h1 class="font-serif text-2xl text-ink mb-2">Quên mật khẩu</h1>
      <p class="text-xs text-muted mb-5 leading-relaxed">Nhập email — bạn sẽ nhận link đặt lại mật khẩu.</p>
      <input id="authForgotEmail" type="email" autocomplete="username" placeholder="Email" class="w-full border-b-2 border-ink/15 focus:border-indigo bg-transparent py-3 text-ink placeholder:text-muted/60 mb-4">
      <p id="authError_forgot" class="text-ledger-red text-xs mb-3 min-h-[1.5em]" role="alert" aria-live="assertive"></p>
      <button id="authForgotBtn" onclick="Session.submitForgotPassword()" class="w-full bg-indigo text-cream py-3.5 rounded font-semibold disabled:opacity-50">Gửi email đặt lại</button>
      <button onclick="Session.showAuthView('login')" class="w-full border border-ink/20 text-ink py-3.5 rounded font-semibold mt-2.5">Quay lại đăng nhập</button>
    </div>

    <div id="authView_reset" class="hidden w-full max-w-[280px]">
      <h1 class="font-serif text-2xl text-ink mb-2">Đặt mật khẩu mới</h1>
      <p class="text-xs text-muted mb-5 leading-relaxed">Nhập mật khẩu mới cho tài khoản của bạn.</p>
      <input id="authResetPassword" type="password" autocomplete="new-password" placeholder="Mật khẩu mới (ít nhất 8 ký tự)" class="w-full border-b-2 border-ink/15 focus:border-indigo bg-transparent py-3 text-ink placeholder:text-muted/60 mb-4">
      <input id="authResetPassword2" type="password" autocomplete="new-password" placeholder="Nhập lại mật khẩu mới" class="w-full border-b-2 border-ink/15 focus:border-indigo bg-transparent py-3 text-ink placeholder:text-muted/60 mb-4">
      <p id="authError_reset" class="text-ledger-red text-xs mb-3 min-h-[1.5em]" role="alert" aria-live="assertive"></p>
      <button id="authResetBtn" onclick="Session.submitResetPassword()" class="w-full bg-indigo text-cream py-3.5 rounded font-semibold disabled:opacity-50">Đặt mật khẩu mới</button>
    </div>
  </div>

  <!-- Main app -->
  <div id="appRoot" class="hidden">
    <div class="max-w-md mx-auto bg-cream min-h-screen border-x border-ink/10">

      <!-- Header / letterhead -->
      <div class="bg-cream sticky top-0 z-40 border-b-2 border-ink/10" style="padding-top:var(--safe-t)">
        <div class="px-6 pt-5 pb-3 flex items-start justify-between">
          <div>
            <p class="text-[10px] tracking-[0.25em] text-indigo font-semibold uppercase">Sổ Chi Tiêu</p>
            <h1 class="font-serif text-xl text-ink -mt-0.5" id="appUserName">Tuấn</h1>
          </div>
          <button onclick="Security.lockApp()" class="icon-btn text-ink/30" aria-label="Khoá ứng dụng" title="Khoá ứng dụng">
            <i class="fas fa-lock text-sm"></i>
          </button>
        </div>
        <div class="flex" role="tablist" aria-label="Điều hướng chính">
          <button onclick="UI.showTab(0)" class="nav-tab active flex-1 py-2.5 flex flex-col items-center gap-1" role="tab" aria-selected="true" aria-controls="tab-0">
            <i class="fas fa-house text-sm" aria-hidden="true"></i><span class="text-[10px] font-medium">Tổng quan</span>
          </button>
          <button onclick="UI.showTab(1)" class="nav-tab flex-1 py-2.5 flex flex-col items-center gap-1" role="tab" aria-selected="false" aria-controls="tab-1">
            <i class="fas fa-plus text-sm" aria-hidden="true"></i><span class="text-[10px] font-medium">Thêm mới</span>
          </button>
          <button onclick="UI.showTab(2)" class="nav-tab flex-1 py-2.5 flex flex-col items-center gap-1" role="tab" aria-selected="false" aria-controls="tab-2">
            <i class="fas fa-hand-holding-dollar text-sm" aria-hidden="true"></i><span class="text-[10px] font-medium">Vay mượn</span>
          </button>
          <button onclick="UI.showTab(3)" class="nav-tab flex-1 py-2.5 flex flex-col items-center gap-1" role="tab" aria-selected="false" aria-controls="tab-3">
            <i class="fas fa-chart-bar text-sm" aria-hidden="true"></i><span class="text-[10px] font-medium">Thống kê</span>
          </button>
          <button onclick="UI.showTab(4)" class="nav-tab flex-1 py-2.5 flex flex-col items-center gap-1" role="tab" aria-selected="false" aria-controls="tab-4">
            <i class="fas fa-gear text-sm" aria-hidden="true"></i><span class="text-[10px] font-medium">Cài đặt</span>
          </button>
        </div>
      </div>

      <!-- Dashboard -->
      <div id="tab-0" class="tab-content" role="tabpanel">
        <div id="dashAlerts" class="px-6 pt-4 space-y-2"></div>

        <div class="px-6 py-7 border-b border-dashed border-ink/20">
          <div class="flex items-center justify-between">
            <p class="text-[11px] tracking-[0.2em] text-muted uppercase">Số dư hiện có</p>
            <button onclick="UI.toggleBalanceVisibility()" id="balanceToggleBtn" class="icon-btn text-ink/30" aria-label="Ẩn/hiện số dư"><i class="fas fa-eye text-xs" aria-hidden="true"></i></button>
          </div>
          <p id="balance" class="font-mono text-4xl text-ink font-medium mt-2">Đang tải...</p>
          <div class="mt-3 h-px bg-ink/25 max-w-[140px]"></div>
          <div class="mt-1 h-px bg-ink/25 max-w-[140px]"></div>
          <p id="todayLabel" class="text-xs text-muted mt-3">Cập nhật hôm nay •</p>

          <div class="grid grid-cols-3 gap-2 mt-5">
            <button onclick="UI.showTab(2,'lend')" class="text-left border-l-2 border-amber pl-2.5 py-0.5">
              <p class="text-[9px] tracking-widest text-muted uppercase">Đang cho vay</p>
              <p id="dashLent" class="font-mono text-sm text-ink mt-0.5">0 ₫</p>
            </button>
            <button onclick="UI.showTab(2,'borrow')" class="text-left border-l-2 border-ledger-red pl-2.5 py-0.5">
              <p class="text-[9px] tracking-widest text-muted uppercase">Đang phải trả</p>
              <p id="dashOwed" class="font-mono text-sm text-ink mt-0.5">0 ₫</p>
            </button>
            <div class="text-left border-l-2 border-indigo pl-2.5 py-0.5">
              <p class="text-[9px] tracking-widest text-muted uppercase">Giá trị ròng</p>
              <p id="dashNetWorth" class="font-mono text-sm text-ink mt-0.5">0 ₫</p>
            </div>
          </div>
        </div>

        <div id="budgetSection" class="hidden px-6 py-4 border-b border-dashed border-ink/20">
          <div class="flex justify-between items-baseline mb-2">
            <p class="text-[10px] tracking-widest text-muted uppercase">Ngân sách tháng này</p>
            <button onclick="UI.showTab(7)" class="text-[11px] text-indigo font-medium">Chi tiết →</button>
          </div>
          <div id="budgetBars" class="space-y-3"></div>
        </div>

        <div class="grid grid-cols-2 divide-x divide-dashed divide-ink/20 border-b border-dashed border-ink/20">
          <div class="px-6 py-4">
            <p class="text-[10px] tracking-widest text-muted uppercase">Thu tháng này</p>
            <p id="incomeMonth" class="font-mono text-lg text-ledger-green mt-1">0 ₫</p>
          </div>
          <div class="px-6 py-4">
            <p class="text-[10px] tracking-widest text-muted uppercase">Chi tháng này</p>
            <p id="expenseMonth" class="font-mono text-lg text-ledger-red mt-1">0 ₫</p>
          </div>
        </div>
        <div class="px-6 py-4 border-b border-dashed border-ink/20 flex items-center justify-between">
          <p class="text-[10px] tracking-widest text-muted uppercase">Tiết kiệm tháng này</p>
          <p id="savedMonth" class="font-mono text-sm text-ink"><span id="savedAmount">0 ₫</span> <span id="savedRate" class="text-muted"></span></p>
        </div>

        <div class="px-6 py-6 border-b border-dashed border-ink/20">
          <canvas id="pieChart" height="180" role="img" aria-label="Biểu đồ thu chi tháng này"></canvas>
          <div class="flex justify-center gap-6 mt-3 text-xs text-muted">
            <span class="flex items-center gap-1.5"><i class="w-2 h-2 rounded-full bg-ledger-green inline-block" aria-hidden="true"></i>Thu</span>
            <span class="flex items-center gap-1.5"><i class="w-2 h-2 rounded-full bg-ledger-red inline-block" aria-hidden="true"></i>Chi</span>
          </div>
        </div>

        <div class="px-6 py-6">
          <div class="flex items-center justify-between mb-3 gap-3">
            <p class="text-[11px] tracking-[0.2em] text-muted uppercase shrink-0">Giao dịch gần đây</p>
            <button onclick="UI.showTab(5)" class="text-[11px] text-indigo font-medium shrink-0">Xem tất cả →</button>
          </div>
          <div id="recentList"></div>
        </div>
      </div>
      <!-- Add / Edit Transaction -->
      <div id="tab-1" class="tab-content hidden px-6 py-6" role="tabpanel">
        <p class="text-[11px] tracking-[0.2em] text-indigo uppercase mb-1">Ghi chép</p>
        <h2 id="txFormTitle" class="font-serif text-xl text-ink mb-5">Thêm giao dịch</h2>

        <div class="relative">
          <label for="type" class="sr-only">Loại giao dịch</label>
          <select id="type" onchange="UI.renderCategoryChipsForForm(); Tx.selectCategory(null)" class="w-full border-b-2 border-ink/15 focus:border-indigo bg-transparent py-3 pr-6 text-ink">
            <option value="income">Thu nhập</option>
            <option value="expense">Chi tiêu</option>
          </select>
          <i class="fas fa-chevron-down absolute right-1 top-1/2 -translate-y-1/2 text-muted text-xs pointer-events-none" aria-hidden="true"></i>
        </div>
        <p class="text-[11px] text-muted mt-2">Muốn ghi khoản <b>cho vay</b> hoặc <b>đi vay</b>? Qua tab <button onclick="UI.showTab(2)" class="text-indigo underline">Vay mượn</button> để được theo dõi đầy đủ và tự đồng bộ số dư.</p>

        <div class="relative mt-4">
          <span class="absolute left-0 top-1/2 -translate-y-1/2 text-muted font-mono" aria-hidden="true">₫</span>
          <label for="amount" class="sr-only">Số tiền</label>
          <input id="amount" type="text" inputmode="numeric" pattern="[0-9]*" placeholder="0" class="w-full border-b-2 border-ink/15 focus:border-indigo bg-transparent py-3 pl-6 pr-10 font-mono text-lg text-ink placeholder:text-muted/60">
          <button type="button" onclick="Modal.openCalculator('amount')" class="absolute right-0 top-1/2 -translate-y-1/2 icon-btn text-indigo" aria-label="Mở máy tính" title="Máy tính"><i class="fas fa-calculator text-sm" aria-hidden="true"></i></button>
        </div>
        <div class="flex flex-wrap gap-2 mt-2.5" id="txQuickAmounts">
          <button type="button" onclick="Util.setQuickAmount('amount',10000)" class="quick-amount-btn px-3 py-1.5 rounded-full border border-ink/15 text-xs text-ink/70">10.000</button>
          <button type="button" onclick="Util.setQuickAmount('amount',20000)" class="quick-amount-btn px-3 py-1.5 rounded-full border border-ink/15 text-xs text-ink/70">20.000</button>
          <button type="button" onclick="Util.setQuickAmount('amount',50000)" class="quick-amount-btn px-3 py-1.5 rounded-full border border-ink/15 text-xs text-ink/70">50.000</button>
          <button type="button" onclick="Util.setQuickAmount('amount',100000)" class="quick-amount-btn px-3 py-1.5 rounded-full border border-ink/15 text-xs text-ink/70">100.000</button>
          <button type="button" onclick="Util.setQuickAmount('amount',200000)" class="quick-amount-btn px-3 py-1.5 rounded-full border border-ink/15 text-xs text-ink/70">200.000</button>
          <button type="button" onclick="Util.setQuickAmount('amount',500000)" class="quick-amount-btn px-3 py-1.5 rounded-full border border-ink/15 text-xs text-ink/70">500.000</button>
        </div>

        <p class="text-[11px] tracking-widest text-muted uppercase mt-5 mb-2">Danh mục</p>
        <div id="txCategoryChips" class="flex flex-wrap gap-2"></div>
        <input type="hidden" id="category">

        <label for="note" class="sr-only">Ghi chú</label>
        <input id="note" maxlength="200" placeholder="Ghi chú" class="w-full border-b-2 border-ink/15 focus:border-indigo bg-transparent py-3 mt-4 text-ink placeholder:text-muted/60">

        <div class="grid grid-cols-2 gap-3 mt-4">
          <div>
            <label for="date" class="sr-only">Ngày</label>
            <input id="date" type="date" class="w-full border-b-2 border-ink/15 focus:border-indigo bg-transparent py-3 text-ink">
          </div>
          <div class="relative">
            <label for="walletId" class="sr-only">Ví tiền</label>
            <select id="walletId" class="w-full border-b-2 border-ink/15 focus:border-indigo bg-transparent py-3 pr-6 text-ink text-sm"></select>
            <i class="fas fa-chevron-down absolute right-1 top-1/2 -translate-y-1/2 text-muted text-xs pointer-events-none" aria-hidden="true"></i>
          </div>
        </div>

        <button onclick="Tx.submit()" id="txSubmitBtn" class="mt-8 w-full bg-indigo hover:bg-indigo-dark text-cream py-4 rounded font-semibold tracking-wide transition-opacity disabled:opacity-50">Lưu giao dịch</button>
      </div>

      <!-- Loans (bidirectional: lend / borrow) -->
      <div id="tab-2" class="tab-content hidden px-6 py-6" role="tabpanel">
        <p class="text-[11px] tracking-[0.2em] text-amber uppercase mb-1">Vay mượn</p>
        <h2 class="font-serif text-xl text-ink mb-4" id="loanTabTitle">Ai đang nợ bạn?</h2>

        <div class="flex border border-ink/15 rounded-full p-1 mb-5" role="tablist" aria-label="Chiều khoản vay">
          <button id="loanDirLendBtn" onclick="Loans.setDirection('lend')" class="flex-1 py-2 rounded-full text-xs font-semibold transition-colors" role="tab" aria-selected="true">Tôi cho vay</button>
          <button id="loanDirBorrowBtn" onclick="Loans.setDirection('borrow')" class="flex-1 py-2 rounded-full text-xs font-semibold transition-colors text-muted" role="tab" aria-selected="false">Tôi đi vay</button>
        </div>

        <div class="border-b-2 border-dashed border-ink/20 pb-5 mb-5">
          <label for="loanName" class="sr-only">Tên người liên quan</label>
          <input id="loanName" maxlength="80" placeholder="Tên người vay" class="w-full border-b-2 border-ink/15 focus:border-amber bg-transparent py-3 text-ink placeholder:text-muted/60">
          <div class="relative mt-3">
            <span class="absolute left-0 top-1/2 -translate-y-1/2 text-muted font-mono" aria-hidden="true">₫</span>
            <label for="loanAmount" class="sr-only">Số tiền gốc</label>
            <input id="loanAmount" type="text" inputmode="numeric" pattern="[0-9]*" placeholder="0" class="w-full border-b-2 border-ink/15 focus:border-amber bg-transparent py-3 pl-6 pr-10 font-mono text-ink placeholder:text-muted/60">
            <button type="button" onclick="Modal.openCalculator('loanAmount')" class="absolute right-0 top-1/2 -translate-y-1/2 icon-btn text-amber" aria-label="Mở máy tính" title="Máy tính"><i class="fas fa-calculator text-sm" aria-hidden="true"></i></button>
          </div>
          <div class="flex flex-wrap gap-2 mt-2.5">
            <button type="button" onclick="Util.setQuickAmount('loanAmount',10000)" class="quick-amount-btn px-3 py-1.5 rounded-full border border-ink/15 text-xs text-ink/70">10.000</button>
            <button type="button" onclick="Util.setQuickAmount('loanAmount',20000)" class="quick-amount-btn px-3 py-1.5 rounded-full border border-ink/15 text-xs text-ink/70">20.000</button>
            <button type="button" onclick="Util.setQuickAmount('loanAmount',50000)" class="quick-amount-btn px-3 py-1.5 rounded-full border border-ink/15 text-xs text-ink/70">50.000</button>
            <button type="button" onclick="Util.setQuickAmount('loanAmount',100000)" class="quick-amount-btn px-3 py-1.5 rounded-full border border-ink/15 text-xs text-ink/70">100.000</button>
            <button type="button" onclick="Util.setQuickAmount('loanAmount',200000)" class="quick-amount-btn px-3 py-1.5 rounded-full border border-ink/15 text-xs text-ink/70">200.000</button>
            <button type="button" onclick="Util.setQuickAmount('loanAmount',500000)" class="quick-amount-btn px-3 py-1.5 rounded-full border border-ink/15 text-xs text-ink/70">500.000</button>
          </div>
          <label class="text-[11px] text-muted mt-3 block">Hạn trả (không bắt buộc)</label>
          <input id="loanDueDate" type="date" class="w-full border-b-2 border-ink/15 focus:border-amber bg-transparent py-2 text-ink text-sm">
          <label for="loanNote" class="text-[11px] text-muted mt-3 block">Ghi chú (không bắt buộc)</label>
          <input id="loanNote" maxlength="200" class="w-full border-b-2 border-ink/15 focus:border-amber bg-transparent py-2 text-ink text-sm">
          <button onclick="Loans.submit()" id="loanSubmitBtn" class="mt-4 w-full bg-amber text-cream py-3.5 rounded font-semibold transition-opacity disabled:opacity-50">+ Thêm khoản cho vay</button>
        </div>

        <div class="flex items-center justify-between mb-3">
          <p class="text-[11px] tracking-[0.2em] text-muted uppercase" id="loanListLabel">Danh sách</p>
          <div class="relative">
            <label for="loanFilter" class="sr-only">Lọc trạng thái</label>
            <select id="loanFilter" onchange="Loans.render()" class="text-xs border-b border-ink/15 bg-transparent py-1 pr-5 text-muted">
              <option value="all">Tất cả</option>
              <option value="active">Chưa xong</option>
              <option value="paid">Đã trả</option>
              <option value="overdue">Quá hạn</option>
            </select>
          </div>
        </div>
        <div id="loansList"></div>
      </div>
      <!-- Statistics -->
      <div id="tab-3" class="tab-content hidden px-6 py-6" role="tabpanel">
        <p class="text-[11px] tracking-[0.2em] text-indigo uppercase mb-1">Thống kê</p>
        <h2 class="font-serif text-xl text-ink mb-4">Bức tranh tài chính</h2>

        <div id="statsPeriodChips" class="flex gap-2 overflow-x-auto pb-1 mb-5" style="scrollbar-width:none"></div>
        <div id="statsCustomRange" class="hidden grid grid-cols-2 gap-3 mb-5">
          <div><label class="text-[11px] text-muted block mb-1">Từ ngày</label><input type="date" id="statsFrom" class="w-full border-b border-ink/15 bg-transparent py-2 text-sm"></div>
          <div><label class="text-[11px] text-muted block mb-1">Đến ngày</label><input type="date" id="statsTo" class="w-full border-b border-ink/15 bg-transparent py-2 text-sm"></div>
          <button onclick="Stats.applyCustomRange()" class="col-span-2 bg-indigo text-cream py-2 rounded text-sm font-semibold">Áp dụng</button>
        </div>

        <div class="grid grid-cols-2 divide-x divide-dashed divide-ink/20 border-y border-dashed border-ink/20 mb-2">
          <div class="px-4 py-3">
            <p class="text-[10px] tracking-widest text-muted uppercase">Tổng thu</p>
            <p id="statsIncomeTotal" class="font-mono text-lg text-ledger-green mt-1">0 ₫</p>
          </div>
          <div class="px-4 py-3">
            <p class="text-[10px] tracking-widest text-muted uppercase">Tổng chi</p>
            <p id="statsExpenseTotal" class="font-mono text-lg text-ledger-red mt-1">0 ₫</p>
          </div>
        </div>
        <div class="grid grid-cols-2 divide-x divide-dashed divide-ink/20 border-b border-dashed border-ink/20 mb-6">
          <div class="px-4 py-3">
            <p class="text-[10px] tracking-widest text-muted uppercase">Tiết kiệm</p>
            <p id="statsSaved" class="font-mono text-lg text-ink mt-1">0 ₫</p>
          </div>
          <div class="px-4 py-3">
            <p class="text-[10px] tracking-widest text-muted uppercase">Tỷ lệ tiết kiệm</p>
            <p id="statsSavedRate" class="font-mono text-lg text-ink mt-1">0%</p>
          </div>
        </div>

        <canvas id="barChart" height="220" role="img" aria-label="Biểu đồ thu chi theo thời gian"></canvas>

        <div class="flex items-center justify-between mt-8 mb-1">
          <p class="text-[11px] tracking-[0.2em] text-indigo uppercase">So với kỳ trước</p>
        </div>
        <div id="statsComparison" class="py-2"></div>

        <p class="text-[11px] tracking-[0.2em] text-indigo uppercase mt-8 mb-3">Dòng tiền theo ngày</p>
        <canvas id="flowChart" height="180" role="img" aria-label="Biểu đồ dòng tiền theo ngày"></canvas>

        <p class="text-[11px] tracking-[0.2em] text-amber uppercase mt-8 mb-3">Chi tiêu theo danh mục</p>
        <div id="categoryBreakdown"></div>

        <p class="text-[11px] tracking-[0.2em] text-indigo uppercase mt-8 mb-3">Con số đáng chú ý</p>
        <div id="smartStats" class="space-y-2.5 pb-4"></div>
      </div>

      <!-- Settings -->
      <div id="tab-4" class="tab-content hidden px-6 py-6" role="tabpanel">
        <p class="text-[11px] tracking-[0.2em] text-muted uppercase mb-4">Cài đặt</p>

        <div id="accountSection"></div>

        <p class="text-[10px] tracking-widest text-muted/70 uppercase mt-2 mb-1">Quản lý</p>
        <button onclick="UI.showTab(7)" class="w-full flex items-center justify-between py-4 border-b border-dashed border-ink/20 text-ink">
          <span class="flex items-center gap-3 text-sm font-medium"><i class="fas fa-wallet text-indigo text-sm w-4" aria-hidden="true"></i> Ngân sách</span>
          <i class="fas fa-chevron-right text-ink/20 text-xs" aria-hidden="true"></i>
        </button>
        <button onclick="UI.showTab(6)" class="w-full flex items-center justify-between py-4 border-b border-dashed border-ink/20 text-ink">
          <span class="flex items-center gap-3 text-sm font-medium"><i class="fas fa-tags text-indigo text-sm w-4" aria-hidden="true"></i> Danh mục</span>
          <i class="fas fa-chevron-right text-ink/20 text-xs" aria-hidden="true"></i>
        </button>
        <button onclick="UI.showTab(8)" class="w-full flex items-center justify-between py-4 border-b border-dashed border-ink/20 text-ink">
          <span class="flex items-center gap-3 text-sm font-medium"><i class="fas fa-building-columns text-indigo text-sm w-4" aria-hidden="true"></i> Ví tiền &amp; tài khoản</span>
          <i class="fas fa-chevron-right text-ink/20 text-xs" aria-hidden="true"></i>
        </button>
        <button onclick="UI.showTab(9)" class="w-full flex items-center justify-between py-4 border-b border-dashed border-ink/20 text-ink">
          <span class="flex items-center gap-3 text-sm font-medium"><i class="fas fa-bullseye text-indigo text-sm w-4" aria-hidden="true"></i> Mục tiêu tiết kiệm</span>
          <i class="fas fa-chevron-right text-ink/20 text-xs" aria-hidden="true"></i>
        </button>
        <button onclick="UI.showTab(10)" class="w-full flex items-center justify-between py-4 border-b border-dashed border-ink/20 text-ink">
          <span class="flex items-center gap-3 text-sm font-medium"><i class="fas fa-rotate text-indigo text-sm w-4" aria-hidden="true"></i> Giao dịch định kỳ</span>
          <i class="fas fa-chevron-right text-ink/20 text-xs" aria-hidden="true"></i>
        </button>

        <p class="text-[10px] tracking-widest text-muted/70 uppercase mt-6 mb-1">Báo cáo &amp; sao lưu</p>
        <button onclick="ExportXlsx.run()" class="w-full flex items-center justify-between py-4 border-b border-dashed border-ink/20 text-ink">
          <span class="flex items-center gap-3 text-sm font-medium"><i class="fas fa-file-excel text-indigo text-sm w-4" aria-hidden="true"></i> Xuất báo cáo Excel</span>
          <i class="fas fa-chevron-right text-ink/20 text-xs" aria-hidden="true"></i>
        </button>
        <button onclick="UI.showTab(11)" class="w-full flex items-center justify-between py-4 border-b border-dashed border-ink/20 text-ink">
          <span class="flex items-center gap-3 text-sm font-medium"><i class="fas fa-print text-indigo text-sm w-4" aria-hidden="true"></i> Báo cáo in / PDF</span>
          <i class="fas fa-chevron-right text-ink/20 text-xs" aria-hidden="true"></i>
        </button>
        <button onclick="Backup.exportBackup()" class="w-full flex items-center justify-between py-4 border-b border-dashed border-ink/20 text-ink">
          <span class="flex items-center gap-3 text-sm font-medium"><i class="fas fa-download text-indigo text-sm w-4" aria-hidden="true"></i> Sao lưu dữ liệu</span>
          <i class="fas fa-chevron-right text-ink/20 text-xs" aria-hidden="true"></i>
        </button>
        <button onclick="Backup.triggerImport()" class="w-full flex items-center justify-between py-4 border-b border-dashed border-ink/20 text-ink">
          <span class="flex items-center gap-3 text-sm font-medium"><i class="fas fa-upload text-indigo text-sm w-4" aria-hidden="true"></i> Khôi phục từ file sao lưu</span>
          <i class="fas fa-chevron-right text-ink/20 text-xs" aria-hidden="true"></i>
        </button>
        <input type="file" id="importFileInput" accept=".json,application/json" class="hidden" onchange="Backup.handleImportFile(event)">

        <p class="text-[10px] tracking-widest text-muted/70 uppercase mt-6 mb-1">Bảo mật</p>
        <button onclick="Security.openChangePinSheet()" class="w-full flex items-center justify-between py-4 border-b border-dashed border-ink/20 text-ink">
          <span class="flex items-center gap-3 text-sm font-medium"><i class="fas fa-lock text-indigo text-sm w-4" aria-hidden="true"></i> Đổi mã PIN</span>
          <i class="fas fa-chevron-right text-ink/20 text-xs" aria-hidden="true"></i>
        </button>
        <button onclick="App.clearDataFlow()" class="w-full flex items-center justify-between py-4 text-ledger-red">
          <span class="flex items-center gap-3 text-sm font-medium"><i class="fas fa-trash text-sm w-4" aria-hidden="true"></i> Xóa toàn bộ dữ liệu</span>
          <i class="fas fa-chevron-right text-ledger-red/40 text-xs" aria-hidden="true"></i>
        </button>
        <p class="text-[11px] text-muted mt-6 leading-relaxed">Mã PIN của bạn dùng để mã hoá dữ liệu ngay trên máy (PBKDF2 + AES‑GCM). Ứng dụng chạy hoàn toàn trong trình duyệt, không có máy chủ riêng — nếu quên mã PIN và không có file sao lưu, dữ liệu đã mã hoá sẽ không thể khôi phục.</p>
        <p class="text-[10px] text-muted/60 mt-3" id="appVersionLabel"></p>
        <p class="text-[10px] text-muted/60 mt-1" id="dataStatusLabel"></p>
      </div>

      <!-- Sub-page: Transaction History -->
      <div id="tab-5" class="tab-content hidden" role="tabpanel">
        <div class="px-6 pt-5 pb-3 flex items-center gap-3 sticky top-[0px] bg-cream z-30 border-b border-dashed border-ink/20">
          <button onclick="UI.showTab(0)" class="icon-btn text-ink/50" aria-label="Quay lại"><i class="fas fa-arrow-left" aria-hidden="true"></i></button>
          <h2 class="font-serif text-lg text-ink">Lịch sử giao dịch</h2>
        </div>
        <div class="px-6 py-4">
          <div class="relative mb-3">
            <i class="fas fa-magnifying-glass absolute left-3 top-1/2 -translate-y-1/2 text-muted text-xs" aria-hidden="true"></i>
            <label for="histSearch" class="sr-only">Tìm kiếm giao dịch</label>
            <input id="histSearch" placeholder="Tìm theo ghi chú, danh mục, số tiền..." class="w-full text-sm border border-ink/15 rounded-full bg-transparent py-2 pl-9 pr-3 text-ink placeholder:text-muted/60">
          </div>
          <div class="flex gap-2 overflow-x-auto pb-1 mb-3" id="histTypeFilters" style="scrollbar-width:none"></div>
          <div class="flex items-center gap-2 mb-4">
            <div class="relative flex-1">
              <select id="histSort" onchange="History.render()" class="w-full text-xs border border-ink/15 rounded-full bg-transparent py-2 px-3 text-muted appearance-none">
                <option value="date_desc">Mới nhất</option>
                <option value="date_asc">Cũ nhất</option>
                <option value="amount_desc">Tiền cao nhất</option>
                <option value="amount_asc">Tiền thấp nhất</option>
              </select>
            </div>
            <button onclick="Modal.openHistoryFilters()" class="text-xs border border-ink/15 rounded-full py-2 px-3 text-muted flex items-center gap-1.5"><i class="fas fa-sliders" aria-hidden="true"></i>Lọc thêm</button>
          </div>
          <div id="histActiveFilters" class="flex flex-wrap gap-1.5 mb-3"></div>
          <div id="histList"></div>
          <div id="histLoadMoreWrap" class="hidden py-4 text-center">
            <button onclick="History.loadMore()" class="text-sm text-indigo font-medium">Xem thêm</button>
          </div>
        </div>
      </div>
      <!-- Sub-page: Categories -->
      <div id="tab-6" class="tab-content hidden" role="tabpanel">
        <div class="px-6 pt-5 pb-3 flex items-center justify-between sticky top-0 bg-cream z-30 border-b border-dashed border-ink/20">
          <div class="flex items-center gap-3"><button onclick="UI.showTab(4)" class="icon-btn text-ink/50" aria-label="Quay lại"><i class="fas fa-arrow-left" aria-hidden="true"></i></button><h2 class="font-serif text-lg text-ink">Danh mục</h2></div>
          <button onclick="Modal.openCategoryEditor()" class="icon-btn text-indigo" aria-label="Thêm danh mục"><i class="fas fa-plus" aria-hidden="true"></i></button>
        </div>
        <div class="px-6 py-4">
          <p class="text-[10px] tracking-widest text-muted uppercase mb-2">Chi tiêu</p>
          <div id="catListExpense" class="mb-6 space-y-1"></div>
          <p class="text-[10px] tracking-widest text-muted uppercase mb-2">Thu nhập</p>
          <div id="catListIncome" class="space-y-1"></div>
        </div>
      </div>

      <!-- Sub-page: Budgets -->
      <div id="tab-7" class="tab-content hidden" role="tabpanel">
        <div class="px-6 pt-5 pb-3 flex items-center justify-between sticky top-0 bg-cream z-30 border-b border-dashed border-ink/20">
          <div class="flex items-center gap-3"><button onclick="UI.showTab(4)" class="icon-btn text-ink/50" aria-label="Quay lại"><i class="fas fa-arrow-left" aria-hidden="true"></i></button><h2 class="font-serif text-lg text-ink">Ngân sách</h2></div>
          <button onclick="Modal.openBudgetEditor()" class="icon-btn text-indigo" aria-label="Thêm ngân sách"><i class="fas fa-plus" aria-hidden="true"></i></button>
        </div>
        <div class="px-6 py-4">
          <div id="budgetFullList" class="space-y-5"></div>
        </div>
      </div>

      <!-- Sub-page: Wallets -->
      <div id="tab-8" class="tab-content hidden" role="tabpanel">
        <div class="px-6 pt-5 pb-3 flex items-center justify-between sticky top-0 bg-cream z-30 border-b border-dashed border-ink/20">
          <div class="flex items-center gap-3"><button onclick="UI.showTab(4)" class="icon-btn text-ink/50" aria-label="Quay lại"><i class="fas fa-arrow-left" aria-hidden="true"></i></button><h2 class="font-serif text-lg text-ink">Ví tiền &amp; tài khoản</h2></div>
          <button onclick="Modal.openWalletEditor()" class="icon-btn text-indigo" aria-label="Thêm ví"><i class="fas fa-plus" aria-hidden="true"></i></button>
        </div>
        <div class="px-6 py-4">
          <div id="walletList" class="space-y-3 mb-5"></div>
          <button onclick="Modal.openTransferSheet()" class="w-full border border-ink/15 rounded-lg py-3 text-sm font-medium text-indigo flex items-center justify-center gap-2"><i class="fas fa-right-left" aria-hidden="true"></i> Chuyển tiền giữa các ví</button>
          <p class="text-[11px] text-muted mt-4 leading-relaxed">Chuyển tiền giữa các ví không được tính là khoản thu hay khoản chi.</p>
          <p class="text-[10px] tracking-widest text-muted uppercase mt-6 mb-2">Lịch sử chuyển tiền</p>
          <div id="transferList" class="space-y-1"></div>
        </div>
      </div>

      <!-- Sub-page: Savings goals -->
      <div id="tab-9" class="tab-content hidden" role="tabpanel">
        <div class="px-6 pt-5 pb-3 flex items-center justify-between sticky top-0 bg-cream z-30 border-b border-dashed border-ink/20">
          <div class="flex items-center gap-3"><button onclick="UI.showTab(4)" class="icon-btn text-ink/50" aria-label="Quay lại"><i class="fas fa-arrow-left" aria-hidden="true"></i></button><h2 class="font-serif text-lg text-ink">Mục tiêu tiết kiệm</h2></div>
          <button onclick="Modal.openGoalEditor()" class="icon-btn text-indigo" aria-label="Thêm mục tiêu"><i class="fas fa-plus" aria-hidden="true"></i></button>
        </div>
        <div class="px-6 py-4">
          <div id="goalList" class="space-y-5"></div>
        </div>
      </div>

      <!-- Sub-page: Recurring transactions -->
      <div id="tab-10" class="tab-content hidden" role="tabpanel">
        <div class="px-6 pt-5 pb-3 flex items-center justify-between sticky top-0 bg-cream z-30 border-b border-dashed border-ink/20">
          <div class="flex items-center gap-3"><button onclick="UI.showTab(4)" class="icon-btn text-ink/50" aria-label="Quay lại"><i class="fas fa-arrow-left" aria-hidden="true"></i></button><h2 class="font-serif text-lg text-ink">Giao dịch định kỳ</h2></div>
          <button onclick="Modal.openRecurringEditor()" class="icon-btn text-indigo" aria-label="Thêm định kỳ"><i class="fas fa-plus" aria-hidden="true"></i></button>
        </div>
        <div class="px-6 py-4">
          <p class="text-[11px] text-muted leading-relaxed mb-4">Ứng dụng chạy hoàn toàn trên trình duyệt nên không thể tự tạo giao dịch khi đang tắt. Giao dịch định kỳ đến hạn sẽ được tạo tự động mỗi khi bạn mở lại ứng dụng, hoặc bấm "Tạo ngay".</p>
          <div id="recurringList" class="space-y-3"></div>
        </div>
      </div>

      <!-- Sub-page: Print report -->
      <div id="tab-11" class="tab-content hidden" role="tabpanel">
        <div class="px-6 pt-5 pb-3 flex items-center gap-3 sticky top-0 bg-cream z-30 border-b border-dashed border-ink/20">
          <button onclick="UI.showTab(4)" class="icon-btn text-ink/50" aria-label="Quay lại"><i class="fas fa-arrow-left" aria-hidden="true"></i></button>
          <h2 class="font-serif text-lg text-ink">Báo cáo in / PDF</h2>
        </div>
        <div class="px-6 py-4">
          <p class="text-[11px] tracking-widest text-muted uppercase mb-2">Kỳ báo cáo</p>
          <div class="flex gap-2 mb-4">
            <button data-report-kind="month" onclick="Report.setKind('month')" class="report-kind-btn flex-1 py-2 rounded-full text-xs font-semibold border border-ink/15">Tháng</button>
            <button data-report-kind="quarter" onclick="Report.setKind('quarter')" class="report-kind-btn flex-1 py-2 rounded-full text-xs font-semibold border border-ink/15">Quý</button>
            <button data-report-kind="year" onclick="Report.setKind('year')" class="report-kind-btn flex-1 py-2 rounded-full text-xs font-semibold border border-ink/15">Năm</button>
          </div>
          <div id="reportPeriodPicker" class="mb-5"></div>
          <div id="reportPreview" class="border border-dashed border-ink/20 rounded-lg p-4 mb-5"></div>
          <button onclick="Report.print()" class="w-full bg-indigo text-cream py-3.5 rounded font-semibold flex items-center justify-center gap-2"><i class="fas fa-print" aria-hidden="true"></i> In báo cáo / Lưu PDF</button>
          <p class="text-[11px] text-muted mt-3 leading-relaxed">Chọn "Lưu dưới dạng PDF" trong hộp thoại in của trình duyệt để xuất file PDF.</p>
        </div>
      </div>

    </div>
  </div>
  <!-- Generic modal mount point (populated dynamically by Modal.open) -->
  <div id="modalRoot"></div>

  <!-- Toast mount point -->
  <div id="toastRoot" aria-live="polite" aria-atomic="false"></div>

  <!-- Signature capture modal (fixed markup: needs persistent canvases) -->
  <div id="sigModal" class="hidden fixed inset-0 bg-ink/40 z-[160] flex items-end sm:items-center justify-center p-0 sm:p-4">
    <div class="bg-cream rounded-t-2xl sm:rounded-lg p-6 max-w-sm w-full max-h-[90vh] overflow-y-auto" style="padding-bottom:calc(24px + var(--safe-b))" role="dialog" aria-modal="true" aria-labelledby="sigModalTitle">
      <div class="modal-handle sm:hidden"></div>
      <h3 id="sigModalTitle" class="font-serif text-lg text-ink mb-1">Ký tên</h3>
      <p class="text-muted text-xs mb-4">Ký bằng ngón tay hoặc chuột. Có thể để trống và ký tay sau khi in.</p>

      <p class="text-[11px] tracking-widest text-muted uppercase mb-1" id="sigLabelA">Chữ ký Bên cho vay (A)</p>
      <canvas id="sigCanvasA" width="300" height="120" class="border border-ink/20 rounded bg-white w-full touch-none" aria-label="Vùng ký tên Bên A"></canvas>
      <button type="button" onclick="Signature.clear('sigCanvasA')" class="text-xs text-muted mt-1 mb-4 underline">Xoá</button>

      <p class="text-[11px] tracking-widest text-muted uppercase mb-1" id="sigLabelB">Chữ ký Bên vay (B)</p>
      <canvas id="sigCanvasB" width="300" height="120" class="border border-ink/20 rounded bg-white w-full touch-none" aria-label="Vùng ký tên Bên B"></canvas>
      <button type="button" onclick="Signature.clear('sigCanvasB')" class="text-xs text-muted mt-1 mb-4 underline">Xoá</button>

      <div class="flex gap-2 mt-2">
        <button type="button" onclick="Signature.closeSigModal()" class="flex-1 py-3 rounded border border-ink/20 text-ink text-sm">Huỷ</button>
        <button type="button" onclick="Signature.confirmPrintWithSignatures()" class="flex-1 py-3 rounded bg-indigo text-cream text-sm font-semibold">In giấy nợ</button>
      </div>
    </div>
  </div>

  <!-- Printable IOU document (generalized for either lending or borrowing direction) -->
  <div id="printArea" class="p-10 font-serif text-black bg-white max-w-2xl mx-auto text-sm">
    <h1 class="text-2xl font-bold text-center mb-1 tracking-wide">GIẤY VAY NỢ</h1>
    <p class="text-center text-xs mb-1">(Biên nhận vay tiền cá nhân)</p>
    <p class="text-center text-xs italic mb-6">Căn cứ vào sự thoả thuận và tự nguyện giữa hai bên</p>

    <p class="mb-4">Hôm nay, <span id="pDate"></span>, chúng tôi gồm:</p>

    <p class="font-semibold mb-1">BÊN CHO VAY (Bên A)</p>
    <p class="mb-0.5">Họ tên: <span id="pNameA"></span></p>
    <p class="mb-0.5">CMND/CCCD số: .......................................................</p>
    <p class="mb-4">Địa chỉ: .......................................................</p>

    <p class="font-semibold mb-1">BÊN VAY (Bên B)</p>
    <p class="mb-0.5">Họ tên: <span id="pNameB"></span></p>
    <p class="mb-0.5">CMND/CCCD số: .......................................................</p>
    <p class="mb-6">Địa chỉ: .......................................................</p>

    <p class="mb-3">Hai bên thống nhất lập giấy vay nợ với các điều khoản sau:</p>

    <p class="font-semibold mb-1">Điều 1. Khoản vay</p>
    <p class="mb-0.5 pl-4">Bên A đồng ý cho Bên B vay số tiền:</p>
    <p class="mb-0.5 pl-4">- Bằng số: <span id="pAmountNum" class="font-semibold"></span></p>
    <p class="mb-0.5 pl-4">- Bằng chữ: <span id="pAmountWords" class="font-semibold"></span></p>
    <p class="mb-4 pl-4">- Mục đích vay: .......................................................</p>

    <p class="font-semibold mb-1">Điều 2. Thời hạn và hình thức trả nợ</p>
    <p class="mb-0.5 pl-4">Bên B cam kết hoàn trả đầy đủ số tiền trên cho Bên A vào <span id="pDueDate"></span>.</p>
    <p class="mb-4 pl-4">Hình thức trả:  ☐ Tiền mặt      ☐ Chuyển khoản</p>

    <p class="font-semibold mb-1">Điều 3. Cam kết chung</p>
    <p class="mb-0.5 pl-4">- Bên B cam kết sử dụng khoản vay đúng mục đích đã nêu và hoàn trả đúng hạn cho Bên A.</p>
    <p class="mb-0.5 pl-4">- Nếu chưa thể trả đúng hạn, Bên B có trách nhiệm báo trước và hai bên cùng thoả thuận phương án xử lý.</p>
    <p class="mb-0.5 pl-4">- Giấy này lập trên tinh thần tự nguyện, hai bên đã đọc, hiểu rõ nội dung và đồng ý ký tên xác nhận.</p>
    <p class="mb-10 pl-4">- Giấy được lập thành 02 (hai) bản có giá trị như nhau, mỗi bên giữ 01 bản làm bằng chứng.</p>

    <div class="flex justify-between text-center">
      <div class="w-[45%]">
        <p class="font-semibold mb-1">Bên cho vay (Bên A)</p>
        <div class="h-16 flex items-end justify-center"><img id="pSigA" class="hidden max-h-16 mx-auto" alt=""></div>
        <div class="border-t border-black w-32 mx-auto"></div>
        <p class="text-xs mt-1">(Ký, ghi rõ họ tên)</p>
      </div>
      <div class="w-[45%]">
        <p class="font-semibold mb-1">Bên vay (Bên B)</p>
        <div class="h-16 flex items-end justify-center"><img id="pSigB" class="hidden max-h-16 mx-auto" alt=""></div>
        <div class="border-t border-black w-32 mx-auto"></div>
        <p class="text-xs mt-1">(Ký, ghi rõ họ tên)</p>
      </div>
    </div>
  </div>

  <!-- Printable period report (populated dynamically) -->
  <div id="reportPrintArea" class="p-10 font-serif text-black bg-white max-w-2xl mx-auto text-sm"></div>

<script>
/* ============================================================================
   Sổ Chi Tiêu — bản nâng cấp toàn diện
   Kiến trúc: các namespace theo domain (CONST, Util, Store, Crypto, Security,
   Session, Auth, Profile, Data, Migration, Sync, Ledger, Tx, Loans, Budgets,
   Categories, Wallets, Goals, Recurring, Stats, Charts, UI, History, Undo,
   Modal, Toast, Backup, ExportXlsx, Report, Signature, Photos, PWA, App).
   Mặc định dữ liệu vẫn ở lại trên máy, mã hoá bằng PIN, không gửi đi đâu cả.
   Nếu bạn cấu hình APPWRITE_CONFIG bên dưới VÀ đăng nhập tài khoản, dữ liệu
   giao dịch (không mã hoá PIN) sẽ được đồng bộ thêm lên project Appwrite của
   bạn để dùng được trên nhiều thiết bị — xem khối cấu hình ngay dưới đây.
   Chưa cấu hình = app hoạt động y hệt bản chỉ-lưu-local trước đây, không có
   màn hình đăng nhập nào cả. Bản gốc (v1) được giữ nguyên logic ở những nơi
   không liên quan tới lỗi/tính năng được yêu cầu sửa.
   ========================================================================= */

// ============================================================================
// CẤU HÌNH APPWRITE — điền endpoint + Project ID vào đây để bật tính năng đăng
// ký/đăng nhập + đồng bộ đám mây. Lấy ở Appwrite Console → project của bạn →
// biểu tượng bánh răng (Settings) → mục "API Credentials" (endpoint thường có
// dạng https://<REGION>.cloud.appwrite.io/v1, KHÔNG phải api key nào cả — SDK
// này chỉ chạy trên trình duyệt nên không có "secret key" để lộ).
// databaseId + 7 tableIds phải khớp CHÍNH XÁC với ID bạn đặt khi tạo Database/
// Table trong Console (xem hướng dẫn APPWRITE_SETUP.md đi kèm).
// Để trống (giữ nguyên placeholder) thì app chạy như bản cũ: chỉ lưu local,
// không có màn hình đăng nhập, không mất chức năng gì.
// ============================================================================
const APPWRITE_CONFIG = {
  endpoint: 'https://nyc.cloud.appwrite.io/v1',
  projectId: '6aa8c5ce001748360089',
  databaseId: 'money_tracker',
  // Các bảng phải tạo với ĐÚNG các id này (xem APPWRITE_SETUP.md):
  tableIds: { transactions: 'transactions', loans: 'loans', budgets: 'budgets', categories: 'categories', wallets: 'wallets', transfers: 'transfers', goals: 'goals', recurring: 'recurring' },
  isConfigured() {
    return !!(APPWRITE_CONFIG.endpoint && APPWRITE_CONFIG.projectId
      && APPWRITE_CONFIG.endpoint !== 'YOUR_APPWRITE_ENDPOINT' && APPWRITE_CONFIG.projectId !== 'YOUR_APPWRITE_PROJECT_ID'
      && typeof Appwrite !== 'undefined' && !window.__appwriteLoadFailed);
  }
};
// Client/Account/TablesDB dùng chung toàn app — null nếu chưa cấu hình hoặc SDK tải lỗi (offline lần đầu).
const appwriteClient = APPWRITE_CONFIG.isConfigured() ? new Appwrite.Client().setEndpoint(APPWRITE_CONFIG.endpoint).setProject(APPWRITE_CONFIG.projectId) : null;
const account = appwriteClient ? new Appwrite.Account(appwriteClient) : null;
const tablesDB = appwriteClient ? new Appwrite.TablesDB(appwriteClient) : null;

if (window.Chart) { Chart.defaults.font.family = "'Be Vietnam Pro', sans-serif"; }

// ---------- Constants ----------
const CONST = Object.freeze({
  SCHEMA_VERSION: 2,
  APP_VERSION: '2.1.0',
  TX: Object.freeze({ INCOME: 'income', EXPENSE: 'expense', LOAN_OUT: 'loan_out', LOAN_IN: 'loan_in' }),
  LOAN_DIR: Object.freeze({ LEND: 'lend', BORROW: 'borrow' }),
  LOAN_STATUS: Object.freeze({ UNPAID: 'unpaid', PARTIAL: 'partial', PAID: 'paid', OVERDUE: 'overdue' }),
  RECUR_FREQ: Object.freeze({ DAILY: 'daily', WEEKLY: 'weekly', MONTHLY: 'monthly', YEARLY: 'yearly' }),
  STORAGE_KEYS: Object.freeze({
    SECURE: 'secureData_v2',
    SALT: 'pinSalt_v2',
    ATTEMPTS: 'pinAttempts_v2',
    LEGACY_PIN: 'pinHash',
    LEGACY_TX: 'transactions',
    LEGACY_LOANS: 'loans',
    LEGACY_BUDGET: 'monthlyBudget'
  }),
  LIMITS: Object.freeze({
    NOTE_MAX: 200,
    CATEGORY_NAME_MAX: 40,
    PERSON_NAME_MAX: 80,
    AMOUNT_MAX: 9999999999, // ~10 tỷ VND — giới hạn hợp lý để chặn lỗi nhập liệu / tràn số
    PIN_MIN_LEN: 4,
    PIN_MAX_LEN: 8
  }),
  PBKDF2_ITERATIONS: 150000,
  INACTIVITY_LOCK_MS: 5 * 60 * 1000,
  UNDO_WINDOW_MS: 6000,
  LOW_BALANCE_THRESHOLD: 50000 // Cảnh báo khi số dư một ví xuống dưới mức này (VND)
});

// ---------- Util ----------
const Util = {
  // Sinh ID ổn định: ưu tiên crypto.randomUUID(), fallback getRandomValues, fallback cuối cùng Math.random
  uuid() {
    try {
      if (window.crypto && typeof crypto.randomUUID === 'function') return crypto.randomUUID();
    } catch (e) { console.warn('crypto.randomUUID không dùng được, chuyển sang phương án dự phòng:', e); }
    try {
      if (window.crypto && crypto.getRandomValues) {
        const b = crypto.getRandomValues(new Uint8Array(16));
        b[6] = (b[6] & 0x0f) | 0x40;
        b[8] = (b[8] & 0x3f) | 0x80;
        const hex = Array.from(b, x => x.toString(16).padStart(2, '0'));
        return `${hex.slice(0,4).join('')}-${hex.slice(4,6).join('')}-${hex.slice(6,8).join('')}-${hex.slice(8,10).join('')}-${hex.slice(10,16).join('')}`;
      }
    } catch (e) { console.warn('crypto.getRandomValues không dùng được, dùng phương án dự phòng cuối:', e); }
    return 'id_' + Date.now().toString(36) + '_' + Math.random().toString(36).slice(2, 10);
  },

  escapeHTML(str) {
    if (str == null) return '';
    return String(str)
      .replace(/&/g, '&amp;').replace(/</g, '&lt;').replace(/>/g, '&gt;')
      .replace(/"/g, '&quot;').replace(/'/g, '&#39;');
  },

  // Safe DOM builder — không bao giờ dùng innerHTML với dữ liệu người dùng.
  // h('div', {class:'x', onclick:fn}, 'text con', childNode, ...)
  h(tag, attrs, ...children) {
    const node = document.createElement(tag);
    attrs = attrs || {};
    for (const k in attrs) {
      const v = attrs[k];
      if (v == null || v === false) continue;
      if (k === 'class') node.className = v;
      else if (k === 'dataset') { for (const dk in v) node.dataset[dk] = v[dk]; }
      else if (k.slice(0, 2) === 'on' && typeof v === 'function') node.addEventListener(k.slice(2).toLowerCase(), v);
      else if (k === 'text') node.textContent = v;
      else if (k in node && k !== 'list') { try { node[k] = v; } catch (e) { node.setAttribute(k, v); } }
      else node.setAttribute(k, v);
    }
    children.flat(Infinity).forEach(c => {
      if (c == null || c === false) return;
      node.appendChild(c.nodeType ? c : document.createTextNode(String(c)));
    });
    return node;
  },

  clearChildren(node) { while (node.firstChild) node.removeChild(node.firstChild); },

  todayStr() {
    const d = new Date();
    return `${d.getFullYear()}-${String(d.getMonth() + 1).padStart(2, '0')}-${String(d.getDate()).padStart(2, '0')}`;
  },

  formatVND(n) {
    const num = Number(n) || 0;
    return num.toLocaleString('vi-VN') + ' ₫';
  },
  formatVNDSigned(n) {
    const num = Number(n) || 0;
    return (num >= 0 ? '+' : '−') + Math.abs(num).toLocaleString('vi-VN') + ' ₫';
  },

  formatDateVN(dateStr) {
    const d = Util.parseDate(dateStr);
    if (!d) return dateStr || '';
    return `ngày ${String(d.getDate()).padStart(2, '0')} tháng ${String(d.getMonth() + 1).padStart(2, '0')} năm ${d.getFullYear()}`;
  },
  formatDateShort(dateStr) {
    const d = Util.parseDate(dateStr);
    if (!d) return dateStr || '';
    return `${String(d.getDate()).padStart(2, '0')}/${String(d.getMonth() + 1).padStart(2, '0')}/${d.getFullYear()}`;
  },
  formatDateHeading(dateStr) {
    const d = Util.parseDate(dateStr);
    if (!d) return dateStr || '';
    const days = ['Chủ nhật', 'Thứ 2', 'Thứ 3', 'Thứ 4', 'Thứ 5', 'Thứ 6', 'Thứ 7'];
    return `${days[d.getDay()]}, ${String(d.getDate()).padStart(2, '0')}/${String(d.getMonth() + 1).padStart(2, '0')}/${d.getFullYear()}`;
  },

  // YYYY-MM-DD hợp lệ + là ngày lịch thật sự (chặn 2026-02-30 v.v.)
  isValidDateStr(s) {
    if (typeof s !== 'string' || !/^\d{4}-\d{2}-\d{2}$/.test(s)) return false;
    const [y, m, d] = s.split('-').map(Number);
    if (m < 1 || m > 12 || d < 1 || d > 31) return false;
    const dt = new Date(y, m - 1, d);
    return dt.getFullYear() === y && dt.getMonth() === m - 1 && dt.getDate() === d;
  },
  parseDate(s) {
    if (!Util.isValidDateStr(s)) { const t = new Date(s); return isNaN(t) ? null : t; }
    const [y, m, d] = s.split('-').map(Number);
    return new Date(y, m - 1, d);
  },

  isThisMonth(dateStr) {
    const now = new Date(); const d = Util.parseDate(dateStr);
    return !!d && d.getMonth() === now.getMonth() && d.getFullYear() === now.getFullYear();
  },
  isSameMonth(dateStr, refDate) {
    const d = Util.parseDate(dateStr);
    return !!d && d.getMonth() === refDate.getMonth() && d.getFullYear() === refDate.getFullYear();
  },
  daysBetween(a, b) { return Math.round((Util.parseDate(b) - Util.parseDate(a)) / 86400000); },
  addDays(dateStr, n) { const d = Util.parseDate(dateStr); d.setDate(d.getDate() + n); return Util.toDateStr(d); },
  addMonths(dateStr, n) { const d = Util.parseDate(dateStr); d.setMonth(d.getMonth() + n); return Util.toDateStr(d); },
  addYears(dateStr, n) { const d = Util.parseDate(dateStr); d.setFullYear(d.getFullYear() + n); return Util.toDateStr(d); },
  toDateStr(d) { return `${d.getFullYear()}-${String(d.getMonth() + 1).padStart(2, '0')}-${String(d.getDate()).padStart(2, '0')}`; },

  clamp(n, min, max) { return Math.min(max, Math.max(min, n)); },

  setQuickAmount(fieldId, val) {
    const el = document.getElementById(fieldId);
    if (el) { el.value = val; el.dispatchEvent(new Event('input', { bubbles: true })); }
  },

  // Trả {ok, value, error}. Chặn NaN, Infinity, âm, 0, vượt trần, ép về số nguyên VND.
  validateAmount(raw) {
    const n = typeof raw === 'string' ? parseFloat(raw.replace(/[,\s]/g, '')) : Number(raw);
    if (raw === '' || raw == null || !isFinite(n) || isNaN(n)) return { ok: false, error: 'Số tiền không hợp lệ' };
    if (n <= 0) return { ok: false, error: 'Số tiền phải lớn hơn 0' };
    if (n > CONST.LIMITS.AMOUNT_MAX) return { ok: false, error: 'Số tiền vượt quá giới hạn cho phép' };
    return { ok: true, value: Math.round(n) };
  },

  sanitizeText(str, maxLen) {
    if (str == null) return '';
    // Bỏ ký tự điều khiển (trừ tab/newline khi cần), cắt độ dài, trim.
    let s = String(str).replace(/[\u0000-\u0008\u000B\u000C\u000E-\u001F\u007F]/g, '');
    s = s.trim();
    if (maxLen && s.length > maxLen) s = s.slice(0, maxLen);
    return s;
  },

  validateType(t) { return Object.values(CONST.TX).includes(t); },

  debounce(fn, ms) {
    let t = null;
    return (...args) => { clearTimeout(t); t = setTimeout(() => fn(...args), ms); };
  },

  // ---------- Số thành chữ tiếng Việt (giữ nguyên logic bản gốc, dùng cho giấy nợ) ----------
  readThreeDigits(n, isLeadingGroup) {
    const units = ['không', 'một', 'hai', 'ba', 'bốn', 'năm', 'sáu', 'bảy', 'tám', 'chín'];
    const h = Math.floor(n / 100);
    const rem = n % 100;
    const t = Math.floor(rem / 10);
    const u = rem % 10;
    let out = '';
    if (h > 0) out += units[h] + ' trăm';
    else if (!isLeadingGroup) out += 'không trăm';
    if (rem > 0) {
      if (t === 0) out += (out ? ' lẻ ' : '') + units[u];
      else if (t === 1) out += (out ? ' ' : '') + 'mười' + (u === 5 ? ' lăm' : u > 0 ? ' ' + units[u] : '');
      else out += (out ? ' ' : '') + units[t] + ' mươi' + (u === 1 ? ' mốt' : u === 5 ? ' lăm' : u > 0 ? ' ' + units[u] : '');
    }
    return out.trim();
  },
  numberToVietnameseWords(num) {
    num = Math.round(Math.abs(num));
    if (num === 0) return 'Không đồng';
    const scaleWords = ['', ' nghìn', ' triệu', ' tỷ'];
    const groups = [];
    let n = num;
    while (n > 0) { groups.push(n % 1000); n = Math.floor(n / 1000); }
    let words = [];
    for (let i = groups.length - 1; i >= 0; i--) {
      const g = groups[i];
      if (g === 0) continue;
      words.push(Util.readThreeDigits(g, i === groups.length - 1) + scaleWords[i]);
    }
    let result = words.join(' ').replace(/\s+/g, ' ').trim();
    result = result.charAt(0).toUpperCase() + result.slice(1);
    return result + ' đồng';
  }
};

// ---------- Store: bọc window.storage (Claude artifact) / localStorage, luôn trả về trạng thái thành công ----------
const Store = {
  hasCloudStorage: typeof window.storage !== 'undefined',

  // Các khoá cần tách riêng theo từng tài khoản khi đã đăng nhập (dữ liệu mã hoá + muối + số lần
  // nhập sai PIN). Khi CHƯA đăng nhập (dùng app không cần tài khoản), giữ nguyên khoá gốc như cũ —
  // 100% tương thích ngược, không đụng gì tới người dùng không muốn tạo tài khoản.
  PER_USER_KEYS: new Set([CONST.STORAGE_KEYS.SECURE, CONST.STORAGE_KEYS.SALT, CONST.STORAGE_KEYS.ATTEMPTS]),
  scopedKey(key) {
    if (Store.PER_USER_KEYS.has(key) && typeof Session !== 'undefined' && Session.currentUser) {
      return key + '__u_' + Session.currentUser.id;
    }
    return key;
  },

  async get(key) {
    key = Store.scopedKey(key);
    try {
      if (Store.hasCloudStorage) {
        const r = await window.storage.get(key, false);
        return r ? r.value : null;
      }
      return localStorage.getItem(key);
    } catch (e) {
      console.error('Store.get lỗi:', key, e);
      return null;
    }
  },
  async set(key, value) {
    key = Store.scopedKey(key);
    try {
      if (Store.hasCloudStorage) {
        const r = await window.storage.set(key, value, false);
        return !!r;
      }
      localStorage.setItem(key, value);
      return true;
    } catch (e) {
      console.error('Store.set lỗi (có thể hết dung lượng):', key, e);
      return false;
    }
  },
  async remove(key) {
    key = Store.scopedKey(key);
    try {
      if (Store.hasCloudStorage) { await window.storage.delete(key, false); }
      else { localStorage.removeItem(key); }
      return true;
    } catch (e) {
      console.error('Store.remove lỗi:', key, e);
      return false;
    }
  },

  // Đọc/ghi KHÔNG qua scoping theo tài khoản — dùng cho Migration (cần đọc thẳng dữ liệu cũ trước
  // khi biết tài khoản nào) và các cờ nhỏ dùng chung. Không dùng cho dữ liệu tài chính thông thường.
  async getRaw(key) {
    try { return Store.hasCloudStorage ? (await window.storage.get(key, false) || {}).value || null : localStorage.getItem(key); }
    catch (e) { return null; }
  },
  async setRaw(key, value) {
    try {
      if (Store.hasCloudStorage) { const r = await window.storage.set(key, value, false); return !!r; }
      localStorage.setItem(key, value); return true;
    } catch (e) { console.error('Store.setRaw lỗi:', key, e); return false; }
  }
};

// ---------- Crypto: PBKDF2 (dẫn khoá từ PIN) + AES-GCM (mã hoá toàn bộ dữ liệu) qua Web Crypto API gốc ----------
const Crypto_ = {
  hasSubtle: !!(window.crypto && window.crypto.subtle),

  randomBytes(n) { const b = new Uint8Array(n); crypto.getRandomValues(b); return b; },
  bufToB64(buf) {
    const bytes = buf instanceof Uint8Array ? buf : new Uint8Array(buf);
    let bin = ''; for (let i = 0; i < bytes.length; i++) bin += String.fromCharCode(bytes[i]);
    return btoa(bin);
  },
  b64ToBuf(b64) {
    const bin = atob(b64);
    const bytes = new Uint8Array(bin.length);
    for (let i = 0; i < bin.length; i++) bytes[i] = bin.charCodeAt(i);
    return bytes;
  },
  randomSaltB64() { return Crypto_.bufToB64(Crypto_.randomBytes(16)); },

  async deriveKey(pin, saltB64, iterations) {
    const enc = new TextEncoder();
    const salt = Crypto_.b64ToBuf(saltB64);
    const baseKey = await crypto.subtle.importKey('raw', enc.encode(pin), 'PBKDF2', false, ['deriveKey']);
    return crypto.subtle.deriveKey(
      { name: 'PBKDF2', salt, iterations: iterations || CONST.PBKDF2_ITERATIONS, hash: 'SHA-256' },
      baseKey, { name: 'AES-GCM', length: 256 }, false, ['encrypt', 'decrypt']
    );
  },
  async encryptJSON(key, obj) {
    const iv = Crypto_.randomBytes(12);
    const encBuf = new TextEncoder().encode(JSON.stringify(obj));
    const cipherBuf = await crypto.subtle.encrypt({ name: 'AES-GCM', iv }, key, encBuf);
    return { iv: Crypto_.bufToB64(iv), data: Crypto_.bufToB64(cipherBuf) };
  },
  async decryptJSON(key, ivB64, dataB64) {
    const iv = Crypto_.b64ToBuf(ivB64);
    const data = Crypto_.b64ToBuf(dataB64);
    const plainBuf = await crypto.subtle.decrypt({ name: 'AES-GCM', iv }, key, data);
    return JSON.parse(new TextDecoder().decode(plainBuf));
  },
  // Chỉ để so khớp mã PIN kiểu CŨ (SHA-256, không salt) lúc migrate — không dùng để mã hoá dữ liệu mới.
  async legacyHashPin(pin) {
    if (Crypto_.hasSubtle) {
      try {
        const buf = await crypto.subtle.digest('SHA-256', new TextEncoder().encode(pin));
        return Array.from(new Uint8Array(buf)).map(b => b.toString(16).padStart(2, '0')).join('');
      } catch (e) { console.warn('SubtleCrypto.digest lỗi, dùng hàm băm dự phòng:', e); }
    }
    let h = 0; for (let i = 0; i < pin.length; i++) h = ((h << 5) - h + pin.charCodeAt(i)) | 0;
    return 'fb_' + h.toString(16);
  }
};

// ---------- Auth: CHỈ chịu trách nhiệm tài khoản đăng nhập (Appwrite Auth) — không đụng PIN/dữ liệu ----------
const Auth = {
  // Dịch lỗi của Appwrite (có mã `type` riêng, đáng tin hơn parse chuỗi message) sang tiếng Việt dễ
  // hiểu. Lỗi lạ khác vẫn hiện nguyên văn — an toàn hơn giấu đi, và không đoán trước hết được.
  translateError(err) {
    const type = err && err.type || '';
    const msg = (err && err.message) || String(err);
    const m = msg.toLowerCase();
    if (type === 'user_invalid_credentials') return 'Email hoặc mật khẩu không đúng';
    if (type === 'user_already_exists' || type === 'user_email_already_exists') return 'Email này đã được đăng ký';
    if (type === 'user_password_mismatch') return 'Mật khẩu hiện tại không đúng';
    if (type === 'general_rate_limit_exceeded') return 'Thao tác quá nhanh — thử lại sau ít phút nhé';
    if (type === 'user_unauthorized' || type === 'general_unauthorized_scope') return 'Phiên đăng nhập đã hết hạn — đăng nhập lại nhé';
    if (m.includes('password') && (m.includes('8') || m.includes('length'))) return 'Mật khẩu cần ít nhất 8 ký tự';
    if (m.includes('invalid email') || m.includes('email') && m.includes('valid')) return 'Email không hợp lệ';
    if (m.includes('failed to fetch') || m.includes('networkerror') || m.includes('load failed')) return 'Mất kết nối mạng — kiểm tra Internet rồi thử lại';
    return msg;
  },
  async signUp(email, password, displayName) {
    if (!account) return { ok: false, error: 'Chưa cấu hình Appwrite (xem APPWRITE_CONFIG đầu file)' };
    try {
      await account.create(Appwrite.ID.unique(), email, password, (displayName || '').slice(0, 128) || undefined);
      await account.createEmailPasswordSession(email, password);
      const user = await account.get();
      return { ok: true, user };
    } catch (e) { return { ok: false, error: Auth.translateError(e) }; }
  },
  async signIn(email, password) {
    if (!account) return { ok: false, error: 'Chưa cấu hình Appwrite (xem APPWRITE_CONFIG đầu file)' };
    try {
      await account.createEmailPasswordSession(email, password);
      const user = await account.get();
      return { ok: true, user };
    } catch (e) { return { ok: false, error: Auth.translateError(e) }; }
  },
  async signOut() {
    if (!account) return { ok: true };
    try { await account.deleteSession('current'); return { ok: true }; }
    catch (e) { return { ok: false, error: Auth.translateError(e) }; }
  },
  async sendPasswordReset(email) {
    if (!account) return { ok: false, error: 'Chưa cấu hình Appwrite' };
    try {
      const redirectUrl = window.location.origin + window.location.pathname;
      await account.createRecovery(email, redirectUrl);
      return { ok: true };
    } catch (e) { return { ok: false, error: Auth.translateError(e) }; }
  },
  // Bước 2 của quên-mật-khẩu: gọi khi người dùng bấm link trong email quay lại app kèm userId+secret.
  async completePasswordRecovery(userId, secret, newPassword) {
    if (!account) return { ok: false, error: 'Chưa cấu hình Appwrite' };
    try {
      await account.updateRecovery(userId, secret, newPassword);
      return { ok: true };
    } catch (e) { return { ok: false, error: Auth.translateError(e) }; }
  },
  async updatePassword(newPassword, currentPassword) {
    if (!account) return { ok: false, error: 'Chưa cấu hình Appwrite' };
    try {
      await account.updatePassword(newPassword, currentPassword);
      return { ok: true };
    } catch (e) { return { ok: false, error: Auth.translateError(e) }; }
  }
};

// ---------- Session: "cổng" duy nhất trước luồng PIN/app hiện có — theo dõi ai đang đăng nhập,
// điều khiển màn hình đăng nhập/đăng ký/quên mật khẩu. Nếu APPWRITE_CONFIG chưa được điền (còn
// placeholder) thì coi như "chế độ khách": bỏ qua hoàn toàn, vào thẳng luồng PIN như bản cũ —
// không màn hình đăng nhập, không mất chức năng offline nào của bản trước. ----------
const Session = {
  currentUser: null, // { id, email } — null nếu chưa đăng nhập / chế độ khách
  authView: 'login',

  isLoggedIn() { return !!Session.currentUser; },

  // true = có thể đi thẳng vào luồng PIN/app ngay (chế độ khách, hoặc đã có sẵn phiên đăng nhập).
  // false = đang hiện màn hình đăng nhập, chờ người dùng — App.continueAfterLogin() sẽ được Session
  // tự gọi lại sau khi đăng nhập xong, không cần App.init() chờ ở đây.
  async init() {
    if (!APPWRITE_CONFIG.isConfigured()) return true;

    document.getElementById('authScreen').classList.remove('hidden');

    // Link "quên mật khẩu" trong email đưa người dùng về đây kèm ?userId=...&secret=... — ưu tiên
    // xử lý việc này trước, không quan tâm có đang đăng nhập sẵn hay không.
    const params = new URLSearchParams(window.location.search);
    if (params.get('userId') && params.get('secret')) {
      Session._recoveryParams = { userId: params.get('userId'), secret: params.get('secret') };
      window.history.replaceState({}, '', window.location.pathname);
      Session.showAuthView('reset');
      return false;
    }

    Session._showLoading('Đang kiểm tra đăng nhập...');
    try {
      const user = await account.get(); // Appwrite: reject nếu chưa có phiên đăng nhập nào
      Session.currentUser = { id: user.$id, email: user.email };
      document.getElementById('authScreen').classList.add('hidden');
      return true;
    } catch (e) {
      // Không có phiên (chưa đăng nhập) rơi vào đây — đây là luồng bình thường, không phải lỗi mạng.
      // Chỉ log khi có vẻ là lỗi mạng thật sự, để không gây nhiễu console ở trường hợp bình thường.
      if (!(e && (e.code === 401 || e.type === 'general_unauthorized_scope'))) console.error('Session.init: lỗi kiểm tra phiên đăng nhập:', e);
    }
    Session.showAuthView('login');
    return false;
  },

  async onLoginSuccess(user) {
    Session.currentUser = { id: user.$id, email: user.email };
    document.getElementById('authScreen').classList.add('hidden');
    await App.continueAfterLogin();
    Migration.checkAndOfferAfterUnlock();
  },

  async logout() {
    const confirmed = await Modal.confirm({
      title: 'Đăng xuất?',
      message: 'Dữ liệu mã hoá trên máy vẫn được giữ nguyên — dữ liệu trên tài khoản của bạn không bị xoá.',
      confirmLabel: 'Đăng xuất', danger: true
    });
    if (!confirmed) return;
    await Auth.signOut();
    Session.currentUser = null;
    Security.sessionKey = null; Security.salt = null; Security.hasLoadedData = false; Data.state = null;
    document.getElementById('appRoot').classList.add('hidden');
    document.getElementById('lockScreen').classList.add('hidden');
    document.getElementById('authScreen').classList.remove('hidden');
    Session.showAuthView('login');
  },

  showAuthView(view) {
    Session.authView = view;
    document.getElementById('authLoading').classList.add('hidden');
    ['login', 'register', 'forgot', 'reset'].forEach(v => document.getElementById('authView_' + v).classList.toggle('hidden', v !== view));
    const errEl = document.getElementById('authError_' + view);
    errEl.textContent = '';
    errEl.classList.remove('text-ledger-green'); errEl.classList.add('text-ledger-red');
  },
  _showLoading(msg) {
    ['login', 'register', 'forgot', 'reset'].forEach(v => document.getElementById('authView_' + v).classList.add('hidden'));
    const el = document.getElementById('authLoading');
    el.classList.remove('hidden'); el.textContent = msg;
  },

  async submitLogin() {
    const email = Util.sanitizeText(document.getElementById('authLoginEmail').value, 200).trim();
    const password = document.getElementById('authLoginPassword').value;
    const errEl = document.getElementById('authError_login');
    const btn = document.getElementById('authLoginBtn');
    errEl.textContent = '';
    if (!email || !password) { errEl.textContent = 'Nhập đủ email và mật khẩu'; return; }
    btn.disabled = true; const original = btn.textContent; btn.textContent = 'Đang đăng nhập...';
    try {
      const res = await Auth.signIn(email, password);
      if (!res.ok) { errEl.textContent = res.error; return; }
      await Session.onLoginSuccess(res.user);
    } finally { btn.disabled = false; btn.textContent = original; }
  },

  async submitRegister() {
    const name = Util.sanitizeText(document.getElementById('authRegisterName').value, CONST.LIMITS.PERSON_NAME_MAX);
    const email = Util.sanitizeText(document.getElementById('authRegisterEmail').value, 200).trim();
    const password = document.getElementById('authRegisterPassword').value;
    const password2 = document.getElementById('authRegisterPassword2').value;
    const agree = document.getElementById('authRegisterAgree').checked;
    const errEl = document.getElementById('authError_register');
    const btn = document.getElementById('authRegisterBtn');
    errEl.textContent = '';
    if (!name) { errEl.textContent = 'Nhập tên hiển thị'; return; }
    if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)) { errEl.textContent = 'Email không hợp lệ'; return; }
    if (password.length < 8) { errEl.textContent = 'Mật khẩu cần ít nhất 8 ký tự'; return; }
    if (password !== password2) { errEl.textContent = 'Hai mật khẩu không khớp'; return; }
    if (!agree) { errEl.textContent = 'Cần đồng ý điều khoản để tiếp tục'; return; }
    btn.disabled = true; const original = btn.textContent; btn.textContent = 'Đang tạo tài khoản...';
    try {
      const res = await Auth.signUp(email, password, name);
      if (!res.ok) { errEl.textContent = res.error; return; }
      Toast.show('Tạo tài khoản thành công!', { type: 'ok' });
      await Session.onLoginSuccess(res.user);
    } finally { btn.disabled = false; btn.textContent = original; }
  },

  async submitForgotPassword() {
    const email = Util.sanitizeText(document.getElementById('authForgotEmail').value, 200).trim();
    const errEl = document.getElementById('authError_forgot');
    const btn = document.getElementById('authForgotBtn');
    if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)) { errEl.textContent = 'Email không hợp lệ'; errEl.classList.remove('text-ledger-green'); errEl.classList.add('text-ledger-red'); return; }
    btn.disabled = true; const original = btn.textContent; btn.textContent = 'Đang gửi...';
    try {
      const res = await Auth.sendPasswordReset(email);
      errEl.classList.toggle('text-ledger-red', !res.ok);
      errEl.classList.toggle('text-ledger-green', res.ok);
      errEl.textContent = res.ok ? 'Đã gửi email đặt lại mật khẩu — kiểm tra hộp thư nhé.' : res.error;
    } finally { btn.disabled = false; btn.textContent = original; }
  },

  // Bước cuối của "quên mật khẩu": người dùng bấm link trong email, quay lại app ở view này.
  async submitResetPassword() {
    const p1 = document.getElementById('authResetPassword').value;
    const p2 = document.getElementById('authResetPassword2').value;
    const errEl = document.getElementById('authError_reset');
    const btn = document.getElementById('authResetBtn');
    errEl.textContent = '';
    if (p1.length < 8) { errEl.textContent = 'Mật khẩu cần ít nhất 8 ký tự'; return; }
    if (p1 !== p2) { errEl.textContent = 'Hai mật khẩu không khớp'; return; }
    if (!Session._recoveryParams) { errEl.textContent = 'Link đã hết hạn — gửi lại email đặt lại mật khẩu nhé'; return; }
    btn.disabled = true; const original = btn.textContent; btn.textContent = 'Đang đặt mật khẩu...';
    try {
      const res = await Auth.completePasswordRecovery(Session._recoveryParams.userId, Session._recoveryParams.secret, p1);
      if (!res.ok) { errEl.textContent = res.error; return; }
      Session._recoveryParams = null;
      Toast.show('Đã đặt mật khẩu mới — đăng nhập lại nhé', { type: 'ok' });
      Session.showAuthView('login');
    } finally { btn.disabled = false; btn.textContent = original; }
  },

  togglePasswordVisibility(inputId, btnEl) {
    const input = document.getElementById(inputId);
    const showing = input.type === 'text';
    input.type = showing ? 'password' : 'text';
    Util.clearChildren(btnEl);
    btnEl.appendChild(Util.h('i', { class: `fas ${showing ? 'fa-eye' : 'fa-eye-slash'} text-xs` }));
  }
};

// ---------- Profile: hồ sơ hiển thị (tên, avatar) — tách riêng khỏi Auth (tài khoản) và Data (số liệu) ----------
const Profile = {
  current: null, // { displayName, lastSyncedAt }

  async fetch() {
    if (!account || !Session.isLoggedIn()) return null;
    try {
      const user = await account.get();
      Profile.current = { displayName: user.name || '', lastSyncedAt: (user.prefs && user.prefs.lastSyncedAt) || null };
      return Profile.current;
    } catch (e) { console.error('Profile.fetch lỗi:', e); return null; }
  },
  async updateDisplayName(name) {
    if (!account || !Session.isLoggedIn()) return { ok: false, error: 'Chưa đăng nhập' };
    const clean = Util.sanitizeText(name, CONST.LIMITS.PERSON_NAME_MAX);
    if (!clean) return { ok: false, error: 'Tên không được để trống' };
    try {
      await account.updateName(clean);
      if (Profile.current) Profile.current.displayName = clean;
      UI.applyProfileNameToHeader();
      return { ok: true };
    } catch (e) { return { ok: false, error: Auth.translateError(e) }; }
  },
  // Appwrite không có bảng "profiles" riêng — account.prefs là kho key-value có sẵn theo mỗi tài
  // khoản, hợp để lưu một mốc thời gian nhỏ như thế này mà không cần thêm bảng nào.
  async updateLastSyncedAt(iso) {
    if (!account || !Session.isLoggedIn()) return;
    try {
      const user = await account.get();
      await account.updatePrefs(Object.assign({}, user.prefs, { lastSyncedAt: iso }));
      if (Profile.current) Profile.current.lastSyncedAt = iso;
    } catch (e) { console.error('Profile.updateLastSyncedAt lỗi (không quan trọng bằng dữ liệu chính, bỏ qua):', e); }
  },
  initials() {
    const src = (Profile.current && Profile.current.displayName) || (Session.currentUser && Session.currentUser.email) || '?';
    return src.trim().charAt(0).toUpperCase();
  }
};

// ---------- Security: thiết lập/mở khoá PIN, migrate từ bản cũ, giới hạn số lần sai, khôi phục ----------
const Security = {
  mode: 'new', // 'new' | 'existing' | 'legacy-migrate' | 'corrupted'
  sessionKey: null,
  salt: null,
  hasLoadedData: false,
  _pendingFirstPin: null,
  _lockoutTicker: null,

  async init() {
    const forgotBtn = document.getElementById('forgotPinBtn');
    const secureRaw = await Store.get(CONST.STORAGE_KEYS.SECURE);
    if (secureRaw) {
      try {
        const parsed = JSON.parse(secureRaw);
        if (!parsed || !parsed.iv || !parsed.data) throw new Error('missing fields');
        Security.salt = await Store.get(CONST.STORAGE_KEYS.SALT);
        if (!Security.salt) throw new Error('missing salt');
        Security.mode = 'existing';
        document.getElementById('lockTitle').textContent = 'Nhập mã PIN';
        document.getElementById('lockSubtitle').textContent = 'Dữ liệu của bạn đang được mã hoá.';
        forgotBtn.classList.remove('hidden');
      } catch (e) {
        console.error('Dữ liệu lưu trữ bị hỏng:', e);
        Security.mode = 'corrupted';
        document.getElementById('lockTitle').textContent = 'Không đọc được dữ liệu';
        document.getElementById('lockSubtitle').textContent = 'Dữ liệu lưu trên máy có vẻ đã bị hỏng. Bạn có thể khôi phục từ file sao lưu hoặc bắt đầu lại.';
        document.getElementById('pinInput').classList.add('hidden');
        document.getElementById('pinSubmitBtn').classList.add('hidden');
        forgotBtn.textContent = 'Khôi phục dữ liệu';
        forgotBtn.classList.remove('hidden');
      }
    } else {
      const legacyHash = await Store.get(CONST.STORAGE_KEYS.LEGACY_PIN);
      if (legacyHash) {
        Security.mode = 'legacy-migrate';
        document.getElementById('lockTitle').textContent = 'Nhập mã PIN';
        document.getElementById('lockSubtitle').textContent = 'Đang nâng cấp bảo mật — nhập mã PIN hiện tại của bạn.';
        forgotBtn.classList.remove('hidden');
      } else {
        Security.mode = 'new';
        document.getElementById('lockTitle').textContent = 'Đặt mã PIN mới';
        document.getElementById('lockSubtitle').textContent = 'Chọn mã PIN từ 4-8 số để mã hoá dữ liệu của bạn.';
        forgotBtn.classList.add('hidden');
      }
    }

    const pinInput = document.getElementById('pinInput');
    pinInput.addEventListener('input', e => { e.target.value = e.target.value.replace(/\D/g, ''); });
    pinInput.addEventListener('keydown', e => { if (e.key === 'Enter') Security.submitPin(); });
    if (Security.mode !== 'corrupted') pinInput.focus();
    Security.renderLockoutState();
  },

  // ---------- Giới hạn số lần nhập sai (rate limit, tăng dần thời gian chờ) ----------
  async getAttempts() {
    const raw = await Store.get(CONST.STORAGE_KEYS.ATTEMPTS);
    if (!raw) return { count: 0, lockUntil: 0 };
    try { const p = JSON.parse(raw); return { count: p.count || 0, lockUntil: p.lockUntil || 0 }; }
    catch (e) { return { count: 0, lockUntil: 0 }; }
  },
  async saveAttempts(a) { return Store.set(CONST.STORAGE_KEYS.ATTEMPTS, JSON.stringify(a)); },
  async checkRateLimit() {
    const a = await Security.getAttempts();
    const wait = Math.ceil((a.lockUntil - Date.now()) / 1000);
    if (a.lockUntil && wait > 0) return { allowed: false, waitSeconds: wait };
    return { allowed: true, waitSeconds: 0 };
  },
  async recordFailedAttempt() {
    const a = await Security.getAttempts();
    a.count = (a.count || 0) + 1;
    const backoff = [0, 0, 0, 0, 30, 60, 120, 300, 600, 900];
    const idx = Util.clamp(a.count, 0, backoff.length - 1);
    if (a.count >= 5) a.lockUntil = Date.now() + backoff[idx] * 1000;
    await Security.saveAttempts(a);
    Security.renderLockoutState();
  },
  async resetAttempts() { await Security.saveAttempts({ count: 0, lockUntil: 0 }); Security.renderLockoutState(); },

  async renderLockoutState() {
    const { allowed, waitSeconds } = await Security.checkRateLimit();
    const btn = document.getElementById('pinSubmitBtn');
    const err = document.getElementById('lockError');
    clearInterval(Security._lockoutTicker);
    if (!allowed) {
      btn.disabled = true;
      let remaining = waitSeconds;
      const tick = () => {
        err.textContent = `Bạn đã nhập sai nhiều lần. Vui lòng thử lại sau ${remaining}s.`;
        remaining--;
        if (remaining < 0) { clearInterval(Security._lockoutTicker); btn.disabled = false; err.textContent = ''; }
      };
      tick();
      Security._lockoutTicker = setInterval(tick, 1000);
    } else {
      btn.disabled = false;
    }
  },

  // ---------- Nộp mã PIN: phân nhánh theo trạng thái tài khoản ----------
  async submitPin() {
    const { allowed } = await Security.checkRateLimit();
    if (!allowed) return;
    const pin = document.getElementById('pinInput').value;
    const errorEl = document.getElementById('lockError');
    if (pin.length < CONST.LIMITS.PIN_MIN_LEN) { errorEl.textContent = `Mã PIN cần ít nhất ${CONST.LIMITS.PIN_MIN_LEN} số`; return; }

    if (Security.mode === 'new') return Security._submitNewAccount(pin);
    if (Security.mode === 'legacy-migrate') return Security._submitLegacyMigrate(pin);
    if (Security.mode === 'existing') return Security._submitExisting(pin);
  },

  async _submitNewAccount(pin) {
    const errorEl = document.getElementById('lockError');
    if (!Security._pendingFirstPin) {
      Security._pendingFirstPin = pin;
      document.getElementById('lockTitle').textContent = 'Nhập lại để xác nhận';
      document.getElementById('pinInput').value = '';
      errorEl.textContent = '';
      return;
    }
    if (pin !== Security._pendingFirstPin) {
      errorEl.textContent = 'Không khớp, thử lại nhé';
      Security._pendingFirstPin = null;
      document.getElementById('lockTitle').textContent = 'Đặt mã PIN mới';
      document.getElementById('pinInput').value = '';
      return;
    }
    Security.salt = Crypto_.randomSaltB64();
    const key = await Crypto_.deriveKey(pin, Security.salt, CONST.PBKDF2_ITERATIONS);
    Data.state = Data.freshState();
    Security.sessionKey = key;
    await Store.set(CONST.STORAGE_KEYS.SALT, Security.salt);
    const ok = await Data.save();
    if (!ok) { errorEl.textContent = 'Không thể lưu dữ liệu trên máy này. Kiểm tra bộ nhớ trống rồi thử lại.'; return; }
    Security.mode = 'existing';
    Security.hasLoadedData = true;
    Security.unlockApp(true);
  },

  async _submitLegacyMigrate(pin) {
    const errorEl = document.getElementById('lockError');
    const legacyHash = await Store.get(CONST.STORAGE_KEYS.LEGACY_PIN);
    const hash = await Crypto_.legacyHashPin(pin);
    if (hash !== legacyHash) {
      await Security.recordFailedAttempt();
      errorEl.textContent = 'Sai mã PIN, thử lại nhé';
      document.getElementById('pinInput').value = '';
      return;
    }
    // Đúng PIN cũ — nâng cấp sang lưu trữ mã hoá mới với cùng mã PIN này.
    let legacyTx = [], legacyLoans = [], legacyBudget = null;
    try { legacyTx = JSON.parse((await Store.get(CONST.STORAGE_KEYS.LEGACY_TX)) || '[]'); } catch (e) { legacyTx = []; }
    try { legacyLoans = JSON.parse((await Store.get(CONST.STORAGE_KEYS.LEGACY_LOANS)) || '[]'); } catch (e) { legacyLoans = []; }
    legacyBudget = await Store.get(CONST.STORAGE_KEYS.LEGACY_BUDGET);

    Data.state = Data.migrateLegacy(legacyTx, legacyLoans, legacyBudget);
    Security.salt = Crypto_.randomSaltB64();
    Security.sessionKey = await Crypto_.deriveKey(pin, Security.salt, CONST.PBKDF2_ITERATIONS);
    await Store.set(CONST.STORAGE_KEYS.SALT, Security.salt);
    const ok = await Data.save();
    if (!ok) { errorEl.textContent = 'Không thể lưu dữ liệu đã nâng cấp. Thử lại nhé.'; return; }
    await Store.remove(CONST.STORAGE_KEYS.LEGACY_PIN);
    await Store.remove(CONST.STORAGE_KEYS.LEGACY_TX);
    await Store.remove(CONST.STORAGE_KEYS.LEGACY_LOANS);
    await Store.remove(CONST.STORAGE_KEYS.LEGACY_BUDGET);
    await Security.resetAttempts();
    Security.mode = 'existing';
    Security.hasLoadedData = true;
    Security._migratedThisSession = true;
    Security.unlockApp(true);
  },

  async _submitExisting(pin) {
    const errorEl = document.getElementById('lockError');
    const raw = await Store.get(CONST.STORAGE_KEYS.SECURE);
    let parsed;
    try { parsed = JSON.parse(raw); } catch (e) { errorEl.textContent = 'Dữ liệu lưu trữ bị hỏng.'; return; }
    try {
      const key = await Crypto_.deriveKey(pin, Security.salt, CONST.PBKDF2_ITERATIONS);
      const state = await Crypto_.decryptJSON(key, parsed.iv, parsed.data);
      Security.sessionKey = key;
      Data.state = Data.migrateIfNeeded(state);
      await Security.resetAttempts();
      const firstThisSession = !Security.hasLoadedData;
      Security.hasLoadedData = true;
      Security.unlockApp(firstThisSession);
    } catch (e) {
      await Security.recordFailedAttempt();
      errorEl.textContent = 'Sai mã PIN, thử lại nhé';
      document.getElementById('pinInput').value = '';
    }
  },

  unlockApp(isFirstLoad) {
    document.getElementById('lockScreen').classList.add('hidden');
    document.getElementById('appRoot').classList.remove('hidden');
    document.getElementById('pinInput').value = '';
    Security.resetInactivityTimer();
    if (isFirstLoad) {
      App.afterFirstUnlock();
      if (Security._migratedThisSession) { Toast.show('Đã nâng cấp bảo mật dữ liệu thành công', { type: 'ok' }); Security._migratedThisSession = false; }
    } else {
      UI.renderDashboard();
    }
  },

  lockApp() {
    if (!Security.hasLoadedData) return;
    document.getElementById('appRoot').classList.add('hidden');
    document.getElementById('lockScreen').classList.remove('hidden');
    document.getElementById('lockTitle').textContent = 'Nhập mã PIN';
    document.getElementById('lockSubtitle').textContent = 'Dữ liệu của bạn đang được mã hoá.';
    document.getElementById('pinInput').value = '';
    document.getElementById('lockError').textContent = '';
    UI.balanceHidden = true;
    const btn = document.getElementById('balanceToggleBtn');
    if (btn) { Util.clearChildren(btn); btn.appendChild(Util.h('i', { class: 'fas fa-eye text-xs', 'aria-hidden': 'true' })); }
    Modal.close();
    document.getElementById('pinInput').focus();
  },

  resetInactivityTimer() {
    clearTimeout(Security._inactivityTimer);
    Security._inactivityTimer = setTimeout(Security.lockApp, CONST.INACTIVITY_LOCK_MS);
  },

  // ---------- Đổi mã PIN (mã hoá lại toàn bộ dữ liệu với khoá mới) ----------
  async changePin(currentPin, newPin) {
    const raw = await Store.get(CONST.STORAGE_KEYS.SECURE);
    let parsed;
    try { parsed = JSON.parse(raw); } catch (e) { return { ok: false, error: 'Dữ liệu lưu trữ bị hỏng.' }; }
    let verifyKey;
    try {
      verifyKey = await Crypto_.deriveKey(currentPin, Security.salt, CONST.PBKDF2_ITERATIONS);
      await Crypto_.decryptJSON(verifyKey, parsed.iv, parsed.data);
    } catch (e) {
      return { ok: false, error: 'Mã PIN hiện tại không đúng.' };
    }
    const newSalt = Crypto_.randomSaltB64();
    const newKey = await Crypto_.deriveKey(newPin, newSalt, CONST.PBKDF2_ITERATIONS);
    Security.salt = newSalt;
    Security.sessionKey = newKey;
    await Store.set(CONST.STORAGE_KEYS.SALT, newSalt);
    const ok = await Data.save();
    if (!ok) return { ok: false, error: 'Không lưu được mã PIN mới. Thử lại nhé.' };
    return { ok: true };
  },

  // ---------- Khôi phục khi quên PIN: không còn xoá ngay lập tức ----------
  openRecoverySheet() {
    if (Security.mode === 'corrupted') return Modal.openCorruptedRecoverySheet();
    Modal.openForgotPinSheet();
  }
};

// ---------- Data: trạng thái trung tâm duy nhất + load/lưu/migrate ----------
const Data = {
  state: null,

  CATEGORY_PALETTE: ['#AE4030', '#3E7C87', '#6B5B95', '#4A6FA5', '#B0544D', '#8B5A2B', '#7A6C5D', '#5C7A5C', '#9C6B30', '#4B7A5E', '#B8863B', '#34456B'],

  defaultCategories() {
    return [
      { id: 'cat_food', name: 'Ăn uống', icon: '🍜', color: '#AE4030', type: 'expense', isDefault: true },
      { id: 'cat_transport', name: 'Đi lại', icon: '🚗', color: '#3E7C87', type: 'expense', isDefault: true },
      { id: 'cat_entertainment', name: 'Giải trí', icon: '🎮', color: '#6B5B95', type: 'expense', isDefault: true },
      { id: 'cat_phone', name: 'Điện thoại', icon: '📱', color: '#4A6FA5', type: 'expense', isDefault: true },
      { id: 'cat_shopping', name: 'Mua sắm', icon: '🛒', color: '#B0544D', type: 'expense', isDefault: true },
      { id: 'cat_study', name: 'Học tập', icon: '📚', color: '#8B5A2B', type: 'expense', isDefault: true },
      { id: 'cat_home', name: 'Nhà ở', icon: '🏠', color: '#7A6C5D', type: 'expense', isDefault: true },
      { id: 'cat_health', name: 'Sức khoẻ', icon: '💊', color: '#5C7A5C', type: 'expense', isDefault: true },
      { id: 'cat_other_exp', name: 'Khác', icon: '💸', color: '#8B8175', type: 'expense', isDefault: true },
      { id: 'cat_salary', name: 'Lương', icon: '💰', color: '#4B7A5E', type: 'income', isDefault: true },
      { id: 'cat_bonus', name: 'Thưởng', icon: '🎁', color: '#B8863B', type: 'income', isDefault: true },
      { id: 'cat_invest', name: 'Đầu tư', icon: '📈', color: '#34456B', type: 'income', isDefault: true },
      { id: 'cat_other_inc', name: 'Khác', icon: '💵', color: '#9C6B30', type: 'income', isDefault: true }
    ];
  },
  defaultWallets() {
    return [
      { id: 'wallet_cash', name: 'Tiền mặt', icon: '💵', isDefault: true, includeInTotal: true },
      { id: 'wallet_bank', name: 'Ngân hàng', icon: '🏦', isDefault: false, includeInTotal: true },
      { id: 'wallet_ewallet', name: 'Ví điện tử', icon: '📲', isDefault: false, includeInTotal: true }
    ];
  },

  freshState() {
    const now = new Date().toISOString();
    return {
      schemaVersion: CONST.SCHEMA_VERSION,
      transactions: [],
      loans: [],
      budgets: [],
      categories: Data.defaultCategories(),
      wallets: Data.defaultWallets(),
      transfers: [],
      goals: [],
      recurring: [],
      pendingDeletes: [], // {table, id, deletedAt} chờ đồng bộ xoá lên Appwrite — chỉ dùng khi có đăng nhập
      settings: { budgetAlertsEnabled: true },
      metadata: { createdAt: now, lastModified: now, appVersion: CONST.APP_VERSION }
    };
  },

  // Kiểm tra & vá cấu trúc phòng thủ — không để thiếu field nào làm crash app, và là nơi
  // đặt các bước nâng cấp schema trong tương lai (if state.schemaVersion < 3) { ... }).
  migrateIfNeeded(state) {
    if (!state || typeof state !== 'object') return Data.freshState();
    const fresh = Data.freshState();
    for (const key of ['transactions', 'loans', 'budgets', 'categories', 'wallets', 'transfers', 'goals', 'recurring', 'pendingDeletes']) {
      if (!Array.isArray(state[key])) state[key] = fresh[key];
    }
    state.settings = Object.assign({}, fresh.settings, state.settings || {});
    state.metadata = Object.assign({}, fresh.metadata, state.metadata || {});
    if (!state.categories.length) state.categories = fresh.categories;
    if (!state.wallets.length) state.wallets = fresh.wallets;
    state.wallets.forEach(w => { if (w.includeInTotal == null) w.includeInTotal = true; });
    state.schemaVersion = CONST.SCHEMA_VERSION;
    return state;
  },

  // Chuyển dữ liệu bản cũ (transactions/loans phẳng, không mã hoá) sang cấu trúc v2.
  // Không xoá hay bỏ sót bất kỳ bản ghi nào; danh mục tự do trước đây được giữ nguyên tên,
  // tự tạo thành danh mục tuỳ chỉnh nếu chưa khớp danh mục mặc định nào.
  migrateLegacy(legacyTx, legacyLoans, legacyBudgetStr) {
    const state = Data.freshState();
    let colorIdx = 0;
    const findOrCreateCategory = (rawName, type) => {
      const name = Util.sanitizeText(rawName, CONST.LIMITS.CATEGORY_NAME_MAX) || 'Khác';
      let cat = state.categories.find(c => c.type === type && c.name.toLowerCase() === name.toLowerCase());
      if (cat) return cat.id;
      cat = { id: Util.uuid(), name, icon: type === 'income' ? '💵' : '💸', color: Data.CATEGORY_PALETTE[colorIdx % Data.CATEGORY_PALETTE.length], type, isDefault: false };
      colorIdx++;
      state.categories.push(cat);
      return cat.id;
    };

    state.transactions = (Array.isArray(legacyTx) ? legacyTx : []).map(t => {
      if (!t || typeof t !== 'object') return null;
      const type = Util.validateType(t.type) ? t.type : CONST.TX.EXPENSE;
      const amtCheck = Util.validateAmount(t.amount);
      if (!amtCheck.ok) return null;
      const statType = (type === CONST.TX.INCOME || type === CONST.TX.LOAN_IN) ? 'income' : 'expense';
      return {
        id: Util.uuid(),
        type,
        amount: amtCheck.value,
        categoryId: findOrCreateCategory(t.category, statType),
        note: Util.sanitizeText(t.note, CONST.LIMITS.NOTE_MAX),
        date: Util.isValidDateStr(t.date) ? t.date : Util.todayStr(),
        walletId: state.wallets[0].id,
        loanId: null,
        isRepayment: false,
        createdAt: new Date().toISOString()
      };
    }).filter(Boolean);

    state.loans = (Array.isArray(legacyLoans) ? legacyLoans : []).map(l => {
      if (!l || typeof l !== 'object') return null;
      const amtCheck = Util.validateAmount(l.amount);
      if (!amtCheck.ok) return null;
      const isPaid = l.status === 'Đã trả';
      return {
        id: Util.uuid(),
        direction: CONST.LOAN_DIR.LEND,
        counterpartyName: Util.sanitizeText(l.name, CONST.LIMITS.PERSON_NAME_MAX) || 'Không rõ tên',
        principal: amtCheck.value,
        paidAmount: isPaid ? amtCheck.value : 0,
        remainingAmount: isPaid ? 0 : amtCheck.value,
        loanDate: Util.isValidDateStr(l.date) ? l.date : Util.todayStr(),
        dueDate: Util.isValidDateStr(l.dueDate) ? l.dueDate : null,
        status: isPaid ? CONST.LOAN_STATUS.PAID : CONST.LOAN_STATUS.UNPAID,
        note: '',
        initialTransactionId: null,
        repayments: [],
        legacyUnlinked: true
      };
    }).filter(Boolean);

    if (legacyBudgetStr) {
      const n = parseInt(legacyBudgetStr, 10);
      if (isFinite(n) && n > 0) {
        state.budgets.push({ id: Util.uuid(), categoryId: null, amount: Math.round(n), alertsEnabled: true, createdAt: new Date().toISOString() });
      }
    }
    return state;
  },

  // Các loại bản ghi có đồng bộ lên tài khoản (khi đăng nhập) — dùng để tự động đóng dấu
  // createdAt/updatedAt mỗi lần lưu, và để Sync biết cần đẩy/kéo những bảng nào.
  SYNCED_ENTITY_KEYS: ['transactions', 'loans', 'budgets', 'categories', 'wallets', 'transfers', 'goals', 'recurring'],
  _lastSavedSnapshot: null,

  // Tự động đóng dấu updatedAt cho bản ghi mới hoặc vừa đổi so với lần lưu trước — tập trung MỘT
  // chỗ duy nhất để Tx/Loans/Wallets/Budgets/Categories/Goals/Recurring/Transfers không cần tự
  // thêm updatedAt thủ công ở từng nơi (đỡ sót, đỡ sai). So sánh bỏ qua chính field updatedAt để
  // không tự coi là "đổi" chỉ vì updatedAt khác nhau. Lần lưu đầu tiên sau khi nâng cấp app (chưa
  // có snapshot trước đó) sẽ đóng dấu toàn bộ dữ liệu hiện có — đúng ý muốn vì đó cũng là dữ liệu
  // "mới" đối với việc đồng bộ lần đầu.
  _stampUpdatedTimestamps() {
    const now = new Date().toISOString();
    const prev = Data._lastSavedSnapshot;
    Data.SYNCED_ENTITY_KEYS.forEach(key => {
      const prevMap = new Map();
      if (prev && Array.isArray(prev[key])) prev[key].forEach(e => { if (e && e.id) prevMap.set(e.id, e); });
      (Data.state[key] || []).forEach(entity => {
        if (!entity || !entity.id) return;
        const before = prevMap.get(entity.id);
        if (!before) {
          if (!entity.createdAt) entity.createdAt = now;
          entity.updatedAt = now;
        } else {
          const { updatedAt: _a, ...curRest } = entity;
          const { updatedAt: _b, ...beforeRest } = before;
          if (JSON.stringify(curRest) !== JSON.stringify(beforeRest)) entity.updatedAt = now;
        }
      });
    });
  },

  // Hàng đợi xoá để đồng bộ lên tài khoản — chỉ theo dõi khi đã đăng nhập (khách dùng local-only
  // không tốn thêm chỗ lưu nào cho việc này). unqueueDelete dùng khi Hoàn tác (Undo) một lượt xoá.
  queueDelete(table, id) {
    if (!Session.isLoggedIn() || !Data.state || !id) return;
    Data.state.pendingDeletes.push({ table, id, deletedAt: new Date().toISOString() });
  },
  unqueueDelete(table, id) {
    if (!Data.state || !Array.isArray(Data.state.pendingDeletes)) return;
    Data.state.pendingDeletes = Data.state.pendingDeletes.filter(d => !(d.table === table && d.id === id));
  },

  async save() {
    if (!Security.sessionKey) { console.error('Data.save: chưa có khoá phiên (ứng dụng đang khoá?)'); return false; }
    if (!Data.state) return false;
    Data._stampUpdatedTimestamps();
    Data.state.metadata.lastModified = new Date().toISOString();
    try {
      const enc = await Crypto_.encryptJSON(Security.sessionKey, Data.state);
      const payload = JSON.stringify({ v: CONST.SCHEMA_VERSION, iv: enc.iv, data: enc.data });
      const ok = await Store.set(CONST.STORAGE_KEYS.SECURE, payload);
      if (!ok) { Toast.show('Không thể lưu dữ liệu — kiểm tra bộ nhớ trên máy', { type: 'err' }); return false; }
      Data._lastSavedSnapshot = Data.snapshot();
      return true;
    } catch (e) {
      console.error('Data.save lỗi mã hoá:', e);
      Toast.show('Lỗi khi lưu dữ liệu', { type: 'err' });
      return false;
    }
  },

  snapshot() { return JSON.parse(JSON.stringify(Data.state)); },
  async restoreSnapshot(snap) { Data.state = snap; return Data.save(); },

  findCategory(id) { return Data.state.categories.find(c => c.id === id) || null; },
  findWallet(id) { return Data.state.wallets.find(w => w.id === id) || null; },
  defaultWalletId() { const w = Data.state.wallets.find(w => w.isDefault); return w ? w.id : (Data.state.wallets[0] && Data.state.wallets[0].id); }
};

// ---------- Migration: chuyển dữ liệu cũ (trước khi có tài khoản) vào tài khoản mới đăng nhập ----------
// Chỉ CHÉP (không xoá, không di chuyển) dữ liệu cũ sang khoá riêng của tài khoản — bản gốc luôn còn
// nguyên như một bản sao lưu ngầm. Việc có đẩy dữ liệu lên Appwrite hay không là một câu hỏi RIÊNG,
// hỏi sau khi đã mở khoá PIN thành công (vì cần giải mã xong mới biết có gì để đồng bộ).
const Migration = {
  _legacyInherited: false,

  // Gọi ngay sau khi đăng nhập, TRƯỚC Security.init(): nếu tài khoản này (theo user id) chưa có
  // dữ liệu local riêng, nhưng máy này đang có dữ liệu cũ (từ trước khi có tính năng đăng nhập, hoặc
  // từ một lần dùng "chỉ trên thiết bị"), thì thừa hưởng nó — nhờ vậy họ mở khoá bằng ĐÚNG mã PIN cũ,
  // không cần luồng nhập PIN riêng biệt nào khác.
  async inheritLegacyLocalDataIfAny() {
    if (!Session.isLoggedIn()) return;
    const ownScopedKey = Store.scopedKey(CONST.STORAGE_KEYS.SECURE);
    if (await Store.getRaw(ownScopedKey)) return; // tài khoản này đã có dữ liệu riêng — không đụng vào
    const legacySecure = await Store.getRaw(CONST.STORAGE_KEYS.SECURE);
    if (!legacySecure) return; // máy này chưa có dữ liệu cũ nào
    const legacySalt = await Store.getRaw(CONST.STORAGE_KEYS.SALT);
    await Store.setRaw(ownScopedKey, legacySecure);
    if (legacySalt) await Store.setRaw(Store.scopedKey(CONST.STORAGE_KEYS.SALT), legacySalt);
    Migration._legacyInherited = true;
  },

  // Gọi sau khi mở khoá PIN thành công lần đầu trong phiên đăng nhập này (App.afterFirstUnlock).
  async checkAndOfferAfterUnlock() {
    if (!Session.isLoggedIn() || !tablesDB || !Migration._legacyInherited) return;
    Migration._legacyInherited = false;
    const decidedFlag = 'migrationDecided_v1__u_' + Session.currentUser.id;
    if (await Store.getRaw(decidedFlag)) return; // đã hỏi/quyết định trước đó rồi — không hỏi lại mỗi lần mở app
    const wantsSync = await Migration._showChoiceModal();
    await Store.setRaw(decidedFlag, '1');
    if (wantsSync) await Migration.syncToAccount();
  },

  _showChoiceModal() {
    return new Promise(resolve => {
      Modal.open({
        title: 'Đã tìm thấy dữ liệu trên thiết bị này',
        bodyNode: Util.h('p', { class: 'text-sm text-ink/80 leading-relaxed' },
          'Có dữ liệu Sổ Chi Tiêu sẵn trên máy này. Đồng bộ lên tài khoản để dùng được trên nhiều thiết bị, hay chỉ giữ trên máy này thôi?'),
        actions: [
          { label: 'Giữ trên thiết bị', onClick: () => { resolve(false); Modal.close(); } },
          { label: 'Đồng bộ lên tài khoản', variant: 'primary', onClick: () => { resolve(true); Modal.close(); } }
        ],
        onClose: () => resolve(false)
      });
    });
  },

  // Sao lưu cục bộ trước, rồi đẩy dữ liệu lên qua Sync (upsert theo id). Không có transaction nhiều
  // bảng như Postgres RPC trước đây, nhưng upsert nghĩa là chạy lại vẫn an toàn — không tạo trùng
  // dù lỡ lỗi giữa chừng. Dữ liệu local không đổi dù thành công hay thất bại.
  async syncToAccount() {
    if (!Data.state || !Session.isLoggedIn() || !tablesDB) return { ok: false };
    try { await Store.setRaw('preMigrationBackup__u_' + Session.currentUser.id + '__' + Date.now(), JSON.stringify(Data.snapshot())); }
    catch (e) { console.error('Sao lưu trước migration lỗi (vẫn tiếp tục):', e); }

    Toast.show('Đang đồng bộ dữ liệu lên tài khoản...', { type: 'info' });
    // Appwrite không có transaction nhiều-bảng như trước (Postgres RPC) — Sync.syncNow() ghi từng
    // dòng kiểu upsert (chạy lại vẫn an toàn, không tạo trùng), nên nếu lỗi giữa chừng chỉ cần bấm
    // "Đồng bộ ngay" ở Cài đặt › Tài khoản để tiếp tục — không mất hay ghi đè nhầm phần đã lên rồi.
    const res = await Sync.syncNow();
    if (res.ok) Toast.show('Đồng bộ thành công!', { type: 'ok' });
    else Toast.show('Đồng bộ thất bại — dữ liệu trên máy vẫn an toàn, thử lại ở Cài đặt › Tài khoản nhé', { type: 'err' });
    return res;
  }
};

// ---------- Sync: đẩy/kéo dữ liệu giữa máy này và Appwrite — chỉ chạy khi đã đăng nhập ----------
// Chiến lược: đẩy TOÀN BỘ dữ liệu local lên trước (upsertRow theo id — chạy lại nhiều lần vẫn an
// toàn, không tạo trùng), xử lý hàng đợi xoá, RỒI MỚI kéo toàn bộ dữ liệu server về hợp nhất theo
// id. Nhờ đẩy trước-kéo sau, một bản ghi local mà bị thiếu khi kéo về chắc chắn là do bị xoá ở máy
// khác (không phải do chưa kịp đẩy) — nên an toàn để dọn khỏi local. Xung đột: updatedAt bản nào
// mới hơn thắng (last-write-wins). Quyền riêng tư mỗi dòng: gắn Permission.read/update/delete cho
// đúng user khi ghi (tương đương RLS/policy bên Postgres) — nhớ bật "Row Security" cho cả 8 bảng trong
// Console, không thì quyền theo dòng này bị bỏ qua (xem APPWRITE_SETUP.md).
const Sync = {
  status: 'idle', // 'idle' | 'syncing' | 'error' | 'offline'
  lastSyncedAt: null,

  // "id" không nằm trong phần data — nó đã là rowId (Appwrite tự trả qua $id khi đọc lại).
  // Appwrite không có kiểu cột JSON/mảng lồng nhau như Postgres jsonb — hai trường này (mảng các
  // object nhỏ) đi qua dưới dạng chuỗi JSON, parse lại khi đọc về.
  _JSON_FIELDS: ['repayments', 'contributions'],
  toRemoteData(entity) {
    const out = {};
    Object.keys(entity).forEach(k => {
      if (k === 'id') return;
      let v = entity[k];
      if (Sync._JSON_FIELDS.includes(k)) v = JSON.stringify(v || []);
      out[k] = v === undefined ? null : v;
    });
    return out;
  },
  toLocalEntity(row) {
    const out = { id: row.$id };
    Object.keys(row).forEach(k => {
      if (k.charAt(0) === '$') return;
      let v = row[k];
      if (Sync._JSON_FIELDS.includes(k) && typeof v === 'string') { try { v = JSON.parse(v); } catch (e) { v = []; } }
      out[k] = v;
    });
    return out;
  },
  _rowPermissions(uid) {
    return [Appwrite.Permission.read(Appwrite.Role.user(uid)), Appwrite.Permission.update(Appwrite.Role.user(uid)), Appwrite.Permission.delete(Appwrite.Role.user(uid))];
  },
  _mergeRemoteIntoLocal(key, remoteRows) {
    const arr = Data.state[key];
    const byId = new Map(arr.map(e => [e.id, e]));
    remoteRows.forEach(row => {
      const remote = Sync.toLocalEntity(row);
      const local = byId.get(remote.id);
      if (!local) { arr.push(remote); byId.set(remote.id, remote); return; }
      const remoteTime = new Date(remote.updatedAt || 0).getTime();
      const localTime = new Date(local.updatedAt || 0).getTime();
      if (remoteTime > localTime) Object.assign(local, remote);
    });
  },

  async _push() {
    const uid = Session.currentUser.id;
    const perms = Sync._rowPermissions(uid);
    for (const key of Data.SYNCED_ENTITY_KEYS) {
      const tableId = APPWRITE_CONFIG.tableIds[key];
      const entities = Data.state[key];
      // Appwrite chưa có kiểu "upsert nhiều dòng trong 1 lần gọi" như trước — gửi song song theo lô
      // nhỏ (8 dòng/lô) để nhanh hơn vòng lặp tuần tự nhưng vẫn không dồn quá nhiều request cùng lúc.
      for (let i = 0; i < entities.length; i += 8) {
        const batch = entities.slice(i, i + 8);
        await Promise.all(batch.map(entity => tablesDB.upsertRow({ databaseId: APPWRITE_CONFIG.databaseId, tableId, rowId: entity.id, data: Sync.toRemoteData(entity), permissions: perms })));
      }
    }
    for (const d of Data.state.pendingDeletes.slice()) {
      const tableId = APPWRITE_CONFIG.tableIds[d.table] || d.table;
      try {
        await tablesDB.deleteRow({ databaseId: APPWRITE_CONFIG.databaseId, tableId, rowId: d.id });
        Data.unqueueDelete(d.table, d.id);
      } catch (e) {
        if (e && e.code === 404) Data.unqueueDelete(d.table, d.id); else throw e; // 404 = vốn đã không có trên server, coi như xong
      }
    }
  },
  async _pull() {
    for (const key of Data.SYNCED_ENTITY_KEYS) {
      const tableId = APPWRITE_CONFIG.tableIds[key];
      const allRows = [];
      let cursor = null;
      for (let guard = 0; guard < 500; guard++) { // guard: chỉ để không lặp vô hạn nếu có gì bất thường
        const queries = [Appwrite.Query.limit(100)];
        if (cursor) queries.push(Appwrite.Query.cursorAfter(cursor));
        const res = await tablesDB.listRows({ databaseId: APPWRITE_CONFIG.databaseId, tableId, queries });
        allRows.push(...res.rows);
        if (res.rows.length < 100) break;
        cursor = res.rows[res.rows.length - 1].$id;
      }
      Sync._mergeRemoteIntoLocal(key, allRows);
      const remoteIds = new Set(allRows.map(r => r.$id));
      Data.state[key] = Data.state[key].filter(e => remoteIds.has(e.id));
    }
  },

  async syncNow(opts) {
    const silent = opts && opts.silent;
    if (!Session.isLoggedIn() || !tablesDB || !Data.state) return { ok: false };
    if (!navigator.onLine) { Sync.status = 'offline'; if (!silent) Toast.show('Đang offline — sẽ đồng bộ khi có mạng lại', { type: 'info' }); return { ok: false }; }
    Sync.status = 'syncing';
    UI.renderSyncStatus();
    try {
      await Sync._push();
      await Sync._pull();
      await Data.save();
      Sync.status = 'idle';
      Sync.lastSyncedAt = new Date().toISOString();
      Profile.updateLastSyncedAt(Sync.lastSyncedAt);
      if (!silent) Toast.show('Đã đồng bộ xong', { type: 'ok' });
      UI.renderDashboard();
      UI.renderSyncStatus();
      return { ok: true };
    } catch (e) {
      console.error('Sync.syncNow lỗi:', e);
      Sync.status = 'error';
      UI.renderSyncStatus();
      if (!silent) Toast.show('Đồng bộ thất bại — ' + Auth.translateError(e), { type: 'err' });
      return { ok: false, error: e };
    }
  }
};

// ---------- Ledger: MỘT hàm duy nhất tính số dư — không nơi nào khác được tự tính lại ----------
const Ledger = {
  totals() {
    const walletById = {};
    (Data.state.wallets || []).forEach(w => walletById[w.id] = w);
    // Ví không tồn tại/không rõ -> mặc định TÍNH vào tổng (an toàn, không âm thầm giấu tiền do lỗi dữ liệu).
    const isIncluded = walletId => !walletId || !walletById[walletId] ? true : walletById[walletId].includeInTotal !== false;

    let cash = 0;
    const walletCash = {};
    (Data.state.wallets || []).forEach(w => walletCash[w.id] = 0);
    (Data.state.transactions || []).forEach(t => {
      const sign = (t.type === CONST.TX.INCOME || t.type === CONST.TX.LOAN_IN) ? 1 : -1;
      if (isIncluded(t.walletId)) cash += sign * t.amount;
      if (t.walletId && Object.prototype.hasOwnProperty.call(walletCash, t.walletId)) walletCash[t.walletId] += sign * t.amount;
    });
    (Data.state.transfers || []).forEach(tr => {
      if (Object.prototype.hasOwnProperty.call(walletCash, tr.fromWalletId)) walletCash[tr.fromWalletId] -= tr.amount;
      if (Object.prototype.hasOwnProperty.call(walletCash, tr.toWalletId)) walletCash[tr.toWalletId] += tr.amount;
      const fromInc = isIncluded(tr.fromWalletId), toInc = isIncluded(tr.toWalletId);
      if (fromInc && !toInc) cash -= tr.amount; else if (!fromInc && toInc) cash += tr.amount;
    });
    let lent = 0, owed = 0;
    (Data.state.loans || []).forEach(l => {
      if (l.direction === CONST.LOAN_DIR.LEND) lent += l.remainingAmount; else owed += l.remainingAmount;
    });
    return { cash, lent, owed, netWorth: cash + lent - owed, walletCash };
  },

  recoveredTotal() {
    return (Data.state.loans || []).filter(l => l.direction === CONST.LOAN_DIR.LEND).reduce((s, l) => s + l.paidAmount, 0);
  },
  repaidTotal() {
    return (Data.state.loans || []).filter(l => l.direction === CONST.LOAN_DIR.BORROW).reduce((s, l) => s + l.paidAmount, 0);
  },

  // Thu/chi trong khoảng ngày [from, to] — không tính các giao dịch trả nợ (isRepayment) vào thu/chi.
  incomeExpenseInRange(fromDate, toDate) {
    let income = 0, expense = 0;
    (Data.state.transactions || []).forEach(t => {
      if (t.isRepayment) return;
      const d = Util.parseDate(t.date);
      if (!d || d < fromDate || d > toDate) return;
      if (t.type === CONST.TX.INCOME) income += t.amount;
      else if (t.type === CONST.TX.EXPENSE) expense += t.amount;
    });
    return { income, expense };
  },

  transactionsInRange(fromDate, toDate, opts) {
    opts = opts || {};
    return (Data.state.transactions || []).filter(t => {
      if (t.isRepayment && !opts.includeRepayments) return false;
      const d = Util.parseDate(t.date);
      return d && d >= fromDate && d <= toDate;
    });
  }
};

// ---------- Tx: thu nhập / chi tiêu (khoản vay tạo qua tab Vay mượn để đảm bảo luôn đồng bộ) ----------
const Tx = {
  editingId: null,

  resetForm() {
    Tx.editingId = null;
    document.getElementById('txFormTitle').textContent = 'Thêm giao dịch';
    document.getElementById('txSubmitBtn').textContent = 'Lưu giao dịch';
    document.getElementById('type').value = CONST.TX.EXPENSE;
    document.getElementById('amount').value = '';
    document.getElementById('note').value = '';
    document.getElementById('date').value = Util.todayStr();
    UI.renderWalletSelect('walletId');
    document.getElementById('walletId').value = Data.defaultWalletId();
    UI.renderCategoryChipsForForm();
  },

  selectCategory(categoryId) {
    document.getElementById('category').value = categoryId || '';
    document.querySelectorAll('#txCategoryChips .cat-chip').forEach(chip => {
      chip.classList.toggle('selected', chip.dataset.catId === categoryId);
    });
  },

  startEdit(id) {
    const t = Data.state.transactions.find(x => x.id === id);
    if (!t) return;
    if (t.isRepayment || t.type === CONST.TX.LOAN_OUT || t.type === CONST.TX.LOAN_IN) {
      Toast.show('Giao dịch vay/trả nợ — sửa trong tab Vay mượn nhé', { type: 'info' });
      UI.showTab(2);
      return;
    }
    Tx.editingId = id;
    UI.showTab(1);
    document.getElementById('txFormTitle').textContent = 'Sửa giao dịch';
    document.getElementById('txSubmitBtn').textContent = 'Cập nhật giao dịch';
    document.getElementById('type').value = t.type;
    document.getElementById('amount').value = t.amount;
    document.getElementById('note').value = t.note || '';
    document.getElementById('date').value = t.date;
    UI.renderWalletSelect('walletId');
    document.getElementById('walletId').value = t.walletId || Data.defaultWalletId();
    UI.renderCategoryChipsForForm();
    Tx.selectCategory(t.categoryId);
  },

  async submit() {
    const btn = document.getElementById('txSubmitBtn');
    const type = document.getElementById('type').value;
    const amountCheck = Util.validateAmount(document.getElementById('amount').value);
    const categoryId = document.getElementById('category').value || null;
    const note = Util.sanitizeText(document.getElementById('note').value, CONST.LIMITS.NOTE_MAX);
    const date = document.getElementById('date').value;
    const walletId = document.getElementById('walletId').value || Data.defaultWalletId();

    if (type !== CONST.TX.INCOME && type !== CONST.TX.EXPENSE) { Toast.show('Loại giao dịch không hợp lệ', { type: 'err' }); return; }
    if (!amountCheck.ok) { Toast.show(amountCheck.error, { type: 'err' }); return; }
    if (!Util.isValidDateStr(date)) { Toast.show('Ngày không hợp lệ', { type: 'err' }); return; }
    if (!categoryId) { Toast.show('Chọn một danh mục nhé', { type: 'err' }); return; }

    btn.disabled = true; const original = btn.textContent; btn.textContent = 'Đang lưu...';
    try {
      let record;
      if (Tx.editingId) {
        record = Data.state.transactions.find(x => x.id === Tx.editingId);
        if (!record) { Toast.show('Không tìm thấy giao dịch', { type: 'err' }); return; }
        Object.assign(record, { type, amount: amountCheck.value, categoryId, note, date, walletId });
      } else {
        record = { id: Util.uuid(), type, amount: amountCheck.value, categoryId, note, date, walletId, loanId: null, isRepayment: false, createdAt: new Date().toISOString() };
        Data.state.transactions.push(record);
      }
      const ok = await Data.save();
      if (!ok) {
        if (!Tx.editingId) Data.state.transactions.pop();
        Toast.show('Lưu thất bại — thử lại nhé', { type: 'err' });
        return;
      }
      Toast.show(Tx.editingId ? 'Đã cập nhật giao dịch' : 'Đã lưu giao dịch', { type: 'ok' });
      const wasEditing = !!Tx.editingId;
      Tx.resetForm();
      UI.showTab(0);
      if (wasEditing) History.render();
    } finally {
      btn.disabled = false; btn.textContent = original;
    }
  },

  async remove(id) {
    const idx = Data.state.transactions.findIndex(x => x.id === id);
    if (idx === -1) return;
    const t = Data.state.transactions[idx];
    if (t.loanId) { Toast.show('Giao dịch gắn với khoản vay — xoá trong tab Vay mượn nhé', { type: 'info' }); return; }
    const cat = Data.findCategory(t.categoryId);
    const confirmed = await Modal.confirm({
      title: 'Xoá giao dịch?',
      message: `${Util.formatVND(t.amount)} ngày ${Util.formatDateShort(t.date)}${cat ? ' · ' + cat.name : ''}${t.note ? ' · ' + t.note : ''}`,
      confirmLabel: 'Xoá', danger: true
    });
    if (!confirmed) return;
    Data.state.transactions.splice(idx, 1);
    Data.queueDelete('transactions', t.id);
    const ok = await Data.save();
    if (!ok) { Data.state.transactions.splice(idx, 0, t); Data.unqueueDelete('transactions', t.id); Toast.show('Xoá thất bại — thử lại nhé', { type: 'err' }); return; }
    UI.renderDashboard(); History.render();
    Undo.offer('Đã xoá giao dịch', async () => {
      Data.state.transactions.splice(idx, 0, t);
      Data.unqueueDelete('transactions', t.id);
      await Data.save();
      UI.renderDashboard(); History.render();
    });
  }
};

// ---------- Loans: cho vay & đi vay — MỌI khoản vay đi qua đây để đảm bảo luôn đồng bộ với số dư ----------
const Loans = {
  direction: CONST.LOAN_DIR.LEND,
  editingId: null,

  isOverdue(loan) {
    if (loan.status === CONST.LOAN_STATUS.PAID || !loan.dueDate) return false;
    return Util.parseDate(loan.dueDate) < Util.parseDate(Util.todayStr());
  },
  isDueSoon(loan) {
    if (loan.status === CONST.LOAN_STATUS.PAID || !loan.dueDate) return false;
    const days = Util.daysBetween(Util.todayStr(), loan.dueDate);
    return days >= 0 && days <= 3;
  },
  displayStatus(loan) {
    if (loan.status === CONST.LOAN_STATUS.PAID) return { label: 'Đã trả', cls: 'text-ledger-green bg-ledger-green/10' };
    if (Loans.isOverdue(loan)) return { label: 'Quá hạn', cls: 'text-ledger-red bg-ledger-red/10' };
    if (loan.status === CONST.LOAN_STATUS.PARTIAL) return { label: 'Đang trả', cls: 'text-amber bg-amber/10' };
    return { label: 'Chưa trả', cls: 'text-muted bg-ink/5' };
  },

  _syncDirectionUI() {
    const isLend = Loans.direction === CONST.LOAN_DIR.LEND;
    const lendBtn = document.getElementById('loanDirLendBtn'), borrowBtn = document.getElementById('loanDirBorrowBtn');
    lendBtn.classList.toggle('bg-amber', isLend); lendBtn.classList.toggle('text-cream', isLend); lendBtn.classList.toggle('text-muted', !isLend);
    borrowBtn.classList.toggle('bg-amber', !isLend); borrowBtn.classList.toggle('text-cream', !isLend); borrowBtn.classList.toggle('text-muted', isLend);
    lendBtn.setAttribute('aria-selected', String(isLend)); borrowBtn.setAttribute('aria-selected', String(!isLend));
    document.getElementById('loanTabTitle').textContent = isLend ? 'Ai đang nợ bạn?' : 'Bạn đang nợ ai?';
    document.getElementById('loanName').placeholder = isLend ? 'Tên người vay' : 'Tên người cho bạn vay';
    document.getElementById('loanListLabel').textContent = isLend ? 'Danh sách cho vay' : 'Danh sách đi vay';
  },
  setDirection(dir) {
    Loans.direction = dir;
    Loans._syncDirectionUI();
    Loans.resetForm();
    Loans.render();
  },
  resetForm() {
    Loans.editingId = null;
    document.getElementById('loanName').value = '';
    document.getElementById('loanAmount').value = '';
    document.getElementById('loanDueDate').value = '';
    document.getElementById('loanNote').value = '';
    document.getElementById('loanSubmitBtn').textContent = Loans.direction === CONST.LOAN_DIR.LEND ? '+ Thêm khoản cho vay' : '+ Thêm khoản đi vay';
  },
  startEdit(id) {
    const l = Data.state.loans.find(x => x.id === id);
    if (!l) return;
    if (Loans.direction !== l.direction) { Loans.direction = l.direction; Loans._syncDirectionUI(); Loans.render(); }
    Loans.editingId = id;
    document.getElementById('loanName').value = l.counterpartyName;
    document.getElementById('loanAmount').value = l.principal;
    document.getElementById('loanDueDate').value = l.dueDate || '';
    document.getElementById('loanNote').value = l.note || '';
    document.getElementById('loanSubmitBtn').textContent = 'Cập nhật khoản vay';
    window.scrollTo({ top: 0, behavior: 'smooth' });
  },

  async submit() {
    const btn = document.getElementById('loanSubmitBtn');
    const name = Util.sanitizeText(document.getElementById('loanName').value, CONST.LIMITS.PERSON_NAME_MAX);
    const amountCheck = Util.validateAmount(document.getElementById('loanAmount').value);
    const dueDate = document.getElementById('loanDueDate').value || null;
    const note = Util.sanitizeText(document.getElementById('loanNote').value, CONST.LIMITS.NOTE_MAX);
    if (!name) { Toast.show('Nhập tên người liên quan', { type: 'err' }); return; }
    if (!amountCheck.ok) { Toast.show(amountCheck.error, { type: 'err' }); return; }
    if (dueDate && !Util.isValidDateStr(dueDate)) { Toast.show('Hạn trả không hợp lệ', { type: 'err' }); return; }

    btn.disabled = true; const original = btn.textContent; btn.textContent = 'Đang lưu...';
    try {
      if (Loans.editingId) {
        const l = Data.state.loans.find(x => x.id === Loans.editingId);
        if (!l) { Toast.show('Không tìm thấy khoản vay', { type: 'err' }); return; }
        const linkedTx = l.initialTransactionId ? Data.state.transactions.find(t => t.id === l.initialTransactionId) : null;
        const loanSnap = JSON.parse(JSON.stringify(l));
        const txSnap = linkedTx ? JSON.parse(JSON.stringify(linkedTx)) : null;
        if (amountCheck.value !== l.principal) {
          if (amountCheck.value < l.paidAmount) { Toast.show(`Số gốc không thể nhỏ hơn số đã trả (${Util.formatVND(l.paidAmount)})`, { type: 'err' }); return; }
          l.principal = amountCheck.value;
          l.remainingAmount = l.principal - l.paidAmount;
          if (linkedTx) linkedTx.amount = amountCheck.value;
        }
        l.counterpartyName = name; l.dueDate = dueDate; l.note = note;
        l.status = l.remainingAmount <= 0 ? CONST.LOAN_STATUS.PAID : (l.paidAmount > 0 ? CONST.LOAN_STATUS.PARTIAL : CONST.LOAN_STATUS.UNPAID);
        const ok = await Data.save();
        if (!ok) { Object.assign(l, loanSnap); if (linkedTx && txSnap) Object.assign(linkedTx, txSnap); Toast.show('Lưu thất bại — thử lại nhé', { type: 'err' }); return; }
        Toast.show('Đã cập nhật khoản vay', { type: 'ok' });
      } else {
        const loan = {
          id: Util.uuid(), direction: Loans.direction, counterpartyName: name,
          principal: amountCheck.value, paidAmount: 0, remainingAmount: amountCheck.value,
          loanDate: Util.todayStr(), dueDate, status: CONST.LOAN_STATUS.UNPAID, note,
          initialTransactionId: null, repayments: [], legacyUnlinked: false
        };
        const tx = {
          id: Util.uuid(), type: Loans.direction === CONST.LOAN_DIR.LEND ? CONST.TX.LOAN_OUT : CONST.TX.LOAN_IN, amount: amountCheck.value,
          categoryId: null, note: Loans.direction === CONST.LOAN_DIR.LEND ? `Cho ${name} vay` : `Vay từ ${name}`,
          date: loan.loanDate, walletId: Data.defaultWalletId(), loanId: loan.id, isRepayment: false, createdAt: new Date().toISOString()
        };
        loan.initialTransactionId = tx.id;
        Data.state.transactions.push(tx);
        Data.state.loans.push(loan);
        const ok = await Data.save();
        if (!ok) { Data.state.loans.pop(); Data.state.transactions.pop(); Toast.show('Lưu thất bại — thử lại nhé', { type: 'err' }); return; }
        Toast.show('Đã thêm khoản vay', { type: 'ok' });
      }
      Loans.resetForm();
      UI.renderDashboard();
      Loans.render();
    } finally {
      btn.disabled = false; btn.textContent = original;
    }
  },

  async remove(id) {
    const idx = Data.state.loans.findIndex(x => x.id === id);
    if (idx === -1) return;
    const loan = Data.state.loans[idx];
    const linkedIds = new Set([loan.initialTransactionId, ...loan.repayments.map(r => r.transactionId)].filter(Boolean));
    const confirmed = await Modal.confirm({
      title: 'Xoá khoản vay?',
      message: `${loan.counterpartyName} · ${Util.formatVND(loan.principal)}${linkedIds.size ? ` — sẽ xoá luôn ${linkedIds.size} giao dịch liên quan (kể cả lịch sử trả nợ)` : ''}`,
      confirmLabel: 'Xoá', danger: true
    });
    if (!confirmed) return;
    const removedLoan = Data.state.loans.splice(idx, 1)[0];
    const removedTx = [];
    Data.state.transactions = Data.state.transactions.filter(t => { if (linkedIds.has(t.id)) { removedTx.push(t); return false; } return true; });
    Data.queueDelete('loans', removedLoan.id);
    linkedIds.forEach(txId => Data.queueDelete('transactions', txId));
    const ok = await Data.save();
    if (!ok) {
      Data.state.loans.splice(idx, 0, removedLoan); Data.state.transactions.push(...removedTx);
      Data.unqueueDelete('loans', removedLoan.id); linkedIds.forEach(txId => Data.unqueueDelete('transactions', txId));
      Toast.show('Xoá thất bại — thử lại nhé', { type: 'err' }); return;
    }
    UI.renderDashboard(); Loans.render();
    Undo.offer('Đã xoá khoản vay', async () => {
      Data.state.loans.splice(idx, 0, removedLoan);
      Data.state.transactions.push(...removedTx);
      Data.unqueueDelete('loans', removedLoan.id); linkedIds.forEach(txId => Data.unqueueDelete('transactions', txId));
      await Data.save();
      UI.renderDashboard(); Loans.render();
    });
  },

  async repay(id, rawAmount) {
    const loan = Data.state.loans.find(x => x.id === id);
    if (!loan) return { ok: false, error: 'Không tìm thấy khoản vay' };
    const check = Util.validateAmount(rawAmount);
    if (!check.ok) return { ok: false, error: check.error };
    if (check.value > loan.remainingAmount) return { ok: false, error: `Không thể trả vượt số còn nợ (${Util.formatVND(loan.remainingAmount)})` };

    const tx = {
      id: Util.uuid(), type: loan.direction === CONST.LOAN_DIR.LEND ? CONST.TX.LOAN_IN : CONST.TX.LOAN_OUT, amount: check.value,
      categoryId: null, note: loan.direction === CONST.LOAN_DIR.LEND ? `${loan.counterpartyName} trả nợ` : `Trả nợ ${loan.counterpartyName}`,
      date: Util.todayStr(), walletId: Data.defaultWalletId(), loanId: loan.id, isRepayment: true, createdAt: new Date().toISOString()
    };
    const rec = { id: Util.uuid(), amount: check.value, date: tx.date, transactionId: tx.id, note: '' };
    Data.state.transactions.push(tx);
    loan.repayments.push(rec);
    loan.paidAmount += check.value;
    loan.remainingAmount -= check.value;
    loan.status = loan.remainingAmount <= 0 ? CONST.LOAN_STATUS.PAID : CONST.LOAN_STATUS.PARTIAL;

    const ok = await Data.save();
    if (!ok) {
      Data.state.transactions.pop(); loan.repayments.pop();
      loan.paidAmount -= check.value; loan.remainingAmount += check.value;
      loan.status = loan.remainingAmount <= 0 ? CONST.LOAN_STATUS.PAID : (loan.paidAmount > 0 ? CONST.LOAN_STATUS.PARTIAL : CONST.LOAN_STATUS.UNPAID);
      return { ok: false, error: 'Lưu thất bại — thử lại nhé' };
    }
    UI.renderDashboard(); Loans.render();
    return { ok: true };
  },

  async linkLegacySync(id) {
    const loan = Data.state.loans.find(x => x.id === id);
    if (!loan) return;
    const confirmed = await Modal.confirm({
      title: 'Đồng bộ khoản vay cũ?',
      message: `Sẽ tạo giao dịch ${loan.direction === CONST.LOAN_DIR.LEND ? 'cho vay' : 'đi vay'} ${Util.formatVND(loan.principal)} vào ${Util.formatDateShort(loan.loanDate)}${loan.paidAmount > 0 ? `, và một giao dịch trả nợ ${Util.formatVND(loan.paidAmount)} vào hôm nay để khớp số đã trả` : ''}. Số dư sẽ thay đổi tương ứng — chỉ bấm nếu khoản vay này CHƯA từng được ghi nhận như một giao dịch riêng trước đó.`,
      confirmLabel: 'Đồng bộ'
    });
    if (!confirmed) return;
    const initTx = {
      id: Util.uuid(), type: loan.direction === CONST.LOAN_DIR.LEND ? CONST.TX.LOAN_OUT : CONST.TX.LOAN_IN, amount: loan.principal,
      categoryId: null, note: `Đồng bộ hồi tố — ${loan.direction === CONST.LOAN_DIR.LEND ? 'cho' : 'từ'} ${loan.counterpartyName}`,
      date: loan.loanDate, walletId: Data.defaultWalletId(), loanId: loan.id, isRepayment: false, createdAt: new Date().toISOString()
    };
    Data.state.transactions.push(initTx);
    loan.initialTransactionId = initTx.id;
    if (loan.paidAmount > 0) {
      const repayTx = {
        id: Util.uuid(), type: loan.direction === CONST.LOAN_DIR.LEND ? CONST.TX.LOAN_IN : CONST.TX.LOAN_OUT, amount: loan.paidAmount,
        categoryId: null, note: 'Đồng bộ hồi tố — khớp số đã trả', date: Util.todayStr(), walletId: Data.defaultWalletId(),
        loanId: loan.id, isRepayment: true, createdAt: new Date().toISOString()
      };
      Data.state.transactions.push(repayTx);
      loan.repayments.push({ id: Util.uuid(), amount: loan.paidAmount, date: repayTx.date, transactionId: repayTx.id, note: 'Đồng bộ hồi tố' });
    }
    loan.legacyUnlinked = false;
    const ok = await Data.save();
    if (!ok) { Toast.show('Đồng bộ thất bại — thử lại nhé', { type: 'err' }); return; }
    Toast.show('Đã đồng bộ khoản vay', { type: 'ok' });
    UI.renderDashboard(); Loans.render();
  },

  render() {
    const filter = document.getElementById('loanFilter') ? document.getElementById('loanFilter').value : 'all';
    let list = Data.state.loans.filter(l => l.direction === Loans.direction);
    if (filter === 'active') list = list.filter(l => l.status !== CONST.LOAN_STATUS.PAID);
    else if (filter === 'paid') list = list.filter(l => l.status === CONST.LOAN_STATUS.PAID);
    else if (filter === 'overdue') list = list.filter(l => Loans.isOverdue(l));
    list = list.slice().sort((a, b) => {
      if ((a.status === CONST.LOAN_STATUS.PAID) !== (b.status === CONST.LOAN_STATUS.PAID)) return a.status === CONST.LOAN_STATUS.PAID ? 1 : -1;
      return new Date(b.loanDate) - new Date(a.loanDate);
    });
    const container = document.getElementById('loansList');
    Util.clearChildren(container);
    const totalRemaining = list.reduce((s, l) => s + (l.status !== CONST.LOAN_STATUS.PAID ? l.remainingAmount : 0), 0);
    const recovered = Loans.direction === CONST.LOAN_DIR.LEND ? Ledger.recoveredTotal() : Ledger.repaidTotal();
    container.appendChild(Util.h('div', { class: 'flex justify-between text-[11px] text-muted mb-4 border-b border-dashed border-ink/15 pb-3' },
      Util.h('span', {}, Loans.direction === CONST.LOAN_DIR.LEND ? 'Còn phải thu: ' : 'Còn phải trả: ', Util.h('b', { class: 'text-ink font-mono' }, Util.formatVND(totalRemaining))),
      Util.h('span', {}, Loans.direction === CONST.LOAN_DIR.LEND ? 'Đã thu hồi: ' : 'Đã trả: ', Util.h('b', { class: 'text-ink font-mono' }, Util.formatVND(recovered)))
    ));
    if (!list.length) { container.appendChild(UI.emptyState(Loans.direction === CONST.LOAN_DIR.LEND ? 'Chưa có khoản cho vay nào' : 'Chưa có khoản đi vay nào', 'fa-hand-holding-dollar')); return; }
    list.forEach(loan => container.appendChild(Loans.renderRow(loan)));
  },

  renderRepayHistory(loan) {
    const details = Util.h('details', { class: 'mt-2' });
    details.appendChild(Util.h('summary', { class: 'text-[11px] text-indigo cursor-pointer select-none' }, `Lịch sử trả nợ (${loan.repayments.length})`));
    const list = Util.h('div', { class: 'mt-1.5 space-y-1 pl-2' });
    loan.repayments.slice().reverse().forEach(r => {
      list.appendChild(Util.h('div', { class: 'flex justify-between text-[11px] text-muted' }, Util.h('span', {}, Util.formatDateShort(r.date)), Util.h('span', { class: 'font-mono' }, Util.formatVND(r.amount))));
    });
    details.appendChild(list);
    return details;
  },

  renderRow(loan) {
    const st = Loans.displayStatus(loan);
    const pct = loan.principal > 0 ? Util.clamp(Math.round((loan.paidAmount / loan.principal) * 100), 0, 100) : 0;
    const actions = [];
    if (loan.status !== CONST.LOAN_STATUS.PAID) actions.push(Util.h('button', { class: 'text-xs font-semibold text-indigo px-2 py-1.5', onclick: () => Modal.openRepaySheet(loan.id) }, 'Ghi nhận trả nợ'));
    actions.push(Util.h('button', { class: 'icon-btn text-ink/40', 'aria-label': 'In giấy nợ', title: 'In giấy nợ', onclick: () => Signature.openSigModal(loan.id) }, Util.h('i', { class: 'fas fa-print text-xs', 'aria-hidden': 'true' })));
    actions.push(Util.h('button', { class: 'icon-btn text-ink/40', 'aria-label': 'Sửa khoản vay', title: 'Sửa', onclick: () => Loans.startEdit(loan.id) }, Util.h('i', { class: 'fas fa-pen text-xs', 'aria-hidden': 'true' })));
    actions.push(Util.h('button', { class: 'icon-btn text-ledger-red/60', 'aria-label': 'Xoá khoản vay', title: 'Xoá', onclick: () => Loans.remove(loan.id) }, Util.h('i', { class: 'fas fa-trash text-xs', 'aria-hidden': 'true' })));
    return Util.h('div', { class: 'border-b border-dashed border-ink/15 py-4' },
      Util.h('div', { class: 'flex items-start justify-between gap-2' },
        Util.h('div', {}, Util.h('p', { class: 'font-medium text-ink text-sm' }, loan.counterpartyName),
          Util.h('p', { class: 'text-[11px] text-muted mt-0.5' }, Util.formatDateShort(loan.loanDate), loan.dueDate ? ` · Hạn ${Util.formatDateShort(loan.dueDate)}` : '')),
        Util.h('span', { class: `text-[10px] font-semibold px-2 py-1 rounded-full whitespace-nowrap ${st.cls}` }, st.label)
      ),
      Util.h('div', { class: 'flex items-baseline justify-between mt-2' },
        Util.h('span', { class: 'font-mono text-base text-ink' }, Util.formatVND(loan.remainingAmount)),
        Util.h('span', { class: 'text-[11px] text-muted' }, 'trên ', Util.formatVND(loan.principal))
      ),
      Util.h('div', { class: 'progress-track mt-2' }, Util.h('div', { class: 'progress-fill', style: `width:${pct}%;background:${loan.status === CONST.LOAN_STATUS.PAID ? '#4B7A5E' : '#B8863B'}` })),
      Loans.isDueSoon(loan) ? Util.h('p', { class: 'text-[11px] text-amber mt-1.5 flex items-center gap-1' }, Util.h('i', { class: 'fas fa-clock', 'aria-hidden': 'true' }), 'Sắp đến hạn') : null,
      loan.note ? Util.h('p', { class: 'text-xs text-muted mt-1.5' }, loan.note) : null,
      loan.legacyUnlinked ? Util.h('button', { class: 'text-[11px] text-indigo underline mt-1.5 block', onclick: () => Loans.linkLegacySync(loan.id) }, 'Khoản vay cũ — Đồng bộ giao dịch ngay') : null,
      loan.repayments.length ? Loans.renderRepayHistory(loan) : null,
      Util.h('div', { class: 'flex items-center gap-1 mt-2 -ml-2' }, ...actions)
    );
  }
};

// ---------- Budgets: nhiều ngân sách theo danh mục + tổng, cảnh báo 50/80/100% ----------
const Budgets = {
  editingId: null,

  currentMonthUsage(budget) {
    const now = new Date();
    const from = new Date(now.getFullYear(), now.getMonth(), 1);
    const to = new Date(now.getFullYear(), now.getMonth() + 1, 0, 23, 59, 59);
    let used = 0;
    Data.state.transactions.forEach(t => {
      if (t.type !== CONST.TX.EXPENSE || t.isRepayment) return;
      const d = Util.parseDate(t.date);
      if (!d || d < from || d > to) return;
      if (budget.categoryId === null || t.categoryId === budget.categoryId) used += t.amount;
    });
    return used;
  },
  pctOf(b) { const used = Budgets.currentMonthUsage(b); return b.amount > 0 ? (used / b.amount) * 100 : 0; },

  async upsert(data) {
    const check = Util.validateAmount(data.amount);
    if (!check.ok) return { ok: false, error: check.error };
    if (Budgets.editingId) {
      const b = Data.state.budgets.find(x => x.id === Budgets.editingId);
      if (!b) return { ok: false, error: 'Không tìm thấy ngân sách' };
      const snap = JSON.parse(JSON.stringify(b));
      b.categoryId = data.categoryId || null; b.amount = check.value; b.alertsEnabled = !!data.alertsEnabled;
      const ok = await Data.save();
      if (!ok) { Object.assign(b, snap); return { ok: false, error: 'Lưu thất bại' }; }
    } else {
      const dup = Data.state.budgets.find(b => (b.categoryId || null) === (data.categoryId || null));
      if (dup) return { ok: false, error: 'Danh mục này đã có ngân sách rồi — sửa ngân sách hiện có nhé' };
      Data.state.budgets.push({ id: Util.uuid(), categoryId: data.categoryId || null, amount: check.value, alertsEnabled: !!data.alertsEnabled, createdAt: new Date().toISOString() });
      const ok = await Data.save();
      if (!ok) { Data.state.budgets.pop(); return { ok: false, error: 'Lưu thất bại' }; }
    }
    return { ok: true };
  },
  async remove(id) {
    const idx = Data.state.budgets.findIndex(x => x.id === id);
    if (idx === -1) return;
    const removed = Data.state.budgets.splice(idx, 1)[0];
    Data.queueDelete('budgets', removed.id);
    const ok = await Data.save();
    if (!ok) { Data.state.budgets.splice(idx, 0, removed); Data.unqueueDelete('budgets', removed.id); Toast.show('Xoá thất bại', { type: 'err' }); return; }
    Toast.show('Đã xoá ngân sách', { type: 'ok' });
    UI.renderDashboard(); Budgets.renderFullList();
  },
  alerts() {
    const out = [];
    Data.state.budgets.forEach(b => {
      if (!b.alertsEnabled) return;
      const pct = Budgets.pctOf(b);
      const cat = b.categoryId ? Data.findCategory(b.categoryId) : null;
      const name = cat ? cat.name : 'Tổng ngân sách';
      if (pct >= 100) out.push({ level: 'over', text: `Đã vượt ngân sách "${name}" (${Math.round(pct)}%)`, target: () => UI.showTab(7) });
      else if (pct >= 80) out.push({ level: 'warn', text: `Sắp vượt ngân sách "${name}" (${Math.round(pct)}%)`, target: () => UI.showTab(7) });
    });
    return out;
  },
  renderDashboardBars() {
    const section = document.getElementById('budgetSection');
    const wrap = document.getElementById('budgetBars');
    Util.clearChildren(wrap);
    if (!Data.state.budgets.length) { section.classList.add('hidden'); return; }
    section.classList.remove('hidden');
    Data.state.budgets.slice().sort((a, b) => Budgets.pctOf(b) - Budgets.pctOf(a)).slice(0, 3).forEach(b => wrap.appendChild(Budgets.renderBar(b)));
  },
  renderBar(b) {
    const used = Budgets.currentMonthUsage(b);
    const pct = Util.clamp(Math.round(Budgets.pctOf(b)), 0, 999);
    const cat = b.categoryId ? Data.findCategory(b.categoryId) : null;
    const name = cat ? cat.name : 'Tổng ngân sách';
    const color = pct >= 100 ? '#AE4030' : pct >= 80 ? '#B8863B' : '#4B7A5E';
    return Util.h('div', {},
      Util.h('div', { class: 'flex justify-between text-xs mb-1' }, Util.h('span', { class: 'text-ink' }, name), Util.h('span', { class: 'font-mono text-muted' }, Util.formatVND(used), ' / ', Util.formatVND(b.amount))),
      Util.h('div', { class: 'progress-track' }, Util.h('div', { class: 'progress-fill', style: `width:${Math.min(pct, 100)}%;background:${color}` }))
    );
  },
  renderFullList() {
    const wrap = document.getElementById('budgetFullList');
    Util.clearChildren(wrap);
    if (!Data.state.budgets.length) { wrap.appendChild(UI.emptyState('Chưa có ngân sách nào', 'fa-wallet')); return; }
    Data.state.budgets.forEach(b => {
      const used = Budgets.currentMonthUsage(b);
      const pct = Util.clamp(Math.round(Budgets.pctOf(b)), 0, 999);
      const remaining = b.amount - used;
      const cat = b.categoryId ? Data.findCategory(b.categoryId) : null;
      const name = cat ? cat.name : 'Tổng ngân sách';
      const icon = cat ? cat.icon : '📊';
      const color = pct >= 100 ? '#AE4030' : pct >= 80 ? '#B8863B' : '#4B7A5E';
      wrap.appendChild(Util.h('div', { class: 'border-b border-dashed border-ink/15 pb-4' },
        Util.h('div', { class: 'flex items-center justify-between mb-1' },
          Util.h('span', { class: 'text-sm font-medium text-ink' }, icon + ' ' + name),
          Util.h('div', { class: 'flex gap-1' },
            Util.h('button', { class: 'icon-btn text-ink/40', 'aria-label': 'Sửa ngân sách', onclick: () => Modal.openBudgetEditor(b) }, Util.h('i', { class: 'fas fa-pen text-xs' })),
            Util.h('button', { class: 'icon-btn text-ledger-red/60', 'aria-label': 'Xoá ngân sách', onclick: () => Budgets.remove(b.id) }, Util.h('i', { class: 'fas fa-trash text-xs' })))),
        Util.h('div', { class: 'flex justify-between text-xs text-muted mb-1.5' },
          Util.h('span', {}, 'Đã dùng ', Util.h('b', { class: 'font-mono text-ink' }, Util.formatVND(used)), ` / ${Util.formatVND(b.amount)}`),
          Util.h('span', { class: pct >= 100 ? 'text-ledger-red font-semibold' : '' }, pct + '%')),
        Util.h('div', { class: 'progress-track' }, Util.h('div', { class: 'progress-fill', style: `width:${Math.min(pct, 100)}%;background:${color}` })),
        Util.h('p', { class: 'text-[11px] text-muted mt-1' }, remaining >= 0 ? `Còn lại ${Util.formatVND(remaining)}` : `Vượt ${Util.formatVND(-remaining)}`)
      ));
    });
  }
};

// ---------- Categories: quản lý danh mục (icon, màu), xoá an toàn (gán lại giao dịch đang dùng) ----------
const Categories = {
  editingId: null,
  forType(type) { return Data.state.categories.filter(c => c.type === type); },

  async upsert(data) {
    const name = Util.sanitizeText(data.name, CONST.LIMITS.CATEGORY_NAME_MAX);
    if (!name) return { ok: false, error: 'Nhập tên danh mục' };
    if (data.type !== 'income' && data.type !== 'expense') return { ok: false, error: 'Loại danh mục không hợp lệ' };
    const dup = Data.state.categories.find(c => c.type === data.type && c.name.toLowerCase() === name.toLowerCase() && c.id !== Categories.editingId);
    if (dup) return { ok: false, error: 'Danh mục này đã tồn tại' };
    if (Categories.editingId) {
      const c = Data.state.categories.find(x => x.id === Categories.editingId);
      if (!c) return { ok: false, error: 'Không tìm thấy danh mục' };
      const snap = JSON.parse(JSON.stringify(c));
      Object.assign(c, { name, icon: data.icon || c.icon, color: data.color || c.color });
      const ok = await Data.save();
      if (!ok) { Object.assign(c, snap); return { ok: false, error: 'Lưu thất bại' }; }
    } else {
      Data.state.categories.push({ id: Util.uuid(), name, icon: data.icon || '🏷️', color: data.color || Data.CATEGORY_PALETTE[Data.state.categories.length % Data.CATEGORY_PALETTE.length], type: data.type, isDefault: false });
      const ok = await Data.save();
      if (!ok) { Data.state.categories.pop(); return { ok: false, error: 'Lưu thất bại' }; }
    }
    return { ok: true };
  },

  async remove(id) {
    const cat = Data.findCategory(id);
    if (!cat) return;
    const inUseCount = Data.state.transactions.filter(t => t.categoryId === id).length;
    const confirmed = await Modal.confirm({
      title: 'Xoá danh mục?',
      message: inUseCount ? `"${cat.name}" đang dùng cho ${inUseCount} giao dịch — các giao dịch này sẽ chuyển sang danh mục "Khác".` : `Xoá danh mục "${cat.name}"?`,
      confirmLabel: 'Xoá', danger: true
    });
    if (!confirmed) return;
    const fallback = Data.state.categories.find(c => c.type === cat.type && c.name === 'Khác' && c.id !== id);
    if (inUseCount && fallback) Data.state.transactions.forEach(t => { if (t.categoryId === id) t.categoryId = fallback.id; });
    const removedBudgetIds = Data.state.budgets.filter(b => b.categoryId === id).map(b => b.id);
    Data.state.budgets = Data.state.budgets.filter(b => b.categoryId !== id);
    Data.state.categories.splice(Data.state.categories.findIndex(c => c.id === id), 1);
    Data.queueDelete('categories', id);
    removedBudgetIds.forEach(bId => Data.queueDelete('budgets', bId));
    const ok = await Data.save();
    if (!ok) { Toast.show('Xoá thất bại — thử lại nhé', { type: 'err' }); return; }
    Toast.show('Đã xoá danh mục', { type: 'ok' });
    Categories.render();
  },

  render() {
    const expWrap = document.getElementById('catListExpense'), incWrap = document.getElementById('catListIncome');
    Util.clearChildren(expWrap); Util.clearChildren(incWrap);
    Categories.forType('expense').forEach(c => expWrap.appendChild(Categories.renderRow(c)));
    Categories.forType('income').forEach(c => incWrap.appendChild(Categories.renderRow(c)));
  },
  renderRow(c) {
    return Util.h('div', { class: 'flex items-center justify-between py-2.5 border-b border-dashed border-ink/10' },
      Util.h('span', { class: 'flex items-center gap-2.5 text-sm text-ink' }, Util.h('span', { class: 'cat-dot', style: `background:${c.color}` }), Util.h('span', {}, c.icon + ' ' + c.name)),
      Util.h('div', { class: 'flex gap-1' },
        Util.h('button', { class: 'icon-btn text-ink/40', 'aria-label': `Sửa ${c.name}`, onclick: () => Modal.openCategoryEditor(c) }, Util.h('i', { class: 'fas fa-pen text-xs' })),
        Util.h('button', { class: 'icon-btn text-ledger-red/50', 'aria-label': `Xoá ${c.name}`, onclick: () => Categories.remove(c.id) }, Util.h('i', { class: 'fas fa-trash text-xs' }))));
  }
};

// ---------- Wallets: nhiều ví/tài khoản + chuyển tiền (không tính là thu/chi) ----------
const Wallets = {
  editingId: null,

  async upsert(data) {
    const name = Util.sanitizeText(data.name, CONST.LIMITS.PERSON_NAME_MAX);
    if (!name) return { ok: false, error: 'Nhập tên ví' };
    if (Wallets.editingId) {
      const w = Data.state.wallets.find(x => x.id === Wallets.editingId);
      if (!w) return { ok: false, error: 'Không tìm thấy ví' };
      const snap = JSON.parse(JSON.stringify(w));
      w.name = name; w.icon = data.icon || w.icon; w.includeInTotal = data.includeInTotal !== false;
      if (data.isDefault) Data.state.wallets.forEach(x => x.isDefault = x.id === w.id);
      const ok = await Data.save();
      if (!ok) { Object.assign(w, snap); return { ok: false, error: 'Lưu thất bại' }; }
    } else {
      const wallet = { id: Util.uuid(), name, icon: data.icon || '💼', isDefault: Data.state.wallets.length === 0, includeInTotal: data.includeInTotal !== false };
      Data.state.wallets.push(wallet);
      const ok = await Data.save();
      if (!ok) { Data.state.wallets.pop(); return { ok: false, error: 'Lưu thất bại' }; }
    }
    return { ok: true };
  },

  async remove(id) {
    if (Data.state.wallets.length <= 1) { Toast.show('Cần giữ lại ít nhất một ví', { type: 'err' }); return; }
    const wallet = Data.findWallet(id);
    if (!wallet) return;
    const inUse = Data.state.transactions.some(t => t.walletId === id) || Data.state.transfers.some(t => t.fromWalletId === id || t.toWalletId === id);
    const confirmed = await Modal.confirm({
      title: 'Xoá ví?',
      message: inUse ? `Ví "${wallet.name}" đang có giao dịch — các giao dịch sẽ chuyển sang ví mặc định.` : `Xoá ví "${wallet.name}"?`,
      confirmLabel: 'Xoá', danger: true
    });
    if (!confirmed) return;
    const idx = Data.state.wallets.findIndex(w => w.id === id);
    const removed = Data.state.wallets.splice(idx, 1)[0];
    if (removed.isDefault && Data.state.wallets[0]) Data.state.wallets[0].isDefault = true;
    const defaultId = Data.defaultWalletId();
    Data.state.transactions.forEach(t => { if (t.walletId === id) t.walletId = defaultId; });
    Data.state.transfers.forEach(t => { if (t.fromWalletId === id) t.fromWalletId = defaultId; if (t.toWalletId === id) t.toWalletId = defaultId; });
    Data.queueDelete('wallets', id);
    const ok = await Data.save();
    if (!ok) { Data.state.wallets.splice(idx, 0, removed); Toast.show('Xoá thất bại', { type: 'err' }); return; }
    Toast.show('Đã xoá ví', { type: 'ok' });
    Wallets.render();
  },

  async transfer(data) {
    const check = Util.validateAmount(data.amount);
    if (!check.ok) return { ok: false, error: check.error };
    if (!data.fromWalletId || !data.toWalletId) return { ok: false, error: 'Chọn đủ ví nguồn và ví đích' };
    if (data.fromWalletId === data.toWalletId) return { ok: false, error: 'Ví nguồn và ví đích phải khác nhau' };
    if (!Util.isValidDateStr(data.date)) return { ok: false, error: 'Ngày không hợp lệ' };
    const t = { id: Util.uuid(), fromWalletId: data.fromWalletId, toWalletId: data.toWalletId, amount: check.value, date: data.date, note: Util.sanitizeText(data.note, CONST.LIMITS.NOTE_MAX), createdAt: new Date().toISOString() };
    Data.state.transfers.push(t);
    const ok = await Data.save();
    if (!ok) { Data.state.transfers.pop(); return { ok: false, error: 'Lưu thất bại' }; }
    UI.renderDashboard(); Wallets.render();
    return { ok: true };
  },

  render() {
    const totals = Ledger.totals();
    const wrap = document.getElementById('walletList');
    Util.clearChildren(wrap);
    Data.state.wallets.forEach(w => {
      wrap.appendChild(Util.h('div', { class: 'flex items-center justify-between border-b border-dashed border-ink/15 pb-3' },
        Util.h('div', { class: 'flex items-center gap-2.5' },
          Util.h('span', { class: 'text-lg' }, w.icon),
          Util.h('div', {},
            Util.h('p', { class: 'text-sm text-ink font-medium' }, w.name, w.isDefault ? Util.h('span', { class: 'text-[10px] text-indigo ml-1.5' }, '· mặc định') : null, w.includeInTotal === false ? Util.h('span', { class: 'text-[10px] text-amber ml-1.5' }, '· không tính vào tổng') : null),
            Util.h('p', { class: 'font-mono text-xs text-muted' }, Util.formatVND(totals.walletCash[w.id] || 0)))),
        Util.h('div', { class: 'flex gap-1' },
          Util.h('button', { class: 'icon-btn text-ledger-green', 'aria-label': `Nạp số dư cho ${w.name}`, title: 'Nạp số dư', onclick: () => Modal.openWalletTopUpSheet(w.id) }, Util.h('i', { class: 'fas fa-circle-plus text-sm' })),
          Util.h('button', { class: 'icon-btn text-ink/40', 'aria-label': `Sửa ví ${w.name}`, onclick: () => Modal.openWalletEditor(w) }, Util.h('i', { class: 'fas fa-pen text-xs' })),
          Util.h('button', { class: 'icon-btn text-ledger-red/50', 'aria-label': `Xoá ví ${w.name}`, onclick: () => Wallets.remove(w.id) }, Util.h('i', { class: 'fas fa-trash text-xs' })))
      ));
    });
    Wallets.renderTransferList();
  },
  renderTransferList() {
    const wrap = document.getElementById('transferList');
    Util.clearChildren(wrap);
    const list = Data.state.transfers.slice().sort((a, b) => new Date(b.date) - new Date(a.date)).slice(0, 20);
    if (!list.length) { wrap.appendChild(UI.emptyState('Chưa có lượt chuyển tiền nào', 'fa-right-left')); return; }
    list.forEach(t => {
      const from = Data.findWallet(t.fromWalletId), to = Data.findWallet(t.toWalletId);
      wrap.appendChild(Util.h('div', { class: 'flex items-center justify-between text-xs py-1.5' },
        Util.h('span', { class: 'text-muted' }, `${from ? from.icon + ' ' + from.name : 'Ví đã xoá'} → ${to ? to.icon + ' ' + to.name : 'Ví đã xoá'}`),
        Util.h('span', { class: 'font-mono text-ink' }, Util.formatVND(t.amount))));
    });
  }
};

// ---------- Goals: mục tiêu tiết kiệm ----------
const Goals = {
  editingId: null,
  async upsert(data) {
    const name = Util.sanitizeText(data.name, CONST.LIMITS.PERSON_NAME_MAX);
    const check = Util.validateAmount(data.targetAmount);
    if (!name) return { ok: false, error: 'Nhập tên mục tiêu' };
    if (!check.ok) return { ok: false, error: check.error };
    if (data.dueDate && !Util.isValidDateStr(data.dueDate)) return { ok: false, error: 'Hạn không hợp lệ' };
    if (Goals.editingId) {
      const g = Data.state.goals.find(x => x.id === Goals.editingId);
      if (!g) return { ok: false, error: 'Không tìm thấy mục tiêu' };
      const snap = JSON.parse(JSON.stringify(g));
      g.name = name; g.targetAmount = check.value; g.dueDate = data.dueDate || null; g.icon = data.icon || g.icon;
      const ok = await Data.save();
      if (!ok) { Object.assign(g, snap); return { ok: false, error: 'Lưu thất bại' }; }
    } else {
      Data.state.goals.push({ id: Util.uuid(), name, targetAmount: check.value, currentAmount: 0, dueDate: data.dueDate || null, icon: data.icon || '🎯', createdAt: new Date().toISOString(), contributions: [] });
      const ok = await Data.save();
      if (!ok) { Data.state.goals.pop(); return { ok: false, error: 'Lưu thất bại' }; }
    }
    return { ok: true };
  },
  async remove(id) {
    const idx = Data.state.goals.findIndex(x => x.id === id);
    if (idx === -1) return;
    const g = Data.state.goals[idx];
    const confirmed = await Modal.confirm({ title: 'Xoá mục tiêu?', message: `"${g.name}" — ${Util.formatVND(g.currentAmount)} đã gom sẽ không bị trừ khỏi số dư, chỉ xoá mục tiêu theo dõi thôi.`, confirmLabel: 'Xoá', danger: true });
    if (!confirmed) return;
    const removed = Data.state.goals.splice(idx, 1)[0];
    Data.queueDelete('goals', removed.id);
    const ok = await Data.save();
    if (!ok) { Data.state.goals.splice(idx, 0, removed); Data.unqueueDelete('goals', removed.id); Toast.show('Xoá thất bại', { type: 'err' }); return; }
    Toast.show('Đã xoá mục tiêu', { type: 'ok' });
    Goals.render();
  },
  async addFunds(id, rawAmount) {
    const g = Data.state.goals.find(x => x.id === id);
    if (!g) return { ok: false, error: 'Không tìm thấy mục tiêu' };
    const check = Util.validateAmount(rawAmount);
    if (!check.ok) return { ok: false, error: check.error };
    g.currentAmount += check.value;
    g.contributions.push({ id: Util.uuid(), amount: check.value, date: Util.todayStr() });
    const ok = await Data.save();
    if (!ok) { g.currentAmount -= check.value; g.contributions.pop(); return { ok: false, error: 'Lưu thất bại' }; }
    Goals.render();
    return { ok: true };
  },
  render() {
    const wrap = document.getElementById('goalList');
    Util.clearChildren(wrap);
    if (!Data.state.goals.length) { wrap.appendChild(UI.emptyState('Chưa có mục tiêu tiết kiệm nào', 'fa-bullseye')); return; }
    Data.state.goals.forEach(g => {
      const pct = g.targetAmount > 0 ? Util.clamp(Math.round((g.currentAmount / g.targetAmount) * 100), 0, 100) : 0;
      const remaining = Math.max(0, g.targetAmount - g.currentAmount);
      wrap.appendChild(Util.h('div', { class: 'border-b border-dashed border-ink/15 pb-4' },
        Util.h('div', { class: 'flex items-center justify-between mb-1' },
          Util.h('span', { class: 'text-sm font-medium text-ink' }, g.icon + ' ' + g.name),
          Util.h('div', { class: 'flex gap-1' },
            Util.h('button', { class: 'icon-btn text-ink/40', 'aria-label': 'Sửa mục tiêu', onclick: () => Modal.openGoalEditor(g) }, Util.h('i', { class: 'fas fa-pen text-xs' })),
            Util.h('button', { class: 'icon-btn text-ledger-red/60', 'aria-label': 'Xoá mục tiêu', onclick: () => Goals.remove(g.id) }, Util.h('i', { class: 'fas fa-trash text-xs' })))),
        Util.h('div', { class: 'flex justify-between text-xs text-muted mb-1.5' },
          Util.h('span', {}, Util.h('b', { class: 'font-mono text-ink' }, Util.formatVND(g.currentAmount)), ` / ${Util.formatVND(g.targetAmount)}`),
          Util.h('span', {}, pct + '%')),
        Util.h('div', { class: 'progress-track' }, Util.h('div', { class: 'progress-fill', style: `width:${pct}%;background:#34456B` })),
        Util.h('div', { class: 'flex items-center justify-between mt-2' },
          Util.h('p', { class: 'text-[11px] text-muted' }, remaining > 0 ? `Còn ${Util.formatVND(remaining)}${g.dueDate ? ` · hạn ${Util.formatDateShort(g.dueDate)}` : ''}` : 'Đã đạt mục tiêu 🎉'),
          Util.h('button', { class: 'text-[11px] text-indigo font-semibold', onclick: () => Modal.openGoalAddFunds(g.id) }, '+ Thêm tiền'))
      ));
    });
  }
};

// ---------- Recurring: giao dịch định kỳ — tạo khi đến hạn lúc mở app (không có tiến trình nền) ----------
const Recurring = {
  editingId: null,
  computeNextDate(fromDate, frequency) {
    switch (frequency) {
      case CONST.RECUR_FREQ.DAILY: return Util.addDays(fromDate, 1);
      case CONST.RECUR_FREQ.WEEKLY: return Util.addDays(fromDate, 7);
      case CONST.RECUR_FREQ.MONTHLY: return Util.addMonths(fromDate, 1);
      case CONST.RECUR_FREQ.YEARLY: return Util.addYears(fromDate, 1);
      default: return Util.addMonths(fromDate, 1);
    }
  },
  async upsert(data) {
    const check = Util.validateAmount(data.amount);
    if (!check.ok) return { ok: false, error: check.error };
    if (data.type !== CONST.TX.INCOME && data.type !== CONST.TX.EXPENSE) return { ok: false, error: 'Loại không hợp lệ' };
    if (!Object.values(CONST.RECUR_FREQ).includes(data.frequency)) return { ok: false, error: 'Tần suất không hợp lệ' };
    if (!Util.isValidDateStr(data.startDate)) return { ok: false, error: 'Ngày bắt đầu không hợp lệ' };
    const note = Util.sanitizeText(data.note, CONST.LIMITS.NOTE_MAX);
    if (Recurring.editingId) {
      const r = Data.state.recurring.find(x => x.id === Recurring.editingId);
      if (!r) return { ok: false, error: 'Không tìm thấy' };
      const snap = JSON.parse(JSON.stringify(r));
      Object.assign(r, { type: data.type, amount: check.value, categoryId: data.categoryId || null, note, walletId: data.walletId, frequency: data.frequency });
      const ok = await Data.save();
      if (!ok) { Object.assign(r, snap); return { ok: false, error: 'Lưu thất bại' }; }
    } else {
      const rule = { id: Util.uuid(), type: data.type, amount: check.value, categoryId: data.categoryId || null, note, walletId: data.walletId || Data.defaultWalletId(), frequency: data.frequency, startDate: data.startDate, nextDate: data.startDate, endDate: data.endDate || null, active: true, lastGeneratedAt: null };
      Data.state.recurring.push(rule);
      const ok = await Data.save();
      if (!ok) { Data.state.recurring.pop(); return { ok: false, error: 'Lưu thất bại' }; }
    }
    return { ok: true };
  },
  async remove(id) {
    const idx = Data.state.recurring.findIndex(x => x.id === id);
    if (idx === -1) return;
    const confirmed = await Modal.confirm({ title: 'Xoá giao dịch định kỳ?', message: 'Các giao dịch đã tạo trước đó vẫn được giữ nguyên, chỉ ngừng tạo mới.', confirmLabel: 'Xoá', danger: true });
    if (!confirmed) return;
    const removed = Data.state.recurring.splice(idx, 1)[0];
    Data.queueDelete('recurring', removed.id);
    const ok = await Data.save();
    if (!ok) { Data.state.recurring.splice(idx, 0, removed); Data.unqueueDelete('recurring', removed.id); Toast.show('Xoá thất bại', { type: 'err' }); return; }
    Toast.show('Đã xoá', { type: 'ok' });
    Recurring.render();
  },
  generateOne(rule) {
    const tx = { id: Util.uuid(), type: rule.type, amount: rule.amount, categoryId: rule.categoryId, note: (rule.note ? rule.note + ' ' : '') + '(định kỳ)', date: rule.nextDate, walletId: rule.walletId, loanId: null, isRepayment: false, createdAt: new Date().toISOString() };
    Data.state.transactions.push(tx);
    rule.nextDate = Recurring.computeNextDate(rule.nextDate, rule.frequency);
    rule.lastGeneratedAt = new Date().toISOString();
  },
  async generateDue() {
    const today = Util.todayStr();
    let count = 0, safety = 0;
    Data.state.recurring.forEach(rule => {
      if (!rule.active) return;
      while (rule.nextDate <= today && safety < 500 && !(rule.endDate && rule.nextDate > rule.endDate)) {
        Recurring.generateOne(rule);
        count++; safety++;
      }
    });
    if (count > 0) { await Data.save(); Toast.show(`Đã tự tạo ${count} giao dịch định kỳ đến hạn`, { type: 'info' }); }
    return count;
  },
  async generateNow(id) {
    const rule = Data.state.recurring.find(x => x.id === id);
    if (!rule) return;
    Recurring.generateOne(rule);
    const ok = await Data.save();
    if (!ok) { Toast.show('Lỗi khi tạo giao dịch', { type: 'err' }); return; }
    Toast.show('Đã tạo giao dịch', { type: 'ok' });
    UI.renderDashboard(); Recurring.render();
  },
  freqLabel(f) { return { daily: 'Hàng ngày', weekly: 'Hàng tuần', monthly: 'Hàng tháng', yearly: 'Hàng năm' }[f] || f; },
  render() {
    const wrap = document.getElementById('recurringList');
    Util.clearChildren(wrap);
    if (!Data.state.recurring.length) { wrap.appendChild(UI.emptyState('Chưa có giao dịch định kỳ nào', 'fa-rotate')); return; }
    Data.state.recurring.forEach(r => {
      const cat = r.categoryId ? Data.findCategory(r.categoryId) : null;
      wrap.appendChild(Util.h('div', { class: 'border-b border-dashed border-ink/15 pb-3' },
        Util.h('div', { class: 'flex items-center justify-between' },
          Util.h('div', {}, Util.h('p', { class: 'text-sm font-medium text-ink' }, (cat ? cat.icon + ' ' : '') + (cat ? cat.name : (r.type === CONST.TX.INCOME ? 'Thu nhập' : 'Chi tiêu')) + (r.note ? ' · ' + r.note : '')),
            Util.h('p', { class: 'text-[11px] text-muted mt-0.5' }, Recurring.freqLabel(r.frequency) + ' · Sắp tạo: ' + Util.formatDateShort(r.nextDate))),
          Util.h('span', { class: `font-mono text-sm ${r.type === CONST.TX.INCOME ? 'text-ledger-green' : 'text-ledger-red'}` }, Util.formatVND(r.amount))),
        Util.h('div', { class: 'flex items-center gap-3 mt-2' },
          Util.h('button', { class: 'text-[11px] text-indigo font-semibold', onclick: () => Recurring.generateNow(r.id) }, 'Tạo ngay'),
          Util.h('button', { class: 'text-[11px] text-muted', onclick: () => Modal.openRecurringEditor(r) }, 'Sửa'),
          Util.h('button', { class: 'text-[11px] text-ledger-red/70', onclick: () => Recurring.remove(r.id) }, 'Xoá'))
      ));
    });
  }
};

// ---------- Stats: thống kê theo kỳ (7 ngày / 30 ngày / 3-6-12 tháng / tuỳ chỉnh) ----------
const Stats = {
  period: { kind: '30d' },
  kinds: [
    { key: '7d', label: '7 ngày' }, { key: '30d', label: '30 ngày' }, { key: '3m', label: '3 tháng' },
    { key: '6m', label: '6 tháng' }, { key: '12m', label: '12 tháng' }, { key: 'custom', label: 'Tuỳ chỉnh' }
  ],

  range() {
    const today = new Date(); today.setHours(23, 59, 59, 999);
    let from = new Date(today);
    switch (Stats.period.kind) {
      case '7d': from.setDate(from.getDate() - 6); from.setHours(0, 0, 0, 0); return { from, to: today };
      case '3m': from.setMonth(from.getMonth() - 3); from.setHours(0, 0, 0, 0); return { from, to: today };
      case '6m': from.setMonth(from.getMonth() - 6); from.setHours(0, 0, 0, 0); return { from, to: today };
      case '12m': from.setFullYear(from.getFullYear() - 1); from.setHours(0, 0, 0, 0); return { from, to: today };
      case 'custom': {
        const f = Util.parseDate(Stats.period.from) || from;
        const t = Util.parseDate(Stats.period.to) || today;
        t.setHours(23, 59, 59, 999);
        return { from: f, to: t };
      }
      default: from.setDate(from.getDate() - 29); from.setHours(0, 0, 0, 0); return { from, to: today };
    }
  },

  setPeriod(kind) {
    Stats.period = { kind };
    document.getElementById('statsCustomRange').classList.toggle('hidden', kind !== 'custom');
    Stats.renderPeriodChips();
    if (kind !== 'custom') Stats.render();
  },
  applyCustomRange() {
    const from = document.getElementById('statsFrom').value, to = document.getElementById('statsTo').value;
    if (!Util.isValidDateStr(from) || !Util.isValidDateStr(to)) { Toast.show('Chọn đủ khoảng ngày hợp lệ', { type: 'err' }); return; }
    if (from > to) { Toast.show('Ngày bắt đầu phải trước ngày kết thúc', { type: 'err' }); return; }
    Stats.period = { kind: 'custom', from, to };
    Stats.render();
  },
  renderPeriodChips() {
    const wrap = document.getElementById('statsPeriodChips');
    Util.clearChildren(wrap);
    Stats.kinds.forEach(k => {
      const active = Stats.period.kind === k.key;
      wrap.appendChild(Util.h('button', { class: `shrink-0 px-3.5 py-2 rounded-full text-xs font-semibold border ${active ? 'bg-indigo text-cream border-indigo' : 'border-ink/15 text-muted'}`, onclick: () => Stats.setPeriod(k.key) }, k.label));
    });
  },

  smartStats(txs, range) {
    const expenses = txs.filter(t => t.type === CONST.TX.EXPENSE);
    const totalExpense = expenses.reduce((s, t) => s + t.amount, 0);
    const numDays = Math.max(1, Math.round((range.to - range.from) / 86400000) + 1);
    const numMonths = Math.max(1, numDays / 30.4);
    const byDay = {};
    expenses.forEach(t => { byDay[t.date] = (byDay[t.date] || 0) + t.amount; });
    let biggestDay = null, biggestDayAmt = 0;
    Object.entries(byDay).forEach(([d, amt]) => { if (amt > biggestDayAmt) { biggestDayAmt = amt; biggestDay = d; } });
    const byCat = {};
    expenses.forEach(t => { byCat[t.categoryId] = (byCat[t.categoryId] || 0) + t.amount; });
    let topCatId = null, topCatAmt = 0;
    Object.entries(byCat).forEach(([id, amt]) => { if (amt > topCatAmt) { topCatAmt = amt; topCatId = id; } });
    let biggestTx = null;
    expenses.forEach(t => { if (!biggestTx || t.amount > biggestTx.amount) biggestTx = t; });
    return { avgPerDay: totalExpense / numDays, avgPerMonth: totalExpense / numMonths, topCategory: topCatId ? Data.findCategory(topCatId) : null, topCategoryAmount: topCatAmt, biggestDay, biggestDayAmount: biggestDayAmt, biggestTx };
  },

  render() {
    const { from, to } = Stats.range();
    const txs = Ledger.transactionsInRange(from, to);
    const { income, expense } = Ledger.incomeExpenseInRange(from, to);
    const saved = income - expense;
    const rate = income > 0 ? Math.round((saved / income) * 100) : 0;
    document.getElementById('statsIncomeTotal').textContent = Util.formatVND(income);
    document.getElementById('statsExpenseTotal').textContent = Util.formatVND(expense);
    document.getElementById('statsSaved').textContent = Util.formatVND(saved);
    document.getElementById('statsSavedRate').textContent = rate + '%';
    Stats.renderComparison(from, to);
    Stats.renderCategoryBreakdown(txs.filter(t => t.type === CONST.TX.EXPENSE));
    Stats.renderSmartStats(txs, { from, to });
    Charts.renderBar(from, to);
    Charts.renderFlow(from, to);
  },

  renderComparison(from, to) {
    const spanMs = to - from;
    const prevTo = new Date(from.getTime() - 86400000);
    const prevFrom = new Date(prevTo.getTime() - spanMs);
    const cur = Ledger.incomeExpenseInRange(from, to);
    const prev = Ledger.incomeExpenseInRange(prevFrom, prevTo);
    const wrap = document.getElementById('statsComparison');
    Util.clearChildren(wrap);
    const diffExpense = cur.expense - prev.expense;
    const pctExpense = prev.expense > 0 ? Math.round((diffExpense / prev.expense) * 100) : (cur.expense > 0 ? 100 : 0);
    const diffIncome = cur.income - prev.income;
    const pctIncome = prev.income > 0 ? Math.round((diffIncome / prev.income) * 100) : (cur.income > 0 ? 100 : 0);
    wrap.appendChild(Util.h('div', { class: 'grid grid-cols-2 gap-3' },
      Util.h('div', { class: 'border border-dashed border-ink/15 rounded-lg p-3' }, Util.h('p', { class: 'text-[10px] text-muted uppercase tracking-widest mb-1' }, 'Chi tiêu'), Util.h('p', { class: `text-xs font-semibold ${diffExpense > 0 ? 'text-ledger-red' : 'text-ledger-green'}` }, (diffExpense >= 0 ? '+' : '') + pctExpense + '% so với kỳ trước')),
      Util.h('div', { class: 'border border-dashed border-ink/15 rounded-lg p-3' }, Util.h('p', { class: 'text-[10px] text-muted uppercase tracking-widest mb-1' }, 'Thu nhập'), Util.h('p', { class: `text-xs font-semibold ${diffIncome >= 0 ? 'text-ledger-green' : 'text-ledger-red'}` }, (diffIncome >= 0 ? '+' : '') + pctIncome + '% so với kỳ trước'))
    ));
  },

  renderCategoryBreakdown(expenseTxs) {
    const wrap = document.getElementById('categoryBreakdown');
    Util.clearChildren(wrap);
    const totals = {};
    expenseTxs.forEach(t => { totals[t.categoryId] = (totals[t.categoryId] || 0) + t.amount; });
    const total = Object.values(totals).reduce((a, b) => a + b, 0);
    const entries = Object.entries(totals).sort((a, b) => b[1] - a[1]);
    if (!entries.length) { wrap.appendChild(UI.emptyState('Chưa có dữ liệu chi tiêu trong kỳ này', 'fa-chart-pie')); return; }
    entries.forEach(([catId, amt]) => {
      const cat = Data.findCategory(catId);
      const pct = total > 0 ? Math.round((amt / total) * 100) : 0;
      wrap.appendChild(Util.h('div', { class: 'mb-3' },
        Util.h('div', { class: 'flex justify-between text-xs mb-1' }, Util.h('span', { class: 'text-ink flex items-center gap-1.5' }, Util.h('span', { class: 'cat-dot', style: `background:${cat ? cat.color : '#8B8175'}` }), cat ? cat.icon + ' ' + cat.name : 'Không rõ'), Util.h('span', { class: 'font-mono text-muted' }, Util.formatVND(amt), ` (${pct}%)`)),
        Util.h('div', { class: 'progress-track' }, Util.h('div', { class: 'progress-fill', style: `width:${pct}%;background:${cat ? cat.color : '#8B8175'}` }))));
    });
  },

  renderSmartStats(txs, range) {
    const s = Stats.smartStats(txs, range);
    const wrap = document.getElementById('smartStats');
    Util.clearChildren(wrap);
    [
      ['Chi trung bình / ngày', Util.formatVND(Math.round(s.avgPerDay))],
      ['Chi trung bình / tháng', Util.formatVND(Math.round(s.avgPerMonth))],
      ['Danh mục chi nhiều nhất', s.topCategory ? `${s.topCategory.icon} ${s.topCategory.name} — ${Util.formatVND(s.topCategoryAmount)}` : '—'],
      ['Ngày chi nhiều nhất', s.biggestDay ? `${Util.formatDateShort(s.biggestDay)} — ${Util.formatVND(s.biggestDayAmount)}` : '—'],
      ['Giao dịch lớn nhất', s.biggestTx ? `${Util.formatVND(s.biggestTx.amount)} · ${Util.formatDateShort(s.biggestTx.date)}` : '—']
    ].forEach(([label, val]) => wrap.appendChild(Util.h('div', { class: 'flex justify-between text-xs border-b border-dashed border-ink/10 pb-2.5' }, Util.h('span', { class: 'text-muted' }, label), Util.h('span', { class: 'text-ink font-medium text-right ml-3' }, val))));
  }
};

// ---------- Charts: bọc Chart.js, tự huỷ trước khi vẽ lại để tránh rò rỉ bộ nhớ, tự ẩn nếu CDN lỗi ----------
const Charts = {
  pie: null, bar: null, flow: null,
  available() { return typeof Chart !== 'undefined' && !window.__chartLoadFailed; },
  showFallback(canvas) {
    if (!canvas) return;
    canvas.style.display = 'none';
    if (!canvas.nextElementSibling || !canvas.nextElementSibling.classList || !canvas.nextElementSibling.classList.contains('chart-fallback-msg')) {
      canvas.parentNode.insertBefore(Util.h('p', { class: 'chart-fallback-msg text-xs text-muted text-center py-6' }, 'Không tải được biểu đồ (cần mạng cho lần đầu) — số liệu chi tiết vẫn đầy đủ bên dưới.'), canvas.nextSibling);
    }
  },
  compactVND(v) {
    const abs = Math.abs(v);
    if (abs >= 1000000) return (v / 1000000).toFixed(1).replace('.0', '') + 'tr';
    if (abs >= 1000) return Math.round(v / 1000) + 'k';
    return String(v);
  },

  renderPie(income, expense) {
    const canvas = document.getElementById('pieChart');
    if (!Charts.available()) { Charts.showFallback(canvas); return; }
    if (Charts.pie) Charts.pie.destroy();
    if (income === 0 && expense === 0) { canvas.style.display = 'none'; return; }
    canvas.style.display = '';
    Charts.pie = new Chart(canvas.getContext('2d'), { type: 'doughnut', data: { labels: ['Thu', 'Chi'], datasets: [{ data: [income, expense], backgroundColor: ['#4B7A5E', '#AE4030'], borderWidth: 0 }] }, options: { cutout: '68%', plugins: { legend: { display: false } } } });
  },

  renderBar(from, to) {
    const canvas = document.getElementById('barChart');
    if (!Charts.available()) { Charts.showFallback(canvas); return; }
    if (Charts.bar) Charts.bar.destroy();
    const buckets = Charts.buildMonthlyBuckets(from, to);
    Charts.bar = new Chart(canvas.getContext('2d'), {
      type: 'bar', data: { labels: buckets.map(b => b.label), datasets: [{ label: 'Thu', data: buckets.map(b => b.income), backgroundColor: '#4B7A5E' }, { label: 'Chi', data: buckets.map(b => b.expense), backgroundColor: '#AE4030' }] },
      options: { plugins: { legend: { display: true, labels: { boxWidth: 10, font: { size: 10 } } } }, scales: { x: { grid: { display: false } }, y: { ticks: { callback: v => Charts.compactVND(v) } } } }
    });
  },

  renderFlow(from, to) {
    const canvas = document.getElementById('flowChart');
    if (!Charts.available()) { Charts.showFallback(canvas); return; }
    if (Charts.flow) Charts.flow.destroy();
    const days = Charts.buildDailyBuckets(from, to);
    Charts.flow = new Chart(canvas.getContext('2d'), {
      type: 'line', data: { labels: days.map(d => d.label), datasets: [{ label: 'Dòng tiền ròng', data: days.map(d => d.net), borderColor: '#34456B', backgroundColor: 'rgba(52,69,107,0.1)', fill: true, tension: 0.3, pointRadius: 0 }] },
      options: { plugins: { legend: { display: false } }, scales: { x: { grid: { display: false }, ticks: { maxTicksLimit: 6 } }, y: { ticks: { callback: v => Charts.compactVND(v) } } } }
    });
  },

  buildMonthlyBuckets(from, to) {
    const buckets = []; let cursor = new Date(from.getFullYear(), from.getMonth(), 1);
    const end = new Date(to.getFullYear(), to.getMonth(), 1); let guard = 0;
    while (cursor <= end && guard < 36) {
      const monthStart = new Date(cursor.getFullYear(), cursor.getMonth(), 1);
      const monthEnd = new Date(cursor.getFullYear(), cursor.getMonth() + 1, 0, 23, 59, 59);
      const { income, expense } = Ledger.incomeExpenseInRange(monthStart < from ? from : monthStart, monthEnd > to ? to : monthEnd);
      buckets.push({ label: `T${cursor.getMonth() + 1}/${String(cursor.getFullYear()).slice(2)}`, income, expense });
      cursor.setMonth(cursor.getMonth() + 1); guard++;
    }
    return buckets;
  },
  buildDailyBuckets(from, to) {
    const days = []; let cursor = new Date(from); let guard = 0;
    const totalDays = Math.round((to - from) / 86400000) + 1;
    const step = Math.max(1, Math.ceil(totalDays / 60));
    while (cursor <= to && guard < 400) {
      const stepEnd = new Date(cursor); stepEnd.setDate(stepEnd.getDate() + step - 1); stepEnd.setHours(23, 59, 59, 999);
      const { income, expense } = Ledger.incomeExpenseInRange(cursor, stepEnd > to ? to : stepEnd);
      days.push({ label: `${cursor.getDate()}/${cursor.getMonth() + 1}`, net: income - expense });
      cursor.setDate(cursor.getDate() + step); guard++;
    }
    return days;
  }
};

// ---------- UI: điều hướng tab + vẽ Dashboard + các mảnh dùng chung (an toàn XSS) ----------
const UI = {
  currentTab: 0,
  balanceHidden: true,
  mainTabCount: 5,

  // ---------- Tài khoản (Cài đặt > Tài khoản) — chỉ có nội dung khi đã cấu hình Appwrite ----------
  async renderAccountSection() {
    const wrap = document.getElementById('accountSection');
    if (!wrap) return;
    Util.clearChildren(wrap);
    if (!APPWRITE_CONFIG.isConfigured()) {
      wrap.appendChild(Util.h('p', { class: 'text-[11px] text-muted/70 italic pb-4 border-b border-dashed border-ink/15 mb-2' }, 'Đăng nhập & đồng bộ đám mây chưa được cấu hình (xem APPWRITE_CONFIG đầu file mã nguồn).'));
      return;
    }
    if (!Session.isLoggedIn()) return;
    if (!Profile.current) await Profile.fetch();
    const name = (Profile.current && Profile.current.displayName) || Session.currentUser.email;
    wrap.appendChild(Util.h('p', { class: 'text-[10px] tracking-widest text-muted/70 uppercase mt-2 mb-1' }, 'Tài khoản'));
    wrap.appendChild(Util.h('div', { class: 'flex items-center gap-3 py-3 border-b border-dashed border-ink/20' },
      Util.h('div', { class: 'w-11 h-11 rounded-full bg-indigo text-cream flex items-center justify-center font-serif text-lg flex-shrink-0' }, Profile.initials()),
      Util.h('div', { class: 'min-w-0' },
        Util.h('p', { class: 'text-sm font-medium text-ink truncate' }, name),
        Util.h('p', { class: 'text-[11px] text-muted truncate' }, Session.currentUser.email),
        Util.h('p', { id: 'syncStatusLabel', class: 'text-[10px] text-muted/70 mt-0.5' }, UI._syncStatusText()))
    ));
    const row = (icon, label, onClick) => Util.h('button', { onclick: onClick, class: 'w-full flex items-center justify-between py-4 border-b border-dashed border-ink/20 text-ink' },
      Util.h('span', { class: 'flex items-center gap-3 text-sm font-medium' }, Util.h('i', { class: `fas ${icon} text-indigo text-sm w-4`, 'aria-hidden': 'true' }), label),
      Util.h('i', { class: 'fas fa-chevron-right text-ink/20 text-xs', 'aria-hidden': 'true' }));
    wrap.appendChild(row('fa-pen', 'Chỉnh sửa hồ sơ', () => Modal.openEditProfileSheet()));
    wrap.appendChild(row('fa-key', 'Đổi mật khẩu', () => Modal.openChangePasswordSheet()));
    wrap.appendChild(row('fa-rotate', 'Đồng bộ ngay', () => Sync.syncNow()));
    wrap.appendChild(Util.h('button', { onclick: () => Session.logout(), class: 'w-full flex items-center gap-3 py-4 text-ledger-red text-sm font-medium' },
      Util.h('i', { class: 'fas fa-arrow-right-from-bracket text-sm w-4', 'aria-hidden': 'true' }), 'Đăng xuất'));
  },
  _syncStatusText() {
    const map = { idle: 'Đã đồng bộ', syncing: 'Đang đồng bộ...', error: 'Đồng bộ lỗi — thử lại sau', offline: 'Đang offline' };
    let text = map[Sync.status] || '';
    const ts = (Profile.current && Profile.current.lastSyncedAt) || Sync.lastSyncedAt;
    if (Sync.status === 'idle' && ts) text += ' · ' + Util.formatDateShort(ts.slice(0, 10));
    return text;
  },
  renderSyncStatus() {
    const el = document.getElementById('syncStatusLabel');
    if (el) el.textContent = UI._syncStatusText();
  },
  applyProfileNameToHeader() {
    const el = document.getElementById('appUserName');
    if (!el) return;
    el.textContent = (Profile.current && Profile.current.displayName) || (Session.currentUser && Session.currentUser.email) || 'Tuấn';
  },

  sortByRecency(list) {
    return list.slice().sort((a, b) => {
      if (a.date !== b.date) return a.date < b.date ? 1 : -1;
      return (a.createdAt || '') < (b.createdAt || '') ? 1 : -1;
    });
  },

  emptyState(text, iconClass) {
    return Util.h('div', { class: 'text-center py-10 text-muted' },
      Util.h('i', { class: `fas ${iconClass} text-2xl mb-3 opacity-30`, 'aria-hidden': 'true' }),
      Util.h('p', { class: 'text-sm italic' }, text));
  },
  skeleton(container, n) {
    Util.clearChildren(container);
    for (let i = 0; i < n; i++) container.appendChild(Util.h('div', { class: 'skel h-12 rounded-lg mb-2' }));
  },

  showTab(n, opt) {
    document.querySelectorAll('.tab-content').forEach(el => el.classList.add('hidden'));
    const target = document.getElementById('tab-' + n);
    if (target) target.classList.remove('hidden');
    if (n < UI.mainTabCount) {
      document.querySelectorAll('.nav-tab').forEach((btn, i) => { btn.classList.toggle('active', i === n); btn.setAttribute('aria-selected', String(i === n)); });
      UI.currentTab = n;
    }
    const scroller = target || document.body;
    scroller.scrollTop = 0; window.scrollTo({ top: 0 });

    if (n === 0) UI.renderDashboard();
    else if (n === 1) { if (!Tx.editingId) Tx.resetForm(); }
    else if (n === 2) { if (opt) Loans.setDirection(opt); else Loans.render(); }
    else if (n === 3) { if (!Stats._chipsRendered) { Stats.renderPeriodChips(); Stats._chipsRendered = true; } Stats.render(); }
    else if (n === 4) UI.renderDataStatus();
    else if (n === 5) History.render();
    else if (n === 6) Categories.render();
    else if (n === 7) Budgets.renderFullList();
    else if (n === 8) Wallets.render();
    else if (n === 9) Goals.render();
    else if (n === 10) Recurring.render();
    else if (n === 11) Report.init();
  },

  toggleBalanceVisibility() {
    UI.balanceHidden = !UI.balanceHidden;
    const btn = document.getElementById('balanceToggleBtn');
    Util.clearChildren(btn);
    btn.appendChild(Util.h('i', { class: `fas ${UI.balanceHidden ? 'fa-eye' : 'fa-eye-slash'} text-xs`, 'aria-hidden': 'true' }));
    UI.renderDashboard();
  },

  renderDashboard() {
    const totals = Ledger.totals();
    const now = new Date();
    const monthStart = new Date(now.getFullYear(), now.getMonth(), 1);
    const monthEnd = new Date(now.getFullYear(), now.getMonth() + 1, 0, 23, 59, 59);
    const { income, expense } = Ledger.incomeExpenseInRange(monthStart, monthEnd);
    const saved = income - expense;
    const rate = income > 0 ? Math.round((saved / income) * 100) : 0;
    const mask = s => UI.balanceHidden ? '•••••• ₫' : s;

    document.getElementById('balance').textContent = mask(Util.formatVND(totals.cash));
    document.getElementById('dashLent').textContent = mask(Util.formatVND(totals.lent));
    document.getElementById('dashOwed').textContent = mask(Util.formatVND(totals.owed));
    document.getElementById('dashNetWorth').textContent = mask(Util.formatVND(totals.netWorth));
    document.getElementById('incomeMonth').textContent = mask(Util.formatVND(income));
    document.getElementById('expenseMonth').textContent = mask(Util.formatVND(expense));
    const savedEl = document.getElementById('savedMonth');
    Util.clearChildren(savedEl);
    savedEl.appendChild(document.createTextNode(mask(Util.formatVND(saved)) + ' '));
    savedEl.appendChild(Util.h('span', { class: 'text-muted' }, income > 0 && !UI.balanceHidden ? `(${rate}%)` : ''));
    document.getElementById('todayLabel').textContent = 'Cập nhật hôm nay • ' + Util.formatDateShort(Util.todayStr());

    Budgets.renderDashboardBars();
    UI.renderAlerts();
    Charts.renderPie(income, expense);
    UI.renderRecentList();
  },

  renderAlerts() {
    const wrap = document.getElementById('dashAlerts');
    Util.clearChildren(wrap);
    const items = [];
    Budgets.alerts().forEach(a => items.push({ text: a.text, cls: a.level === 'over' ? 'bg-ledger-red/10 text-ledger-red' : 'bg-amber/10 text-amber', icon: 'fa-wallet', onClick: a.target }));
    const walletCash = Ledger.totals().walletCash;
    Data.state.wallets.forEach(w => {
      const bal = walletCash[w.id] || 0;
      if (bal < CONST.LOW_BALANCE_THRESHOLD) items.push({ text: `Ví "${w.name}" sắp cạn (còn ${Util.formatVND(bal)})`, cls: 'bg-amber/10 text-amber', icon: 'fa-wallet', onClick: () => UI.showTab(8) });
    });
    Data.state.loans.forEach(l => {
      if (l.status === CONST.LOAN_STATUS.PAID) return;
      const dirLabel = l.direction === CONST.LOAN_DIR.LEND ? 'cho vay' : 'đi vay';
      if (Loans.isOverdue(l)) items.push({ text: `Khoản ${dirLabel} của ${l.counterpartyName} đã quá hạn`, cls: 'bg-ledger-red/10 text-ledger-red', icon: 'fa-triangle-exclamation', onClick: () => UI.showTab(2, l.direction) });
      else if (Loans.isDueSoon(l)) items.push({ text: `Khoản ${dirLabel} của ${l.counterpartyName} sắp đến hạn`, cls: 'bg-amber/10 text-amber', icon: 'fa-clock', onClick: () => UI.showTab(2, l.direction) });
    });
    items.forEach(it => wrap.appendChild(Util.h('button', { class: `w-full flex items-center gap-2.5 text-left text-xs px-3.5 py-2.5 rounded-lg ${it.cls}`, onclick: it.onClick || (() => {}) }, Util.h('i', { class: `fas ${it.icon}`, 'aria-hidden': 'true' }), Util.h('span', {}, it.text))));
  },

  renderTransactionRow(t) {
    const cat = t.categoryId ? Data.findCategory(t.categoryId) : null;
    const isIncome = t.type === CONST.TX.INCOME || t.type === CONST.TX.LOAN_IN;
    const linkedLoan = t.loanId ? Data.state.loans.find(l => l.id === t.loanId) : null;
    const icon = t.loanId ? (t.isRepayment ? '↩️' : (t.type === CONST.TX.LOAN_OUT ? '📤' : '📥')) : (cat ? cat.icon : (isIncome ? '💵' : '💸'));
    const label = linkedLoan ? linkedLoan.counterpartyName : (cat ? cat.name : (isIncome ? 'Thu nhập' : 'Chi tiêu'));
    const sub = linkedLoan ? (t.isRepayment ? 'Trả nợ' : (linkedLoan.direction === CONST.LOAN_DIR.LEND ? 'Cho vay' : 'Đi vay')) : (t.note || '');
    return Util.h('div', { class: 'flex items-center justify-between py-3 border-b border-dashed border-ink/10' },
      Util.h('div', { class: 'flex items-center gap-3 min-w-0' },
        Util.h('span', { class: 'text-lg shrink-0' }, icon),
        Util.h('div', { class: 'min-w-0' },
          Util.h('p', { class: 'text-sm text-ink truncate' }, label),
          Util.h('p', { class: 'text-[11px] text-muted truncate' }, Util.formatDateShort(t.date), sub ? ' · ' + sub : ''))),
      Util.h('div', { class: 'flex items-center gap-1.5 shrink-0' },
        Util.h('span', { class: `font-mono text-sm ${isIncome ? 'text-ledger-green' : 'text-ledger-red'}` }, (isIncome ? '+' : '−') + Util.formatVND(t.amount)),
        !t.loanId ? Util.h('button', { class: 'icon-btn text-ink/25', 'aria-label': 'Ảnh hoá đơn', title: 'Ảnh hoá đơn', onclick: () => Modal.openReceiptSheet(t.id) }, Util.h('i', { class: 'fas fa-camera text-[10px]', 'aria-hidden': 'true' })) : null,
        !t.loanId ? Util.h('button', { class: 'icon-btn text-ink/25', 'aria-label': 'Sửa giao dịch', onclick: () => Tx.startEdit(t.id) }, Util.h('i', { class: 'fas fa-pen text-[10px]', 'aria-hidden': 'true' })) : null,
        !t.loanId ? Util.h('button', { class: 'icon-btn text-ink/25', 'aria-label': 'Xoá giao dịch', onclick: () => Tx.remove(t.id) }, Util.h('i', { class: 'fas fa-trash text-[10px]', 'aria-hidden': 'true' })) : null)
    );
  },
  renderRecentList() {
    const wrap = document.getElementById('recentList');
    Util.clearChildren(wrap);
    const list = UI.sortByRecency(Data.state.transactions).slice(0, 5);
    if (!list.length) { wrap.appendChild(UI.emptyState('Chưa có giao dịch nào — thêm giao dịch đầu tiên nhé', 'fa-receipt')); return; }
    list.forEach(t => wrap.appendChild(UI.renderTransactionRow(t)));
  },

  renderCategoryChipsForForm() {
    const type = document.getElementById('type').value;
    const wrap = document.getElementById('txCategoryChips');
    Util.clearChildren(wrap);
    Categories.forType(type).forEach(c => wrap.appendChild(Util.h('button', { type: 'button', class: 'cat-chip', dataset: { catId: c.id }, onclick: () => Tx.selectCategory(c.id) }, Util.h('span', { class: 'cat-dot', style: `background:${c.color}` }), c.icon + ' ' + c.name)));
  },

  renderWalletSelect(selectId) {
    const sel = document.getElementById(selectId);
    const prev = sel.value;
    Util.clearChildren(sel);
    Data.state.wallets.forEach(w => { const opt = document.createElement('option'); opt.value = w.id; opt.textContent = w.icon + ' ' + w.name; sel.appendChild(opt); });
    if (prev) sel.value = prev;
  },

  dataCounts() {
    return {
      'giao dịch': Data.state.transactions.length, 'khoản vay': Data.state.loans.length, 'ngân sách': Data.state.budgets.length,
      'danh mục': Data.state.categories.length, 'ví': Data.state.wallets.length, 'lượt chuyển tiền': Data.state.transfers.length,
      'mục tiêu': Data.state.goals.length, 'định kỳ': Data.state.recurring.length
    };
  },
  renderDataStatus() {
    const el = document.getElementById('dataStatusLabel');
    if (!el) return;
    const c = UI.dataCounts();
    el.textContent = `Đang lưu: ${c['giao dịch']} giao dịch · ${c['khoản vay']} khoản vay · ${c['ví']} ví · ${c['ngân sách']} ngân sách · ${c['danh mục']} danh mục · ${c['mục tiêu']} mục tiêu · ${c['định kỳ']} định kỳ`;
  }
};

// ---------- History: trang "Lịch sử giao dịch" — tìm kiếm mở rộng, lọc, sắp xếp, nhóm theo ngày, phân trang ----------
const History = {
  filters: { type: 'all', from: null, to: null, minAmount: null, maxAmount: null, categoryId: null },
  visibleCount: 30,
  pageSize: 30,
  typeOptions: [{ key: 'all', label: 'Tất cả' }, { key: 'income', label: 'Thu' }, { key: 'expense', label: 'Chi' }, { key: 'loan_out', label: 'Cho vay' }, { key: 'loan_in', label: 'Đi vay' }],

  renderTypeChips() {
    const wrap = document.getElementById('histTypeFilters');
    Util.clearChildren(wrap);
    History.typeOptions.forEach(o => {
      const active = History.filters.type === o.key;
      wrap.appendChild(Util.h('button', { class: `shrink-0 px-3 py-1.5 rounded-full text-xs font-semibold border ${active ? 'bg-indigo text-cream border-indigo' : 'border-ink/15 text-muted'}`, onclick: () => { History.filters.type = o.key; History.visibleCount = History.pageSize; History.render(); } }, o.label));
    });
  },

  matchesFilters(t) {
    const f = History.filters;
    if (f.type !== 'all' && t.type !== f.type) return false;
    if (f.from && t.date < f.from) return false;
    if (f.to && t.date > f.to) return false;
    if (f.minAmount != null && t.amount < f.minAmount) return false;
    if (f.maxAmount != null && t.amount > f.maxAmount) return false;
    if (f.categoryId && t.categoryId !== f.categoryId) return false;
    const q = (document.getElementById('histSearch').value || '').trim().toLowerCase();
    if (q) {
      const cat = t.categoryId ? Data.findCategory(t.categoryId) : null;
      const loan = t.loanId ? Data.state.loans.find(l => l.id === t.loanId) : null;
      const haystack = [t.note || '', cat ? cat.name : '', loan ? loan.counterpartyName : '', String(t.amount), t.date, Util.formatDateShort(t.date)].join(' ').toLowerCase();
      if (!haystack.includes(q)) return false;
    }
    return true;
  },

  sortList(list) {
    const mode = document.getElementById('histSort').value;
    const arr = list.slice();
    if (mode === 'date_asc') return arr.sort((a, b) => a.date === b.date ? (a.createdAt || '').localeCompare(b.createdAt || '') : (a.date < b.date ? -1 : 1));
    if (mode === 'amount_desc') return arr.sort((a, b) => b.amount - a.amount);
    if (mode === 'amount_asc') return arr.sort((a, b) => a.amount - b.amount);
    return UI.sortByRecency(arr);
  },

  render() {
    History.renderTypeChips();
    History.renderActiveFilterChips();
    const all = Data.state.transactions.filter(History.matchesFilters);
    const sorted = History.sortList(all);
    const wrap = document.getElementById('histList');
    Util.clearChildren(wrap);
    if (!sorted.length) { wrap.appendChild(UI.emptyState('Không tìm thấy giao dịch nào khớp', 'fa-magnifying-glass')); document.getElementById('histLoadMoreWrap').classList.add('hidden'); return; }

    const visible = sorted.slice(0, History.visibleCount);
    const mode = document.getElementById('histSort').value;
    if (mode === 'date_desc' || mode === 'date_asc') {
      let lastDate = null;
      visible.forEach(t => {
        if (t.date !== lastDate) {
          lastDate = t.date;
          const dayTxs = sorted.filter(x => x.date === t.date);
          const dayIncome = dayTxs.filter(x => x.type === CONST.TX.INCOME || x.type === CONST.TX.LOAN_IN).reduce((s, x) => s + x.amount, 0);
          const dayExpense = dayTxs.filter(x => x.type === CONST.TX.EXPENSE || x.type === CONST.TX.LOAN_OUT).reduce((s, x) => s + x.amount, 0);
          wrap.appendChild(Util.h('div', { class: 'flex justify-between items-baseline pt-4 pb-1.5' },
            Util.h('p', { class: 'text-[11px] font-semibold text-ink' }, Util.formatDateHeading(t.date)),
            Util.h('p', { class: 'text-[10px] text-muted font-mono' }, dayIncome > 0 ? `+${Util.formatVND(dayIncome)} ` : '', dayExpense > 0 ? `−${Util.formatVND(dayExpense)}` : '')));
        }
        wrap.appendChild(UI.renderTransactionRow(t));
      });
    } else {
      visible.forEach(t => wrap.appendChild(UI.renderTransactionRow(t)));
    }
    document.getElementById('histLoadMoreWrap').classList.toggle('hidden', sorted.length <= History.visibleCount);
  },

  loadMore() { History.visibleCount += History.pageSize; History.render(); },

  renderActiveFilterChips() {
    const wrap = document.getElementById('histActiveFilters');
    Util.clearChildren(wrap);
    const f = History.filters;
    const chips = [];
    if (f.from || f.to) chips.push({ label: `${f.from ? Util.formatDateShort(f.from) : '...'} → ${f.to ? Util.formatDateShort(f.to) : '...'}`, clear: () => { f.from = null; f.to = null; } });
    if (f.minAmount != null || f.maxAmount != null) chips.push({ label: `${f.minAmount != null ? Util.formatVND(f.minAmount) : '0'} – ${f.maxAmount != null ? Util.formatVND(f.maxAmount) : '∞'}`, clear: () => { f.minAmount = null; f.maxAmount = null; } });
    if (f.categoryId) { const c = Data.findCategory(f.categoryId); chips.push({ label: c ? c.icon + ' ' + c.name : 'Danh mục', clear: () => { f.categoryId = null; } }); }
    chips.forEach(c => wrap.appendChild(Util.h('button', { class: 'flex items-center gap-1.5 text-[11px] bg-indigo/10 text-indigo px-2.5 py-1 rounded-full', onclick: () => { c.clear(); History.visibleCount = History.pageSize; History.render(); } }, c.label, Util.h('i', { class: 'fas fa-xmark text-[9px]', 'aria-hidden': 'true' }))));
  }
};

// ---------- Undo: hoàn tác thao tác xoá gần nhất qua nút trên toast ----------
const Undo = {
  offer(message, restoreFn) {
    Toast.show(message, { type: 'ok', actionLabel: 'Hoàn tác', onAction: restoreFn, duration: CONST.UNDO_WINDOW_MS });
  }
};

// ---------- Toast: thông báo ngắn thay cho alert() ----------
const Toast = {
  show(message, opts) {
    opts = opts || {};
    const root = document.getElementById('toastRoot');
    const el = document.createElement('div');
    el.className = `toast ${opts.type === 'err' ? 'err' : opts.type === 'ok' ? 'ok' : ''}`;
    el.setAttribute('role', 'status');
    el.appendChild(Util.h('span', { class: 'flex-1' }, message));
    let dismissed = false;
    const dismiss = () => { if (dismissed) return; dismissed = true; clearTimeout(timer); el.classList.add('leaving'); setTimeout(() => el.remove(), 200); };
    if (opts.actionLabel) el.appendChild(Util.h('button', { class: 'toast-action', onclick: async () => { dismiss(); if (opts.onAction) await opts.onAction(); } }, opts.actionLabel));
    root.appendChild(el);
    const timer = setTimeout(dismiss, opts.duration || 3200);
  }
};

// ---------- Modal: hộp thoại chung thay cho alert()/confirm()/prompt(), hỗ trợ ESC + khoá focus ----------
const Modal = {
  _keydownHandler: null,
  _lastFocused: null,
  _onCloseCb: null,

  _field(label, inputEl) { return Util.h('div', { class: 'mb-4' }, Util.h('label', { class: 'text-[11px] text-muted uppercase tracking-widest block mb-1.5' }, label), inputEl); },
  _amountField(label, inputEl) {
    inputEl.classList.add('pr-10');
    const wrap = Util.h('div', { class: 'relative' }, inputEl, Util.h('button', { type: 'button', class: 'absolute right-0 bottom-2 icon-btn text-indigo', 'aria-label': 'Mở máy tính', title: 'Máy tính', onclick: () => Modal.openCalculator(inputEl) }, Util.h('i', { class: 'fas fa-calculator text-sm', 'aria-hidden': 'true' })));
    return Util.h('div', { class: 'mb-4' }, Util.h('label', { class: 'text-[11px] text-muted uppercase tracking-widest block mb-1.5' }, label), wrap);
  },

  open({ title, bodyNode, actions, onClose }) {
    Modal.close();
    Modal._lastFocused = document.activeElement;
    const root = document.getElementById('modalRoot');
    const overlay = Util.h('div', { class: 'modal-overlay' });
    overlay.addEventListener('click', e => { if (e.target === overlay) Modal.close(); });
    const card = Util.h('div', { class: 'modal-card', role: 'dialog', 'aria-modal': 'true' });
    card.appendChild(Util.h('div', { class: 'modal-handle' }));
    if (title) card.appendChild(Util.h('h3', { class: 'font-serif text-lg text-ink mb-4' }, title));
    card.appendChild(bodyNode);
    if (actions && actions.length) card.appendChild(Util.h('div', { class: 'flex gap-2 mt-6' }, ...actions.map(a => Util.h('button', { class: `flex-1 py-3 rounded font-semibold text-sm ${a.variant === 'danger' ? 'bg-ledger-red text-cream' : a.variant === 'primary' ? 'bg-indigo text-cream' : 'border border-ink/20 text-ink'}`, onclick: a.onClick }, a.label))));
    overlay.appendChild(card);
    root.appendChild(overlay);
    Modal._onCloseCb = onClose || null;
    Modal._keydownHandler = e => { if (e.key === 'Escape') Modal.close(); else if (e.key === 'Tab') Modal._trapFocus(e, card); };
    document.addEventListener('keydown', Modal._keydownHandler);
    const firstInput = card.querySelector('input:not([type=hidden]), textarea, select');
    setTimeout(() => (firstInput || card.querySelector('button')).focus(), 30);
  },
  _trapFocus(e, card) {
    const f = card.querySelectorAll('button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])');
    if (!f.length) return;
    const first = f[0], last = f[f.length - 1];
    if (e.shiftKey && document.activeElement === first) { e.preventDefault(); last.focus(); }
    else if (!e.shiftKey && document.activeElement === last) { e.preventDefault(); first.focus(); }
  },
  close() {
    const root = document.getElementById('modalRoot');
    if (!root.firstChild) return;
    Util.clearChildren(root);
    if (Modal._keydownHandler) { document.removeEventListener('keydown', Modal._keydownHandler); Modal._keydownHandler = null; }
    if (Modal._lastFocused && Modal._lastFocused.focus) Modal._lastFocused.focus();
    const cb = Modal._onCloseCb; Modal._onCloseCb = null;
    if (cb) cb();
  },
  confirm({ title, message, confirmLabel, cancelLabel, danger }) {
    return new Promise(resolve => {
      Modal.open({
        title: title || 'Xác nhận', bodyNode: Util.h('p', { class: 'text-sm text-ink/80 leading-relaxed' }, message),
        actions: [{ label: cancelLabel || 'Huỷ', onClick: () => Modal.close() }, { label: confirmLabel || 'Xác nhận', variant: danger ? 'danger' : 'primary', onClick: () => { resolve(true); Modal.close(); } }],
        onClose: () => resolve(false)
      });
    });
  },

  // ---------- Danh mục ----------
  openCategoryEditor(existing) {
    Categories.editingId = existing ? existing.id : null;
    const nameInput = Util.h('input', { class: 'w-full border-b-2 border-ink/15 focus:border-indigo bg-transparent py-2.5 text-ink', value: existing ? existing.name : '', maxlength: CONST.LIMITS.CATEGORY_NAME_MAX, placeholder: 'Tên danh mục' });
    const typeSelect = Util.h('select', { class: 'w-full border-b-2 border-ink/15 bg-transparent py-2.5 text-ink' }, Util.h('option', { value: 'expense', selected: !existing || existing.type === 'expense' }, 'Chi tiêu'), Util.h('option', { value: 'income', selected: existing && existing.type === 'income' }, 'Thu nhập'));
    const emojiChoices = ['🍜', '🚗', '🎮', '📱', '🛒', '📚', '🏠', '💊', '💵', '💰', '🎁', '📈', '✈️', '🐾', '🧾', '🎓', '🏥', '🎵', '👕', '🔧', '☕', '🍺', '🏋️', '💇', '📺', '🧸'];
    let selectedIcon = existing ? existing.icon : emojiChoices[0];
    const iconWrap = Util.h('div', { class: 'flex flex-wrap gap-2' });
    emojiChoices.forEach(e => iconWrap.appendChild(Util.h('button', { type: 'button', class: 'w-9 h-9 rounded-full border text-base flex items-center justify-center', style: e === selectedIcon ? 'border-color:#34456B;background:#F3E8D2' : 'border-color:rgba(43,36,32,0.15)', onclick: ev => { selectedIcon = e; iconWrap.querySelectorAll('button').forEach(b => b.style.cssText = 'border-color:rgba(43,36,32,0.15)'); ev.currentTarget.style.cssText = 'border-color:#34456B;background:#F3E8D2'; } }, e)));
    const colorChoices = Data.CATEGORY_PALETTE;
    let selectedColor = existing ? existing.color : colorChoices[0];
    const colorWrap = Util.h('div', { class: 'flex flex-wrap gap-2' });
    colorChoices.forEach((c, i) => colorWrap.appendChild(Util.h('button', { type: 'button', class: 'w-7 h-7 rounded-full', style: `background:${c};box-shadow:${c === selectedColor ? '0 0 0 2px white, 0 0 0 4px ' + c : 'none'}`, onclick: () => { selectedColor = c; Array.from(colorWrap.children).forEach((b, j) => b.style.boxShadow = colorChoices[j] === selectedColor ? '0 0 0 2px white, 0 0 0 4px ' + colorChoices[j] : 'none'); } })));
    const errorP = Util.h('p', { class: 'text-ledger-red text-xs mb-2 hidden' });
    const body = Util.h('div', {}, Modal._field('Tên danh mục', nameInput), Modal._field('Loại', typeSelect), Modal._field('Biểu tượng', iconWrap), Modal._field('Màu sắc', colorWrap), errorP);
    Modal.open({
      title: existing ? 'Sửa danh mục' : 'Thêm danh mục', bodyNode: body, actions: [
        { label: 'Huỷ', onClick: () => Modal.close() },
        {
          label: existing ? 'Cập nhật' : 'Thêm', variant: 'primary', onClick: async () => {
            const res = await Categories.upsert({ name: nameInput.value, type: typeSelect.value, icon: selectedIcon, color: selectedColor });
            if (!res.ok) { errorP.textContent = res.error; errorP.classList.remove('hidden'); return; }
            Modal.close(); Categories.render(); UI.renderCategoryChipsForForm();
            Toast.show(existing ? 'Đã cập nhật danh mục' : 'Đã thêm danh mục', { type: 'ok' });
          }
        }
      ]
    });
  },

  // ---------- Ngân sách ----------
  openBudgetEditor(existing) {
    Budgets.editingId = existing ? existing.id : null;
    const catSelect = Util.h('select', { class: 'w-full border-b-2 border-ink/15 bg-transparent py-2.5 text-ink' });
    catSelect.appendChild(Util.h('option', { value: '' }, '— Tổng ngân sách (mọi danh mục) —'));
    Categories.forType('expense').forEach(c => catSelect.appendChild(Util.h('option', { value: c.id, selected: existing && existing.categoryId === c.id }, c.icon + ' ' + c.name)));
    const amountInput = Util.h('input', { type: 'text', inputmode: 'numeric', pattern: '[0-9]*', class: 'w-full border-b-2 border-ink/15 bg-transparent py-2.5 font-mono text-ink', value: existing ? existing.amount : '', placeholder: '0' });
    const alertsCheckbox = Util.h('input', { type: 'checkbox', checked: !existing || existing.alertsEnabled, class: 'w-4 h-4 mr-2 align-middle' });
    const errorP = Util.h('p', { class: 'text-ledger-red text-xs mb-2 hidden' });
    const body = Util.h('div', {}, Modal._field('Áp dụng cho', catSelect), Modal._amountField('Hạn mức mỗi tháng', amountInput), Util.h('label', { class: 'flex items-center text-sm text-ink mb-2' }, alertsCheckbox, 'Bật cảnh báo 80% / 100%'), errorP);
    Modal.open({
      title: existing ? 'Sửa ngân sách' : 'Thêm ngân sách', bodyNode: body, actions: [
        { label: 'Huỷ', onClick: () => Modal.close() },
        {
          label: existing ? 'Cập nhật' : 'Thêm', variant: 'primary', onClick: async () => {
            const res = await Budgets.upsert({ categoryId: catSelect.value || null, amount: amountInput.value, alertsEnabled: alertsCheckbox.checked });
            if (!res.ok) { errorP.textContent = res.error; errorP.classList.remove('hidden'); return; }
            Modal.close(); Budgets.renderFullList(); UI.renderDashboard();
            Toast.show(existing ? 'Đã cập nhật ngân sách' : 'Đã thêm ngân sách', { type: 'ok' });
          }
        }
      ]
    });
  },

  // ---------- Ví ----------
  openWalletEditor(existing) {
    Wallets.editingId = existing ? existing.id : null;
    const nameInput = Util.h('input', { class: 'w-full border-b-2 border-ink/15 bg-transparent py-2.5 text-ink', value: existing ? existing.name : '', maxlength: CONST.LIMITS.PERSON_NAME_MAX, placeholder: 'Tên ví' });
    const icons = ['💵', '🏦', '📲', '💳', '🪙', '💼', '🏧', '👛'];
    let selectedIcon = existing ? existing.icon : icons[0];
    const iconWrap = Util.h('div', { class: 'flex flex-wrap gap-2' });
    icons.forEach(e => iconWrap.appendChild(Util.h('button', { type: 'button', class: 'w-9 h-9 rounded-full border text-base flex items-center justify-center', style: e === selectedIcon ? 'border-color:#34456B;background:#F3E8D2' : 'border-color:rgba(43,36,32,0.15)', onclick: ev => { selectedIcon = e; iconWrap.querySelectorAll('button').forEach(b => b.style.cssText = 'border-color:rgba(43,36,32,0.15)'); ev.currentTarget.style.cssText = 'border-color:#34456B;background:#F3E8D2'; } }, e)));
    const defaultCheckbox = Util.h('input', { type: 'checkbox', checked: existing ? existing.isDefault : false, class: 'w-4 h-4 mr-2 align-middle' });
    const includeCheckbox = Util.h('input', { type: 'checkbox', checked: existing ? existing.includeInTotal !== false : true, class: 'mt-0.5 mr-2 shrink-0' });
    const errorP = Util.h('p', { class: 'text-ledger-red text-xs mb-2 hidden' });
    const body = Util.h('div', {}, Modal._field('Tên ví', nameInput), Modal._field('Biểu tượng', iconWrap),
      Util.h('label', { class: 'flex items-center text-sm text-ink mb-4' }, defaultCheckbox, 'Đặt làm ví mặc định'),
      Util.h('label', { class: 'flex items-start text-sm text-ink mb-2' }, includeCheckbox, Util.h('span', {}, 'Tính vào số dư tổng', Util.h('span', { class: 'block text-xs text-muted mt-0.5' }, 'Tắt nếu đây là ví bạn chỉ muốn theo dõi riêng (VD: điểm thưởng, tiền chưa rõ ràng) — số dư ví này sẽ không cộng vào tổng ở tab Tổng quan.'))),
      errorP);
    Modal.open({
      title: existing ? 'Sửa ví' : 'Thêm ví', bodyNode: body, actions: [
        { label: 'Huỷ', onClick: () => Modal.close() },
        {
          label: existing ? 'Cập nhật' : 'Thêm', variant: 'primary', onClick: async () => {
            const res = await Wallets.upsert({ name: nameInput.value, icon: selectedIcon, isDefault: defaultCheckbox.checked, includeInTotal: includeCheckbox.checked });
            if (!res.ok) { errorP.textContent = res.error; errorP.classList.remove('hidden'); return; }
            Modal.close(); Wallets.render(); UI.renderDashboard();
            Toast.show(existing ? 'Đã cập nhật ví' : 'Đã thêm ví', { type: 'ok' });
          }
        }
      ]
    });
  },
  openTransferSheet() {
    const fromSelect = Util.h('select', { class: 'w-full border-b-2 border-ink/15 bg-transparent py-2.5 text-ink' });
    const toSelect = Util.h('select', { class: 'w-full border-b-2 border-ink/15 bg-transparent py-2.5 text-ink' });
    Data.state.wallets.forEach(w => { fromSelect.appendChild(Util.h('option', { value: w.id }, w.icon + ' ' + w.name)); toSelect.appendChild(Util.h('option', { value: w.id }, w.icon + ' ' + w.name)); });
    if (Data.state.wallets[1]) toSelect.value = Data.state.wallets[1].id;
    const amountInput = Util.h('input', { type: 'text', inputmode: 'numeric', pattern: '[0-9]*', class: 'w-full border-b-2 border-ink/15 bg-transparent py-2.5 font-mono text-ink', placeholder: '0' });
    const dateInput = Util.h('input', { type: 'date', value: Util.todayStr(), class: 'w-full border-b-2 border-ink/15 bg-transparent py-2.5 text-ink' });
    const noteInput = Util.h('input', { maxlength: CONST.LIMITS.NOTE_MAX, class: 'w-full border-b-2 border-ink/15 bg-transparent py-2.5 text-ink', placeholder: 'Ghi chú (không bắt buộc)' });
    const errorP = Util.h('p', { class: 'text-ledger-red text-xs mb-2 hidden' });
    const body = Util.h('div', {}, Modal._field('Từ ví', fromSelect), Modal._field('Đến ví', toSelect), Modal._amountField('Số tiền', amountInput), Modal._field('Ngày', dateInput), Modal._field('Ghi chú', noteInput), errorP);
    Modal.open({
      title: 'Chuyển tiền giữa các ví', bodyNode: body, actions: [
        { label: 'Huỷ', onClick: () => Modal.close() },
        {
          label: 'Chuyển', variant: 'primary', onClick: async () => {
            const res = await Wallets.transfer({ fromWalletId: fromSelect.value, toWalletId: toSelect.value, amount: amountInput.value, date: dateInput.value, note: noteInput.value });
            if (!res.ok) { errorP.textContent = res.error; errorP.classList.remove('hidden'); return; }
            Modal.close(); Toast.show('Đã chuyển tiền', { type: 'ok' });
          }
        }
      ]
    });
  },

  // Nạp số dư cho một ví cụ thể — 2 kiểu: "Tiền mới" (cộng vào ví VÀ vào tổng, ghi nhận như một khoản thu),
  // hoặc "Từ ví khác" (chuyển nội bộ — cộng vào ví này, trừ ví nguồn, KHÔNG đổi tổng).
  openWalletTopUpSheet(walletId) {
    const wallet = Data.findWallet(walletId);
    if (!wallet) return;
    const included = wallet.includeInTotal !== false;

    const amountInput = Util.h('input', { type: 'text', inputmode: 'numeric', pattern: '[0-9]*', class: 'w-full border-b-2 border-ink/15 bg-transparent py-2.5 font-mono text-lg text-ink', placeholder: '0' });
    const noteInput = Util.h('input', { maxlength: CONST.LIMITS.NOTE_MAX, class: 'w-full border-b-2 border-ink/15 bg-transparent py-2.5 text-ink', placeholder: 'Số dư ban đầu' });
    const catSelect = Util.h('select', { class: 'w-full border-b-2 border-ink/15 bg-transparent py-2.5 text-ink' });
    Categories.forType('income').forEach(c => catSelect.appendChild(Util.h('option', { value: c.id }, c.icon + ' ' + c.name)));

    const statusNote = Util.h('p', { class: `text-xs rounded-lg p-3 mb-4 ${included ? 'bg-indigo/10 text-indigo' : 'bg-amber/10 text-amber'}` },
      included ? 'Ví này đang TÍNH vào số dư tổng — số tiền nạp sẽ cộng vào cả tổng.' : 'Ví này đang KHÔNG tính vào số dư tổng — số tiền nạp sẽ chỉ hiện ở riêng ví này.',
      Util.h('button', { class: 'underline font-semibold ml-1', onclick: () => { Modal.close(); Modal.openWalletEditor(wallet); } }, 'Đổi ở đây'));

    const errorP = Util.h('p', { class: 'text-ledger-red text-xs mt-2 hidden' });
    const body = Util.h('div', {},
      Util.h('p', { class: 'text-sm text-ink mb-2' }, `Nạp số dư cho "${wallet.icon} ${wallet.name}"`),
      statusNote,
      Modal._amountField('Số tiền', amountInput), Modal._field('Ghi nhận vào danh mục', catSelect), Modal._field('Ghi chú', noteInput), errorP);

    Modal.open({
      title: 'Nạp số dư', bodyNode: body, actions: [
        { label: 'Huỷ', onClick: () => Modal.close() },
        {
          label: 'Nạp', variant: 'primary', onClick: async () => {
            const amountCheck = Util.validateAmount(amountInput.value);
            if (!amountCheck.ok) { errorP.textContent = amountCheck.error; errorP.classList.remove('hidden'); return; }
            if (!catSelect.value) { errorP.textContent = 'Chọn danh mục'; errorP.classList.remove('hidden'); return; }
            const tx = { id: Util.uuid(), type: CONST.TX.INCOME, amount: amountCheck.value, categoryId: catSelect.value, note: Util.sanitizeText(noteInput.value, CONST.LIMITS.NOTE_MAX) || 'Số dư ban đầu', date: Util.todayStr(), walletId, loanId: null, isRepayment: false, createdAt: new Date().toISOString() };
            Data.state.transactions.push(tx);
            const ok = await Data.save();
            if (!ok) { Data.state.transactions.pop(); errorP.textContent = 'Lưu thất bại — thử lại nhé'; errorP.classList.remove('hidden'); return; }
            Modal.close();
            Toast.show(included ? 'Đã nạp số dư — có tính vào tổng' : 'Đã nạp số dư — không tính vào tổng', { type: 'ok' });
            UI.renderDashboard(); Wallets.render();
          }
        }
      ]
    });
  },

  // ---------- Mục tiêu tiết kiệm ----------
  openGoalEditor(existing) {
    Goals.editingId = existing ? existing.id : null;
    const nameInput = Util.h('input', { class: 'w-full border-b-2 border-ink/15 bg-transparent py-2.5 text-ink', value: existing ? existing.name : '', maxlength: CONST.LIMITS.PERSON_NAME_MAX, placeholder: 'VD: Mua điện thoại' });
    const targetInput = Util.h('input', { type: 'text', inputmode: 'numeric', pattern: '[0-9]*', class: 'w-full border-b-2 border-ink/15 bg-transparent py-2.5 font-mono text-ink', value: existing ? existing.targetAmount : '', placeholder: '0' });
    const dueInput = Util.h('input', { type: 'date', value: existing && existing.dueDate ? existing.dueDate : '', class: 'w-full border-b-2 border-ink/15 bg-transparent py-2.5 text-ink' });
    const errorP = Util.h('p', { class: 'text-ledger-red text-xs mb-2 hidden' });
    const body = Util.h('div', {}, Modal._field('Tên mục tiêu', nameInput), Modal._amountField('Số tiền cần đạt', targetInput), Modal._field('Hạn (không bắt buộc)', dueInput), errorP);
    Modal.open({
      title: existing ? 'Sửa mục tiêu' : 'Thêm mục tiêu tiết kiệm', bodyNode: body, actions: [
        { label: 'Huỷ', onClick: () => Modal.close() },
        {
          label: existing ? 'Cập nhật' : 'Thêm', variant: 'primary', onClick: async () => {
            const res = await Goals.upsert({ name: nameInput.value, targetAmount: targetInput.value, dueDate: dueInput.value || null });
            if (!res.ok) { errorP.textContent = res.error; errorP.classList.remove('hidden'); return; }
            Modal.close(); Goals.render();
            Toast.show(existing ? 'Đã cập nhật mục tiêu' : 'Đã thêm mục tiêu', { type: 'ok' });
          }
        }
      ]
    });
  },
  openGoalAddFunds(id) {
    const g = Data.state.goals.find(x => x.id === id);
    if (!g) return;
    const amountInput = Util.h('input', { type: 'text', inputmode: 'numeric', pattern: '[0-9]*', class: 'w-full border-b-2 border-ink/15 bg-transparent py-2.5 font-mono text-ink text-lg', placeholder: '0' });
    const errorP = Util.h('p', { class: 'text-ledger-red text-xs mb-2 hidden' });
    const body = Util.h('div', {}, Util.h('p', { class: 'text-xs text-muted mb-3' }, `Còn thiếu ${Util.formatVND(Math.max(0, g.targetAmount - g.currentAmount))} để đạt "${g.name}"`), Modal._amountField('Số tiền thêm vào', amountInput), errorP);
    Modal.open({
      title: 'Thêm tiền vào mục tiêu', bodyNode: body, actions: [
        { label: 'Huỷ', onClick: () => Modal.close() },
        {
          label: 'Thêm', variant: 'primary', onClick: async () => {
            const res = await Goals.addFunds(id, amountInput.value);
            if (!res.ok) { errorP.textContent = res.error; errorP.classList.remove('hidden'); return; }
            Modal.close(); Toast.show('Đã thêm vào mục tiêu', { type: 'ok' });
          }
        }
      ]
    });
  },

  // ---------- Giao dịch định kỳ ----------
  openRecurringEditor(existing) {
    Recurring.editingId = existing ? existing.id : null;
    const typeSelect = Util.h('select', { class: 'w-full border-b-2 border-ink/15 bg-transparent py-2.5 text-ink' }, Util.h('option', { value: CONST.TX.EXPENSE, selected: !existing || existing.type === CONST.TX.EXPENSE }, 'Chi tiêu'), Util.h('option', { value: CONST.TX.INCOME, selected: existing && existing.type === CONST.TX.INCOME }, 'Thu nhập'));
    const catSelect = Util.h('select', { class: 'w-full border-b-2 border-ink/15 bg-transparent py-2.5 text-ink' });
    const fillCats = () => { Util.clearChildren(catSelect); Categories.forType(typeSelect.value).forEach(c => catSelect.appendChild(Util.h('option', { value: c.id, selected: existing && existing.categoryId === c.id }, c.icon + ' ' + c.name))); };
    fillCats();
    typeSelect.addEventListener('change', fillCats);
    const amountInput = Util.h('input', { type: 'text', inputmode: 'numeric', pattern: '[0-9]*', class: 'w-full border-b-2 border-ink/15 bg-transparent py-2.5 font-mono text-ink', value: existing ? existing.amount : '', placeholder: '0' });
    const freqSelect = Util.h('select', { class: 'w-full border-b-2 border-ink/15 bg-transparent py-2.5 text-ink' }, Util.h('option', { value: 'daily', selected: existing && existing.frequency === 'daily' }, 'Hàng ngày'), Util.h('option', { value: 'weekly', selected: existing && existing.frequency === 'weekly' }, 'Hàng tuần'), Util.h('option', { value: 'monthly', selected: !existing || existing.frequency === 'monthly' }, 'Hàng tháng'), Util.h('option', { value: 'yearly', selected: existing && existing.frequency === 'yearly' }, 'Hàng năm'));
    const startInput = Util.h('input', { type: 'date', value: existing ? existing.startDate : Util.todayStr(), class: 'w-full border-b-2 border-ink/15 bg-transparent py-2.5 text-ink', disabled: !!existing });
    const noteInput = Util.h('input', { maxlength: CONST.LIMITS.NOTE_MAX, value: existing ? existing.note : '', class: 'w-full border-b-2 border-ink/15 bg-transparent py-2.5 text-ink', placeholder: 'Ghi chú (VD: tiền điện)' });
    const walletSelect = Util.h('select', { class: 'w-full border-b-2 border-ink/15 bg-transparent py-2.5 text-ink' });
    Data.state.wallets.forEach(w => walletSelect.appendChild(Util.h('option', { value: w.id, selected: existing ? existing.walletId === w.id : w.isDefault }, w.icon + ' ' + w.name)));
    const errorP = Util.h('p', { class: 'text-ledger-red text-xs mb-2 hidden' });
    const body = Util.h('div', {}, Modal._field('Loại', typeSelect), Modal._field('Danh mục', catSelect), Modal._amountField('Số tiền mỗi lần', amountInput), Modal._field('Tần suất', freqSelect), Modal._field(existing ? 'Ngày bắt đầu (đã khoá)' : 'Ngày bắt đầu', startInput), Modal._field('Ví', walletSelect), Modal._field('Ghi chú', noteInput), errorP);
    Modal.open({
      title: existing ? 'Sửa giao dịch định kỳ' : 'Thêm giao dịch định kỳ', bodyNode: body, actions: [
        { label: 'Huỷ', onClick: () => Modal.close() },
        {
          label: existing ? 'Cập nhật' : 'Thêm', variant: 'primary', onClick: async () => {
            const res = await Recurring.upsert({ type: typeSelect.value, categoryId: catSelect.value || null, amount: amountInput.value, frequency: freqSelect.value, startDate: startInput.value, walletId: walletSelect.value, note: noteInput.value });
            if (!res.ok) { errorP.textContent = res.error; errorP.classList.remove('hidden'); return; }
            Modal.close(); Recurring.render();
            Toast.show(existing ? 'Đã cập nhật' : 'Đã thêm giao dịch định kỳ', { type: 'ok' });
          }
        }
      ]
    });
  },

  // ---------- Ghi nhận trả nợ ----------
  openRepaySheet(loanId) {
    const loan = Data.state.loans.find(l => l.id === loanId);
    if (!loan) return;
    const amountInput = Util.h('input', { type: 'text', inputmode: 'numeric', pattern: '[0-9]*', class: 'w-full border-b-2 border-ink/15 bg-transparent py-2.5 font-mono text-lg text-ink', placeholder: '0' });
    const quickWrap = Util.h('div', { class: 'flex flex-wrap gap-2 mt-2' },
      Util.h('button', { type: 'button', class: 'quick-amount-btn px-3 py-1.5 rounded-full border border-ink/15 text-xs', onclick: () => { amountInput.value = loan.remainingAmount; } }, 'Trả hết (' + Util.formatVND(loan.remainingAmount) + ')'),
      Util.h('button', { type: 'button', class: 'quick-amount-btn px-3 py-1.5 rounded-full border border-ink/15 text-xs', onclick: () => { amountInput.value = Math.round(loan.remainingAmount / 2); } }, 'Trả nửa'));
    const errorP = Util.h('p', { class: 'text-ledger-red text-xs mt-2 hidden' });
    const body = Util.h('div', {}, Util.h('p', { class: 'text-sm text-ink mb-1' }, loan.counterpartyName), Util.h('p', { class: 'text-xs text-muted mb-4' }, `Còn nợ ${Util.formatVND(loan.remainingAmount)} / ${Util.formatVND(loan.principal)}`), Modal._amountField(loan.direction === CONST.LOAN_DIR.LEND ? 'Số tiền đã nhận' : 'Số tiền đã trả', amountInput), quickWrap, errorP);
    Modal.open({
      title: 'Ghi nhận trả nợ', bodyNode: body, actions: [
        { label: 'Huỷ', onClick: () => Modal.close() },
        {
          label: 'Ghi nhận', variant: 'primary', onClick: async () => {
            const res = await Loans.repay(loanId, amountInput.value);
            if (!res.ok) { errorP.textContent = res.error; errorP.classList.remove('hidden'); return; }
            Modal.close(); Toast.show('Đã ghi nhận trả nợ', { type: 'ok' });
          }
        }
      ]
    });
  },

  // ---------- Lọc thêm trong Lịch sử ----------
  openHistoryFilters() {
    const f = History.filters;
    const fromInput = Util.h('input', { type: 'date', value: f.from || '', class: 'w-full border-b border-ink/15 bg-transparent py-2 text-sm' });
    const toInput = Util.h('input', { type: 'date', value: f.to || '', class: 'w-full border-b border-ink/15 bg-transparent py-2 text-sm' });
    const minInput = Util.h('input', { type: 'text', inputmode: 'numeric', pattern: '[0-9]*', value: f.minAmount != null ? f.minAmount : '', class: 'w-full border-b border-ink/15 bg-transparent py-2 text-sm font-mono', placeholder: '0' });
    const maxInput = Util.h('input', { type: 'text', inputmode: 'numeric', pattern: '[0-9]*', value: f.maxAmount != null ? f.maxAmount : '', class: 'w-full border-b border-ink/15 bg-transparent py-2 text-sm font-mono', placeholder: 'Không giới hạn' });
    const catSelect = Util.h('select', { class: 'w-full border-b border-ink/15 bg-transparent py-2 text-sm' });
    catSelect.appendChild(Util.h('option', { value: '' }, 'Tất cả danh mục'));
    Data.state.categories.forEach(c => catSelect.appendChild(Util.h('option', { value: c.id, selected: f.categoryId === c.id }, c.icon + ' ' + c.name)));
    const body = Util.h('div', {}, Util.h('div', { class: 'grid grid-cols-2 gap-3' }, Modal._field('Từ ngày', fromInput), Modal._field('Đến ngày', toInput)), Util.h('div', { class: 'grid grid-cols-2 gap-3' }, Modal._field('Số tiền từ', minInput), Modal._field('Đến', maxInput)), Modal._field('Danh mục', catSelect));
    Modal.open({
      title: 'Lọc thêm', bodyNode: body, actions: [
        { label: 'Xoá lọc', onClick: () => { const type = History.filters.type; History.filters = { type, from: null, to: null, minAmount: null, maxAmount: null, categoryId: null }; Modal.close(); History.visibleCount = History.pageSize; History.render(); } },
        {
          label: 'Áp dụng', variant: 'primary', onClick: () => {
            f.from = fromInput.value || null; f.to = toInput.value || null;
            f.minAmount = minInput.value !== '' ? Number(minInput.value) : null;
            f.maxAmount = maxInput.value !== '' ? Number(maxInput.value) : null;
            f.categoryId = catSelect.value || null;
            Modal.close(); History.visibleCount = History.pageSize; History.render();
          }
        }
      ]
    });
  },

  // ---------- Quên PIN / khôi phục ----------
  openForgotPinSheet() {
    const body = Util.h('div', {},
      Util.h('p', { class: 'text-sm text-ink/80 leading-relaxed mb-4' }, 'Dữ liệu của bạn được mã hoá bằng mã PIN. Nếu không nhớ mã PIN, cách duy nhất để lấy lại dữ liệu là khôi phục từ file sao lưu (.json) đã xuất trước đó. Nếu không có file sao lưu, dữ liệu hiện tại sẽ không thể khôi phục.'),
      Util.h('input', { type: 'file', accept: '.json,application/json', id: 'recoveryFileInput', class: 'hidden', onchange: e => Modal._handleRecoveryFile(e) }),
      Util.h('button', { class: 'w-full py-3 rounded bg-indigo text-cream font-semibold text-sm mb-2.5', onclick: () => document.getElementById('recoveryFileInput').click() }, 'Khôi phục từ file sao lưu'),
      Util.h('button', { class: 'w-full py-3 rounded border border-ledger-red text-ledger-red font-semibold text-sm', onclick: () => Modal.openWipeConfirmSheet() }, 'Xoá dữ liệu và bắt đầu lại'));
    Modal.open({ title: 'Quên mã PIN?', bodyNode: body, actions: [{ label: 'Đóng', onClick: () => Modal.close() }] });
  },
  openCorruptedRecoverySheet() {
    const body = Util.h('div', {},
      Util.h('p', { class: 'text-sm text-ink/80 leading-relaxed mb-4' }, 'Không đọc được dữ liệu đã lưu trên máy. Bạn có thể khôi phục từ file sao lưu, hoặc xoá để bắt đầu lại (mất dữ liệu cũ nếu không có file sao lưu).'),
      Util.h('input', { type: 'file', accept: '.json,application/json', id: 'recoveryFileInput', class: 'hidden', onchange: e => Modal._handleRecoveryFile(e) }),
      Util.h('button', { class: 'w-full py-3 rounded bg-indigo text-cream font-semibold text-sm mb-2.5', onclick: () => document.getElementById('recoveryFileInput').click() }, 'Khôi phục từ file sao lưu'),
      Util.h('button', { class: 'w-full py-3 rounded border border-ledger-red text-ledger-red font-semibold text-sm', onclick: () => Modal.openWipeConfirmSheet() }, 'Xoá dữ liệu và bắt đầu lại'));
    Modal.open({ title: 'Không đọc được dữ liệu', bodyNode: body, actions: [] });
  },
  openWipeConfirmSheet() {
    const confirmInput = Util.h('input', { class: 'w-full border-b-2 border-ledger-red/40 bg-transparent py-2 text-ink', placeholder: 'Gõ chữ XOÁ để xác nhận' });
    const errorP = Util.h('p', { class: 'text-ledger-red text-xs mt-2 hidden' });
    const body = Util.h('div', {}, Util.h('p', { class: 'text-sm text-ink/80 mb-4' }, 'Hành động này xoá vĩnh viễn toàn bộ dữ liệu trên máy này và không thể hoàn tác.'), Modal._field('Xác nhận', confirmInput), errorP);
    Modal.open({
      title: 'Xoá toàn bộ dữ liệu?', bodyNode: body, actions: [
        { label: 'Huỷ', onClick: () => Modal.close() },
        {
          label: 'Xoá vĩnh viễn', variant: 'danger', onClick: async () => {
            const v = confirmInput.value.trim().toUpperCase();
            if (v !== 'XOÁ' && v !== 'XOA') { errorP.textContent = 'Gõ đúng chữ "XOÁ" để xác nhận'; errorP.classList.remove('hidden'); return; }
            await App.wipeEverything();
            Modal.close();
            location.reload();
          }
        }
      ]
    });
  },
  async _handleRecoveryFile(e) {
    const file = e.target.files[0];
    if (!file) return;
    let json;
    try { json = JSON.parse(await file.text()); } catch (err) { Toast.show('File không hợp lệ (không đọc được JSON)', { type: 'err' }); return; }
    const validation = Backup.validateImportedData(json);
    if (!validation.valid) { Toast.show('File sao lưu không hợp lệ: ' + validation.errors[0], { type: 'err' }); return; }
    Modal.openRecoveryNewPinSheet(validation.state);
  },
  openRecoveryNewPinSheet(recoveredState) {
    const p1 = Util.h('input', { type: 'tel', inputmode: 'numeric', maxlength: 8, class: 'pin-mask w-full border-b-2 border-ink/15 bg-transparent py-2.5 font-mono text-center text-2xl tracking-[0.4em]', placeholder: '••••' });
    const p2 = Util.h('input', { type: 'tel', inputmode: 'numeric', maxlength: 8, class: 'pin-mask w-full border-b-2 border-ink/15 bg-transparent py-2.5 font-mono text-center text-2xl tracking-[0.4em]', placeholder: '••••' });
    const errorP = Util.h('p', { class: 'text-ledger-red text-xs mt-2 hidden' });
    const body = Util.h('div', {}, Util.h('p', { class: 'text-sm text-ink/80 mb-4' }, 'Đặt mã PIN mới cho dữ liệu vừa khôi phục.'), Modal._field('Mã PIN mới', p1), Modal._field('Nhập lại', p2), errorP);
    Modal.open({
      title: 'Đặt mã PIN mới', bodyNode: body, actions: [
        { label: 'Huỷ', onClick: () => Modal.close() },
        {
          label: 'Xác nhận', variant: 'primary', onClick: async () => {
            if (p1.value.length < CONST.LIMITS.PIN_MIN_LEN) { errorP.textContent = 'Mã PIN cần ít nhất 4 số'; errorP.classList.remove('hidden'); return; }
            if (p1.value !== p2.value) { errorP.textContent = 'Hai mã PIN không khớp'; errorP.classList.remove('hidden'); return; }
            Security.salt = Crypto_.randomSaltB64();
            Security.sessionKey = await Crypto_.deriveKey(p1.value, Security.salt, CONST.PBKDF2_ITERATIONS);
            Data.state = Data.migrateIfNeeded(recoveredState);
            await Store.set(CONST.STORAGE_KEYS.SALT, Security.salt);
            const ok = await Data.save();
            if (!ok) { errorP.textContent = 'Không lưu được — thử lại nhé'; errorP.classList.remove('hidden'); return; }
            await Security.resetAttempts();
            Security.mode = 'existing';
            Security.hasLoadedData = true;
            Modal.close();
            Security.unlockApp(true);
            Toast.show('Đã khôi phục dữ liệu từ file sao lưu', { type: 'ok' });
          }
        }
      ]
    });
  }
};

// Đổi mã PIN — gắn vào Security (được định nghĩa ở phần trước) vì cần Modal đã sẵn sàng.
Security.openChangePinSheet = function () {
  const curInput = Util.h('input', { type: 'tel', inputmode: 'numeric', maxlength: 8, class: 'pin-mask w-full border-b-2 border-ink/15 bg-transparent py-2.5 font-mono text-center text-xl tracking-[0.3em]', placeholder: 'PIN hiện tại' });
  const p1 = Util.h('input', { type: 'tel', inputmode: 'numeric', maxlength: 8, class: 'pin-mask w-full border-b-2 border-ink/15 bg-transparent py-2.5 font-mono text-center text-xl tracking-[0.3em]', placeholder: 'PIN mới' });
  const p2 = Util.h('input', { type: 'tel', inputmode: 'numeric', maxlength: 8, class: 'pin-mask w-full border-b-2 border-ink/15 bg-transparent py-2.5 font-mono text-center text-xl tracking-[0.3em]', placeholder: 'Nhập lại PIN mới' });
  const errorP = Util.h('p', { class: 'text-ledger-red text-xs mt-2 hidden' });
  const body = Util.h('div', {}, Modal._field('Mã PIN hiện tại', curInput), Modal._field('Mã PIN mới', p1), Modal._field('Nhập lại mã PIN mới', p2), errorP);
  Modal.open({
    title: 'Đổi mã PIN', bodyNode: body, actions: [
      { label: 'Huỷ', onClick: () => Modal.close() },
      {
        label: 'Đổi mã PIN', variant: 'primary', onClick: async () => {
          if (p1.value.length < CONST.LIMITS.PIN_MIN_LEN) { errorP.textContent = 'Mã PIN mới cần ít nhất 4 số'; errorP.classList.remove('hidden'); return; }
          if (p1.value !== p2.value) { errorP.textContent = 'Hai mã PIN mới không khớp'; errorP.classList.remove('hidden'); return; }
          const res = await Security.changePin(curInput.value, p1.value);
          if (!res.ok) { errorP.textContent = res.error; errorP.classList.remove('hidden'); return; }
          Modal.close();
          Toast.show('Đã đổi mã PIN', { type: 'ok' });
        }
      }
    ]
  });
};

// ---------- Calculator: máy tính 4 phép cạnh mọi ô nhập số tiền — không dùng eval(), chỉ tính tuần tự như máy tính thật ----------
const Calculator = {
  state: { display: '0', prevValue: null, pendingOp: null, justEvaluated: false },
  reset() { Calculator.state = { display: '0', prevValue: null, pendingOp: null, justEvaluated: false }; },
  inputDigit(d) {
    const s = Calculator.state;
    if (s.justEvaluated) { s.display = '0'; s.justEvaluated = false; }
    if (d === '.') { if (!s.display.includes('.')) s.display += '.'; return; }
    if (s.display === '0') s.display = d;
    else if (s.display.replace('-', '').replace('.', '').length < 12) s.display += d;
  },
  inputOp(op) {
    const s = Calculator.state;
    if (s.pendingOp && !s.justEvaluated) Calculator.evaluate();
    s.prevValue = parseFloat(s.display);
    s.pendingOp = op;
    s.justEvaluated = true;
  },
  evaluate() {
    const s = Calculator.state;
    if (s.pendingOp == null || s.prevValue == null) return;
    const cur = parseFloat(s.display);
    let result = cur;
    if (s.pendingOp === '+') result = s.prevValue + cur;
    else if (s.pendingOp === '-') result = s.prevValue - cur;
    else if (s.pendingOp === '*') result = s.prevValue * cur;
    else if (s.pendingOp === '/') result = cur !== 0 ? s.prevValue / cur : s.prevValue;
    else if (s.pendingOp === '%') result = s.prevValue * (cur / 100);
    s.display = String(Math.round(result * 100) / 100);
    s.prevValue = null; s.pendingOp = null; s.justEvaluated = true;
  },
  backspace() { const s = Calculator.state; s.display = s.display.length > 1 ? s.display.slice(0, -1) : '0'; },
  clear() { Calculator.reset(); }
};

// Mở máy tính cho một ô nhập cụ thể — target có thể là id (string) hoặc chính phần tử input.
Modal.openCalculator = function (target) {
  const el = typeof target === 'string' ? document.getElementById(target) : target;
  if (!el) return;
  Calculator.reset();
  const startVal = parseFloat(String(el.value).replace(/[^\d.-]/g, ''));
  if (isFinite(startVal) && startVal !== 0) Calculator.state.display = String(startVal);

  const display = Util.h('div', { class: 'text-right font-mono text-3xl text-ink py-4 px-2 mb-3 border-b-2 border-ink/15 truncate' }, Calculator.state.display);
  const renderDisplay = () => { display.textContent = Calculator.state.display; };
  // Rung nhẹ khi nhấn phím (nếu thiết bị hỗ trợ) — phản hồi cảm ứng ngắn, không gây khó chịu.
  const buzz = () => { try { if (navigator.vibrate) navigator.vibrate(10); } catch (e) { /* trình duyệt chặn rung — bỏ qua, không ảnh hưởng chức năng */ } };
  const btn = (label, cls, onClick, ariaLabel) => Util.h('button', { type: 'button', class: `py-4 rounded-lg text-lg font-semibold ${cls}`, 'aria-label': ariaLabel || undefined, onclick: () => { buzz(); onClick(); renderDisplay(); } }, label);

  const grid = Util.h('div', { class: 'grid grid-cols-4 gap-2', role: 'group', 'aria-label': 'Bàn phím số' },
    btn('C', 'bg-ink/5 text-ledger-red', () => Calculator.clear(), 'Xoá hết'),
    btn('⌫', 'bg-ink/5 text-ink', () => Calculator.backspace(), 'Xoá một ký tự'),
    btn('%', 'bg-ink/5 text-ink', () => Calculator.inputOp('%'), 'Phần trăm'),
    btn('÷', 'bg-amber/10 text-amber', () => Calculator.inputOp('/'), 'Chia'),
    btn('7', 'bg-cream text-ink', () => Calculator.inputDigit('7'), 'Số 7'),
    btn('8', 'bg-cream text-ink', () => Calculator.inputDigit('8'), 'Số 8'),
    btn('9', 'bg-cream text-ink', () => Calculator.inputDigit('9'), 'Số 9'),
    btn('×', 'bg-amber/10 text-amber', () => Calculator.inputOp('*'), 'Nhân'),
    btn('4', 'bg-cream text-ink', () => Calculator.inputDigit('4'), 'Số 4'),
    btn('5', 'bg-cream text-ink', () => Calculator.inputDigit('5'), 'Số 5'),
    btn('6', 'bg-cream text-ink', () => Calculator.inputDigit('6'), 'Số 6'),
    btn('−', 'bg-amber/10 text-amber', () => Calculator.inputOp('-'), 'Trừ'),
    btn('1', 'bg-cream text-ink', () => Calculator.inputDigit('1'), 'Số 1'),
    btn('2', 'bg-cream text-ink', () => Calculator.inputDigit('2'), 'Số 2'),
    btn('3', 'bg-cream text-ink', () => Calculator.inputDigit('3'), 'Số 3'),
    btn('+', 'bg-amber/10 text-amber', () => Calculator.inputOp('+'), 'Cộng'),
    btn('0', 'bg-cream text-ink col-span-2', () => Calculator.inputDigit('0'), 'Số 0'),
    btn('.', 'bg-cream text-ink', () => Calculator.inputDigit('.'), 'Dấu thập phân'),
    btn('=', 'bg-indigo text-cream', () => Calculator.evaluate(), 'Bằng')
  );

  Modal.open({
    title: 'Máy tính', bodyNode: Util.h('div', {}, display, grid), actions: [
      { label: 'Huỷ', onClick: () => Modal.close() },
      {
        label: 'Dùng số này', variant: 'primary', onClick: () => {
          const result = Math.max(0, Math.round(parseFloat(Calculator.state.display) || 0));
          el.value = result;
          el.dispatchEvent(new Event('input', { bubbles: true }));
          Modal.close();
        }
      }
    ]
  });
};

// ---------- Tài khoản: sửa hồ sơ / đổi mật khẩu — gọi từ Cài đặt > Tài khoản ----------
Modal.openEditProfileSheet = function () {
  const nameInput = Util.h('input', { class: 'w-full border-b-2 border-ink/15 focus:border-indigo bg-transparent py-2.5 text-ink', value: (Profile.current && Profile.current.displayName) || '', maxlength: CONST.LIMITS.PERSON_NAME_MAX, placeholder: 'Tên hiển thị' });
  Modal.open({
    title: 'Chỉnh sửa hồ sơ',
    bodyNode: Util.h('div', {}, Modal._field('Tên hiển thị', nameInput)),
    actions: [
      { label: 'Huỷ', onClick: () => Modal.close() },
      {
        label: 'Lưu', variant: 'primary', onClick: async () => {
          const res = await Profile.updateDisplayName(nameInput.value);
          if (!res.ok) { Toast.show(res.error, { type: 'err' }); return; }
          Toast.show('Đã cập nhật hồ sơ', { type: 'ok' });
          Modal.close();
          UI.renderAccountSection();
        }
      }
    ]
  });
};

Modal.openChangePasswordSheet = function () {
  const pwCur = Util.h('input', { type: 'password', autocomplete: 'current-password', class: 'w-full border-b-2 border-ink/15 focus:border-indigo bg-transparent py-2.5 text-ink', placeholder: 'Mật khẩu hiện tại' });
  const pw1 = Util.h('input', { type: 'password', autocomplete: 'new-password', class: 'w-full border-b-2 border-ink/15 focus:border-indigo bg-transparent py-2.5 text-ink', placeholder: 'Ít nhất 8 ký tự' });
  const pw2 = Util.h('input', { type: 'password', autocomplete: 'new-password', class: 'w-full border-b-2 border-ink/15 focus:border-indigo bg-transparent py-2.5 text-ink', placeholder: 'Nhập lại mật khẩu mới' });
  Modal.open({
    title: 'Đổi mật khẩu',
    bodyNode: Util.h('div', {}, Modal._field('Mật khẩu hiện tại', pwCur), Modal._field('Mật khẩu mới', pw1), Modal._field('Xác nhận mật khẩu mới', pw2)),
    actions: [
      { label: 'Huỷ', onClick: () => Modal.close() },
      {
        label: 'Đổi mật khẩu', variant: 'primary', onClick: async () => {
          if (!pwCur.value) { Toast.show('Nhập mật khẩu hiện tại', { type: 'err' }); return; }
          if (pw1.value.length < 8) { Toast.show('Mật khẩu mới cần ít nhất 8 ký tự', { type: 'err' }); return; }
          if (pw1.value !== pw2.value) { Toast.show('Hai mật khẩu mới không khớp', { type: 'err' }); return; }
          const res = await Auth.updatePassword(pw1.value, pwCur.value);
          if (!res.ok) { Toast.show(res.error, { type: 'err' }); return; }
          Toast.show('Đã đổi mật khẩu', { type: 'ok' });
          Modal.close();
        }
      }
    ]
  });
};

// ---------- Kết quả sao lưu: hiện rõ số lượng từng loại đã gồm trong file + nhiều cách chắc chắn lấy được file ----------
// (Một số trình duyệt/app nhúng (WebView của trình xem file, v.v.) có thể âm thầm không tải file khi bấm link download —
// nên ngoài cách tải thông thường còn có Chia sẻ (Web Share API) và Sao chép nội dung làm phương án dự phòng chắc ăn.)
Modal.openBackupResultSheet = function (json, filename, counts) {
  const sizeKb = Math.max(1, Math.round(new Blob([json]).size / 1024));

  const tryDownload = () => {
    try {
      const blob = new Blob([json], { type: 'application/json' });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url; a.download = filename;
      document.body.appendChild(a); a.click(); document.body.removeChild(a);
      setTimeout(() => URL.revokeObjectURL(url), 4000);
      Toast.show('Đã yêu cầu tải file — không thấy hộp thoại lưu thì dùng các cách bên dưới nhé', { type: 'ok' });
    } catch (e) {
      console.error('Lỗi tạo file tải xuống:', e);
      Toast.show('Tải trực tiếp không được — dùng "Chia sẻ" hoặc "Sao chép" bên dưới', { type: 'err' });
    }
  };
  tryDownload();

  const summary = Util.h('div', { class: 'grid grid-cols-2 gap-x-3 gap-y-1 text-xs mb-4 border border-dashed border-ink/15 rounded-lg p-3' },
    ...Object.entries(counts).map(([label, n]) => Util.h('div', { class: 'flex justify-between' }, Util.h('span', { class: 'text-muted' }, label), Util.h('span', { class: 'font-mono text-ink font-semibold' }, String(n)))));

  const actions = Util.h('div', { class: 'space-y-2' });
  if (navigator.share) {
    actions.appendChild(Util.h('button', {
      class: 'w-full py-3 rounded bg-indigo text-cream text-sm font-semibold flex items-center justify-center gap-2', onclick: async () => {
        try {
          const file = new File([json], filename, { type: 'application/json' });
          if (navigator.canShare && !navigator.canShare({ files: [file] })) throw new Error('không hỗ trợ chia sẻ file trên trình duyệt này');
          await navigator.share({ files: [file], title: filename });
        } catch (e) {
          if (e && e.name !== 'AbortError') { console.error('Lỗi chia sẻ:', e); Toast.show('Không chia sẻ được — thử "Sao chép nội dung" nhé', { type: 'err' }); }
        }
      }
    }, Util.h('i', { class: 'fas fa-share-nodes', 'aria-hidden': 'true' }), 'Chia sẻ / Lưu file'));
  }
  actions.appendChild(Util.h('button', {
    class: 'w-full py-3 rounded border border-ink/20 text-ink text-sm font-semibold', onclick: async () => {
      let copied = false;
      try { if (navigator.clipboard && navigator.clipboard.writeText) { await navigator.clipboard.writeText(json); copied = true; } } catch (e) { console.warn('Clipboard API lỗi:', e); }
      if (copied) Toast.show('Đã sao chép nội dung sao lưu — dán vào một file .json để lưu', { type: 'ok' });
      else Modal.openBackupTextSheet(json, filename);
    }
  }, 'Sao chép nội dung (nếu tải/chia sẻ không được)'));
  actions.appendChild(Util.h('button', { class: 'w-full py-2.5 rounded text-muted text-xs', onclick: () => tryDownload() }, 'Thử tải file lại'));

  const body = Util.h('div', {},
    Util.h('p', { class: 'text-sm text-ink mb-1' }, `${filename} · ~${sizeKb} KB`),
    Util.h('p', { class: 'text-[11px] text-muted mb-3' }, 'Đã gồm đầy đủ trong file:'),
    summary, actions);
  Modal.open({ title: 'Sao lưu dữ liệu', bodyNode: body, actions: [{ label: 'Xong', onClick: () => Modal.close() }] });
};

// Phương án cuối cùng khi cả tải file lẫn Clipboard API đều không dùng được: chọn thủ công bằng cử chỉ chọn văn bản của hệ điều hành.
Modal.openBackupTextSheet = function (json, filename) {
  const textarea = Util.h('textarea', { readonly: true, class: 'w-full h-48 border border-ink/20 rounded p-2 font-mono text-[10px] text-ink leading-relaxed', value: json });
  const body = Util.h('div', {},
    Util.h('p', { class: 'text-xs text-muted mb-2' }, `Chạm vào ô bên dưới, chọn tất cả rồi sao chép thủ công, dán vào một file .json để lưu (gợi ý tên: ${filename}).`),
    textarea);
  Modal.open({ title: 'Nội dung sao lưu', bodyNode: body, actions: [{ label: 'Đóng', onClick: () => Modal.close() }] });
  setTimeout(() => { textarea.focus(); textarea.select(); }, 100);
};

// ---------- Backup: xuất/nhập dữ liệu đầy đủ, kiểm tra nghiêm ngặt, không gộp mù quáng ----------
const Backup = {
  exportBackup() {
    const payload = {
      appVersion: CONST.APP_VERSION, schemaVersion: CONST.SCHEMA_VERSION, exportedAt: new Date().toISOString(),
      transactions: Data.state.transactions, loans: Data.state.loans, budgets: Data.state.budgets,
      categories: Data.state.categories, wallets: Data.state.wallets, transfers: Data.state.transfers,
      goals: Data.state.goals, recurring: Data.state.recurring, settings: Data.state.settings, metadata: Data.state.metadata
    };
    const counts = {
      'Giao dịch': payload.transactions.length, 'Khoản vay': payload.loans.length, 'Ngân sách': payload.budgets.length,
      'Danh mục': payload.categories.length, 'Ví': payload.wallets.length, 'Lượt chuyển tiền': payload.transfers.length,
      'Mục tiêu tiết kiệm': payload.goals.length, 'Định kỳ': payload.recurring.length
    };
    const json = JSON.stringify(payload, null, 2);
    const filename = `so-chi-tieu-backup-${Util.todayStr()}.json`;
    Modal.openBackupResultSheet(json, filename, counts);
  },

  triggerImport() { document.getElementById('importFileInput').click(); },

  async handleImportFile(event) {
    const file = event.target.files[0];
    event.target.value = '';
    if (!file) return;
    let json;
    try { json = JSON.parse(await file.text()); } catch (e) { Toast.show('File không hợp lệ (không đọc được JSON)', { type: 'err' }); return; }
    const validation = Backup.validateImportedData(json);
    if (!validation.valid) { Toast.show('File sao lưu không hợp lệ: ' + validation.errors[0], { type: 'err' }); return; }
    Modal.openImportChoiceSheet(validation);
  },

  validateImportedData(json) {
    const errors = [];
    if (!json || typeof json !== 'object') return { valid: false, errors: ['Định dạng file không đúng'] };

    const validTx = []; let txErr = 0;
    (Array.isArray(json.transactions) ? json.transactions : []).forEach(t => {
      if (!t || typeof t !== 'object' || !Util.validateType(t.type)) { txErr++; return; }
      const amtCheck = Util.validateAmount(t.amount);
      if (!amtCheck.ok || !Util.isValidDateStr(t.date)) { txErr++; return; }
      validTx.push({ id: (typeof t.id === 'string' && t.id) ? t.id : Util.uuid(), type: t.type, amount: amtCheck.value, categoryId: typeof t.categoryId === 'string' ? t.categoryId : null, note: Util.sanitizeText(t.note, CONST.LIMITS.NOTE_MAX), date: t.date, walletId: typeof t.walletId === 'string' ? t.walletId : null, loanId: typeof t.loanId === 'string' ? t.loanId : null, isRepayment: !!t.isRepayment, createdAt: typeof t.createdAt === 'string' ? t.createdAt : new Date().toISOString() });
    });

    const validLoans = []; let loanErr = 0;
    (Array.isArray(json.loans) ? json.loans : []).forEach(l => {
      if (!l || typeof l !== 'object') { loanErr++; return; }
      const principalCheck = Util.validateAmount(l.principal);
      if (!principalCheck.ok || (l.direction !== CONST.LOAN_DIR.LEND && l.direction !== CONST.LOAN_DIR.BORROW)) { loanErr++; return; }
      const paid = Util.clamp(Math.round(Number(l.paidAmount) || 0), 0, principalCheck.value);
      validLoans.push({
        id: (typeof l.id === 'string' && l.id) ? l.id : Util.uuid(), direction: l.direction,
        counterpartyName: Util.sanitizeText(l.counterpartyName, CONST.LIMITS.PERSON_NAME_MAX) || 'Không rõ tên',
        principal: principalCheck.value, paidAmount: paid, remainingAmount: principalCheck.value - paid,
        loanDate: Util.isValidDateStr(l.loanDate) ? l.loanDate : Util.todayStr(), dueDate: Util.isValidDateStr(l.dueDate) ? l.dueDate : null,
        status: [CONST.LOAN_STATUS.UNPAID, CONST.LOAN_STATUS.PARTIAL, CONST.LOAN_STATUS.PAID].includes(l.status) ? l.status : (paid >= principalCheck.value ? CONST.LOAN_STATUS.PAID : (paid > 0 ? CONST.LOAN_STATUS.PARTIAL : CONST.LOAN_STATUS.UNPAID)),
        note: Util.sanitizeText(l.note, CONST.LIMITS.NOTE_MAX), initialTransactionId: typeof l.initialTransactionId === 'string' ? l.initialTransactionId : null,
        repayments: Array.isArray(l.repayments) ? l.repayments.filter(r => r && Util.validateAmount(r.amount).ok).map(r => ({ id: (typeof r.id === 'string' && r.id) ? r.id : Util.uuid(), amount: Util.validateAmount(r.amount).value, date: Util.isValidDateStr(r.date) ? r.date : Util.todayStr(), transactionId: typeof r.transactionId === 'string' ? r.transactionId : null, note: Util.sanitizeText(r.note, CONST.LIMITS.NOTE_MAX) })) : [],
        legacyUnlinked: !!l.legacyUnlinked
      });
    });

    const HEX_COLOR_RE = /^#[0-9a-fA-F]{6}$/;
    const validCats = (Array.isArray(json.categories) ? json.categories : []).filter(c => c && typeof c.name === 'string' && (c.type === 'income' || c.type === 'expense')).map(c => ({ id: (typeof c.id === 'string' && c.id) ? c.id : Util.uuid(), name: Util.sanitizeText(c.name, CONST.LIMITS.CATEGORY_NAME_MAX), icon: (typeof c.icon === 'string' && c.icon.length <= 8) ? c.icon : '🏷️', color: (typeof c.color === 'string' && HEX_COLOR_RE.test(c.color)) ? c.color : '#8B8175', type: c.type, isDefault: !!c.isDefault }));
    const validWallets = (Array.isArray(json.wallets) ? json.wallets : []).filter(w => w && typeof w.name === 'string').map(w => ({ id: (typeof w.id === 'string' && w.id) ? w.id : Util.uuid(), name: Util.sanitizeText(w.name, CONST.LIMITS.PERSON_NAME_MAX), icon: typeof w.icon === 'string' ? w.icon : '💼', isDefault: !!w.isDefault, includeInTotal: w.includeInTotal !== false }));
    const validBudgets = (Array.isArray(json.budgets) ? json.budgets : []).filter(b => b && Util.validateAmount(b.amount).ok).map(b => ({ id: (typeof b.id === 'string' && b.id) ? b.id : Util.uuid(), categoryId: typeof b.categoryId === 'string' ? b.categoryId : null, amount: Util.validateAmount(b.amount).value, alertsEnabled: b.alertsEnabled !== false, createdAt: typeof b.createdAt === 'string' ? b.createdAt : new Date().toISOString() }));
    const validTransfers = (Array.isArray(json.transfers) ? json.transfers : []).filter(t => t && Util.validateAmount(t.amount).ok && typeof t.fromWalletId === 'string' && typeof t.toWalletId === 'string').map(t => ({ id: (typeof t.id === 'string' && t.id) ? t.id : Util.uuid(), fromWalletId: t.fromWalletId, toWalletId: t.toWalletId, amount: Util.validateAmount(t.amount).value, date: Util.isValidDateStr(t.date) ? t.date : Util.todayStr(), note: Util.sanitizeText(t.note, CONST.LIMITS.NOTE_MAX), createdAt: typeof t.createdAt === 'string' ? t.createdAt : new Date().toISOString() }));
    const validGoals = (Array.isArray(json.goals) ? json.goals : []).filter(g => g && typeof g.name === 'string' && Util.validateAmount(g.targetAmount).ok).map(g => ({ id: (typeof g.id === 'string' && g.id) ? g.id : Util.uuid(), name: Util.sanitizeText(g.name, CONST.LIMITS.PERSON_NAME_MAX), targetAmount: Util.validateAmount(g.targetAmount).value, currentAmount: Math.max(0, Math.round(Number(g.currentAmount) || 0)), dueDate: Util.isValidDateStr(g.dueDate) ? g.dueDate : null, icon: typeof g.icon === 'string' ? g.icon : '🎯', createdAt: typeof g.createdAt === 'string' ? g.createdAt : new Date().toISOString(), contributions: Array.isArray(g.contributions) ? g.contributions.filter(c => c && Util.validateAmount(c.amount).ok).map(c => ({ id: (typeof c.id === 'string' && c.id) ? c.id : Util.uuid(), amount: Util.validateAmount(c.amount).value, date: Util.isValidDateStr(c.date) ? c.date : Util.todayStr() })) : [] }));
    const validRecurring = (Array.isArray(json.recurring) ? json.recurring : []).filter(r => r && Util.validateAmount(r.amount).ok && (r.type === CONST.TX.INCOME || r.type === CONST.TX.EXPENSE) && Object.values(CONST.RECUR_FREQ).includes(r.frequency)).map(r => ({ id: (typeof r.id === 'string' && r.id) ? r.id : Util.uuid(), type: r.type, amount: Util.validateAmount(r.amount).value, categoryId: typeof r.categoryId === 'string' ? r.categoryId : null, note: Util.sanitizeText(r.note, CONST.LIMITS.NOTE_MAX), walletId: typeof r.walletId === 'string' ? r.walletId : null, frequency: r.frequency, startDate: Util.isValidDateStr(r.startDate) ? r.startDate : Util.todayStr(), nextDate: Util.isValidDateStr(r.nextDate) ? r.nextDate : (Util.isValidDateStr(r.startDate) ? r.startDate : Util.todayStr()), endDate: Util.isValidDateStr(r.endDate) ? r.endDate : null, active: r.active !== false, lastGeneratedAt: typeof r.lastGeneratedAt === 'string' ? r.lastGeneratedAt : null }));

    if (!validTx.length && !validLoans.length && !validCats.length && !validWallets.length) errors.push('Không tìm thấy dữ liệu hợp lệ nào trong file');

    const state = Data.migrateIfNeeded({
      schemaVersion: CONST.SCHEMA_VERSION, transactions: validTx, loans: validLoans, budgets: validBudgets,
      categories: validCats.length ? validCats : Data.defaultCategories(), wallets: validWallets.length ? validWallets : Data.defaultWallets(),
      transfers: validTransfers, goals: validGoals, recurring: validRecurring,
      settings: (json.settings && typeof json.settings === 'object') ? json.settings : {}, metadata: (json.metadata && typeof json.metadata === 'object') ? json.metadata : {}
    });

    return { valid: errors.length === 0, errors, state, counts: { txValid: validTx.length, txError: txErr, loanValid: validLoans.length, loanError: loanErr, catValid: validCats.length, walletValid: validWallets.length, budgetValid: validBudgets.length, goalValid: validGoals.length, recurringValid: validRecurring.length } };
  },

  async performImport(validation, mode) {
    const snapshot = Data.snapshot();
    if (mode === 'replace') {
      Data.state = validation.state;
    } else {
      const merged = Data.snapshot();
      const mergeArr = (existingArr, incomingArr) => {
        const byId = new Map(existingArr.map(x => [x.id, x]));
        incomingArr.forEach(item => { if (byId.has(item.id)) Object.assign(byId.get(item.id), item); else { existingArr.push(item); byId.set(item.id, item); } });
      };
      mergeArr(merged.transactions, validation.state.transactions);
      mergeArr(merged.loans, validation.state.loans);
      mergeArr(merged.categories, validation.state.categories);
      mergeArr(merged.wallets, validation.state.wallets);
      mergeArr(merged.budgets, validation.state.budgets);
      mergeArr(merged.transfers, validation.state.transfers);
      mergeArr(merged.goals, validation.state.goals);
      mergeArr(merged.recurring, validation.state.recurring);
      Data.state = merged;
    }
    const ok = await Data.save();
    if (!ok) { Data.state = snapshot; return { ok: false, error: 'Lưu thất bại — đã khôi phục dữ liệu trước đó' }; }
    return { ok: true, rollbackSnapshot: snapshot };
  }
};

// Sheet chọn Gộp / Thay thế / Huỷ — gắn vào Modal (đã định nghĩa ở phần trước).
Modal.openImportChoiceSheet = function (validation) {
  const c = validation.counts;
  const summary = Util.h('div', { class: 'text-xs text-muted mb-5 space-y-1 border border-dashed border-ink/15 rounded-lg p-3' },
    Util.h('p', {}, `${c.txValid} giao dịch hợp lệ` + (c.txError ? `, ${c.txError} bản ghi lỗi bị bỏ qua` : '')),
    Util.h('p', {}, `${c.loanValid} khoản vay hợp lệ` + (c.loanError ? `, ${c.loanError} bản ghi lỗi bị bỏ qua` : '')),
    Util.h('p', {}, `${c.catValid} danh mục · ${c.walletValid} ví · ${c.budgetValid} ngân sách · ${c.goalValid} mục tiêu · ${c.recurringValid} định kỳ`));
  const body = Util.h('div', {}, summary,
    Util.h('button', { class: 'w-full py-3 rounded bg-indigo text-cream font-semibold text-sm mb-2.5', onclick: async () => Modal._doImport(validation, 'merge') }, 'Gộp vào dữ liệu hiện tại'),
    Util.h('button', {
      class: 'w-full py-3 rounded border border-ledger-red text-ledger-red font-semibold text-sm mb-2.5', onclick: async () => {
        const sure = await Modal.confirm({ title: 'Thay thế toàn bộ dữ liệu?', message: 'Toàn bộ dữ liệu hiện tại sẽ bị thay thế bằng dữ liệu trong file sao lưu. Có thể hoàn tác ngay sau đó nếu cần.', confirmLabel: 'Thay thế', danger: true });
        if (sure) await Modal._doImport(validation, 'replace');
      }
    }, 'Thay thế toàn bộ dữ liệu'),
    Util.h('button', { class: 'w-full py-3 rounded text-muted text-sm', onclick: () => Modal.close() }, 'Huỷ'));
  Modal.open({ title: 'Khôi phục dữ liệu', bodyNode: body, actions: [] });
};
Modal._doImport = async function (validation, mode) {
  const res = await Backup.performImport(validation, mode);
  Modal.close();
  if (!res.ok) { Toast.show(res.error, { type: 'err' }); return; }
  Toast.show(mode === 'replace' ? 'Đã thay thế toàn bộ dữ liệu' : 'Đã gộp dữ liệu sao lưu', { type: 'ok' });
  UI.renderDashboard();
  Undo.offer('Có thể hoàn tác việc khôi phục này', async () => { await Data.restoreSnapshot(res.rollbackSnapshot); UI.renderDashboard(); Toast.show('Đã hoàn tác khôi phục', { type: 'ok' }); });
};

// ---------- ExportXlsx: xuất Excel nhiều sheet ----------
const ExportXlsx = {
  available() { return typeof XLSX !== 'undefined' && !window.__xlsxLoadFailed; },
  run() {
    if (!ExportXlsx.available()) {
      Toast.show('Không tải được thư viện Excel (cần mạng) — dùng CSV thay thế', { type: 'info' });
      ExportXlsx.exportCSV();
      return;
    }
    try {
      const wb = XLSX.utils.book_new();
      const typeLabel = { income: 'Thu nhập', expense: 'Chi tiêu', loan_out: 'Cho vay', loan_in: 'Đi vay' };

      const txRows = Data.state.transactions.map(t => ({ 'Ngày': t.date, 'Loại': typeLabel[t.type] || t.type, 'Danh mục': (t.categoryId && Data.findCategory(t.categoryId)) ? Data.findCategory(t.categoryId).name : '', 'Số tiền (VND)': t.amount, 'Ghi chú': t.note || '', 'Ví': (t.walletId && Data.findWallet(t.walletId)) ? Data.findWallet(t.walletId).name : '', 'Trả nợ?': t.isRepayment ? 'Có' : '' }));
      const wsTx = XLSX.utils.json_to_sheet(txRows);
      wsTx['!cols'] = [{ wch: 12 }, { wch: 12 }, { wch: 16 }, { wch: 14 }, { wch: 30 }, { wch: 14 }, { wch: 8 }];
      XLSX.utils.book_append_sheet(wb, wsTx, 'Giao dịch');

      const loanRows = Data.state.loans.map(l => ({ 'Người liên quan': l.counterpartyName, 'Chiều': l.direction === CONST.LOAN_DIR.LEND ? 'Cho vay' : 'Đi vay', 'Gốc (VND)': l.principal, 'Đã trả (VND)': l.paidAmount, 'Còn nợ (VND)': l.remainingAmount, 'Ngày vay': l.loanDate, 'Hạn trả': l.dueDate || '', 'Trạng thái': Loans.displayStatus(l).label, 'Ghi chú': l.note || '' }));
      const wsLoans = XLSX.utils.json_to_sheet(loanRows);
      wsLoans['!cols'] = [{ wch: 18 }, { wch: 10 }, { wch: 14 }, { wch: 14 }, { wch: 14 }, { wch: 12 }, { wch: 12 }, { wch: 12 }, { wch: 24 }];
      XLSX.utils.book_append_sheet(wb, wsLoans, 'Khoản vay');

      const repayRows = [];
      Data.state.loans.forEach(l => l.repayments.forEach(r => repayRows.push({ 'Người liên quan': l.counterpartyName, 'Chiều': l.direction === CONST.LOAN_DIR.LEND ? 'Cho vay' : 'Đi vay', 'Ngày trả': r.date, 'Số tiền (VND)': r.amount })));
      const wsRepay = XLSX.utils.json_to_sheet(repayRows);
      wsRepay['!cols'] = [{ wch: 18 }, { wch: 10 }, { wch: 12 }, { wch: 14 }];
      XLSX.utils.book_append_sheet(wb, wsRepay, 'Lịch sử trả nợ');

      const budgetRows = Data.state.budgets.map(b => ({ 'Áp dụng': b.categoryId && Data.findCategory(b.categoryId) ? Data.findCategory(b.categoryId).name : 'Tổng ngân sách', 'Hạn mức/tháng (VND)': b.amount, 'Đã dùng tháng này (VND)': Budgets.currentMonthUsage(b), 'Cảnh báo': b.alertsEnabled ? 'Bật' : 'Tắt' }));
      const wsBudget = XLSX.utils.json_to_sheet(budgetRows);
      wsBudget['!cols'] = [{ wch: 18 }, { wch: 18 }, { wch: 20 }, { wch: 10 }];
      XLSX.utils.book_append_sheet(wb, wsBudget, 'Ngân sách');

      const catRows = Data.state.categories.map(c => ({ 'Tên': c.name, 'Loại': c.type === 'income' ? 'Thu nhập' : 'Chi tiêu', 'Icon': c.icon, 'Màu': c.color }));
      const wsCat = XLSX.utils.json_to_sheet(catRows);
      wsCat['!cols'] = [{ wch: 18 }, { wch: 10 }, { wch: 8 }, { wch: 10 }];
      XLSX.utils.book_append_sheet(wb, wsCat, 'Danh mục');

      const totals = Ledger.totals();
      const now = new Date();
      const { income, expense } = Ledger.incomeExpenseInRange(new Date(now.getFullYear(), now.getMonth(), 1), new Date(now.getFullYear(), now.getMonth() + 1, 0, 23, 59, 59));
      const wsSummary = XLSX.utils.json_to_sheet([
        { 'Chỉ số': 'Số dư hiện tại (VND)', 'Giá trị': totals.cash }, { 'Chỉ số': 'Đang cho vay (VND)', 'Giá trị': totals.lent },
        { 'Chỉ số': 'Đang phải trả (VND)', 'Giá trị': totals.owed }, { 'Chỉ số': 'Giá trị ròng (VND)', 'Giá trị': totals.netWorth },
        { 'Chỉ số': 'Thu tháng này (VND)', 'Giá trị': income }, { 'Chỉ số': 'Chi tháng này (VND)', 'Giá trị': expense },
        { 'Chỉ số': 'Ngày xuất file', 'Giá trị': Util.formatDateShort(Util.todayStr()) }
      ]);
      wsSummary['!cols'] = [{ wch: 24 }, { wch: 16 }];
      XLSX.utils.book_append_sheet(wb, wsSummary, 'Tổng quan');

      XLSX.writeFile(wb, `so-chi-tieu-${Util.todayStr()}.xlsx`);
      Toast.show('Đã xuất file Excel', { type: 'ok' });
    } catch (e) {
      console.error('Lỗi xuất Excel:', e);
      Toast.show('Có lỗi khi xuất Excel — đang thử xuất CSV thay thế', { type: 'err' });
      ExportXlsx.exportCSV();
    }
  },

  // ---------- Dự phòng CSV: JS thuần, không phụ thuộc thư viện/CDN nào — dùng khi Excel không tải được hoặc lỗi ----------
  csvEscape(v) {
    const s = v == null ? '' : String(v);
    return /[",\r\n]/.test(s) ? '"' + s.replace(/"/g, '""') + '"' : s;
  },
  exportCSV() {
    try {
      const typeLabel = { income: 'Thu nhập', expense: 'Chi tiêu', loan_out: 'Cho vay', loan_in: 'Đi vay' };
      const header = ['Ngày', 'Loại', 'Danh mục', 'Số tiền (VND)', 'Ghi chú', 'Ví', 'Trả nợ?'];
      const rows = Data.state.transactions.map(t => [
        t.date, typeLabel[t.type] || t.type,
        (t.categoryId && Data.findCategory(t.categoryId)) ? Data.findCategory(t.categoryId).name : '',
        t.amount, t.note || '',
        (t.walletId && Data.findWallet(t.walletId)) ? Data.findWallet(t.walletId).name : '',
        t.isRepayment ? 'Có' : ''
      ]);
      const csv = [header].concat(rows).map(r => r.map(ExportXlsx.csvEscape).join(',')).join('\r\n');
      // BOM ở đầu file để Excel nhận đúng UTF-8, không lỗi font dấu tiếng Việt.
      const blob = new Blob(['\uFEFF' + csv], { type: 'text/csv;charset=utf-8;' });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url; a.download = `giao-dich-${Util.todayStr()}.csv`;
      document.body.appendChild(a); a.click(); a.remove();
      setTimeout(() => URL.revokeObjectURL(url), 1000);
      Toast.show('Đã xuất file CSV (giao dịch)', { type: 'ok' });
    } catch (e) {
      console.error('Lỗi xuất CSV:', e);
      Toast.show('Không thể xuất file — thử lại sau nhé', { type: 'err' });
    }
  }
};

// ---------- Report: báo cáo theo tháng/quý/năm, xem trước rồi in (Lưu thành PDF qua hộp thoại in của trình duyệt) ----------
const Report = {
  kind: 'month', year: new Date().getFullYear(), month: new Date().getMonth(), quarter: Math.floor(new Date().getMonth() / 3),
  _kindInit: false,

  init() { if (!Report._kindInit) { Report._kindInit = true; Report.setKind(Report.kind); } else { Report.renderPicker(); Report.renderPreview(); } },
  setKind(k) {
    Report.kind = k;
    document.querySelectorAll('.report-kind-btn').forEach(b => { const active = b.dataset.reportKind === k; b.classList.toggle('bg-indigo', active); b.classList.toggle('text-cream', active); b.classList.toggle('border-indigo', active); });
    Report.renderPicker(); Report.renderPreview();
  },
  range() {
    if (Report.kind === 'month') return { from: new Date(Report.year, Report.month, 1), to: new Date(Report.year, Report.month + 1, 0, 23, 59, 59), label: `Tháng ${Report.month + 1}/${Report.year}` };
    if (Report.kind === 'quarter') return { from: new Date(Report.year, Report.quarter * 3, 1), to: new Date(Report.year, Report.quarter * 3 + 3, 0, 23, 59, 59), label: `Quý ${Report.quarter + 1}/${Report.year}` };
    return { from: new Date(Report.year, 0, 1), to: new Date(Report.year, 11, 31, 23, 59, 59), label: `Năm ${Report.year}` };
  },
  renderPicker() {
    const wrap = document.getElementById('reportPeriodPicker');
    Util.clearChildren(wrap);
    const sel = Util.h('select', { class: 'w-full border-b-2 border-ink/15 bg-transparent py-2.5 text-ink' });
    if (Report.kind === 'month') { for (let y = Report.year - 2; y <= Report.year + 1; y++) for (let m = 0; m < 12; m++) sel.appendChild(Util.h('option', { value: `${y}-${m}`, selected: y === Report.year && m === Report.month }, `Tháng ${m + 1}/${y}`)); sel.addEventListener('change', () => { const [y, m] = sel.value.split('-').map(Number); Report.year = y; Report.month = m; Report.renderPreview(); }); }
    else if (Report.kind === 'quarter') { for (let y = Report.year - 2; y <= Report.year + 1; y++) for (let q = 0; q < 4; q++) sel.appendChild(Util.h('option', { value: `${y}-${q}`, selected: y === Report.year && q === Report.quarter }, `Quý ${q + 1}/${y}`)); sel.addEventListener('change', () => { const [y, q] = sel.value.split('-').map(Number); Report.year = y; Report.quarter = q; Report.renderPreview(); }); }
    else { for (let y = Report.year - 4; y <= Report.year + 1; y++) sel.appendChild(Util.h('option', { value: y, selected: y === Report.year }, `Năm ${y}`)); sel.addEventListener('change', () => { Report.year = Number(sel.value); Report.renderPreview(); }); }
    wrap.appendChild(sel);
  },
  renderPreview() {
    const { from, to, label } = Report.range();
    const { income, expense } = Ledger.incomeExpenseInRange(from, to);
    const txs = Ledger.transactionsInRange(from, to);
    const byCat = {};
    txs.filter(t => t.type === CONST.TX.EXPENSE).forEach(t => { byCat[t.categoryId] = (byCat[t.categoryId] || 0) + t.amount; });
    const top = Object.entries(byCat).sort((a, b) => b[1] - a[1]).slice(0, 5);
    const wrap = document.getElementById('reportPreview');
    Util.clearChildren(wrap);
    wrap.appendChild(Util.h('p', { class: 'font-serif text-base text-ink mb-3' }, label));
    wrap.appendChild(Util.h('div', { class: 'grid grid-cols-2 gap-3 text-xs mb-3' },
      Util.h('div', {}, Util.h('p', { class: 'text-muted' }, 'Thu'), Util.h('p', { class: 'font-mono text-ledger-green' }, Util.formatVND(income))),
      Util.h('div', {}, Util.h('p', { class: 'text-muted' }, 'Chi'), Util.h('p', { class: 'font-mono text-ledger-red' }, Util.formatVND(expense))),
      Util.h('div', {}, Util.h('p', { class: 'text-muted' }, 'Tiết kiệm'), Util.h('p', { class: 'font-mono text-ink' }, Util.formatVND(income - expense))),
      Util.h('div', {}, Util.h('p', { class: 'text-muted' }, 'Số giao dịch'), Util.h('p', { class: 'font-mono text-ink' }, String(txs.length)))));
    if (top.length) {
      wrap.appendChild(Util.h('p', { class: 'text-[10px] tracking-widest text-muted uppercase mb-1.5' }, 'Top danh mục chi tiêu'));
      top.forEach(([id, amt]) => { const cat = Data.findCategory(id); wrap.appendChild(Util.h('div', { class: 'flex justify-between text-xs py-1' }, Util.h('span', {}, cat ? cat.icon + ' ' + cat.name : 'Khác'), Util.h('span', { class: 'font-mono' }, Util.formatVND(amt)))); });
    }
  },
  print() {
    const { from, to, label } = Report.range();
    const { income, expense } = Ledger.incomeExpenseInRange(from, to);
    const txs = Ledger.transactionsInRange(from, to);
    const byCat = {};
    txs.filter(t => t.type === CONST.TX.EXPENSE).forEach(t => { byCat[t.categoryId] = (byCat[t.categoryId] || 0) + t.amount; });
    const top = Object.entries(byCat).sort((a, b) => b[1] - a[1]).slice(0, 8);
    const activeLoans = Data.state.loans.filter(l => l.status !== CONST.LOAN_STATUS.PAID);

    const area = document.getElementById('reportPrintArea');
    Util.clearChildren(area);
    area.appendChild(Util.h('h1', { class: 'text-2xl font-bold text-center mb-1' }, 'BÁO CÁO TÀI CHÍNH CÁ NHÂN'));
    area.appendChild(Util.h('p', { class: 'text-center text-sm mb-8' }, label));
    area.appendChild(Util.h('div', { class: 'grid grid-cols-2 gap-4 mb-6 text-sm' },
      Util.h('p', {}, 'Tổng thu: ', Util.h('b', {}, Util.formatVND(income))), Util.h('p', {}, 'Tổng chi: ', Util.h('b', {}, Util.formatVND(expense))),
      Util.h('p', {}, 'Tiết kiệm: ', Util.h('b', {}, Util.formatVND(income - expense))), Util.h('p', {}, 'Số dư hiện tại: ', Util.h('b', {}, Util.formatVND(Ledger.totals().cash)))));
    if (top.length) {
      area.appendChild(Util.h('h2', { class: 'font-semibold mb-2 mt-4' }, 'Top danh mục chi tiêu'));
      const list = Util.h('div', { class: 'mb-6' });
      top.forEach(([id, amt]) => { const cat = Data.findCategory(id); list.appendChild(Util.h('p', { class: 'text-sm flex justify-between border-b border-gray-300 py-1' }, Util.h('span', {}, cat ? cat.icon + ' ' + cat.name : 'Khác'), Util.h('span', {}, Util.formatVND(amt)))); });
      area.appendChild(list);
    }
    if (activeLoans.length) {
      area.appendChild(Util.h('h2', { class: 'font-semibold mb-2 mt-4' }, 'Khoản vay chưa tất toán'));
      const list = Util.h('div', { class: 'mb-6' });
      activeLoans.forEach(l => list.appendChild(Util.h('p', { class: 'text-sm flex justify-between border-b border-gray-300 py-1' }, Util.h('span', {}, (l.direction === CONST.LOAN_DIR.LEND ? 'Cho vay: ' : 'Đi vay: ') + l.counterpartyName), Util.h('span', {}, Util.formatVND(l.remainingAmount)))));
      area.appendChild(list);
    }
    area.appendChild(Util.h('p', { class: 'text-xs text-gray-500 mt-10 text-center' }, 'Xuất từ Sổ Chi Tiêu · ' + Util.formatDateShort(Util.todayStr())));
    area.classList.add('printing');
    window.print();
    setTimeout(() => area.classList.remove('printing'), 500);
  }
};

// ---------- Signature: vẽ chữ ký tay + in giấy vay nợ (tổng quát hoá cho cả hai chiều vay/cho vay) ----------
const Signature = {
  contexts: {},
  currentLoanId: null,

  setup(canvasId) {
    const canvas = document.getElementById(canvasId);
    if (!canvas) return;
    const ctx = canvas.getContext('2d');
    ctx.lineWidth = 2; ctx.lineCap = 'round'; ctx.strokeStyle = '#2B2420';
    const state = { drawing: false, hasDrawn: false };
    Signature.contexts[canvasId] = { ctx, canvas, state };
    const getPos = e => {
      const rect = canvas.getBoundingClientRect();
      const scaleX = canvas.width / rect.width, scaleY = canvas.height / rect.height;
      const p = e.touches ? e.touches[0] : e;
      return { x: (p.clientX - rect.left) * scaleX, y: (p.clientY - rect.top) * scaleY };
    };
    const start = e => { e.preventDefault(); state.drawing = true; state.hasDrawn = true; const p = getPos(e); ctx.beginPath(); ctx.moveTo(p.x, p.y); };
    const move = e => { if (!state.drawing) return; e.preventDefault(); const p = getPos(e); ctx.lineTo(p.x, p.y); ctx.stroke(); };
    const end = () => { state.drawing = false; };
    canvas.addEventListener('mousedown', start);
    canvas.addEventListener('mousemove', move);
    window.addEventListener('mouseup', end);
    canvas.addEventListener('touchstart', start, { passive: false });
    canvas.addEventListener('touchmove', move, { passive: false });
    canvas.addEventListener('touchend', end);
  },
  clear(canvasId) {
    const e = Signature.contexts[canvasId];
    if (!e) return;
    e.ctx.clearRect(0, 0, e.canvas.width, e.canvas.height);
    e.state.hasDrawn = false;
  },
  isBlank(canvasId) { const e = Signature.contexts[canvasId]; return !e || !e.state.hasDrawn; },

  openSigModal(loanId) {
    const loan = Data.state.loans.find(l => l.id === loanId);
    if (!loan) return;
    Signature.currentLoanId = loanId;
    Signature.clear('sigCanvasA'); Signature.clear('sigCanvasB');
    const isLend = loan.direction === CONST.LOAN_DIR.LEND;
    document.getElementById('sigLabelA').textContent = 'Chữ ký Bên cho vay (A)' + (isLend ? ' — Bạn' : '');
    document.getElementById('sigLabelB').textContent = 'Chữ ký Bên vay (B)' + (!isLend ? ' — Bạn' : '');
    document.getElementById('sigModal').classList.remove('hidden');
  },
  closeSigModal() { document.getElementById('sigModal').classList.add('hidden'); Signature.currentLoanId = null; },
  confirmPrintWithSignatures() {
    if (!Signature.currentLoanId) return;
    Signature.printLoanPaper(Signature.currentLoanId);
    Signature.closeSigModal();
  },

  printLoanPaper(loanId) {
    const loan = Data.state.loans.find(l => l.id === loanId);
    if (!loan) return;
    const isLend = loan.direction === CONST.LOAN_DIR.LEND;
    document.getElementById('pDate').textContent = Util.formatDateVN(Util.todayStr());
    document.getElementById('pNameA').textContent = isLend ? 'Tuấn' : loan.counterpartyName;
    document.getElementById('pNameB').textContent = isLend ? loan.counterpartyName : 'Tuấn';
    document.getElementById('pAmountNum').textContent = Util.formatVND(loan.principal);
    document.getElementById('pAmountWords').textContent = Util.numberToVietnameseWords(loan.principal);
    document.getElementById('pDueDate').textContent = loan.dueDate ? Util.formatDateVN(loan.dueDate) : 'theo thoả thuận giữa hai bên';
    const sigA = document.getElementById('pSigA'), sigB = document.getElementById('pSigB');
    if (!Signature.isBlank('sigCanvasA')) { sigA.src = document.getElementById('sigCanvasA').toDataURL('image/png'); sigA.classList.remove('hidden'); } else sigA.classList.add('hidden');
    if (!Signature.isBlank('sigCanvasB')) { sigB.src = document.getElementById('sigCanvasB').toDataURL('image/png'); sigB.classList.remove('hidden'); } else sigB.classList.add('hidden');
    const area = document.getElementById('printArea');
    area.classList.add('printing');
    window.print();
    setTimeout(() => area.classList.remove('printing'), 500);
  }
};

// ---------- Photos: ảnh hoá đơn lưu trong IndexedDB (tách khỏi khối dữ liệu mã hoá chính để tránh phình to) ----------
const Photos = {
  dbPromise: null,
  openDB() {
    if (Photos.dbPromise) return Photos.dbPromise;
    Photos.dbPromise = new Promise((resolve, reject) => {
      if (!window.indexedDB) { reject(new Error('IndexedDB không khả dụng')); return; }
      const req = indexedDB.open('SoChiTieuPhotos', 1);
      req.onupgradeneeded = () => { req.result.createObjectStore('receipts', { keyPath: 'transactionId' }); };
      req.onsuccess = () => resolve(req.result);
      req.onerror = () => reject(req.error);
    });
    return Photos.dbPromise;
  },
  resizeImage(file, maxDim) {
    return new Promise((resolve, reject) => {
      const img = new Image();
      const reader = new FileReader();
      reader.onload = () => { img.src = reader.result; };
      reader.onerror = reject;
      img.onload = () => {
        let w = img.width, h = img.height;
        if (w > h && w > maxDim) { h = Math.round(h * maxDim / w); w = maxDim; } else if (h > maxDim) { w = Math.round(w * maxDim / h); h = maxDim; }
        const canvas = document.createElement('canvas');
        canvas.width = w; canvas.height = h;
        canvas.getContext('2d').drawImage(img, 0, 0, w, h);
        resolve(canvas.toDataURL('image/jpeg', 0.72));
      };
      img.onerror = reject;
      reader.readAsDataURL(file);
    });
  },
  async attach(transactionId, file) {
    if (!file.type || !file.type.startsWith('image/')) { Toast.show('Chỉ nhận file ảnh', { type: 'err' }); return false; }
    try {
      const dataUrl = await Photos.resizeImage(file, 1200);
      const db = await Photos.openDB();
      await new Promise((resolve, reject) => { const tx = db.transaction('receipts', 'readwrite'); tx.objectStore('receipts').put({ transactionId, dataUrl, savedAt: new Date().toISOString() }); tx.oncomplete = resolve; tx.onerror = () => reject(tx.error); });
      Toast.show('Đã đính kèm ảnh hoá đơn', { type: 'ok' });
      return true;
    } catch (e) {
      console.error('Lỗi lưu ảnh hoá đơn:', e);
      Toast.show('Không lưu được ảnh trên trình duyệt này', { type: 'err' });
      return false;
    }
  },
  async get(transactionId) {
    try { const db = await Photos.openDB(); return await new Promise((resolve, reject) => { const tx = db.transaction('receipts', 'readonly'); const req = tx.objectStore('receipts').get(transactionId); req.onsuccess = () => resolve(req.result || null); req.onerror = () => reject(req.error); }); }
    catch (e) { return null; }
  },
  async remove(transactionId) {
    try { const db = await Photos.openDB(); await new Promise((resolve, reject) => { const tx = db.transaction('receipts', 'readwrite'); tx.objectStore('receipts').delete(transactionId); tx.oncomplete = resolve; tx.onerror = () => reject(tx.error); }); return true; }
    catch (e) { return false; }
  },
  async clearAll() {
    try { const db = await Photos.openDB(); await new Promise((resolve, reject) => { const tx = db.transaction('receipts', 'readwrite'); tx.objectStore('receipts').clear(); tx.oncomplete = resolve; tx.onerror = () => reject(tx.error); }); }
    catch (e) { console.warn('Không xoá được ảnh hoá đơn trong IndexedDB:', e); }
  }
};
Modal.openReceiptSheet = async function (transactionId) {
  const body = Util.h('div', { class: 'text-center' }, Util.h('p', { class: 'text-xs text-muted py-6' }, 'Đang tải...'));
  Modal.open({ title: 'Ảnh hoá đơn', bodyNode: body, actions: [{ label: 'Đóng', onClick: () => Modal.close() }] });
  const existing = await Photos.get(transactionId);
  if (!document.body.contains(body)) return; // modal đã bị đóng trong lúc tải
  Util.clearChildren(body);
  if (existing) {
    body.appendChild(Util.h('img', { src: existing.dataUrl, class: 'w-full rounded-lg mb-3', alt: 'Ảnh hoá đơn đã đính kèm' }));
    body.appendChild(Util.h('button', { class: 'w-full py-2.5 rounded border border-ledger-red text-ledger-red text-sm font-semibold mb-2', onclick: async () => { await Photos.remove(transactionId); Modal.close(); Toast.show('Đã xoá ảnh hoá đơn', { type: 'ok' }); } }, 'Xoá ảnh'));
  } else {
    body.appendChild(Util.h('p', { class: 'text-xs text-muted mb-3' }, 'Chưa có ảnh hoá đơn cho giao dịch này.'));
  }
  const fileInput = Util.h('input', { type: 'file', accept: 'image/*', class: 'hidden', onchange: async e => { const f = e.target.files[0]; if (!f) return; const ok = await Photos.attach(transactionId, f); if (ok) Modal.openReceiptSheet(transactionId); } });
  body.appendChild(fileInput);
  body.appendChild(Util.h('button', { class: 'w-full py-2.5 rounded bg-indigo text-cream text-sm font-semibold', onclick: () => fileInput.click() }, existing ? 'Thay ảnh khác' : 'Đính kèm ảnh'));
};

// ---------- PWA: manifest đã liên kết ở <head>; đăng ký service worker MỘT CÁCH TUỲ CHỌN nếu có sẵn ----------
// Lưu ý: trình duyệt chỉ cho phép service worker khi trang được phục vụ qua http(s), không hoạt động khi mở
// trực tiếp file HTML (file://) trên máy — đây là giới hạn bảo mật của trình duyệt, không phải của ứng dụng.
const PWA = {
  init() {
    if ('serviceWorker' in navigator && location.protocol.indexOf('http') === 0) {
      navigator.serviceWorker.register('./sw.js').catch(() => { /* không có sw.js cạnh file, hoặc môi trường không hỗ trợ — bỏ qua, không ảnh hưởng chức năng chính */ });
    }
  }
};

// ---------- App: khởi động, gắn sự kiện toàn cục, tự khoá khi nền/không hoạt động, xoá dữ liệu an toàn ----------
const App = {
  // Trước đây init() chạy thẳng Security.init(). Giờ có thêm một "cổng" Session ở trước: nếu chưa
  // cấu hình Appwrite (APPWRITE_CONFIG còn placeholder), Session.init() trả về true ngay lập tức và
  // hành vi hệt như bản cũ. Nếu đã cấu hình mà chưa đăng nhập, màn hình đăng nhập hiện ra và chờ —
  // continueAfterLogin() được Session tự gọi lại sau khi đăng nhập xong.
  async init() {
    const canProceed = await Session.init();
    if (!canProceed) return;
    await App.continueAfterLogin();
  },

  async continueAfterLogin() {
    document.getElementById('lockScreen').classList.remove('hidden');
    document.getElementById('authScreen').classList.add('hidden');
    if (Session.isLoggedIn()) await Migration.inheritLegacyLocalDataIfAny();

    await Security.init();
    const verLabel = document.getElementById('appVersionLabel');
    if (verLabel) verLabel.textContent = `Phiên bản ${CONST.APP_VERSION} · ` + (Session.isLoggedIn() ? 'Dữ liệu mã hoá trên máy này, có đồng bộ lên tài khoản của bạn' : 'Dữ liệu chỉ lưu trên máy này, không gửi lên máy chủ nào');
    const lockNote = document.getElementById('lockFooterNote');
    if (lockNote) lockNote.textContent = Session.isLoggedIn() ? 'Dữ liệu được mã hoá ngay trên máy bằng mã PIN của bạn. PIN này tách riêng với mật khẩu tài khoản.' : 'Dữ liệu được mã hoá ngay trên máy bằng mã PIN của bạn. Không có máy chủ nào lưu hay xem được dữ liệu này.';
    Signature.setup('sigCanvasA');
    Signature.setup('sigCanvasB');
    PWA.init();
    if (Session.isLoggedIn()) { await Profile.fetch(); UI.applyProfileNameToHeader(); }
    UI.renderAccountSection();

    document.getElementById('histSearch').addEventListener('input', Util.debounce(() => { History.visibleCount = History.pageSize; History.render(); }, 200));

    ['mousedown', 'keydown', 'touchstart', 'scroll'].forEach(evt => document.addEventListener(evt, () => { if (Security.hasLoadedData) Security.resetInactivityTimer(); }, { passive: true }));
    document.addEventListener('visibilitychange', () => { if (document.hidden && Security.hasLoadedData) Security.lockApp(); });
    window.addEventListener('online', () => { if (Session.isLoggedIn() && Security.hasLoadedData) Sync.syncNow({ silent: true }); });

    // Đồng bộ nhiều tab (chỉ áp dụng khi lưu bằng localStorage): tab khác lưu dữ liệu mới thì tab này thử
    // giải mã bằng khoá phiên hiện có và làm mới Dashboard. Nếu giải mã lỗi (VD: PIN vừa đổi ở tab kia) thì bỏ qua lặng lẽ.
    window.addEventListener('storage', async (e) => {
      if (e.key !== Store.scopedKey(CONST.STORAGE_KEYS.SECURE) || !e.newValue || Store.hasCloudStorage) return;
      if (!Security.hasLoadedData || !Security.sessionKey) return;
      try {
        const parsed = JSON.parse(e.newValue);
        Data.state = Data.migrateIfNeeded(await Crypto_.decryptJSON(Security.sessionKey, parsed.iv, parsed.data));
        UI.renderDashboard();
        Toast.show('Đã đồng bộ dữ liệu từ tab khác', { type: 'info' });
      } catch (err) { /* khoá phiên không còn khớp (VD: PIN vừa đổi ở tab kia) — bỏ qua, tab tự khoá lại khi cần */ }
    });
  },

  async afterFirstUnlock() {
    UI.renderWalletSelect('walletId');
    Tx.resetForm();
    await Recurring.generateDue();
    UI.showTab(0);
    if (Session.isLoggedIn()) {
      await Migration.checkAndOfferAfterUnlock();
      Sync.syncNow({ silent: true });
    }
  },

  clearDataFlow() { Modal.openWipeConfirmSheet(); },

  async wipeEverything() {
    await Store.remove(CONST.STORAGE_KEYS.SECURE);
    await Store.remove(CONST.STORAGE_KEYS.SALT);
    await Store.remove(CONST.STORAGE_KEYS.ATTEMPTS);
    await Store.remove(CONST.STORAGE_KEYS.LEGACY_PIN);
    await Store.remove(CONST.STORAGE_KEYS.LEGACY_TX);
    await Store.remove(CONST.STORAGE_KEYS.LEGACY_LOANS);
    await Store.remove(CONST.STORAGE_KEYS.LEGACY_BUDGET);
    await Photos.clearAll();
    Security.sessionKey = null; Security.salt = null; Security.hasLoadedData = false;
    Data.state = null;
  }
};

App.init();

</script>
</body>
</html>
