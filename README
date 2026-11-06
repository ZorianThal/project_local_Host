# project_local_Host
Сервер: 
# Мультихост: один сервер раздаёт ВСЁ, что лежит в папке hosts/.
#
# 1. Кладите проекты в hosts/<имя>/index.html (+ css/js/картинки, что угодно).
# 2. Запустите: python server.py
# 3. Откройте http://localhost:8000 - увидите список всех проектов.
#
# Публичную ссылку наружу поднимаем отдельной командой в терминале, когда нужно:
#   ssh -R 80:localhost:8000 serveo.net

import http.server, os, urllib.parse  # всё нужное - из стандартной библиотеки

PORT = 8000            # на этом порту работает сайт
HOSTS_DIR = "hosts"      # папка, куда студенты кладут свои проекты


class MultiHostHandler(http.server.SimpleHTTPRequestHandler):
    def do_GET(self):  # вызывается автоматически на каждый запрос браузера
        self.send_project_list() if self.path in ("/", "") else super().do_GET()
        # ^ если открыли главную - показываем список проектов, иначе отдаём файл как обычно

    def send_project_list(self):  # страница со списком всех папок внутри hosts/
        names = sorted(d for d in os.listdir(HOSTS_DIR) if os.path.isdir(f"{HOSTS_DIR}/{d}"))
        items = "".join(f'<li><a href="/{n}/">{n}</a></li>' for n in names) \
            or "<li>Пусто - добавьте проект в hosts/</li>"
        body = f'<meta charset="utf-8"><h1>Проекты студентов</h1><ul>{items}</ul>'.encode("utf-8")
        self.send_response(200)
        self.send_header("Content-Type", "text/html; charset=utf-8")  # без этого браузер путает кодировку
        self.send_header("Content-Length", str(len(body)))
        self.end_headers()
        self.wfile.write(body)

    def translate_path(self, path):  # "/проект/файл" -> реальный путь hosts/проект/файл
        path = urllib.parse.unquote(path.split("?")[0].split("#")[0])
        project, _, rest = path.strip("/").partition("/")       # делим путь на имя проекта и остальное
        base = os.path.abspath(f"{HOSTS_DIR}/{project}")          # папка именно этого проекта
        full = os.path.normpath(os.path.join(base, rest))
        return full if full.startswith(base) else base          # защита от выхода через "../"


os.makedirs(HOSTS_DIR, exist_ok=True)              # создаём hosts/, если его ещё нет
print(f"Локальная ссылка: http://localhost:{PORT}")
http.server.ThreadingHTTPServer.allow_reuse_address = True  # не падать, если порт только что освободился
http.server.ThreadingHTTPServer(("0.0.0.0", PORT), MultiHostHandler).serve_forever()  # сам сервер, навсегда

#Хост самого сервера на сервонет - ssh -R 80:localhost:8000 serveo.net

Страница для теста 
<!doctype html>
<html>
<head>
<meta charset="utf-8">
<title>Магазин</title>
<style>
  body { font-family: sans-serif; max-width: 480px; margin: 30px auto; padding: 0 15px; color: #222; }
  h1 { font-size: 22px; margin-bottom: 4px; }
  .muted { color: #777; font-size: 14px; margin-bottom: 20px; }
  .product { display: flex; justify-content: space-between; align-items: center;
             border-bottom: 1px solid #eee; padding: 12px 0; }
  .price { color: #555; }
  button { padding: 6px 12px; border: 1px solid #ccc; border-radius: 6px;
           background: #f7f7f7; cursor: pointer; }
  button:hover { background: #ededed; }
  #cart-count { font-weight: bold; }
  form { margin-top: 10px; }
  input { display: block; width: 100%; margin: 6px 0; padding: 6px; box-sizing: border-box; }
  .toast { margin-top: 14px; padding: 10px; border-radius: 6px; background: #f0f0f0; font-size: 14px; }
</style>
</head>
<body>

<h1>Книжный магазин</h1>
<p class="muted">В корзине: <span id="cart-count">0</span> товара(ов)</p>

<div class="product">
  <span>«Чистый код»</span>
  <span class="price">990 ₽</span>
  <button onclick="addToCart(1)">В корзину</button>
</div>
<div class="product">
  <span>«Как устроен интернет»</span>
  <span class="price">650 ₽</span>
  <button onclick="addToCart(2)">В корзину</button>
</div>
<div class="product">
  <span>«Python с нуля»</span>
  <span class="price">1190 ₽</span>
  <button onclick="addToCart(3)">В корзину</button>
</div>

<h3>Промокод</h3>
<input id="promo" placeholder="Введите код">
<button onclick="applyPromo()">Применить</button>

<h3>Вход в личный кабинет</h3>
<form onsubmit="return login(event)">
  <input id="email" placeholder="Email">
  <input id="password" type="password" placeholder="Пароль">
  <button type="submit">Войти</button>
</form>

<button onclick="checkout()" style="margin-top:20px;">Оформить заказ</button>

<div class="toast" id="toast">Жмите на кнопки и смотрите в консоль сервера</div>

<script>
let cartCount = 0;

function show(text) {
  document.getElementById("toast").textContent = text;
}

function addToCart(id) {
  cartCount++;
  document.getElementById("cart-count").textContent = cartCount;
  fetch(`/cart/add?id=${id}`);                 // обычная ссылка с параметрами - видно в консоли и в адресе
  show("Товар #" + id + " добавлен (отправлен GET-запрос)");
}

function applyPromo() {
  const code = document.getElementById("promo").value;
  fetch(`/promo?code=${encodeURIComponent(code)}`);
  show("Промокод отправлен в адресе запроса - его видно всем");
}

function login(event) {
  event.preventDefault();                       // не даём странице перезагрузиться
  const email = document.getElementById("email").value;
  const password = document.getElementById("password").value;
  fetch("/login", {
    method: "POST",
    body: `email=${encodeURIComponent(email)}&password=${encodeURIComponent(password)}`
  });
  show("Отправлен POST-запрос - в адресе скрыт, но в консоли сервера виден целиком");
  return false;
}

function checkout() {
  fetch("/checkout", { method: "POST", body: `items=${cartCount}` });
  show("Заказ оформлен POST-запросом на сумму из " + cartCount + " товара(ов)");
}
</script>

</body>
</html>
