<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>بوابة الإدارة - النادي الرياضي</title>
  <style>
    body {
      font-family: system-ui, -apple-system, sans-serif;
      background: #f3f4f6;
      margin: 0;
      padding: 20px;
      color: #111827;
    }
    .card {
      max-width: 420px;
      margin: 60px auto;
      background: #ffffff;
      padding: 24px;
      border-radius: 12px;
      box-shadow: 0 4px 16px rgba(0, 0, 0, 0.08);
    }
    .dashboard {
      max-width: 900px;
      margin: 20px auto;
      background: #ffffff;
      padding: 24px;
      border-radius: 12px;
      box-shadow: 0 4px 16px rgba(0, 0, 0, 0.08);
      display: none;
    }
    label {
      display: block;
      margin-bottom: 6px;
      font-weight: 600;
      font-size: 14px;
    }
    input, select {
      width: 100%;
      padding: 10px 12px;
      margin-bottom: 14px;
      border: 1px solid #d1d5db;
      border-radius: 8px;
      box-sizing: border-box;
      font-size: 15px;
    }
    button {
      padding: 10px 16px;
      border: none;
      border-radius: 8px;
      cursor: pointer;
      font-weight: 600;
      font-size: 14px;
    }
    .btn-primary {
      width: 100%;
      background: #0f766e;
      color: #fff;
    }
    .btn-approve {
      background: #15803d;
      color: #fff;
    }
    .btn-logout {
      background: #b91c1c;
      color: #fff;
    }
    .error {
      color: #b91c1c;
      font-size: 13px;
      margin-bottom: 10px;
      min-height: 18px;
    }
    .topbar {
      display: flex;
      justify-content: space-between;
      align-items: center;
      border-bottom: 1px solid #e5e7eb;
      padding-bottom: 14px;
      margin-bottom: 20px;
      gap: 10px;
      flex-wrap: wrap;
    }
    .player-row {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 12px;
      border: 1px solid #e5e7eb;
      border-radius: 8px;
      margin-bottom: 10px;
      gap: 12px;
      flex-wrap: wrap;
    }
    .badge {
      padding: 4px 10px;
      border-radius: 999px;
      font-size: 12px;
      font-weight: 600;
      background: #fef3c7;
      color: #92400e;
    }
    .badge.approved {
      background: #dcfce7;
      color: #166534;
    }
  </style>
</head>
<body>

  <!-- شاشة تسجيل الدخول -->
  <div id="loginScreen" class="card">
    <h2 style="margin-top: 0;">دخول الإدارة والمدرب</h2>
    <p style="color: #4b5563; font-size: 14px;">
      هذه المساحة مخصصة للمدير والمدرب فقط لحماية بيانات وملفات اللاعبين.
    </p>
    <form id="loginForm">
      <label for="username">اسم الحساب</label>
      <input type="text" id="username" required autocomplete="username" />

      <label for="password">كلمة المرور</label>
      <input type="password" id="password" required autocomplete="current-password" />

      <div id="loginError" class="error"></div>
      <button type="submit" class="btn-primary">تسجيل الدخول</button>
    </form>
  </div>

  <!-- لوحة الإدارة المحمية -->
  <div id="adminDashboard" class="dashboard">
    <div class="topbar">
      <div>
        <h2 style="margin: 0;">ملفات اللاعبين والموافقة الإدارية</h2>
        <small id="currentUserInfo" style="color: #4b5563;"></small>
      </div>
      <button type="button" id="logoutBtn" class="btn-logout">تسجيل الخروج</button>
    </div>

    <div id="playersList"></div>
  </div>

  <script>
    // حسابات تجريبية: غيّر أسماء المستخدمين وكلمات المرور قبل الاستخدام الفعلي
    const ACCOUNTS = {
      "director": { pass: "1234", role: "director", label: "المدير" },
      "coach":    { pass: "0000", role: "coach",    label: "المدرب" }
    };

    const STORAGE_KEY_SESSION = "club_admin_session_v1";
    const STORAGE_KEY_FILES   = "club_player_files_v1";

    const defaultFiles = [
      { id: 1, name: "ملف لاعب 1 - شهادة طبية ووثائق", status: "pending" },
      { id: 2, name: "ملف لاعب 2 - استمارة انخراط", status: "pending" }
    ];

    function getFiles() {
      const saved = localStorage.getItem(STORAGE_KEY_FILES);
      return saved ? JSON.parse(saved) : defaultFiles;
    }

    function saveFiles(files) {
      localStorage.setItem(STORAGE_KEY_FILES, JSON.stringify(files));
    }

    function approveFile(id) {
      const session = JSON.parse(sessionStorage.getItem(STORAGE_KEY_SESSION) || "null");
      if (!session || session.role !== "director") {
        alert("الموافقة على الملفات متاحة لحساب المدير فقط.");
        return;
      }
      const files = getFiles().map(item =>
        item.id === id ? { ...item, status: "approved" } : item
      );
      saveFiles(files);
      renderDashboard(session);
    }

    function renderDashboard(session) {
      document.getElementById("loginScreen").style.display = "none";
      const dash = document.getElementById("adminDashboard");
      dash.style.display = "block";

      document.getElementById("currentUserInfo").textContent =
        "المتصل حالياً: " + session.label + " (" + session.username + ")";

      const listEl = document.getElementById("playersList");
      const files = getFiles();
      listEl.innerHTML = "";

      files.forEach(file => {
        const isApproved = file.status === "approved";
        const row = document.createElement("div");
        row.className = "player-row";

        const infoDiv = document.createElement("div");
        infoDiv.innerHTML =
          "<strong>" + file.name + "</strong><br>" +
          "<span class='badge " + (isApproved ? "approved" : "") + "'>" +
          (isApproved ? "تمت الموافقة" : "بانتظار الموافقة") +
          "</span>";

        row.appendChild(infoDiv);

        if (session.role === "director" && !isApproved) {
          const btn = document.createElement("button");
          btn.className = "btn-approve";
          btn.textContent = "موافقة على الملف";
          btn.onclick = () => approveFile(file.id);
          row.appendChild(btn);
        }

        listEl.appendChild(row);
      });
    }

    function showLogin() {
      sessionStorage.removeItem(STORAGE_KEY_SESSION);
      document.getElementById("adminDashboard").style.display = "none";
      document.getElementById("loginScreen").style.display = "block";
      document.getElementById("loginForm").reset();
      document.getElementById("loginError").textContent = "";
    }

    document.getElementById("loginForm").addEventListener("submit", function (e) {
      e.preventDefault();
      const u = document.getElementById("username").value.trim();
      const p = document.getElementById("password").value;
      const errEl = document.getElementById("loginError");

      const acc = ACCOUNTS[u];
      if (!acc || acc.pass !== p) {
        errEl.textContent = "اسم الحساب أو كلمة المرور غير صحيحة.";
        return;
      }

      const session = { username: u, role: acc.role, label: acc.label };
      sessionStorage.setItem(STORAGE_KEY_SESSION, JSON.stringify(session));
      renderDashboard(session);
    });

    document.getElementById("logoutBtn").addEventListener("click", showLogin);

    // تحقق تلقائي عند فتح الصفحة
    const existingSession = JSON.parse(sessionStorage.getItem(STORAGE_KEY_SESSION) || "null");
    if (existingSession && ACCOUNTS[existingSession.username]) {
      renderDashboard(existingSession);
    } else {
      showLogin();
    }
  </script>
</body>
</html>
