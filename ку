<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <title>Канбан-доска броней (17 столов)</title>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }
    body { font-family: sans-serif; background: #eef2f5; padding: 20px; line-height: 1.4; }
    h2 { margin-bottom: 16px; color: #333; }
    .controls { display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 20px; }
    .controls input, .controls select, .controls button { padding: 6px 10px; border: 1px solid #bbb; border-radius: 4px; background: #fff; }
    .controls button { background: #4078c0; color: #fff; border-color: #305a8c; cursor: pointer; }
    .controls button:hover { background: #305a8c; }
    .clear-btn { background: #c0392b; border-color: #922b21; }
    .clear-btn:hover { background: #922b21; }
    .board { display: grid; grid-template-columns: repeat(auto-fill, minmax(200px, 1fr)); gap: 12px; }
    .cell { background: #fff; border: 1px solid #dde2e8; border-radius: 6px; box-shadow: 0 2px 4px rgba(0,0,0,0.1); display: flex; flex-direction: column; overflow: hidden; }
    .cell-header { background: #4078c0; color: #fff; text-align: center; padding: 8px; font-weight: bold; font-size: 14px; }
    .cell-body { flex: 1; padding: 8px; overflow-y: auto; }
    .cell-footer { background: #f5f7fa; color: #555; text-align: center; padding: 6px; font-size: 13px; border-top: 1px solid #dde2e8; }
    .card { background: #ffeeba; border-left: 6px solid #d39e00; border-radius: 4px; padding: 8px; margin-bottom: 8px; box-shadow: 0 1px 2px rgba(0,0,0,0.1); cursor: move; transition: transform .1s; font-size: 13px; }
    .card:hover { transform: translateY(-2px); }
    .green { background: #d4edda; border-left-color: #28a745 !important; }
    .status-btn, .delete-btn { margin-top: 6px; background: transparent; border: none; color: #555; cursor: pointer; font-size: 12px; padding: 2px 4px; }
    .delete-btn { color: #c0392b; }
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

<!-- Supabase SDK -->
<script type="module">
  import { createClient } from 'https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2.0.0/dist/umd/index.min.js';

  // Инициализация Supabase
  const supabaseUrl = 'https://jkzgrmdwfdrddwpzqiuz.supabase.co';
  const supabaseKey = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImpremdybWR3ZmRyZGR3cHpxaXV6Iiwicm9sZSI6ImFub24iLCJpYXQiOjE3NDYzMDg5ODIsImV4cCI6MjA2MTg4NDk4Mn0.pgGseeSNqlEZO1TItHM8BIWyEiATxyRDpo7XXMNAiIU';
  const supabase = createClient(supabaseUrl, supabaseKey);

  // Генерация столов
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
    cell.ondrop = e => { e.preventDefault(); const id = e.dataTransfer.getData('text'); document.querySelector(`#table-${i} .cell-body`).append(document.getElementById(id)); updateTaskTable(id, i); };
    cell.innerHTML = `<div class="cell-header">Стол ${i}</div><div class="cell-body"></div><div class="cell-footer">${cap}</div>`;
    board.append(cell);
  }

  // Drag & Drop
  window.drag = ev => ev.dataTransfer.setData('text', ev.target.id);

  // Загрузка состояния
  async function loadState() {
    const { data, error } = await supabase.from('bookings').select('*');
    if (error) return console.error(error);
    document.querySelectorAll('.card').forEach(c => c.remove());
    data.forEach(item => {
      const cell = document.querySelector(`#table-${item.table_id} .cell-body`);
      if (!cell) return;
      const card = document.createElement('div');
      card.id = 'card-' + item.id; card.className = 'card ' + (item.status==='done'?'green':'red');
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
    const name = nameInp.value.trim(), phone = phoneInp.value.trim(),
          table = +tableSelect.value, time = timeInp.value,
          guests = +guestsInp.value, source = sourceSel.value,
          hookah = hookahChk.checked, vr = vrChk.checked;
    if(!name||!phone||!table||!time||!guests||!source) return;
    const { data, error } = await supabase.from('bookings').insert([{
      name, phone, table_id:table, time, guests, source,
      hookah:hookah?'Кальян':'', vr:vr?'VR':'', status:'to-do'
    }]);
    if(error) return console.error(error);
    loadState();
  };

  // Удаление
  window.deleteCard = async id => {
    const { error } = await supabase.from('bookings').delete().eq('id', id);
    if(error) return console.error(error);
    loadState();
  };

  // Переключение статуса
  window.toggleStatus = async id => {
    const row = (await supabase.from('bookings').select('status').eq('id',id).single()).data;
    const newStatus = row.status==='to-do'?'done':'to-do';
    const { error } = await supabase.from('bookings').update({status:newStatus}).eq('id', id);
    if(error) return console.error(error);
    loadState();
  };

  // Обновление столика при drop
  async function updateTaskTable(elId, newTable) {
    const id = +elId.split('-')[1];
    const { error } = await supabase.from('bookings').update({table_id:newTable}).eq('id', id);
    if(error) console.error(error); else loadState();
  }

  // Очистка всех
  window.clearAll = async () => {
    if(!confirm("Очистить все?")) return;
    const { error } = await supabase.from('bookings').delete();
    if(error) console.error(error); else loadState();
  };
</script>

</body>
</html>
