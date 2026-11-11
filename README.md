<!DOCTYPE html>
<html lang="zh-Hant">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>愛心</title>
<style>
  :root { --fg:#37352f; --muted:#8a8780; --line:#d9d6cf; --heart:#e0556b; --tint:rgba(224,85,107,.1); }
  @media (prefers-color-scheme: dark) {
    :root { --fg:#e6e4df; --muted:#9b988f; --line:#4a4843; --tint:rgba(224,85,107,.18); }
  }
  html, body { margin:0; height:100%; background:transparent; }
  body {
    display:flex; align-items:center; justify-content:center;
    font:15px/1.4 -apple-system, "Noto Sans TC", "PingFang TC", "Microsoft JhengHei", sans-serif;
    color:var(--fg);
  }
  button {
    display:inline-flex; align-items:center; gap:10px;
    padding:10px 20px; border-radius:999px;
    border:1px solid var(--line); background:transparent; color:inherit;
    font:inherit; cursor:pointer;
    transition:transform .15s ease, border-color .15s ease, background .15s ease;
  }
  button:hover { border-color:var(--heart); }
  button:active { transform:scale(.96); }
  button:focus-visible { outline:2px solid var(--heart); outline-offset:2px; }
  button[disabled] { opacity:.6; cursor:wait; }
  svg { width:20px; height:20px; fill:none; stroke:currentColor; stroke-width:2; transition:fill .15s ease, stroke .15s ease; }
  button.liked { border-color:var(--heart); background:var(--tint); }
  button.liked svg { fill:var(--heart); stroke:var(--heart); }
  .count { min-width:1.5ch; font-variant-numeric:tabular-nums; }
  .label { color:var(--muted); }
  @media (prefers-reduced-motion: reduce) { * { transition:none !important; } }
</style>
</head>
<body>
<button id="btn" type="button" aria-pressed="false">
  <svg viewBox="0 0 24 24" aria-hidden="true"><path d="M20.8 4.6a5.5 5.5 0 0 0-7.8 0L12 5.7l-1-1.1a5.5 5.5 0 0 0-7.8 7.8l1 1.1L12 21l7.8-7.5 1-1.1a5.5 5.5 0 0 0 0-7.8z"/></svg>
  <span class="count" id="count">–</span>
  <span class="label" id="label">喜歡這篇？按個愛心</span>
</button>
 
<script>
  // ── 設定 ─────────────────────────────────────────────
  // NAMESPACE：改成只屬於你的獨特名稱，避免和別人的計數器撞名。
  var NAMESPACE = 'a-vision-of-drowl-hearts';
  var API = 'https://api.counterapi.dev/v1/';
  // ────────────────────────────────────────────────────
 
  // 每個頁面用網址的 ?id=... 區分，例如 like.html?id=電影評論-星際效應
  var params = new URLSearchParams(location.search);
  var id = (params.get('id') || 'default').toLowerCase().replace(/[^a-z0-9\u4e00-\u9fff_-]+/g, '-').slice(0, 100);
 
  var btn = document.getElementById('btn');
  var countEl = document.getElementById('count');
  var labelEl = document.getElementById('label');
  var storageKey = 'heart:' + NAMESPACE + ':' + id;
  var liked = false, count = 0, busy = false;
 
  try { liked = localStorage.getItem(storageKey) === '1'; } catch (e) {}
 
  function url(action) {
    return API + encodeURIComponent(NAMESPACE) + '/' + encodeURIComponent(id) + '/' + (action || '');
  }
  function readCount(data) {
    if (!data) return 0;
    var n = data.count !== undefined ? data.count : data.value;
    return Number(n) || 0;
  }
  function paint() {
    btn.classList.toggle('liked', liked);
    btn.setAttribute('aria-pressed', liked ? 'true' : 'false');
    countEl.textContent = count;
    labelEl.textContent = liked ? '謝謝你的愛心' : '喜歡這篇？按個愛心';
  }
 
  // 讀取目前愛心數（計數器尚未建立時視為 0）
  fetch(url(''))
    .then(function (r) { return r.ok ? r.json() : null; })
    .then(function (d) { count = readCount(d); paint(); })
    .catch(function () { count = 0; paint(); });
 
  btn.addEventListener('click', function () {
    if (busy) return;
    busy = true; btn.disabled = true;
    var action = liked ? 'down' : 'up';
    fetch(url(action))
      .then(function (r) { if (!r.ok) throw new Error(r.status); return r.json(); })
      .then(function (d) {
        liked = !liked;
        count = Math.max(0, readCount(d));
        try { liked ? localStorage.setItem(storageKey, '1') : localStorage.removeItem(storageKey); } catch (e) {}
      })
      .catch(function () { labelEl.textContent = '暫時無法連線，稍後再試'; })
      .then(function () { busy = false; btn.disabled = false; if (labelEl.textContent.indexOf('無法') === -1) paint(); });
  });
 
  paint();
</script>
</body>
</html>
 
