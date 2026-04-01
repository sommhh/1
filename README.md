<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1.0">
<title>写真ログ</title>

<link rel="manifest" href="manifest.json">

<style>
body{font-family:sans-serif;margin:0;background:#f2f2f7}
header{padding:10px;background:#007aff;color:#fff;text-align:center}
button{padding:10px;margin:5px;border:none;border-radius:8px}
img{width:100%;border-radius:10px}
.card{background:#fff;margin:10px;padding:10px;border-radius:12px}
</style>
</head>

<body>

<header>📷 写真ログ</header>

<input type="file" accept="image/*" capture="environment" id="file">

<input type="text" id="memo" placeholder="メモ">

<button onclick="save()">保存</button>

<div id="list"></div>

<script>
let db;

const request = indexedDB.open("photoDB",1);

request.onupgradeneeded = e=>{
  db = e.target.result;
  db.createObjectStore("photos",{keyPath:"id"});
};

request.onsuccess = e=>{
  db = e.target.result;
  load();
};

function save(){
  const file = document.getElementById("file").files[0];
  if(!file) return;

  const reader = new FileReader();
  reader.onload = ()=>{
    const tx = db.transaction("photos","readwrite");
    const store = tx.objectStore("photos");

    store.add({
      id: Date.now(),
      img: reader.result,
      memo: document.getElementById("memo").value,
      date: new Date().toLocaleString()
    });

    load();
  };
  reader.readAsDataURL(file);
}

function load(){
  const tx = db.transaction("photos","readonly");
  const store = tx.objectStore("photos");

  const req = store.getAll();
  req.onsuccess = ()=>{
    const list = document.getElementById("list");
    list.innerHTML = "";

    req.result.reverse().forEach(p=>{
      list.innerHTML += `
        <div class="card">
          <img src="${p.img}">
          <div>${p.date}</div>
          <div>${p.memo}</div>
        </div>
      `;
    });
  };
}

// PWA
if("serviceWorker" in navigator){
  navigator.serviceWorker.register("sw.js");
}
</script>

</body>
</html>
