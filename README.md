<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Sítio Brisa do Vale</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<style>
*{
  box-sizing:border-box;
  margin:0;
  padding:0;
  font-family:Segoe UI,Tahoma,sans-serif
}

body{
  background:linear-gradient(135deg,#0d47a1,#e3f2fd);
  color:#0d1b2a;
}

header{
  background:linear-gradient(135deg,#0d47a1,#08306b);
  color:#fff;
  padding:30px;
  text-align:center;
}

main{
  max-width:1100px;
  margin:30px auto;
  padding:20px
}

.card{
  background:#ffffff;
  border-radius:18px;
  padding:25px;
  box-shadow:0 15px 35px rgba(13,71,161,.35)
}

.calendar{
  display:grid;
  grid-template-columns:repeat(7,1fr);
  gap:10px
}

.day-name{
  text-align:center;
  font-weight:bold;
  color:#0d47a1
}

.day{
  padding:15px;
  text-align:center;
  border-radius:12px;
  background:#e3f2fd;
  cursor:pointer;
  transition:.2s
}

.day:hover{background:#bbdefb}

.day.selected{
  background:#0d47a1;
  color:#fff;
  font-weight:bold
}

.day.booked{
  background:#b71c1c;
  color:#fff;
  font-weight:bold;
  cursor:not-allowed
}

button{
  padding:12px;
  border-radius:12px;
  border:none;
  cursor:pointer;
  font-weight:bold
}

button.green{
  background:linear-gradient(135deg,#0d47a1,#1565c0);
  color:#fff
}

button.red{
  background:linear-gradient(135deg,#b71c1c,#7f0000);
  color:#fff
}

.booking input{
  padding:12px;
  border-radius:12px;
  border:2px solid #0d47a1;
  width:100%;
  margin-top:8px;
  font-weight:500
}

.booking-item{
  background:#e3f2fd;
  border-left:6px solid #0d47a1;
  padding:12px;
  margin-bottom:10px;
  border-radius:10px;
  box-shadow:0 5px 12px rgba(13,71,161,.2)
}

.booking-name{
  font-size:1.1rem;
  font-weight:700;
  color:#0d47a1
}

.booking-phone{
  font-size:1rem;
  font-weight:600;
  color:#08306b
}

.booking-date{
  font-weight:bold;
  color:#000
}

.booking-item button{
  margin-top:8px;
  background:#b71c1c;
  color:#fff;
  padding:6px 10px;
  border-radius:8px
}
</style>
</head>
<body>
<header>
<h1>Sítio Brisa do Vale</h1>
<p>Agenda interna</p>
</header>

<main>
<section class="card">

<div style="display:flex;gap:10px;margin-bottom:15px">
<select id="month"></select>
<select id="year"></select>
<button class="red" onclick="clearAll()">Limpar Agenda</button>
</div>

<div class="calendar" id="calendar"></div>

<div class="booking" style="margin-top:15px">
<input id="selectedDate" placeholder="Data selecionada" readonly>
<input id="clientName" placeholder="Nome da pessoa">
<input id="clientPhone" placeholder="Telefone">
<input type="time" id="startTime">
<input type="time" id="endTime">
<button class="green" onclick="saveBooking()">Marcar como Alugado</button>
</div>

<h3 style="margin-top:20px">Datas Agendadas (ordem crescente)</h3>
<ul id="bookingList"></ul>

<h3 style="margin-top:20px">Logs</h3>
<button class="red" onclick="clearLogs()">🗑 Limpar Logs</button>
<div id="logPanel" style="margin-top:10px;background:#f1f8f4;padding:15px;border-radius:12px"></div>

</section>
</main>

<script>
const calendar = document.getElementById('calendar')
const month = document.getElementById('month')
const year = document.getElementById('year')
const selected = document.getElementById('selectedDate')
const startTime = document.getElementById('startTime')
const endTime = document.getElementById('endTime')
const clientName = document.getElementById('clientName')
const clientPhone = document.getElementById('clientPhone')
const bookingList = document.getElementById('bookingList')
const logPanel = document.getElementById('logPanel')

const months = ['Janeiro','Fevereiro','Março','Abril','Maio','Junho','Julho','Agosto','Setembro','Outubro','Novembro','Dezembro']
const days = ['Dom','Seg','Ter','Qua','Qui','Sex','Sáb']

months.forEach((m,i)=>month.innerHTML+=`<option value="${i}">${m}</option>`)
for(let y=2026;y<=2035;y++) year.innerHTML+=`<option>${y}</option>`
month.value=0;year.value=2026

function getStorageKey(){return `bookings_${year.value}_${month.value}`}

month.onchange=year.onchange=()=>{selected.value='';renderCalendar();renderBookings()}

function ordenarPorData(bookings){
  return bookings.sort((a,b)=>{
    const [da,ma,ya]=a.date.split('/')
    const [db,mb,yb]=b.date.split('/')
    return new Date(ya,ma-1,da)-new Date(yb,mb-1,db)
  })
}

function renderCalendar(){
  calendar.innerHTML=''
  days.forEach(d=>calendar.innerHTML+=`<div class="day-name">${d}</div>`)
  const first=new Date(year.value,month.value,1).getDay()
  const total=new Date(year.value,Number(month.value)+1,0).getDate()
  for(let i=0;i<first;i++) calendar.innerHTML+='<div></div>'
  const bookings=JSON.parse(localStorage.getItem(getStorageKey())||'[]')
  for(let d=1;d<=total;d++){
    const date=`${d}/${Number(month.value)+1}/${year.value}`
    const booked=bookings.some(b=>b.date===date)
    calendar.innerHTML+=`<div class="day ${booked?'booked':''}" onclick="selectDay('${date}',this)">${d}</div>`
  }
}

function selectDay(date,el){
  if(el.classList.contains('booked'))return alert('Dia já alugado')
  document.querySelectorAll('.day').forEach(d=>d.classList.remove('selected'))
  el.classList.add('selected')
  selected.value=date
}

function saveBooking(){
  if(!selected.value||!startTime.value||!endTime.value||!clientName.value||!clientPhone.value)
    return alert('Preencha tudo')
  const key=getStorageKey()
  let bookings=JSON.parse(localStorage.getItem(key)||'[]')
  bookings.push({
    id:Date.now(),
    date:selected.value,
    start:startTime.value,
    end:endTime.value,
    name:clientName.value,
    phone:clientPhone.value
  })
  localStorage.setItem(key,JSON.stringify(bookings))
  addLog(`Reserva: ${selected.value} - ${clientName.value}`)
  selected.value=startTime.value=endTime.value=clientName.value=clientPhone.value=''
  renderCalendar();renderBookings()
}

function renderBookings(){
  bookingList.innerHTML=''
  let bookings=JSON.parse(localStorage.getItem(getStorageKey())||'[]')
  bookings=ordenarPorData(bookings)
  bookings.forEach(b=>{
    bookingList.innerHTML+=`
    <li class="booking-item">
      <div class="booking-date">📅 ${b.date} — ⏰ ${b.start} às ${b.end}</div>
      <div class="booking-name">👤 ${b.name}</div>
      <div class="booking-phone">📞 ${b.phone}</div>
      <button onclick="removeBooking(${b.id})">❌ Remover</button>
    </li>`
  })
}

function removeBooking(id){
  const key=getStorageKey()
  let bookings=JSON.parse(localStorage.getItem(key)||'[]')
  const r=bookings.find(b=>b.id===id)
  bookings=bookings.filter(b=>b.id!==id)
  localStorage.setItem(key,JSON.stringify(bookings))
  if(r)addLog(`Reserva removida: ${r.date} - ${r.name}`)
  renderCalendar();renderBookings()
}

function addLog(text){
  const logs=JSON.parse(localStorage.getItem('logs')||'[]')
  logs.unshift(`${new Date().toLocaleString()} — ${text}`)
  localStorage.setItem('logs',JSON.stringify(logs))
  renderLogs()
}

function renderLogs(){
  const logs=JSON.parse(localStorage.getItem('logs')||'[]')
  logPanel.innerHTML=logs.length?logs.join('<br>'):'Nenhum log'
}

function clearLogs(){if(confirm('Apagar logs?')){localStorage.removeItem('logs');renderLogs()}}
function clearAll(){if(confirm('Apagar tudo deste mês?')){localStorage.removeItem(getStorageKey());renderCalendar();renderBookings()}}

renderCalendar();renderBookings();renderLogs()
</script>
</body>
</html>
