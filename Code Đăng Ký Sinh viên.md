<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Đăng ký thông tin sinh viên</title>
  <style>
    body { font-family: Arial, sans-serif; background: #f5f5f5; margin:0; padding:20px; }
    .form-container { max-width: 420px; margin:auto; background:#fff; padding:20px; border-radius:10px; box-shadow:0 2px 8px rgba(0,0,0,0.1); }
    h2 { text-align:center; margin-bottom:15px; }
    label { display:block; margin-top:10px; font-weight:bold; }
    input, select, button { width:100%; padding:8px; margin-top:5px; border:1px solid #ccc; border-radius:5px; box-sizing:border-box; }
    button { cursor:pointer; background:#4CAF50; color:white; border:none; padding:10px; }
    button:hover { background:#45a049; }
  </style>
</head>
<body>
  <div class="form-container">
    <h2>Đăng ký thông tin sinh viên</h2>
    <form id="studentForm">
      <label for="name">Tên:</label>
      <input type="text" id="name" name="name" required>

      <label for="mssv">MSSV:</label>
      <input type="text" id="mssv" name="mssv" required>

      <label for="class">Lớp:</label>
      <input type="text" id="class" name="class" required>

      <label for="subject">Môn học:</label>
      <select id="subject" name="subject" required>
        <option value="">--Chọn môn--</option>
        <option>IOT</option>
      </select>

      <label for="day">Thứ (2 - 8):</label>
      <select id="day" name="day" required>
        <option value="">--Chọn thứ--</option>
        <option value="2">2</option>
        <option value="3">3</option>
        <option value="4">4</option>
        <option value="5">5</option>
        <option value="6">6</option>
        <option value="7">7</option>
        <option value="8">8</option>
      </select>

      <label for="shift">Tiết:</label>
      <select id="shift" name="shift" required>
        <option value="">--Chọn tiết--</option>
        <option>1</option>
        <option>4</option>
        <option>7</option>
        <option>10</option>
      </select>

      <button type="submit" id="submitBtn">Gửi thông tin</button>
    </form>
  </div>

  <!-- Firebase SDK -->
  <script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/12.5.0/firebase-app.js";
    import { getAnalytics } from "https://www.gstatic.com/firebasejs/12.5.0/firebase-analytics.js";
    import { getDatabase, ref, push, set } from "https://www.gstatic.com/firebasejs/12.5.0/firebase-database.js";

    // Cấu hình Firebase mới
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

    // Khởi tạo Firebase
    const app = initializeApp(firebaseConfig);
    const analytics = getAnalytics(app);
    const db = getDatabase(app);

    // --- Xử lý form ---
    const form = document.getElementById("studentForm");
    form.addEventListener("submit", async (e) => {
      e.preventDefault();

      const nameValue = document.getElementById("name").value.toUpperCase();
      const classValue = document.getElementById("class").value.toUpperCase();
      const mssvValue = document.getElementById("mssv").value;
      const subjectValue = document.getElementById("subject").value;
      const thuValue = document.getElementById("day").value;
      const tietValue = document.getElementById("shift").value;
      const thoigian = new Date().toLocaleString("vi-VN");

      const data = {
        ten: nameValue,
        mssv: mssvValue,
        lop: classValue,
        mon: subjectValue,
        thu: thuValue,
        tiet: tietValue,
        thoigian: thoigian
      };

      try {
        const newRef = push(ref(db, "DangKySinhVien"));
        await set(newRef, data);

        alert("✅ Đăng ký thành công! Đang chuyển hướng...");
        // Chuyển hướng đến Google Form
        window.location.href = "https://docs.google.com/forms/d/e/1FAIpQLSfnhGsgLTVcx0LmyHBBbfQgtqXCSRhCbpuIvRR6lsH1T8xHkg/viewform";
      } catch (err) {
        console.error("Lỗi lưu Firebase:", err);
        alert("❌ Có lỗi xảy ra khi lưu dữ liệu!");
      }
    });
  </script>
</body>
</html>
