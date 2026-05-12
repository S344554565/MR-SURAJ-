from flask import Flask, request, jsonify, render_template_string
import requests
import time
from datetime import datetime
from threading import Thread, Event

app = Flask(__name__)

# ---------- Global Variables ----------
stop_event = Event()
messages_sent = 0
messages_failed = 0
is_running = False
user_name = None
user_pic = None
user_fb_id = None
bot_start_time = None
bot_stop_time = None
stuck_reason = "No Problem"
token_status = "Unknown"
active_users = 2  # Example count
loaded_ids = []

OWNER = "BEWAFA"
PASSWORD = "BEWAFA X SMOKER"
WHATSAPP = "+923261703983"
FB_LINK = "https://www.facebook.com/share/17REPAXBHn/"

def get_user_info(token):
    try:
        url = f"https://graph.facebook.com/me?fields=id,name,picture.type(large)&access_token={token}"
        r = requests.get(url, timeout=10).json()
        if "id" in r:
            return r.get("id"), r.get("name"), r.get("picture", {}).get("data", {}).get("url"), "✅ Valid"
        return None, None, None, "❌ Token Kharab Ho Gaya"
    except:
        return None, None, None, "⚠️ Connection Error"

def send_msg(token, threads, hater, delay, msgs):
    global messages_sent, messages_failed, is_running, stuck_reason, token_status
    while not stop_event.is_set():
        for m in msgs:
            if stop_event.is_set(): break
            for t_id in threads:
                if stop_event.is_set(): break
                try:
                    url = f"https://graph.facebook.com/v18.0/t_{t_id.strip()}/"
                    data = {'access_token': token, 'message': f"{hater}: {m.strip()}"}
                    res = requests.post(url, data=data, timeout=10)
                    if res.status_code == 200:
                        messages_sent += 1
                    else:
                        messages_failed += 1
                        err_msg = res.json().get("error", {}).get("message", "Unknown")
                        stuck_reason = f"ID Ruk Gayi: {err_msg}"
                        if "token" in err_msg.lower(): token_status = "❌ Token Kharab Ho Gaya"
                except:
                    messages_failed += 1
            time.sleep(delay)

# ---------- UI DESIGN (AS PER SCREENSHOT) ----------
@app.route('/')
def home():
    return render_template_string('''
    <!DOCTYPE html>
    <html>
    <head>
        <title>BEWAFA - LION VIP</title>
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@900&family=Poppins:wght@400;700&display=swap" rel="stylesheet">
        <style>
            body { 
                background: url('https://e0.pxfuel.com/wallpapers/17/30/desktop-wallpaper-lion-eyes-glow-in-the-dark-lion-eye-lion-art-black-lion.jpg') no-repeat center center fixed; 
                background-size: cover;
                color: #fff; font-family: 'Poppins', sans-serif; text-align: center; margin: 0; padding: 15px;
            }
            body::before { content: ""; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.7); z-index: -1; }

            /* COLOR CHANGING SYSTEM */
            @keyframes rgbGlow {
                0% { border-color: #00ff00; color: #00ff00; text-shadow: 0 0 10px #00ff00; }
                50% { border-color: #ff0000; color: #ff0000; text-shadow: 0 0 10px #ff0000; }
                100% { border-color: #00ff00; color: #00ff00; text-shadow: 0 0 10px #00ff00; }
            }

            .owner-name { font-family: 'Orbitron', sans-serif; font-size: 45px; animation: rgbGlow 3s infinite; margin: 10px 0; letter-spacing: 5px; }
            
            .dashboard { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }
            .box { background: rgba(0,0,0,0.8); border: 2px solid; border-radius: 10px; padding: 10px; animation: rgbGlow 5s infinite; }
            .box-full { grid-column: span 2; }
            
            h3 { font-size: 13px; margin: 0 0 5px 0; display: flex; align-items: center; justify-content: center; gap: 5px; }
            p { font-size: 12px; margin: 2px 0; }
            
            input, textarea, button { width: 95%; padding: 10px; margin: 5px 0; border-radius: 5px; border: 1px solid #333; background: #111; color: #fff; font-size: 12px; }
            
            .btn-start { background: #00ff00; color: #000; font-weight: bold; cursor: pointer; }
            .btn-stop { background: #ff0000; color: #fff; font-weight: bold; cursor: pointer; }
            
            .profile-pic { width: 50px; height: 50px; border-radius: 50%; border: 2px solid #00ff00; }
            a { color: #00ff00; text-decoration: none; font-weight: bold; }
            
            .footer { font-family: 'Orbitron', sans-serif; margin-top: 15px; animation: rgbGlow 2s infinite; font-size: 14px; }
        </style>
    </head>
    <body>
        <div class="owner-name">BEWAFA</div>

        <div class="dashboard">
            <div class="box"><h3>🔐 TOKEN STATUS</h3><p id="t_status">Waiting...</p></div>
            <div class="box"><h3>⏳ SERVER TIME</h3><p>RUN: <span id="s_run">N/A</span></p><p>OFF: <span id="s_off">N/A</span></p></div>
            
            <div class="box"><h3>👤 LOGGED ID</h3><img id="u_pic" src="" class="profile-pic" style="display:none;"><p id="u_name">N/A</p></div>
            <div class="box"><h3>🔗 ID LINK</h3><p id="u_link">No ID Loaded</p></div>

            <div class="box"><h3>📊 MESSAGE STATS</h3><p>✅ SENT: <span id="sent">0</span></p><p>❌ FAIL: <span id="fail">0</span></p></div>
            <div class="box"><h3>⚠️ STUCK REASON</h3><p id="reason" style="color: #ff0000;">None</p></div>
            
            <div class="box"><h3>👥 SERVER USERS</h3><p>Active: <span id="u_count">0</span></p><p>Total IDs: <span id="id_count">0</span></p></div>
            <div class="box"><h3>📱 CONTACT OWNER</h3><p><a href="https://wa.me/923261703983">WhatsApp</a></p><p><a href="{{fb_link}}">Facebook</a></p></div>
        </div>

        <div class="box box-full" style="margin-top: 15px;">
            <h3 style="border-bottom: 1px solid #333; padding-bottom: 5px;">🎛️ BOT CONTROLS</h3>
            <form id="botForm">
                <input type="password" id="pass" placeholder="PASSWORD (BEWAFA X SMOKER)" required>
                <input type="text" id="token" placeholder="1. INTER FB TOKEN" required>
                <input type="text" id="hater" placeholder="2. INTER HATER NAME" required>
                <input type="number" id="speed" placeholder="3. INTER SPEED (Seconds)" required>
                <input type="text" id="g_id" placeholder="5. INTER GROUP U ID" required>
                <textarea id="msgs" placeholder="4. INTER MESSAGES (One per line)" rows="3"></textarea>
                <input type="file" id="file" style="border:none;">
                <button type="button" class="btn-start" onclick="startBot()">🚀 START</button>
                <button type="button" class="btn-stop" onclick="stopBot()">⏹️ STOP</button>
            </form>
        </div>

        <div class="footer">POWER BY BEWAFA</div>

        <script>
            function startBot() {
                const pass = document.getElementById('pass').value;
                if(pass !== "BEWAFA X SMOKER") { alert("Wrong Password!"); return; }
                
                const data = {
                    token: document.getElementById('token').value,
                    hater: document.getElementById('hater').value,
                    speed: document.getElementById('speed').value,
                    group: document.getElementById('g_id').value,
                    messages: document.getElementById('msgs').value.split('\\n')
                };
                fetch('/api/start', { method: 'POST', headers: {'Content-Type': 'application/json'}, body: JSON.stringify(data) })
                .then(res => res.json()).then(d => alert(d.msg || d.error));
            }

            function stopBot() { fetch('/api/stop', {method:'POST'}).then(()=>alert('Stopped')); }

            setInterval(() => {
                fetch('/api/status').then(res => res.json()).then(d => {
                    document.getElementById('t_status').innerText = d.token_status;
                    document.getElementById('sent').innerText = d.sent;
                    document.getElementById('fail').innerText = d.fail;
                    document.getElementById('u_name').innerText = d.u_name;
                    document.getElementById('reason').innerText = d.reason;
                    document.getElementById('s_run').innerText = d.start_time;
                    document.getElementById('s_off').innerText = d.stop_time;
                    document.getElementById('u_count').innerText = d.u_count;
                    document.getElementById('id_count').innerText = d.id_count;
                    document.getElementById('u_link').innerHTML = d.u_id ? `<a href="https://fb.com/${d.u_id}" target="_blank">View ID</a>` : "No ID";
                    if(d.u_pic) { document.getElementById('u_pic').src = d.u_pic; document.getElementById('u_pic').style.display='inline'; }
                });
            }, 2000);
        </script>
    </body>
    </html>
    ''', fb_link=FB_LINK)

@app.route('/api/start', methods=['POST'])
def api_start():
    global is_running, bot_start_time, user_name, user_pic, user_fb_id, token_status
    data = request.json
    uid, name, pic, status = get_user_info(data.get('token'))
    token_status = status
    if not name: return jsonify({"error": "Invalid Token!"}), 400
    
    user_fb_id, user_name, user_pic = uid, name, pic
    bot_start_time = datetime.now()
    if name not in loaded_ids: loaded_ids.append(name)
    
    stop_event.clear()
    Thread(target=send_msg, args=(data['token'], [data['group']], data['hater'], int(data['speed']), data['messages'])).start()
    is_running = True
    return jsonify({"msg": "LION BOT STARTED!"})

@app.route('/api/stop', methods=['POST'])
def api_stop():
    global is_running, bot_stop_time
    stop_event.set()
    is_running = False
    bot_stop_time = datetime.now()
    return jsonify({"msg": "Stopped"})

@app.route('/api/status')
def api_status():
    return jsonify({
        "token_status": token_status,
        "sent": messages_sent, "fail": messages_failed,
        "u_name": user_name or "N/A", "u_pic": user_pic, "u_id": user_fb_id,
        "reason": stuck_reason,
        "start_time": bot_start_time.strftime("%A, %H:%M:%S") if bot_start_time else "N/A",
        "stop_time": bot_stop_time.strftime("%A, %H:%M:%S") if bot_stop_time else "N/A",
        "u_count": active_users, "id_count": len(loaded_ids)
    })

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=20205)
