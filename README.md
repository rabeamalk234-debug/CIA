<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>GTA Intelligence Portal</title>
    <style>
        :root {
            --primary: #00d2ff;
            --accent: #38bdf8;
            --glass: rgba(15, 23, 42, 0.85);
            --text: #f8fafc;
        }

        body {
            font-family: 'Segoe UI', system-ui, sans-serif;
            background: radial-gradient(circle at top right, #1e293b, #0f172a);
            color: var(--text);
            margin: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            overflow: hidden;
        }

        /* الحاوية الزجاجية الكبيرة */
        .glass-card {
            background: var(--glass);
            backdrop-filter: blur(20px);
            border: 1px solid rgba(255, 255, 255, 0.1);
            border-radius: 30px;
            padding: 40px;
            width: 450px;
            box-shadow: 0 25px 50px rgba(0,0,0,0.6);
            text-align: center;
            position: relative;
        }

        .hidden { display: none; }

        /* المدخلات */
        input {
            width: 100%;
            padding: 15px;
            margin: 10px 0;
            background: rgba(255, 255, 255, 0.05);
            border: 1px solid rgba(255, 255, 255, 0.1);
            border-radius: 12px;
            color: white;
            box-sizing: border-box;
        }

        .btn {
            background: linear-gradient(135deg, #0ea5e9, #2563eb);
            border: none;
            padding: 15px;
            border-radius: 12px;
            color: white;
            width: 100%;
            cursor: pointer;
            font-weight: bold;
            transition: 0.3s;
        }
        .btn:hover { transform: translateY(-2px); box-shadow: 0 10px 20px rgba(37, 99, 235, 0.4); }

        /* البصمة */
        .finger-icon { font-size: 70px; margin: 30px 0; cursor: pointer; transition: 0.3s; filter: drop-shadow(0 0 10px var(--primary)); }

        /* الصناديق الاستخباراتية */
        .grid {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 15px;
            margin-top: 25px;
        }
        .box-card {
            background: rgba(255, 255, 255, 0.03);
            border: 1px solid rgba(255, 255, 255, 0.1);
            padding: 20px;
            border-radius: 18px;
            cursor: pointer;
            transition: 0.3s;
        }
        .box-card:hover { background: rgba(56, 189, 248, 0.1); border-color: var(--primary); transform: scale(1.05); }

        /* نافذة الـ AI */
        #ai-window {
            position: fixed;
            bottom: 20px;
            left: 20px;
            width: 300px;
            background: var(--glass);
            border: 1px solid var(--primary);
            border-radius: 20px;
            display: flex;
            flex-direction: column;
            padding: 15px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.5);
        }
        #chat-log { height: 150px; overflow-y: auto; font-size: 13px; text-align: right; margin-bottom: 10px; }
        .ai-msg { color: var(--primary); margin-bottom: 5px; }
    </style>
</head>
<body>

    <div class="glass-card">
        <div id="step-1">
            <img src="https://cdn-icons-png.flaticon.com/512/702/702455.png" width="80" style="margin-bottom: 15px;">
            <h2>استخبارات GTA</h2>
            <input type="text" placeholder="معرف العميل">
            <input type="password" placeholder="كلمة المرور">
            <button class="btn" onclick="navTo('step-2')">تسجيل الدخول</button>
        </div>

        <div id="step-2" class="hidden">
            <h3>التحقق البيومتري</h3>
            <div class="finger-icon" onclick="navTo('step-3')">🧬</div>
            <p style="color: #94a3b8;">اضغط للمسح الضوئي</p>
        </div>

        <div id="step-3" class="hidden">
            <h3>لوحة الاستخبارات</h3>
            <div class="grid">
                <div class="box-card" onclick="showData(0)">📁<br>العصابات</div>
                <div class="box-card" onclick="showData(1)">⚔️<br>الأسلحة</div>
                <div class="box-card" onclick="showData(2)">👮<br>الشرطة</div>
                <div class="box-card" onclick="showData(3)">💰<br>المهمات</div>
            </div>
        </div>

        <div id="step-4" class="hidden">
            <h2 id="info-title" style="color: var(--primary);"></h2>
            <p id="info-body" style="background: rgba(0,0,0,0.2); padding: 15px; border-radius: 10px; line-height: 1.6;"></p>
            <button class="btn" onclick="navTo('step-3')">رجوع</button>
        </div>
    </div>

    <div id="ai-window" class="hidden">
        <div style="font-weight: bold; font-size: 12px; margin-bottom: 10px; color: var(--primary);">🤖 مساعد الاستخبارات</div>
        <div id="chat-log">
            <div class="ai-msg">أهلاً أيها العميل. اسألني عن أي هدف في لوس سانتوس؟</div>
        </div>
        <input type="text" id="ai-in" placeholder="اسأل الـ AI..." onkeypress="handleAI(event)" style="padding: 8px; font-size: 12px;">
    </div>

    <script>
        // بيانات استخبارات GTA
        const gtaIntel = [
            { t: "عصابة البالاس", b: "تسيطر على منطقة شرق لوس سانتوس. يرتدون اللون الأرجواني. عدوهم اللدود: عصابة العائلات." },
            { t: "مخازن الأسلحة", b: "تم رصد شحنة صواريخ في الميناء الجنوبي. الأمان مكثف جداً في المنطقة." },
            { t: "تحركات الشرطة", b: "تم رصد زيادة في دوريات الـ LSPD بالقرب من بنك Maze. حالة الاستعداد: برتقالي." },
            { t: "المهمات الكبرى", b: "المهمة القادمة: السطو على كازينو دايموند. يتطلب فريقاً من 4 أشخاص." }
        ];

        function navTo(id) {
            document.querySelectorAll('.glass-card > div').forEach(d => d.classList.add('hidden'));
            document.getElementById(id).classList.remove('hidden');
            if(id === 'step-3') document.getElementById('ai-window').classList.remove('hidden');
        }

        function showData(i) {
            document.getElementById('info-title').innerText = gtaIntel[i].t;
            document.getElementById('info-body').innerText = gtaIntel[i].b;
            navTo('step-4');
        }

        function handleAI(e) {
            if (e.key === 'Enter') {
                const input = document.getElementById('ai-in');
                const log = document.getElementById('chat-log');
                const val = input.value;
                if(!val) return;

                log.innerHTML += `<div style="color:white; margin-bottom:5px;">أنت: ${val}</div>`;
                
                let reply = "لم أجد معلومات عن هذا في ملفات الاستخبارات.";
                gtaIntel.forEach(item => {
                    if(val.includes(item.t.split(' ')[0]) || val.includes(item.t.split(' ')[1])) {
                        reply = item.b;
                    }
                });

                setTimeout(() => {
                    log.innerHTML += `<div class="ai-msg">AI: ${reply}</div>`;
                    log.scrollTop = log.scrollHeight;
                }, 500);
                input.value = "";
            }
        }
    </script>
</body>
</html># CIA
