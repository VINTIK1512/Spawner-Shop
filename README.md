# Spawner-Shop
<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Спавнеры Minecraft</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body {
    font-family: -apple-system, Arial, sans-serif;
    background: #0f0f1a; color: #fff; min-height: 100vh;
  }
  header {
    padding: 40px 20px; text-align: center;
    background: #1a1a2e; border-bottom: 1px solid #2a2a40;
  }
  header h1 { font-size: 28px; margin-bottom: 8px; }
  header p { color: #a1a1aa; font-size: 14px; }
  .grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(220px, 1fr));
    gap: 20px; padding: 30px 20px;
    max-width: 1200px; margin: 0 auto;
  }
  .card {
    background: #1a1a2e; border: 1px solid #2a2a40;
    border-radius: 14px; overflow: hidden;
    display: flex; flex-direction: column;
  }
  .card img {
    width: 100%; height: 160px; object-fit: contain;
    background: #000; padding: 10px;
  }
  .card-body { padding: 14px; display: flex; flex-direction: column; gap: 10px; flex: 1; }
  .card-body h3 { font-size: 16px; }
  .card-body .price { color: #4ade80; font-weight: bold; font-size: 16px; }
  .buy-btn {
    margin-top: auto; padding: 12px;
    background: #4ade80; color: #000; border: none;
    border-radius: 10px; font-size: 15px; font-weight: bold;
    cursor: pointer;
  }
  .buy-btn:active { background: #22c55e; }
  .modal {
    position: fixed; inset: 0; background: rgba(0,0,0,0.9);
    display: none; align-items: center; justify-content: center;
    padding: 20px; z-index: 100;
  }
  .modal.open { display: flex; }
  .modal-box {
    background: #1a1a2e; border: 1px solid #2a2a40;
    border-radius: 14px; padding: 24px;
    max-width: 440px; width: 100%; max-height: 90vh; overflow-y: auto;
    position: relative;
  }
  .close-btn {
    position: absolute; top: 12px; right: 12px;
    background: none; border: none; color: #a1a1aa;
    font-size: 24px; cursor: pointer;
  }
  .modal-box h2 { font-size: 20px; margin-bottom: 16px; }
  .preview {
    display: flex; gap: 12px; align-items: center;
    background: #0f0f1a; padding: 10px; border-radius: 10px; margin-bottom: 16px;
  }
  .preview img { width: 60px; height: 60px; object-fit: contain; background: #000; border-radius: 8px; }
  .preview h3 { font-size: 15px; margin-bottom: 4px; }
  .preview p { color: #4ade80; font-weight: bold; font-size: 14px; }
  label { display: block; font-size: 13px; color: #a1a1aa; margin-bottom: 12px; }
  input, textarea {
    width: 100%; margin-top: 5px; padding: 11px;
    background: #0f0f1a; border: 1px solid #2a2a40;
    border-radius: 8px; color: #fff; font-size: 16px;
    font-family: inherit; outline: none;
  }
  input:focus, textarea:focus { border-color: #4ade80; }
  .total {
    background: #0f0f1a; border: 1px dashed #4ade80;
    border-radius: 8px; padding: 12px; margin-bottom: 14px;
    display: flex; justify-content: space-between; font-size: 15px;
  }
  .total strong { color: #4ade80; font-size: 18px; }
  .submit-btn {
    width: 100%; padding: 14px; background: #4ade80; color: #000;
    border: none; border-radius: 10px; font-size: 16px; font-weight: bold;
    cursor: pointer;
  }
  .submit-btn:disabled { opacity: 0.6; }
  .toast {
    position: fixed; bottom: 24px; left: 50%;
    transform: translateX(-50%);
    padding: 12px 22px; border-radius: 10px;
    font-weight: 500; z-index: 200; display: none;
    max-width: 90%; text-align: center;
  }
  .toast.show { display: block; }
  .toast.ok { background: #4ade80; color: #000; }
  .toast.err { background: #ef4444; color: #fff; }
</style>
</head>
<body>

<header>
  <h1>⛏️ Спавнеры Minecraft</h1>
  <p>Закажи спавнер любого моба</p>
</header>

<div class="grid" id="catalog"></div>

<div class="modal" id="modal">
  <div class="modal-box">
    <button class="close-btn" id="closeBtn">✕</button>
    <h2>Оформление заказа</h2>
    <div class="preview">
      <img id="pImg" src="" alt="">
      <div>
        <h3 id="pName"></h3>
        <p id="pPrice"></p>
      </div>
    </div>
    <form id="orderForm">
      <input type="hidden" id="fSpawner">
      <label>Ник в Minecraft
        <input type="text" id="fNick" required placeholder="Steve">
      </label>
      <label>Контакт (Telegram/Discord)
        <input type="text" id="fContact" required placeholder="@username">
      </label>
      <label>Количество
        <input type="number" id="fQty" min="1" value="1" required>
      </label>
      <div class="total">
        <span>Итого:</span>
        <strong id="fTotal">0 кк монет</strong>
      </div>
      <label>Комментарий
        <textarea id="fComment" rows="2" placeholder="Координаты, мир..."></textarea>
      </label>
      <button type="submit" class="submit-btn" id="submitBtn">Отправить заказ</button>
    </form>
  </div>
</div>

<div class="toast" id="toast"></div>

<script>
/* ==========================================================
   ⚙️⚙️⚙️  НАСТРОЙКИ — ВСТАВЬ СВОИ ДАННЫЕ СЮДА  ⚙️⚙️⚙️
   ==========================================================
   
   📌 КАК ЗАПОЛНИТЬ:
   
   1) TELEGRAM_TOKEN — токен бота из @BotFather
      Пример: "8906795805:AAEhRGHEEFB3NizXkiuCKpWjUNMXfgt74Tg"
   
   2) TELEGRAM_CHAT_ID — "7862746520"
   
   ⚠️ Кавычки НЕ убирай!
   ⚠️ Между кавычками — только твои данные, без пробелов
   ========================================================== */

var TELEGRAM_TOKEN   = "ВСТАВЬ_СЮДА_ТОКЕН_БОТА";
var TELEGRAM_CHAT_ID = "ВСТАВЬ_СЮДА_CHAT_ID";

/* ==========================================================
   ДАЛЬШЕ НЕ ТРОГАЙ
   ========================================================== */

var CURRENCY = "кк монет";

var SPAWNERS = [
  { id: "zombie",   name: "Спавнер зомби",     price: 2.9, image: "https://minecraft.wiki/images/Zombie_JE5_BE2.png" },
  { id: "spider",   name: "Спавнер паука",     price: 6.3, image: "https://minecraft.wiki/images/Spider_JE4_BE3.png" },
  { id: "skeleton", name: "Спавнер скелета",   price: 3.3, image: "https://minecraft.wiki/images/Skeleton_JE5_BE3.png" },
  { id: "ghast",    name: "Спавнер гаста",     price: 6.6, image: "https://minecraft.wiki/images/Ghast_JE2_BE2.png" },
  { id: "creeper",  name: "Спавнер крипера",   price: 6.7, image: "https://minecraft.wiki/images/Creeper_JE3_BE2.png" },
  { id: "phantom",  name: "Спавнер фантома",   price: 2.8, image: "https://minecraft.wiki/images/Phantom_JE2_BE2.png" },
  { id: "guardian", name: "Спавнер стража",    price: 6.5, image: "https://minecraft.wiki/images/Guardian_JE3_BE2.png" },
  { id: "enderman", name: "Спавнер эндермена", price: 3.4, image: "https://minecraft.wiki/images/Enderman_JE5_BE3.png" },
  { id: "rabbit",   name: "Спавнер кролика",   price: 2.1, image: "https://minecraft.wiki/images/Rabbit_JE3_BE2.png" },
  { id: "sheep",    name: "Спавнер овцы",      price: 2.0, image: "https://minecraft.wiki/images/Sheep_JE3_BE3.png" },
  { id: "cow",      name: "Спавнер коровы",    price: 2.3, image: "https://minecraft.wiki/images/Cow_JE5_BE3.png" },
  { id: "blaze",    name: "Спавнер ифрита",    price: 6.1, image: "https://minecraft.wiki/images/Blaze_JE2_BE2.png" }
];

var current = null;
var catalog = document.getElementById('catalog');
var modal = document.getElementById('modal');
var toast = document.getElementById('toast');

// Рисуем каталог
var html = '';
for (var i = 0; i < SPAWNERS.length; i++) {
  var s = SPAWNERS[i];
  html += '<div class="card">';
  html += '<img src="' + s.image + '" alt="' + s.name + '">';
  html += '<div class="card-body">';
  html += '<h3>' + s.name + '</h3>';
  html += '<p class="price">🪙 ' + s.price + ' ' + CURRENCY + '</p>';
  html += '<button class="buy-btn" data-i="' + i + '">🛒 Заказать</button>';
  html += '</div></div>';
}
catalog.innerHTML = html;

// Кнопки "Заказать"
var btns = document.querySelectorAll('.buy-btn');
for (var j = 0; j < btns.length; j++) {
  btns[j].onclick = function() {
    var idx = parseInt(this.getAttribute('data-i'));
    current = SPAWNERS[idx];
    document.getElementById('pImg').src = current.image;
    document.getElementById('pName').textContent = current.name;
    document.getElementById('pPrice').textContent = '🪙 ' + current.price + ' ' + CURRENCY;
    document.getElementById('fSpawner').value = current.name;
    document.getElementById('fQty').value = 1;
    document.getElementById('fTotal').textContent = current.price + ' ' + CURRENCY;
    modal.classList.add('open');
  };
}

// Пересчёт
document.getElementById('fQty').oninput = function() {
  if (!current) return;
  var q = parseInt(this.value) || 1;
  var total = (current.price * q).toFixed(1);
  document.getElementById('fTotal').textContent = total + ' ' + CURRENCY;
};

// Закрытие
document.getElementById('closeBtn').onclick = function() {
  modal.classList.remove('open');
};
modal.onclick = function(e) {
  if (e.target === modal) modal.classList.remove('open');
};

// Отправка
document.getElementById('orderForm').onsubmit = function(e) {
  e.preventDefault();

  // ПРОВЕРКА: заполнены ли настройки
  if (TELEGRAM_TOKEN === "ВСТАВЬ_СЮДА_ТОКЕН_БОТА" || TELEGRAM_TOKEN === "") {
    showToast('⚠️ Токен бота не вставлен в код', 'err');
    return;
  }
  if (TELEGRAM_CHAT_ID === "ВСТАВЬ_СЮДА_CHAT_ID" || TELEGRAM_CHAT_ID === "") {
    showToast('⚠️ Chat ID не вставлен в код', 'err');
    return;
  }

  var btn = document.getElementById('submitBtn');
  btn.disabled = true;
  btn.textContent = 'Отправка...';

  var q = parseInt(document.getElementById('fQty').value) || 1;
  var total = (current.price * q).toFixed(1);

  var text = '🛒 Новый заказ!\n\n';
  text += '📦 ' + current.name + '\n';
  text += '🔢 Кол-во: ' + q + '\n';
  text += '💰 Цена: ' + current.price + ' ' + CURRENCY + '\n';
  text += '💵 Итого: ' + total + ' ' + CURRENCY + '\n';
  text += '👤 Ник: ' + document.getElementById('fNick').value + '\n';
  text += '💬 Контакт: ' + document.getElementById('fContact').value + '\n';
  text += '📝 Коммент: ' + (document.getElementById('fComment').value || '—');

  var url = 'https://api.telegram.org/bot' + TELEGRAM_TOKEN + '/sendMessage';

  fetch(url, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ chat_id: TELEGRAM_CHAT_ID, text: text })
  })
  .then(function(r) { return r.json(); })
  .then(function(data) {
    if (data.ok) {
      showToast('✅ Заказ отправлен!', 'ok');
      document.getElementById('orderForm').reset();
      modal.classList.remove('open');
    } else {
      showToast('❌ ' + (data.description || 'Ошибка'), 'err');
    }
  })
  .catch(function() {
    showToast('❌ Ошибка сети', 'err');
  })
  .finally(function() {
    btn.disabled = false;
    btn.textContent = 'Отправить заказ';
  });
};

function showToast(text, type) {
  toast.textContent = text;
  toast.className = 'toast show ' + type;
  setTimeout(function() { toast.className = 'toast'; }, 4000);
}
</script>

</body>
</html>