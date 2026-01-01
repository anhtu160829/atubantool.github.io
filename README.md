tele:anhtubantool
<html lang="vi">
<head>
<meta charset="UTF-8">
<title>anhtubantool</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<link href="https://fonts.googleapis.com/css2?family=Cinzel:wght@600;800&display=swap" rel="stylesheet">
<style>
body{
  margin:0;
  font-family:Segoe UI,Arial;
  background:linear-gradient(270deg,red,orange,yellow,green,cyan,blue,violet);
  background-size:1400% 1400%;
  animation:rainbow 18s ease infinite;
  color:#fff;
}
@keyframes rainbow{
  0%{background-position:0% 50%}
  50%{background-position:100% 50%}
  100%{background-position:0% 50%}
}
h1{
  text-align:center;
  font-family:'Cinzel',serif;
  font-size:34px;
  margin:14px 0;
  letter-spacing:2px;
  color:transparent;
  background:linear-gradient(#fff,#ffd700);
  -webkit-background-clip:text;
  -webkit-text-stroke:1px white;
}
.tabs{display:flex;justify-content:center;gap:10px;margin-bottom:10px}
.tab{padding:10px 16px;border-radius:12px;background:rgba(0,0,0,.4);border:1px solid #fff;cursor:pointer;font-weight:bold}
.tab.active{background:#ffd700;color:#000}
.card{max-width:520px;margin:0 auto 20px;background:rgba(0,0,0,.45);border-radius:18px;padding:18px;border:1px solid #fff;display:none}
.card.active{display:block}
table{width:100%;border-collapse:collapse;margin-top:10px}
td{padding:6px;border-bottom:1px dashed rgba(255,255,255,.5)}
input,button{width:100%;padding:10px;border-radius:10px;border:none;margin-top:6px}
button{background:#ffd700;font-weight:bold;cursor:pointer}
.history{margin-top:10px;font-size:12px;max-height:200px;overflow:auto}
.history div{border-bottom:1px dotted rgba(255,255,255,.4);padding:2px 0}
</style>
</head>

<body>
<h1>anhtubantool</h1>

<div class="tabs">
  <div class="tab active" onclick="openTab('sicbo')">SICBO</div>
  <div class="tab" onclick="openTab('chanle')">CHẴN LẺ</div>
  <div class="tab" onclick="openTab('sunwin')">SUNWIN</div>
</div>

<!-- SICBO (GIỮ NGUYÊN) -->
<div id="sicbo" class="card active">
<h2>🎲 SICBO AI</h2>
<input id="sicbo_md5" placeholder="Dán MD5 32 ký tự hex">
<button onclick="analyzeSicbo()">PHÂN TÍCH</button>
<table>
<tr><td>Dự đoán</td><td id="sb_pred">--</td></tr>
<tr><td>Vị lót</td><td id="sb_lot">--</td></tr>
<tr><td>Entropy</td><td id="sb_ent">--</td></tr>
<tr><td>Deep</td><td id="sb_deep">--</td></tr>
</table>
<div class="history" id="sb_history"></div>
</div>

<!-- CHẴN LẺ (GIỮ NGUYÊN) -->
<div id="chanle" class="card">
<h2>🔐 CHẴN / LẺ AI</h2>
<input id="cl_md5" placeholder="Dán MD5 32 ký tự hex">
<button onclick="analyzeChanLe()">PHÂN TÍCH</button>
<table>
<tr><td>Kết quả</td><td id="cl_res">--</td></tr>
<tr><td>Vị lót</td><td id="cl_lot">--</td></tr>
<tr><td>Tỉ lệ</td><td id="cl_conf">--</td></tr>
<tr><td>Lời khuyên</td><td id="cl_adv">--</td></tr>
</table>
<div class="history" id="cl_history"></div>
</div>

<!-- SUNWIN (AUTO UPDATE) -->
<div id="sunwin" class="card">
<h2>🎰 SUNWIN AI</h2>

<table>
<tr><td>Phiên hiện tại</td><td id="phien">--</td></tr>
<tr><td>Xúc xắc</td><td id="xx">--</td></tr>
<tr><td>Tổng</td><td id="tong">--</td></tr>
<tr><td>Kết quả</td><td id="ketqua">--</td></tr>
<tr><td>Phiên dự đoán</td><td id="phien_next">--</td></tr>
<tr><td>Dự đoán</td><td id="dudoan">--</td></tr>
<tr><td>Cầu</td><td id="sw_cau">--</td></tr>
<tr><td>Đánh giá</td><td id="sw_rs">--</td></tr>
</table>

<div class="history" id="sw_history"></div>
</div>

<script>
function openTab(id){
  document.querySelectorAll('.tab').forEach(t=>t.classList.remove('active'));
  document.querySelectorAll('.card').forEach(c=>c.classList.remove('active'));
  event.target.classList.add('active');
  document.getElementById(id).classList.add('active');
}

/* ===== SUNWIN AUTO UPDATE ===== */
let swHistory=[];
let lastPhien=null;

async function loadSunwin(){
  try{
    const r = await fetch("https://sunwinsaygex-tzz9.onrender.com/api/sun");
    const d = await r.json();

    if(d.phien === lastPhien) return; // KHÔNG CÓ PHIÊN MỚI → BỎ QUA
    lastPhien = d.phien;

    phien.innerText = d.phien;
    phien_next.innerText = d.phien + 1;
    sw_cau.innerText = d.cau || "--";

    xx.innerText = `${d.xuc_xac_1}-${d.xuc_xac_2}-${d.xuc_xac_3}`;
    tong.innerText = d.tong;
    ketqua.innerText = d.ket_qua;
    dudoan.innerText = d.du_doan;

    const rs = (d.ket_qua === d.du_doan) ? "✅ THẮNG" : "❌ THUA";
    sw_rs.innerText = rs;

    swHistory.unshift(`#${d.phien} | ${d.cau || "?"} | ${d.du_doan} → ${rs}`);
    if(swHistory.length > 20) swHistory.pop();
    sw_history.innerHTML = swHistory.map(x=>`<div>${x}</div>`).join("");

  }catch(e){
    ketqua.innerText = "Lỗi API";
  }
}

/* poll API mỗi 3 giây */
setInterval(loadSunwin, 3000);
loadSunwin();
</script>
</body>
</html>
