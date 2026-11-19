<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>نبهني - نظام تنبيه السيارات الذكي</title>
    <link href="https://fonts.googleapis.com/css2?family=Cairo:wght@400;600;700;800&display=swap" rel="stylesheet">
    <!-- مكتبة توليد الباركود -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>
    <style>
        :root {
            --primary: #e63946;
            --primary-hover: #d62828;
            --bg: #f4f6f9;
            --card: #ffffff;
            --text: #1d3557;
            --muted: #6c757d;
            --success: #2a9d8f;
        }
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: 'Cairo', sans-serif;
        }
        body {
            background-color: var(--bg);
            color: var(--text);
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            padding: 20px;
        }
        .container {
            background: var(--card);
            width: 100%;
            max-width: 420px;
            border-radius: 24px;
            padding: 30px 24px;
            box-shadow: 0 12px 35px rgba(0, 0, 0, 0.08);
            text-align: center;
        }
        .logo {
            font-size: 28px;
            font-weight: 800;
            color: var(--primary);
            margin-bottom: 8px;
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 8px;
        }
        .subtitle {
            color: var(--muted);
            font-size: 14px;
            margin-bottom: 24px;
        }
        .car-badge {
            display: inline-block;
            background: #edf2f4;
            color: var(--text);
            font-weight: 700;
            padding: 6px 16px;
            border-radius: 12px;
            font-size: 16px;
            margin-bottom: 20px;
            border: 1px solid #d8e2dc;
        }
        input[type="text"], textarea {
            width: 100%;
            padding: 14px;
            border: 2px solid #e0e0e0;
            border-radius: 14px;
            font-size: 15px;
            margin-bottom: 14px;
            text-align: center;
            transition: border-color 0.2s;
        }
        input[type="text"]:focus, textarea:focus {
            outline: none;
            border-color: var(--primary);
        }
        .btn {
            width: 100%;
            padding: 14px;
            border: none;
            border-radius: 14px;
            font-size: 16px;
            font-weight: 700;
            cursor: pointer;
            transition: transform 0.1s, background 0.2s;
            margin-bottom: 10px;
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 8px;
        }
        .btn:active {
            transform: scale(0.98);
        }
        .btn-primary {
            background: var(--primary);
            color: white;
        }
        .btn-primary:hover {
            background: var(--primary-hover);
        }
        .btn-outline {
            background: #f8f9fa;
            color: var(--text);
            border: 2px solid #e9ecef;
        }
        .btn-outline:hover {
            background: #e9ecef;
            border-color: #ced4da;
        }
        .btn-success {
            background: var(--success);
            color: white;
        }
        .qr-box {
            margin: 20px auto;
            padding: 16px;
            background: #ffffff;
            border: 2px dashed #cbd5e1;
            border-radius: 16px;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
        }
        #qrcode {
            margin: 10px auto;
        }
        .status-msg {
            margin-top: 15px;
            padding: 12px;
            border-radius: 12px;
            font-size: 14px;
            font-weight: 600;
            display: none;
        }
        .status-success {
            background: #d8f3dc;
            color: #1b4332;
            display: block;
        }
        .status-error {
            background: #ffe5ec;
            color: #9b2226;
            display: block;
        }
        .hidden {
            display: none !important;
        }
        .divider {
            height: 1px;
            background: #eee;
            margin: 20px 0;
        }
    </style>
</head>
<body>

<div class="container">
    <div class="logo">🚗 نبهني</div>

    <!-- ========================================== -->
    <!-- الواجهة 1: واجهة مالك السيارة (الإعداد والباركود) -->
    <!-- ========================================== -->
    <div id="owner-view" class="hidden">
        <p class="subtitle">لوحة تحكم مالك السيارة - فعّل التنبيهات واطبع الباركود</p>

        <input type="text" id="car-input" placeholder="اكتب رقم سيارتك هنا (مثال: CAR-001)" value="CAR-001">
        
        <button class="btn btn-primary" onclick="setupCar()">
            🔔 تفعيل الإشعارات وتوليد الباركود
        </button>

        <div id="owner-status" class="status-msg"></div>

        <div id="qr-section" class="hidden">
            <div class="divider"></div>
            <p style="font-size: 14px; font-weight: 700; margin-bottom: 6px;">الباركود الخاص بسيارتك جاهز!</p>
            <p style="font-size: 12px; color: var(--muted);">احفظ الصورة واطبعها وضعها على زجاج السيارة</p>
            
            <div class="qr-box">
                <div id="qrcode"></div>
                <div id="qr-label" style="font-weight: 800; font-size: 14px; margin-top: 8px;"></div>
            </div>

            <button class="btn btn-success" onclick="downloadQR()">
                📥 حفظ صورة الباركود (PNG)
            </button>
            <button class="btn btn-outline" onclick="testAlertView()">
                👁️ معاينة صفحة الزائر (اختبار)
            </button>
        </div>
    </div>

    <!-- ========================================== -->
    <!-- الواجهة 2: واجهة الشخص بالشارع (إرسال التنبيه) -->
    <!-- ========================================== -->
    <div id="visitor-view" class="hidden">
        <p class="subtitle">يرجى اختيار سبب تنبيه صاحب هذه السيارة</p>
        <div class="car-badge">رقم السيارة: <span id="display-car-id">---</span></div>

        <button class="btn btn-outline" onclick="sendAlert('🚫 سيارتك تغلق الطريق أو تعيق الحركة!')">
            🚫 سيارتك تغلق الطريق
        </button>
        <button class="btn btn-outline" onclick="sendAlert('💡 أضواء سيارتك لا تزال مشتعلة!')">
            💡 الأضواء مشتعلة
        </button>
        <button class="btn btn-outline" onclick="sendAlert('🪟 نافذة أو باب السيارة مفتوح!')">
            🪟 نافذة أو باب مفتوح
        </button>
        <button class="btn btn-outline" onclick="sendAlert('⚠️ يوجد طارئ بخصوص سيارتك، يرجى الحضور فوراً!')">
            ⚠️ طارئ / يرجى الحضور فوراً
        </button>

        <div class="divider"></div>
        <textarea id="custom-msg" rows="2" placeholder="أو اكتب رسالة مخصصة لصاحب السيارة..."></textarea>
        <button class="btn btn-primary" onclick="sendCustomAlert()">
            📤 إرسال التنبيه الآن
        </button>

        <div id="visitor-status" class="status-msg"></div>
    </div>
</div>

<script>
    // قراءة البيانات من الرابط (URL Parameters)
    const urlParams = new URLSearchParams(window.location.search);
    const carParam = urlParams.get('car');
    const modeParam = urlParams.get('mode');

    const ownerView = document.getElementById('owner-view');
    const visitorView = document.getElementById('visitor-view');

    // تحديد أي واجهة ستظهر
    if (modeParam === 'alert' && carParam) {
        // إظهار واجهة الزائر (الشخص بالشارع)
        visitorView.classList.remove('hidden');
        document.getElementById('display-car-id').textContent = decodeURIComponent(carParam);
    } else {
        // إظهار واجهة المالك
        ownerView.classList.remove('hidden');
        const savedCar = localStorage.getItem('nabihni_car');
        if (savedCar) {
            document.getElementById('car-input').value = savedCar;
            setupCar(true);
        }
    }

    // دالة إعداد السيارة وتوليد الباركود للمالك
    function setupCar(isAuto = false) {
        const carId = document.getElementById('car-input').value.trim();
        if (!carId) {
            alert('يرجى إدخال رقم السيارة أولاً');
            return;
        }

        localStorage.setItem('nabihni_car', carId);

        // طلب إذن الإشعارات في المتصفح
        if ("Notification" in window) {
            Notification.requestPermission().then(permission => {
                const statusEl = document.getElementById('owner-status');
                if (permission === "granted") {
                    statusEl.className = 'status-msg status-success';
                    statusEl.innerHTML = `✅ تم تفعيل الإشعارات بنجاح لسيارة (${carId})!<br>أبقِ هذه الصفحة مفتوحة أو في الخلفية لاستلام التنبيهات فوراً.`;
                    startListening(carId);
                } else {
                    statusEl.className = 'status-msg status-error';
                    statusEl.textContent = '⚠️ يرجى السماح بالإشعارات من إعدادات المتصفح لتصلك التنبيهات.';
                }
            });
        }

        // توليد رابط الباركود الخاص بالزوار
        const baseUrl = window.location.origin + window.location.pathname;
        const alertUrl = `${baseUrl}?car=${encodeURIComponent(carId)}&mode=alert`;

        // رسم الباركود
        const qrContainer = document.getElementById('qrcode');
        qrContainer.innerHTML = '';
        new QRCode(qrContainer, {
            text: alertUrl,
            width: 180,
            height: 180,
            colorDark: "#1d3557",
            colorLight: "#ffffff",
            correctLevel: QRCode.CorrectLevel.H
        });

        document.getElementById('qr-label').textContent = `امسح للتنبيه - ${carId}`;
        document.getElementById('qr-section').classList.remove('hidden');
    }

    // الاستماع الفوري للتنبيهات القادمة (عبر خدمة ntfy المجانية بدون سيرفر)
    let eventSource = null;
    function startListening(carId) {
        if (eventSource) eventSource.close();
        const topic = 'nabihni_iq_' + carId.replace(/[^a-zA-Z0-9]/g, '_');
        
        eventSource = new EventSource(`https://ntfy.sh/${topic}/sse`);
        eventSource.onmessage = function(e) {
            const data = JSON.parse(e.data);
            if (data.event === 'message') {
                // إظهار إشعار المتصفح
                new Notification(`🚨 تنبيه لسيارتك (${carId})`, {
                    body: data.message,
                    icon: 'https://cdn-icons-png.flaticon.com/512/744/744465.png'
                });
                // تشغيل صوت تنبيه بسيط
                const audio = new Audio('https://actions.google.com/sounds/v1/alarms/beep_short.ogg');
                audio.play().catch(() => {});
            }
        };
    }

    // دالة إرسال التنبيه من واجهة الزائر
    function sendAlert(message) {
        const statusEl = document.getElementById('visitor-status');
        statusEl.className = 'status-msg';
        statusEl.style.display = 'block';
        statusEl.style.background = '#e2e8f0';
        statusEl.textContent = '⏳ جاري إرسال التنبيه لصاحب السيارة...';

        const topic = 'nabihni_iq_' + carParam.replace(/[^a-zA-Z0-9]/g, '_');

        fetch(`https://ntfy.sh/${topic}`, {
            method: 'POST',
            body: message,
            headers: {
                'Title': `تنبيه بخصوص السيارة ${carParam}`
            }
        })
        .then(res => {
            if (res.ok) {
                statusEl.className = 'status-msg status-success';
                statusEl.textContent = '✅ تم إرسال التنبيه بنجاح! سيصل الإشعار لصاحب السيارة فوراً.';
            } else {
                throw new Error('فشل الإرسال');
            }
        })
        .catch(() => {
            statusEl.className = 'status-msg status-error';
            statusEl.textContent = '❌ حدث خطأ أثناء الإرسال، يرجى المحاولة مرة أخرى.';
        });
    }

    function sendCustomAlert() {
        const msg = document.getElementById('custom-msg').value.trim();
        if (!msg) {
            alert('يرجى كتابة رسالة أولاً');
            return;
        }
        sendAlert(msg);
        document.getElementById('custom-msg').value = '';
    }

    // تحميل الباركود كصورة PNG
    function downloadQR() {
        const img = document.querySelector('#qrcode img');
        if (img) {
            const link = document.createElement('a');
            link.href = img.src;
            link.download = `QR-${document.getElementById('car-input').value}.png`;
            link.click();
        }
    }

    // زر لمعاينة واجهة الزائر بصفحة جديدة
    function testAlertView() {
        const carId = document.getElementById('car-input').value.trim();
        const baseUrl = window.location.origin + window.location.pathname;
        window.open(`${baseUrl}?car=${encodeURIComponent(carId)}&mode=alert`, '_blank');
    }
</script>

</body>
</html>
