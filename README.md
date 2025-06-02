# 安裝必要的庫
!pip install flask ngrok==5.0.0

# 匯入必要的模組
from flask import Flask, render_template, request, redirect, url_for
from datetime import datetime, timedelta
from google.colab.output import eval_js
import threading
import time

# 設定 ngrok 隧道
# 在 Colab 中，您可以使用 eval_js 來獲取公開的 URL
print(eval_js("google.colab.kernel.proxyPort(5000)"))

# 初始化 Flask 應用程式
app = Flask(__name__)

# 儲存睡眠紀錄的列表
sleep_records = []

# 網站主頁的路由
@app.route('/', methods=['GET', 'POST'])
def index():
    message = None
    if request.method == 'POST':
        date = request.form['date']
        start_time = request.form['start_time']
        end_time = request.form['end_time']

        try:
            fmt = "%H:%M"
            start = datetime.strptime(start_time, fmt)
            end = datetime.strptime(end_time, fmt)

            # 如果結束時間早於開始時間，表示跨越午夜
            if end < start:
                end += timedelta(days=1)

            duration = end - start
            # 計算睡眠時數並四捨五入到小數點後兩位
            hours = round(duration.total_seconds() / 3600, 2)

            # 將紀錄添加到列表中
            sleep_records.append({
                "日期": date,
                "就寢": start_time,
                "起床": end_time,
                "睡眠時數": hours
            })
            message = "✅ 睡眠紀錄已新增！"
        except ValueError:
            message = "⚠️ 時間格式錯誤，請使用 HH:MM 格式！"

    return render_template('index.html', records=sleep_records, message=message)

# 建立一個簡單的 HTML 模板文件
# 在 Colab 中，您可以直接在這裡定義 HTML 內容
# 或者將其保存為 'index.html' 文件
html_template = """
<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <title>睡眠時數計算器</title>
    <style>
        body { font-family: Arial; max-width: 600px; margin: auto; padding: 20px; }
        h1 { color: #333; }
        form { margin-bottom: 20px; }
        input, button { padding: 8px; margin: 5px 0; width: 100%; }
        .message { color: green; }
        .error { color: red; }
        table { width: 100%; border-collapse: collapse; margin-top: 20px; }
        th, td { border: 1px solid #ccc; padding: 8px; text-align: center; }
    </style>
</head>
<body>
    <h1>睡眠時數計算器</h1>
    {% if message %}
        <p class="{{ 'error' if '⚠️' in message else 'message' }}">{{ message }}</p>
    {% endif %}
    <form method="post">
        <label>日期：<input type="date" name="date" required></label><br>
        <label>就寢時間（HH:MM）：<input type="time" name="start_time" required></label><br>
        <label>起床時間（HH:MM）：<input type="time" name="end_time" required></label><br>
        <button type="submit">送出</button>
    </form>

    <h2>睡眠紀錄</h2>
    <table>
        <tr>
            <th>日期</th>
            <th>就寢</th>
            <th>起床</th>
            <th>睡眠時數</th>
        </tr>
        {% for record in records %}
        <tr>
            <td>{{ record['日期'] }}</td>
            <td>{{ record['就寢'] }}</td>
            <td>{{ record['起床'] }}</td>
            <td>{{ record['睡眠時數'] }} 小時</td>
        </tr>
        {% endfor %}
    </table>
</body>
</html>
"""

# 在 Colab 中創建一個模擬的 'templates' 資料夾並寫入 HTML 內容
import os
if not os.path.exists('templates'):
    os.makedirs('templates')

with open('templates/index.html', 'w', encoding='utf-8') as f:
    f.write(html_template)

# 在一個單獨的線程中運行 Flask 伺服器，以免阻塞 Colab 執行
def run_flask():
    app.run(port=5000, debug=True, use_reloader=False)

# 啟動 Flask 伺服器線程
flask_thread = threading.Thread(target=run_flask)
flask_thread.daemon = True
flask_thread.start()

# 讓 Colab 保持運行，以便 Flask 應用程式可以運行
# 您可能需要手動停止執行
while True:
    time.sleep(1)
