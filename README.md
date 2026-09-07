
<html lang="es">
<head>
<meta charset="utf-8"><meta name="viewport" content="width=device-width,initial-scale=1">
<title>Organizador de puestos</title>
<style>
:root{font-family:system-ui,-apple-system,Segoe UI,Roboto,Arial,sans-serif;color:#172033;background:#f4f6f8}*{box-sizing:border-box}body{margin:0}.wrap{max-width:1000px;margin:auto;padding:20px}.card{background:#fff;border:1px solid #dfe4ea;border-radius:16px;padding:18px;margin-bottom:16px;box-shadow:0 3px 14px #00000008}h1{margin:0 0 6px;font-size:25px}p{color:#667085}.row{display:flex;gap:10px;flex-wrap:wrap}.btn{border:0;border-radius:10px;padding:11px 15px;font-weight:700;cursor:pointer;background:#172033;color:white}.btn.alt{background:#e9edf2;color:#172033}.btn.go{background:#087f5b}.btn.small{padding:8px 11px}.hint{font-size:13px;color:#667085}textarea{width:100%;min-height:230px;border:1px solid #cfd6df;border-radius:12px;padding:13px;font:14px ui-monospace,SFMono-Regular,Menlo,monospace;resize:vertical}.summary{display:flex;gap:10px;flex-wrap:wrap;margin:12px 0}.pill{background:#eef2f6;padding:8px 11px;border-radius:999px;font-size:13px}.table{width:100%;border-collapse:collapse}.table th,.table td{text-align:left;padding:10px 8px;border-bottom:1px solid #edf0f3;vertical-align:top}.table th{font-size:12px;color:#667085}.order{font-weight:800;width:45px}.pm{font-weight:800;white-space:nowrap}.actions{white-space:nowrap}.empty{text-align:center;color:#667085;padding:25px}.notice{background:#eaf8f1;border:1px solid #b9e6cf;border-radius:12px;padding:12px;color:#146c43;font-size:13px}@media(max-width:650px){.wrap{padding:12px}.card{padding:13px}.table th:nth-child(2),.table td:nth-child(2){display:none}.btn{width:100%}.actions .btn{width:auto}.table{font-size:14px}}
</style>
</head>
<body>
<div class="wrap">
<div class="card">
<h1>📍 Organizador de puestos</h1>
<p>Pegá el listado que te manda monitoreo. La herramienta detecta puestos, números y direcciones, y genera accesos directos a Google Maps.</p>
<div class="row"><button class="btn" id="process">Procesar listado</button><button class="btn alt" id="example">Cargar ejemplo</button><button class="btn alt" id="clear">Limpiar</button></div>
</div>
<div class="card">
<textarea id="input" placeholder="Pegá acá el texto de WhatsApp, Excel o una lista de direcciones..."></textarea>
<div class="hint" style="margin-top:8px">Si no hay código de puesto, se usa el número o simplemente la dirección. Podés pegar texto desprolijo: se intenta detectar automáticamente. El botón "Cargar ejemplo" solo muestra el formato con datos ficticios, no guarda ni recuerda tu listado real.</div>
</div>
<div class="card">
<div id="summary" class="summary"></div>
<div id="notice" class="notice" style="display:none"></div>
<div style="overflow:auto"><table class="table"><thead><tr><th>#</th><th>PM</th><th>Ubicación</th><th>Acción</th></tr></thead><tbody id="results"><tr><td colspan="4" class="empty">Todavía no hay puestos procesados.</td></tr></tbody></table></div>
</div>
<div class="card"><div class="hint"><b>Uso:</b> la app no guarda ni envía tu listado a ningún servidor. Funciona como una página estática y genera enlaces de Google Maps en el navegador. Para búsquedas de calles se agrega automáticamente la localidad seleccionada.</div><label style="display:block;margin-top:12px;font-size:13px;font-weight:700">📍 Localidad para completar direcciones</label><select id="city" style="width:100%;padding:10px;border:1px solid #cfd6df;border-radius:10px;margin-top:5px;background:#fff"><option value="Rosario, Santa Fe, Argentina">Rosario, Santa Fe</option><option value="Villa Gobernador Gálvez, Santa Fe, Argentina" selected>Villa Gobernador Gálvez, Santa Fe</option><option value="Funes, Santa Fe, Argentina">Funes, Santa Fe</option><option value="Roldán, Santa Fe, Argentina">Roldán, Santa Fe</option><option value="__custom__">Otra localidad...</option></select><input id="customCity" placeholder="Escribí la localidad, provincia y país" style="display:none;width:100%;padding:10px;border:1px solid #cfd6df;border-radius:10px;margin-top:8px"></div>
</div>
<script>
const example=`PM\tOrden lógico\tUbicacion\nAAA-001\t1\tCalle Uno y Calle Dos\nAAA-002\t2\tAv. Ejemplo y Calle Tres\nAAA-003\t3\tCalle Cuatro y Calle Cinco\nAAA-004\t4\t-33.0149, -60.6224`;
const input=document.getElementById('input'), results=document.getElementById('results'), summary=document.getElementById('summary'), notice=document.getElementById('notice');
function getCity(){const sel=document.getElementById('city');return sel.value==='__custom__'?document.getElementById('customCity').value.trim():sel.value.trim()}
function mapsUrl(loc){let s=loc.trim();let isCoord=/^-?\d+(?:\.\d+)?\s*,\s*-?\d+(?:\.\d+)?$/.test(s);let city=getCity();let q=isCoord?s:(city?(s+', '+city):s);return 'https://www.google.com/maps/search/?api=1&query='+encodeURIComponent(q)}
function parse(text){
 const lines=text.split(/\r?\n/).map(x=>x.trim()).filter(Boolean); const out=[];
 for(const line of lines){
   if(/^(pm|orden\s*l[oó]gico|ubicaci[oó]n|anillo|puesto\s*$)/i.test(line.replace(/\s+/g,' '))) continue;
   let cells=line.split(/\t+|\s{2,}|\s*\|\s*/).map(x=>x.trim()).filter(Boolean);
   if(cells.length>=3){
      let pm=cells[0], ord=cells[1], loc=cells.slice(2).join(' ');
      if(!/\d/.test(ord) && /^\d+$/.test(pm)){ord=pm;pm='';}
      if(/^(pm|orden|ubicacion)$/i.test(pm)) continue;
      out.push({pm,ord:/^\d+$/.test(ord)?+ord:null,loc}); continue;
   }
   let m=line.match(/^(?:([A-Za-zÁÉÍÓÚÜÑáéíóúüñ0-9_-]+)\s+)?(?:([0-9]+)[\s.-]+)?(.+)$/);
   if(m){let pm=m[1]||'', ord=m[2]?+m[2]:null, loc=(m[3]||'').trim();
      if(/^https?:\/\//i.test(loc)||/^(pm|orden|ubicaci[oó]n)$/i.test(loc)) continue;
      if(pm && /^(anillo|puesto)$/i.test(pm)){pm='';}
      out.push({pm,ord,loc});}
 }
 return out.filter(x=>x.loc.length>2);
}
function render(){const data=parse(input.value);results.innerHTML=''; if(!data.length){results.innerHTML='<tr><td colspan="4" class="empty">No pude detectar ubicaciones. Probá pegando una dirección por línea o el formato PM / orden / ubicación.</td></tr>';summary.innerHTML='';notice.style.display='none';return;}
 data.forEach((x,i)=>{let tr=document.createElement('tr');let n=x.ord||i+1;tr.innerHTML=`<td class="order">${n}</td><td class="pm">${esc(x.pm||'—')}</td><td>${esc(x.loc)}</td><td class="actions"><a class="btn go small" target="_blank" rel="noopener" href="${mapsUrl(x.loc)}">🧭 IR</a></td>`;results.appendChild(tr);});
 summary.innerHTML=`<span class="pill">📍 ${data.length} destinos</span><span class="pill">🔎 ${data.filter(x=>/^-?\d+(?:\.\d+)?\s*,\s*-?\d+(?:\.\d+)?$/.test(x.loc)).length} con coordenadas</span>`;
 notice.style.display='block'; notice.textContent='Listo. "IR" abre Google Maps en una pestaña nueva con el destino.';
}
function esc(s){return s.replace(/[&<>"']/g,c=>({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[c]))}
document.getElementById('process').onclick=render;document.getElementById('example').onclick=()=>{input.value=example;render()};document.getElementById('clear').onclick=()=>{input.value='';results.innerHTML='<tr><td colspan="4" class="empty">Todavía no hay puestos procesados.</td></tr>';summary.innerHTML='';notice.style.display='none'};input.addEventListener('paste',()=>setTimeout(render,50));
const city=document.getElementById('city'), customCity=document.getElementById('customCity');
city.addEventListener('change',()=>{customCity.style.display=city.value==='__custom__'?'block':'none';if(city.value==='__custom__')customCity.focus();if(input.value.trim())render()});
customCity.addEventListener('input',()=>{if(input.value.trim())render()});
</script></body></html>
