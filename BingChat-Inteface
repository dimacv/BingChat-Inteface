import configparser

config = configparser.ConfigParser()
config.read('config.ini')

proxy = config.get('Settings', 'proxy', fallback=None)
cookie_file = config.get('Settings', 'cookie_file')
protocol = config.get('Settings', 'protocol', fallback='http')

from flask import Flask, request, render_template_string, send_file
from EdgeGPT.EdgeUtils import Query, Cookie

# Функция для отправки запроса к Bing Chat
def send_request_to_bingchat(user_input):
    q = Query(user_input,
              style="precise",  # or: 'creative', 'balanced', 'precise'
              cookie_file=cookie_file,
              proxy=proxy)

    return [q.output, q.suggestions]

app = Flask(__name__)

def check_auth(username, password):
    # Замените на свою логику проверки имени пользователя и пароля
    return username == 'dimacv' and password == 'vadgra'

def save_to_file(data):
    with open('history.txt', 'a') as f:
        f.write("___________________________________________________\n")
        f.write("  ВОПРОС: " + data[0] + "\n")
        f.write("  ОТВЕТ: " + data[1] + "\n")

def read_from_file():
    with open('history.txt', 'r') as f:
        return f.read()

def clear_file():
    open('history.txt', 'w').close()

@app.route('/', methods=['GET', 'POST'])
def index():
    if request.authorization and check_auth(request.authorization.username, request.authorization.password):
        result = ["", ""]
        show_history = False
        history = ""
        if request.method == 'POST':
            if 'data' in request.form:
                data = request.form['data']
                result = send_request_to_bingchat(data)
                save_to_file([data, result[0]])
            elif 'show_history' in request.form:
                show_history = True
                history = read_from_file()
            elif 'hide_history' in request.form:
                show_history = False
            elif 'clear_history' in request.form:
                clear_file()
        return render_template_string('''
            <form method="post" onsubmit="showLoadingMessage()">
                Введите данные:<br>
                <textarea name="data" rows="4" cols="50"></textarea><br>
                <input type="submit" value="Отправить">
            </form>
            <h2>Ответ:</h2>
            <pre id="answer">{{result[0]}}</pre>
            <h2>Рекомендуемые дальнейшие запросы:</h2>
            <p>{{result[1]}}</p>
            {% if not show_history %}
                <form method="post">
                    <input type="hidden" name="show_history" value="1">
                    <input type="submit" value="Показать историю">
                </form>
            {% else %}
                <form method="post">
                    <input type="hidden" name="hide_history" value="1">
                    <input type="submit" value="Скрыть историю">
                </form>
                <h2>История:</h2>
                <pre>{{history}}</pre>
                <a href="/download">Скачать историю</a><br>
                <form method="post">
                    <input type="hidden" name="clear_history" value="1">
                    <input type="submit" value="Очистить историю">
                </form>
            {% endif %}
            <script>
                function showLoadingMessage() {
                    document.getElementById("answer").innerHTML = "Формируется ответ от Bing Chat";
                }
            </script>
        ''', result=result, show_history=show_history, history=history)
    return 'Авторизация не пройдена', 401, {'WWW-Authenticate': 'Basic realm="Login Required"'}

@app.route('/download')
def download():
    return send_file('history.txt', as_attachment=True)

if __name__ == '__main__':
    if protocol == 'https':
        app.run(host='0.0.0.0', port=443, ssl_context=(config.get('Settings', 'cert_file'), config.get('Settings', 'key_file')))
    else:
        app.run(host='0.0.0.0', port=8080)
