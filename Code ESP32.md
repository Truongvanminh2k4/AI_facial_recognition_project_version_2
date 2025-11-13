#include <WiFi.h>
#include <Firebase_ESP_Client.h>
#include "addons/TokenHelper.h"
#include "addons/RTDBHelper.h"

//Định nghĩa nút nhấn
#define BUTTON_PIN 21


// ================= Cấu hình WiFi =================
#define WIFI_SSID "?????_EXT"
#define WIFI_PASSWORD "luongloan7282@@"

//==================API kết nối Python=============
const char* host = "192.168.1.13";   // IP máy chạy Python
const uint16_t port = 5000;
WiFiClient client;

// ================= Firebase =================
#define DATABASE_URL "https://aifacialrecognitionproje-3ed64-default-rtdb.firebaseio.com/"
#define DATABASE_SECRET "7EoByCLbigS7Ht8Z1Crei9qh3QRXmNOvBZoPJ5AY"

// ================= Biến toàn cục =================
FirebaseData fbdo;
FirebaseAuth auth;
FirebaseConfig config;

int giu;
uint8_t quet_van_tay = 0;
uint8_t ma_van_tay;
uint8_t flag = 0;
String so_id;
int so_id_can_xoa;
String ds_sinh_vien[50] = {};
String ds_xoa_sinh_vien[50] = {};
int idx = 1;

//----------------Nguyên mẫu hàm-------------------------

// ================= Setup =================
void setup() {
  Serial.begin(9600);

  pinMode(BUTTON_PIN, INPUT_PULLUP);  // Bật điện trở kéo lên nội bộ

  // Kết nối WiFi
  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  Serial.print("Dang ket noi WiFi");
  while (WiFi.status() != WL_CONNECTED) {
    Serial.print(".");
    delay(300);
  }
  Serial.println();
  Serial.print("Da ket noi voi IP: ");
  Serial.println(WiFi.localIP());

  // Cấu hình Firebase
  config.database_url = DATABASE_URL;
  config.signer.tokens.legacy_token = DATABASE_SECRET;  // Dùng Database Secret

  Firebase.reconnectNetwork(true);
  Firebase.begin(&config, &auth);

  Serial.println("Firebase connected!");


  //Lưu trường thông tin sinh viên vào mảng
  if (Firebase.RTDB.getShallowData(&fbdo, "/DangKySinhVien")) {
  FirebaseJson *json = fbdo.to<FirebaseJson *>();
  
  // Lấy iterator
  size_t count = json->iteratorBegin();
  String key, value;
  
  for (size_t i = 0; i < count; i++) {
    int type = 0;
    json->iteratorGet(i, type, key, value);
    ds_sinh_vien[idx] = "/DangKySinhVien/" + key + "/id";
    ds_xoa_sinh_vien[idx++] = "/DangKySinhVien/" + key;
    //Serial.println("Key: " + key);
  }
  
  json->iteratorEnd();
}
else {
  Serial.println("Lỗi đọc Firebase: " + fbdo.errorReason());
}
  Serial.println(ds_sinh_vien[1]); 
  Serial.println(ds_sinh_vien[2]); 
  Serial.println(ds_xoa_sinh_vien[1]); 
  Serial.println(ds_xoa_sinh_vien[2]);


  
}

// ================= Loop =================
void loop() {
  //===Kết nối Python===
  if (!client.connected()) {
    if (client.connect(host, port)) {
      Serial.println("Đã kết nối server Python");
    } else {
      Serial.println("Không kết nối được server");
      delay(1000);
      return;
    }
  }
  //===Nhận data từ python=== 
  if (client.available()) {
    String msg = client.readStringUntil('\n');
    msg.trim();
    if (msg.length() > 0) {
      Serial.print("Nhận từ Python: ");
      Serial.println(msg);
      giu = msg.toInt();
      Firebase.RTDB.setInt(&fbdo,"/DiemDanh/id", giu);
    }
  }
  delay(50); // Chống nhiễu (debounce nhẹ)


  //===Đọc FireBase===
  if (Firebase.RTDB.getInt(&fbdo, "/xoa_sinhvien/id")){
    if (fbdo.dataType() == "string"){
      so_id = fbdo.stringData();
      Serial.print("so_id_can_xoa: ");
      Serial.println(so_id_can_xoa);
    }else{
      Serial.print("Loi doc du lieu: ");
      Serial.println(fbdo.errorReason());
    }
    
    so_id_can_xoa = so_id.toInt();

    //Duyệt để tìm
    Firebase.RTDB.getString(&fbdo, ds_sinh_vien[so_id_can_xoa]);
    String id_sinh_vien = fbdo.stringData();
    Serial.print("so_id_sinh_vien: ");
    Serial.println(id_sinh_vien);

    //===XÓa Firebase===
    if (Firebase.RTDB.deleteNode(&fbdo, ds_xoa_sinh_vien[so_id_can_xoa])) {
      Serial.println("✅ Da xoa thanh cong!");
      Firebase.RTDB.deleteNode(&fbdo, "/xoa_sinhvien/id");
    } else {
      Serial.println("❌ Loi xoa: " + fbdo.errorReason());
    }
  }


}
