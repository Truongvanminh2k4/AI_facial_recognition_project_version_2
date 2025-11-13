<!doctype html>
<html lang="vi">
<head>
  <meta charset="utf-8">
  <title>Quản lý sinh viên & Điểm danh</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; font-family: Arial, sans-serif; }
    body { display: flex; min-height: 100vh; background-color: #f4f6f8; }

    .sidebar {
      width: 220px;
      background: #2b7a0b;
      color: white;
      padding-top: 20px;
      display: flex;
      flex-direction: column;
    }
    .sidebar h2 { text-align: center; margin-bottom: 20px; font-size: 20px; }
    .sidebar a {
      padding: 12px 20px;
      text-decoration: none;
      color: white;
      display: block;
      transition: 0.3s;
    }
    .sidebar a:hover, .sidebar a.active { background: #3fa015; }

    .main { flex: 1; padding: 20px; overflow-x: auto; }
    h2 { color: #2b7a0b; margin-bottom: 15px; }

    table {
      width: 100%; border-collapse: collapse; background: white;
      border-radius: 8px; overflow: hidden;
      box-shadow: 0 2px 6px rgba(0,0,0,0.15);
    }
    th, td { padding: 10px 12px; border-bottom: 1px solid #eee; text-align: left; }
    th { background: #2b7a0b; color: white; }
    tr:hover { background: #eaffea; }

    button {
      background: #2b7a0b; border: none; color: white;
      padding: 8px 14px; border-radius: 6px; cursor: pointer; margin-right: 5px;
    }
    button:hover { background: #3fa015; }

    .schedule-table {
      border-collapse: collapse; width: 100%;
      background: #1b1b1b; color: white; text-align: center;
      border-radius: 8px; overflow: hidden;
    }
    .schedule-table th, .schedule-table td {
      border: 1px solid #3a3a3a; padding: 10px; height: 50px;
    }
    .schedule-table th { background: #009879; }
    .schedule-cell {
      background-color: #003b2f; border-radius: 6px;
      padding: 5px; cursor: pointer;
    }
    .schedule-cell:hover { background-color: #015f4b; }
    .schedule-cell .subject { font-weight: bold; color: #00ffcc; }
  </style>
</head>

<body>
  <div class="sidebar">
    <h2>📚 Menu</h2>
    <a href="#" id="overviewBtn" class="active">Tổng quan sinh viên</a>
    <a href="#" id="attendanceBtn">📅 Quản lý điểm danh</a>
  </div>

  <div class="main">
    <div id="overviewSection">
      <h2>📋 Tổng quan sinh viên</h2>
      <table id="studentTable">
        <thead>
          <tr>
            <th>STT</th><th>ID</th><th>Tên</th><th>MSSV</th><th>Lớp</th><th>Môn học</th>
            <th>Thứ</th><th>Tiết</th><th>Hành động</th>
          </tr>
        </thead>
        <tbody id="studentBody"><tr><td colspan="9" style="text-align:center;">Đang tải dữ liệu...</td></tr></tbody>
      </table>
    </div>

    <div id="attendanceSection" style="display:none;">
      <h2>📅 Quản lý điểm danh</h2>
      <table class="schedule-table" id="scheduleTable">
        <thead>
          <tr>
            <th>Tiết / Thứ</th>
            <th>Thứ 2</th><th>Thứ 3</th><th>Thứ 4</th><th>Thứ 5</th>
            <th>Thứ 6</th><th>Thứ 7</th><th>CN</th>
          </tr>
        </thead>
        <tbody id="scheduleBody"></tbody>
      </table>
    </div>
  </div>

  <script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/12.5.0/firebase-app.js";
    import { getDatabase, ref, onValue, set, remove } from "https://www.gstatic.com/firebasejs/12.5.0/firebase-database.js";
    import { getAnalytics } from "https://www.gstatic.com/firebasejs/12.5.0/firebase-analytics.js";

    const firebaseConfig = {
      apiKey: "AIzaSyByKYepQZaor6Opszs-sxveKNvuptQIkpM",
      authDomain: "aifacialrecognitionproje-3ed64.firebaseapp.com",
      databaseURL: "https://aifacialrecognitionproje-3ed64-default-rtdb.firebaseio.com",
      projectId: "aifacialrecognitionproje-3ed64",
      storageBucket: "aifacialrecognitionproje-3ed64.firebasestorage.app",
      messagingSenderId: "11220301418",
      appId: "1:11220301418:web:f807dbb21b560241d83c13",
      measurementId: "G-H6XZ3YXDZY"
    };

    const app = initializeApp(firebaseConfig);
    const analytics = getAnalytics(app);
    const db = getDatabase(app);

    const studentRef = ref(db, "DangKySinhVien");
    const attendanceRef = ref(db, "DiemDanh");

    let studentData = [];
    let currentAttendanceID = null;
    let attendanceHistory = {};

    const tbody = document.getElementById("studentBody");
    let scheduleBody = document.getElementById("scheduleBody");

    // --- Tải danh sách sinh viên ---
    onValue(studentRef, (snapshot) => {
      const data = snapshot.val();
      tbody.innerHTML = "";
      studentData = [];

      if (!data) {
        tbody.innerHTML = `<tr><td colspan="9" style="text-align:center;">Chưa có sinh viên nào đăng ký.</td></tr>`;
        return;
      }

      // 🔹 Chuyển object -> mảng + sắp xếp theo tên (theo thứ tự chữ cái)
      const sortedStudents = Object.entries(data)
        .map(([key, sv]) => ({ key, ...sv }))
        .sort((a, b) => a.ten.localeCompare(b.ten, 'vi', { sensitivity: 'base' }));

      let stt = 1;
      sortedStudents.forEach(({ key, ...sv }) => {
        const storedId = sv.id || "";
        tbody.innerHTML += `
          <tr>
            <td>${stt}</td>
            <td>${storedId}</td>
            <td>${sv.ten || ""}</td>
            <td>${sv.mssv || ""}</td>
            <td>${sv.lop || ""}</td>
            <td>${sv.mon || ""}</td>
            <td>${sv.thu || ""}</td>
            <td>${sv.tiet || ""}</td>
            <td><button onclick="xoaSinhVien('${key}','${sv.ten || ""}')">🗑️ Xóa</button></td>
          </tr>`;
        studentData.push({ stt, key, ...sv });
        stt++;
      });

      capNhatLich();
    });

    // --- Lắng nghe điểm danh ---
    onValue(attendanceRef, (snapshot) => {
      const data = snapshot.val();
      if (!data || !data.id) return;

      const newID = String(data.id);
      const thoigian = data.thoigian || new Date().toLocaleString("vi-VN");

      if (newID !== currentAttendanceID) {
        currentAttendanceID = newID;
        attendanceHistory[newID] = thoigian;

        const sv = studentData.find(s => String(s.id) === newID);
        if (sv) {
          capNhatBangDiemDanh(newID, thoigian);
          return;
        }

        const num = Number(newID);
        const svByStt = Number.isInteger(num) ? studentData.find(s => s.stt === num) : null;
        if (svByStt) {
          if (!svByStt.id) {
            set(ref(db, `DangKySinhVien/${svByStt.key}/id`), newID)
              .then(() => {
                svByStt.id = newID;
                attendanceHistory[newID] = thoigian;
                alert(`✅ Đã lưu ID "${newID}" cho sinh viên ${svByStt.ten}`);
                capNhatBangDiemDanh(newID, thoigian);
              });
          } else capNhatBangDiemDanh(newID, thoigian);
          return;
        }

        alert(`⚠️ Không tìm thấy sinh viên có ID "${newID}".`);
      }
    });

    function capNhatBangDiemDanh(id, time) {
      const sv = studentData.find(s => String(s.id) === String(id));
      if (!sv) return;
      alert(`✅ ${sv.ten} (${sv.mssv}) đã điểm danh lúc ${time}`);
      xemChiTiet(sv.mon, sv.thu, sv.tiet);
    }

    function capNhatLich() {
      let html = "";
      for (let tiet = 1; tiet <= 16; tiet++) {
        html += `<tr><th>Tiết ${tiet}</th>`;
        for (let thu = 2; thu <= 8; thu++) {
          const lop = studentData.find(s => Number(s.thu) === thu && Number(s.tiet) === tiet);
          html += lop
            ? `<td><div class="schedule-cell" onclick="xemChiTiet('${lop.mon}','${lop.thu}','${lop.tiet}')"><div class="subject">${lop.mon}</div></div></td>`
            : "<td></td>";
        }
        html += "</tr>";
      }
      scheduleBody.innerHTML = html;
    }

    window.xemChiTiet = (mon, thu, tiet) => {
      const danhSach = studentData.filter(s => s.mon === mon && s.thu == thu && s.tiet == tiet);
      const section = document.getElementById("attendanceSection");
      let html = `
        <h2>📘 Sinh viên môn: ${mon} (Thứ ${thu}, Tiết ${tiet})</h2>
        <table><thead><tr>
          <th>STT</th><th>Tên</th><th>MSSV</th><th>Lớp</th><th>ID</th>
          <th>Trạng thái điểm danh</th><th>Thời gian</th>
        </tr></thead><tbody>`;

      danhSach.forEach(sv => {
        const daDD = attendanceHistory[sv.id];
        html += `
          <tr>
            <td>${sv.stt}</td><td>${sv.ten}</td><td>${sv.mssv}</td>
            <td>${sv.lop}</td><td>${sv.id || "(Chưa có)"}</td>
            <td>${daDD ? "✅ Đã điểm danh" : "❌ Chưa điểm danh"}</td>
            <td>${daDD || "--"}</td>
          </tr>`;
      });

      html += `</tbody></table>
        <div style="margin-top:15px;">
          <button onclick="exportExcel('${mon}','${thu}','${tiet}')">📦 Xuất Excel</button>
          <button onclick="quayLaiDiemDanh()">⬅️ Quay lại</button>
        </div>`;
      section.innerHTML = html;
    };

    window.exportExcel = (mon, thu, tiet) => {
      const danhSach = studentData.filter(s => s.mon === mon && s.thu == thu && s.tiet == tiet);
      const rows = danhSach.map(sv => ({
        "Tên sinh viên": sv.ten,
        "MSSV": sv.mssv,
        "Lớp": sv.lop,
        "Môn học": sv.mon,
        "Thứ": sv.thu,
        "Tiết": sv.tiet,
        "ID": sv.id,
        "Trạng thái điểm danh": attendanceHistory[sv.id] ? "✅ Đã điểm danh" : "❌ Chưa điểm danh",
        "Thời gian điểm danh": attendanceHistory[sv.id] || "--"
      }));

      if (!rows.length) return alert("⚠️ Không có dữ liệu để xuất!");

      const ws = XLSX.utils.json_to_sheet(rows);
      const wb = XLSX.utils.book_new();
      XLSX.utils.book_append_sheet(wb, ws, mon);
      const fileName = `DiemDanh_${mon}_Thu${thu}_Tiet${tiet}.xlsx`;
      XLSX.writeFile(wb, fileName);

      remove(attendanceRef)
        .then(() => alert(`✅ Đã xuất ${fileName} và xóa dữ liệu điểm danh Firebase!`));
    };

    window.xoaSinhVien = (key, ten) => {
      if (!confirm(`Bạn có chắc muốn xóa sinh viên "${ten}" không?`)) return;
      remove(ref(db, `DangKySinhVien/${key}`))
        .then(() => alert(`✅ Đã xóa sinh viên "${ten}" khỏi Firebase.`))
        .catch(() => alert("❌ Lỗi khi xóa sinh viên!"));
    };

    window.quayLaiDiemDanh = () => {
      const section = document.getElementById("attendanceSection");
      section.innerHTML = `<h2>📅 Quản lý điểm danh</h2>
      <table class="schedule-table" id="scheduleTable">
        <thead><tr>
          <th>Tiết / Thứ</th><th>Thứ 2</th><th>Thứ 3</th><th>Thứ 4</th><th>Thứ 5</th>
          <th>Thứ 6</th><th>Thứ 7</th><th>CN</th>
        </tr></thead><tbody id="scheduleBody"></tbody></table>`;
      scheduleBody = document.getElementById("scheduleBody");
      capNhatLich();
    };

    // --- Nút chuyển trang ---
    const overviewBtn = document.getElementById("overviewBtn");
    const attendanceBtn = document.getElementById("attendanceBtn");
    const overviewSection = document.getElementById("overviewSection");
    const attendanceSection = document.getElementById("attendanceSection");

    overviewBtn.addEventListener("click", () => {
      overviewBtn.classList.add("active");
      attendanceBtn.classList.remove("active");
      overviewSection.style.display = "block";
      attendanceSection.style.display = "none";
    });

    attendanceBtn.addEventListener("click", () => {
      attendanceBtn.classList.add("active");
      overviewBtn.classList.remove("active");
      overviewSection.style.display = "none";
      attendanceSection.style.display = "block";
      capNhatLich();
    });
  </script>
</body>
</html>
