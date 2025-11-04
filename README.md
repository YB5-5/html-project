[index.html.html](https://github.com/user-attachments/files/23341514/index.html.html)
<!DOCTYPE html>
<html lang="ar">
<head>
  <meta charset="UTF-8">
  <title>عداد الطواف التجريبي - GPS</title>
  <style>
    body {
      font-family: "Cairo", sans-serif;
      background-color: #f8f9fa;
      text-align: center;
      direction: rtl;
    }
    .card {
      margin: 50px auto;
      background: white;
      padding: 30px;
      width: 360px;
      border-radius: 20px;
      box-shadow: 0 4px 10px rgba(0,0,0,0.2);
    }
    h1 {
      color: #007bff;
    }
    .count {
      font-size: 60px;
      color: #28a745;
      margin: 20px 0;
    }
  </style>
</head>
<body>
  <div class="card">
    <h1>🏃‍♂️ عداد الطواف التجريبي</h1>
    <p>عدد اللفات المكتملة:</p>
    <div id="laps" class="count">0</div>
    <p id="status">جاري تحديد موقعك...</p>
  </div>

  <script>
    // 📍 نقطة البداية
    const startPoint = { lat: 24.8280278, lon: 46.7434167 };
    // 📍 نقطة النهاية
    const endPoint = { lat: 24.8277222, lon: 46.7431667 };
    // 📍 المركز التجريبي (تقريباً في منتصف المسافة)
    const centerPoint = { lat: (24.8280278 + 24.8277222) / 2, lon: (46.7434167 + 46.7431667) / 2 };

    let started = false;
    let lastAngle = null;
    let totalAngle = 0;
    let laps = 0;

    // دالة حساب المسافة بين نقطتين (بالمتر)
    function distance(lat1, lon1, lat2, lon2) {
      const R = 6371000; // نصف قطر الأرض بالمتر
      const dLat = (lat2 - lat1) * Math.PI / 180;
      const dLon = (lon2 - lon1) * Math.PI / 180;
      const a = Math.sin(dLat/2)**2 +
                Math.cos(lat1 * Math.PI/180) * Math.cos(lat2 * Math.PI/180) *
                Math.sin(dLon/2)**2;
      const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
      return R * c;
    }

    if ("geolocation" in navigator) {
      navigator.geolocation.watchPosition(updatePosition, showError, {
        enableHighAccuracy: true,
        maximumAge: 0,
        timeout: 5000
      });
    } else {
      document.getElementById("status").textContent = "GPS غير مدعوم على هذا الجهاز.";
    }

    function updatePosition(pos) {
      const lat = pos.coords.latitude;
      const lon = pos.coords.longitude;
      document.getElementById("status").textContent = `📍 موقعك الحالي: ${lat.toFixed(5)}, ${lon.toFixed(5)}`;

      // تحقق من الاقتراب من نقطة البداية
      const distToStart = distance(lat, lon, startPoint.lat, startPoint.lon);
      const distToEnd = distance(lat, lon, endPoint.lat, endPoint.lon);

      if (!started && distToStart < 10) { // ضمن 10 متر
        started = true;
        totalAngle = 0;
        laps = 0;
        alert("🚀 بدأ العدّ من نقطة البداية!");
      }

      if (started) {
        const angle = Math.atan2(lon - centerPoint.lon, lat - centerPoint.lat);

        if (lastAngle !== null) {
          let diff = angle - lastAngle;
          if (diff > Math.PI) diff -= 2 * Math.PI;
          if (diff < -Math.PI) diff += 2 * Math.PI;
          totalAngle += diff;

          // تحقق إذا المستخدم رجع إلى نقطة النهاية بعد دورة كاملة
          if (Math.abs(totalAngle) >= 2 * Math.PI && distToEnd < 15) {
            laps++;
            totalAngle = 0;
            document.getElementById("laps").textContent = laps;
            alert(`✅ تم إكمال لفة رقم ${laps}!`);
          }
        }

        lastAngle = angle;
      }
    }

    function showError(err) {
      document.getElementById("status").textContent = "⚠️ لم يتم تحديد الموقع. فعّل GPS.";
    }
  </script>
</body>
</html>
