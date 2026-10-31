<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>IoT Smart Home Dashboard</title>
    <!-- โหลดไลบรารี MQTT สำหรับเบราว์เซอร์ -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/paho-mqtt/1.0.1/mqttws31.min.js"></script>
    <style>
        body { font-family: Arial, sans-serif; text-align: center; background-color: #f4f4f9; padding: 20px; }
        .card { background: white; padding: 20px; margin: 10px; border-radius: 10px; box-shadow: 0 4px 8px rgba(0,0,0,0.1); display: inline-block; width: 150px; }
        h1 { color: #333; }
        .value { font-size: 24px; font-weight: bold; color: #007BFF; }
        button { padding: 15px 30px; font-size: 18px; margin: 10px; cursor: pointer; border: none; border-radius: 5px; color: white; font-weight: bold; }
        .btn-on { background-color: #28a745; }
        .btn-off { background-color: #dc3545; }
    </style>
</head>
<body>

    <h1>🏠 Smart Home Dashboard</h1>
    <p id="status" style="color: gray;">กำลังเชื่อมต่อ MQTT...</p>

    <!-- กรอบแสดงค่าเซ็นเซอร์ -->
    <div>
        <div class="card">
            <h3>🌡️ อุณหภูมิ</h3>
            <p class="value"><span id="temp">--</span> °C</p>
        </div>
        <div class="card">
            <h3>💧 ความชื้น</h3>
            <p class="value"><span id="hum">--</span> %</p>
        </div>
        <div class="card">
            <h3>☀️ แสงสว่าง</h3>
            <p class="value"><span id="light">--</span> lux</p>
        </div>
    </div>

    <!-- ปุ่มควบคุม Relay -->
    <div style="margin-top: 30px;">
        <h3>💡 ควบคุมไฟ (Relay)</h3>
        <button class="btn-on" onclick="sendControl('1')">เปิดไฟ (ON)</button>
        <button class="btn-off" onclick="sendControl('0')">ปิดไฟ (OFF)</button>
    </div>

    <script>
        // 1. ตั้งค่าการเชื่อมต่อ (ใช้พอร์ต 8083 สำหรับเว็บ)
        const broker = "broker.emqx.io";
        const port = 8083; 
        const clientId = "web-dashboard-" + Math.random().toString(16).substr(2, 8);
        
        const client = new Paho.MQTT.Client(broker, port, clientId);

        // 2. เมื่อขาดการเชื่อมต่อ
        client.onConnectionLost = function(responseObject) {
            document.getElementById("status").innerText = "❌ ขาดการเชื่อมต่อ MQTT";
            document.getElementById("status").style.color = "red";
        };

        // 3. เมื่อได้รับข้อมูลจาก ESP32
        client.onMessageArrived = function(message) {
            const topic = message.destinationName;
            const payload = message.payloadString;
            
            if (topic === "pkru/iot/temp") document.getElementById("temp").innerText = payload;
            if (topic === "pkru/iot/hum") document.getElementById("hum").innerText = payload;
            if (topic === "pkru/iot/light") document.getElementById("light").innerText = payload;
        };

        // 4. สั่งเชื่อมต่อ
        client.connect({
            onSuccess: function() {
                document.getElementById("status").innerText = "✅ เชื่อมต่อ MQTT สำเร็จ";
                document.getElementById("status").style.color = "green";
                
                // ติดตาม Topic เพื่อรอรับค่า
                client.subscribe("pkru/iot/temp");
                client.subscribe("pkru/iot/hum");
                client.subscribe("pkru/iot/light");
            }
        });

        // 5. ฟังก์ชันส่งคำสั่งเปิด-ปิด ไปที่ ESP32
        function sendControl(command) {
            const message = new Paho.MQTT.Message(command);
            message.destinationName = "pkru/iot/relay"; // Topic สำหรับ Relay
            client.send(message);
        }
    </script>
</body>
</html>
