<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <title>Канбан-доска броней (17 столов)</title>
  <style>
    /* ... (ваши стили без изменений) ... */
  </style>
</head>
<body>

<h2>Канбан-доска броней (17 столов)</h2>

<div class="controls">
  <input id="name" placeholder="Имя">
  <input id="phone" placeholder="Телефон">
  <select id="table"><option disabled selected>Стол</option></select>
  <input id="time" type="time">
  <input id="guests" type="number" placeholder="Гостей">
  <select id="source">
    <option disabled selected>Источник</option>
    <option value="Звонок">Звонок</option>
    <option value="Лично">Лично</option>
    <option value="Онлайн">Онлайн</option>
  </select>
  <label><input id="hookah" type="checkbox"> Кальян</label>
  <label><input id="vr" type="checkbox"> VR</label>
  <button onclick="addCard()">Добавить</button>
  <button class="clear-btn" onclick="clearAll()">Очистить все</button>
</div>

<div class="board" id="board"></div>

<!-- 1) Supabase UMD-бандл -->
<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js/dist/umd/index.min.js"></script>

<script>
// 2) Инициализация Supabase (UMD)
const supabaseUrl = 'https://jkzgrmdwfdrddwpzqiuz.supabase.co';
const supabaseKey = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImpremdybWR3ZmRyZGR3cHpxaXV6Iiwicm9sZSI6ImFub24iLCJpYXQiOjE3NDYzMDg5ODIsImV4cCI6MjA2MTg4NDk4Mn0.pgGseeSNqlEZO1TItHM8BIWyEiATxyRDpo7XXMNAiIU';
const supabase = supabaseJs.createClient(supabaseUrl, supabaseKey);

// 3) Генерация доски и селекта
const tableSelect = document.getElementById('table');
const board = document.getElementById('board');
for (let i = 1; i <= 17; i++) {
  let cap = i <= 8 ? '4–6 чел.' :
            [9,12,16].includes(i) ? '4 чел.' :
            [13,14].includes(i) ? '2 чел.' :
            [10,11,15].includes(i) ? '8 чел.' : 'VIP 12+ чел.';
  const opt = document.createElement('option'); opt.value = i; opt.textContent = i;
  tableSelect.append(opt);
  const cell = document.createElement('div');
  cell.className = 'cell'; cell.id = 'table-' + i;
  cell.ondragover = e => e.preventDefault();
  cell.ondrop = e => {
    e.preventDefault();
    const id = e.dataTransfer.getData('text');
    document.querySelector(`#table-${i} .cell-body`).append(document.getElementById(id));
    updateTaskTable(id, i);
  };
  cell.innerHTML = `
    <div class="cell-header">Стол ${i}</div>
    <div class="cell-body"></div>
    <div class="cell-footer">${cap}</div>
  `;
  board.append(cell);
}

// Drag handler (глобально)
window.drag = ev => ev.dataTransfer.setData('text', ev.target.id);

// Загрузка состояния из Supabase
async function loadState() {
  const { data, error } = await supabase.from('bookings').select('*');
  if (error) return console.error(error);
  document.querySelectorAll('.card').forEach(c => c.remove());
  data.forEach(item => {
    const cell = document.querySelector(`#table-${item.table_id} .cell-body`);
    if (!cell) return;
    const card = document.createElement('div');
    card.id = 'card-'+item.id;
    card.className = 'card ' + (item.status==='done'?'green':'red');
    card.draggable = true; card.ondragstart = drag;
    card.innerHTML = `
      <strong>${item.name}</strong><br>
      ${item.time} · ${item.guests} чел.<br>
      тел. ${item.phone}<br>
      src: ${item.source}${item.hookah?'<br>🍹':''}${item.vr?'<br>🎮':''}
      <button class="status-btn" onclick="toggleStatus(${item.id})">Статус</button>
      <button class="delete-btn" onclick="deleteCard(${item.id})">Удалить</button>
    `;
    cell.append(card);
  });
}
window.onload = loadState;

// Добавление карточки
window.addCard = async () => {
  const name   = name.value.trim(),
        phone  = phone.value.trim(),
        table  = +tableSelect.value,
        time   = time.value,
        guests = +guests.value,
        source = source.value,
        hookah = hookah.checked,
        vr     = vr.checked;
  if(!name||!phone||!table||!time||!guests||!source) return;
  let { error } = await supabase.from('bookings').insert([{
    name, phone, table_id:table, time, guests, source,
    hookah:hookah?'Кальян':'', vr:vr?'VR':'', status:'to-do'
  }]);
  if(error) return console.error(error);
  loadState();
};

// Удаление
window.deleteCard = async id => {
  let { error } = await supabase.from('bookings').delete().eq('id', id);
  if(error) return console.error(error);
  loadState();
};

// Переключение статуса
window.toggleStatus = async id => {
  let { data:[row], error } = await supabase.from('bookings').select('status').eq('id',id);
  if(error) return console.error(error);
  const newStatus = row.status==='to-do'?'done':'to-do';
  ({ error } = await supabase.from('bookings').update({status:newStatus}).eq('id',id));
  if(error) return console.error(error);
  loadState();
};

// Обновление столика при drop
async function updateTaskTable(elId, newTable) {
  const id = +elId.split('-')[1];
  let { error } = await supabase.from('bookings').update({table_id:newTable}).eq('id',id);
  if(error) console.error(error);
}

// Очистка всех
window.clearAll = async () => {
  if(!confirm("Очистить все?")) return;
  let { error } = await supabase.from('bookings').delete();
  if(error) return console.error(error);
  loadState();
};
</script>

</body>
</html>
