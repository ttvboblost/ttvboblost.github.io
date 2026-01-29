# ttvboblost.github.io
Site de armas metas para o codm
<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>CODM Meta BR</title>
<style>
body { font-family: Arial,sans-serif; background:#111; color:#fff; margin:0; padding:0; }
header { background:#222; padding:10px; display:flex; justify-content:space-between; align-items:center; }
header h1 { margin:0; color:#00ffea; font-size:20px; }
button { padding:5px 10px; margin:2px; border:none; border-radius:4px; cursor:pointer; background:#00ffea; color:#111; font-weight:bold; }
#menuHamburguer { display:flex; flex-wrap:wrap; margin:10px 0; }
#menuHamburguer button { flex:1; margin:2px; }
#listaArmas, #meusLoadouts { padding:10px; }
.arma { background:#222; margin:5px 0; padding:5px; border-radius:5px; }
.loadoutDetalhe { font-size:0.9em; margin-left:10px; cursor:pointer; }
.abasLoadouts button { margin-right:5px; }
.hidden { display:none; }
#dashboardAdmin { background:#333; padding:10px; margin:10px 0; border-radius:5px; }
input, select { padding:5px; margin:2px; border-radius:4px; border:1px solid #00ffea; }
#perfil { position:fixed; top:10px; right:10px; }
.tooltip { background:#333; color:#00ffea; padding:5px; border-radius:4px; display:inline-block; position:absolute; z-index:10; font-size:0.85em; }
</style>
</head>
<body>

<header>
<h1>CODM Meta BR</h1>
<div id="perfil">
<button onclick="togglePerfilMenu()">Perfil</button>
<div id="perfilMenu" class="hidden">
<p id="nomeUsuario">Usuário: Visitante</p>
<button id="btnDashboard" class="hidden" onclick="toggleDashboard()">Dashboard Admin</button>
</div>
</div>
</header>

<div id="menuHamburguer">
<button onclick="mostrarClasse('AR')">AR</button>
<button onclick="mostrarClasse('SMG')">SMG</button>
<button onclick="mostrarClasse('SHOTGUN')">Shotgun</button>
<button onclick="mostrarClasse('SNIPER')">Sniper</button>
<button onclick="mostrarClasse('LMG')">LMG</button>
<button onclick="mostrarClasse('MARKSMAN')">Marksman</button>
</div>

<div>
<input type="text" id="pesquisaArma" placeholder="Pesquisar arma..." oninput="pesquisarArma()">
</div>

<div id="listaArmas"></div>

<div id="meusLoadouts">
<h2>Meus Loadouts</h2>
<div class="abasLoadouts">
<button onclick="mostrarMeusLoadouts('BR')">BR</button>
<button onclick="mostrarMeusLoadouts('MJ')">MJ</button>
<button onclick="mostrarMeusLoadouts('DMZ')">DMZ</button>
</div>
<div id="listaMeusLoadouts"></div>
</div>

<div id="dashboardAdmin" class="hidden">
<h2>Dashboard Admin</h2>
<p>Funções administrativas simuladas aqui.</p>
<p>Adicionar/editar armas, loadouts e perks.</p>
</div>

<div id="tooltip" class="tooltip hidden"></div>

<script>
let usuario={nome:"Bob Lost",tipo:"super_admin"};

function togglePerfilMenu(){
  const menu=document.getElementById('perfilMenu');
  menu.classList.toggle('hidden');
  const btnDash=document.getElementById('btnDashboard');
  if(usuario.tipo==="admin"||usuario.tipo==="super_admin"||usuario.tipo==="criador"){btnDash.classList.remove('hidden');}
  else{btnDash.classList.add('hidden');}
  document.getElementById('nomeUsuario').textContent=`Usuário: ${usuario.nome}`;
}

/* ===== Armas por Categoria ===== */
const armas={
  AR:["AK-47","AK117","ASM10","AS VAL","BP50","BK57","DR-H","EM2","FFAR 1","FR .556","Grau 5.56","HVK-30","ICR-1","Kilo 141","Krig 6","LK24","M4","M13","Maddox","Oden","Ram-7","Swordfish","Type 19","Vargo-S"],
  SMG:["AGR 556","CBR4","Chicom","Cordite","CX-9","Fennec","GKS","HG40","ISO","KSP 45","LAPA","MAC-10","MX9","OTs 9","PDW-57","Pharo","PP19 Bizon","PPSh-41","QXR","RUS-79U","Razorback","USS 9","VMP","LC10","TEC-9","MSMC"],
  LMG:["Chopper","Dingo","Holger 26","Hades","M4LMG","PKM","RPD","S36","UL736","MG42"],
  SNIPER:["Arctic .50","DL Q33","HDR","Koshka","Locus","LW3-Tundra","M21 EBR","NA-45","Outlaw","Rytec AMR","SVD","XPR-50","ZRG 20mm"],
  MARKSMAN:["SKS","Type 63","SO-14 Marksman Rifle","Kilo Bolt-Action"],
  SHOTGUN:["BY15","HS0405","HS2126","JAK-12","KRM-262","R9-0","Striker","Echo","Expedite 12"]
};

/* ===== Loadouts padrão ===== */
const loadouts={};
Object.keys(armas).forEach(classe=>{
  armas[classe].forEach(arma=>{
    loadouts[arma]={
      BOCA:"Supressor",
      CANO:"OWC Marksman",
      LASER:"Laser OWC Tático",
      CARREGADOR:"Estendido",
      EMPUNHADURA:"Fita Granulada",
      MIRA:"Holográfica 1",
      DLC1:"Omega 108",
      DLC2:"Termita",
      DLC3:"Omega 212",
      PERK1:"RM",
      PERK2:"Lacerante",
      PERK3:"Fantasma"
    };
  });
});

/* ===== Acessórios e Perks ===== */
const accessories={
"Supressor":"Reduz ruído e recoil",
"Tático":"Silenciador que mantém ADS equilibrado",
"Marksman":"Cano de precisão para maior alcance",
"OWC Marksman":"Cano de precisão OWC",
"Longo":"Cano que aumenta alcance",
"Mais Longo":"Cano sniper longo",
"Estendido":"Aumenta munição no carregador",
"Fita Granulada":"Melhora controle ao mirar",
"Fita Aderente":"Melhora mobilidade e estabilidade",
"Choke":"Reduz espalhamento",
"5mW":"Laser de combate para precisão",
"Laser OWC Tático":"Laser OWC tático",
"Holográfica 1":"Mira holográfica de média precisão",
"Omega 108":"Aumenta dano a longa distância",
"Termita":"Projétil incendiário",
"Omega 212":"Aumenta cadência de tiro",
"RM":"Reduz recuo",
"Lacerante":"Aumenta penetração",
"Fantasma":"Mantém invisível no radar"
};

/* ===== Tooltip efeito ===== */
const tooltip=document.getElementById('tooltip');
function showTooltip(event,text){
  tooltip.textContent=text;
  tooltip.style.left=event.pageX+10+'px';
  tooltip.style.top=event.pageY+10+'px';
  tooltip.classList.remove('hidden');
}
function hideTooltip(){tooltip.classList.add('hidden');}

/* ===== Funções JS ===== */
function mostrarClasse(classe){
  const listaDiv=document.getElementById('listaArmas');
  listaDiv.innerHTML='';
  armas[classe].forEach(nome=>{
    const div=document.createElement('div');
    div.className='arma';
    const load=loadouts[nome]||{};
    let detalhe='';
    for(const key in load){
      if(load[key]!=="-") detalhe+=`<div class="loadoutDetalhe" onmouseover="showTooltip(event,'${accessories[load[key]]||''}')" onmouseout="hideTooltip()">${key}: ${load[key]}</div>`;
    }
    div.innerHTML=`<strong>${nome}</strong>${detalhe}`;
    listaDiv.appendChild(div);
  });
}

function pesquisarArma(){
  const input=document.getElementById('pesquisaArma').value.toLowerCase();
  const listaDiv=document.getElementById('listaArmas');
  listaDiv.innerHTML='';
  for(const classe in armas){
    armas[classe].forEach(nome=>{
      if(nome.toLowerCase().includes(input)){
        const div=document.createElement('div');
        div.className='arma';
        const load=loadouts[nome]||{};
        let detalhe='';
        for(const key in load){
          if(load[key]!=="-") detalhe+=`<div class="loadoutDetalhe" onmouseover="showTooltip(event,'${accessories[load[key]]||''}')" onmouseout="hideTooltip()">${key}: ${load[key]}</div>`;
        }
        div.innerHTML=`<strong>${nome}</strong>${detalhe}`;
        listaDiv.appendChild(div);
      }
    });
  }
}

function toggleDashboard(){document.getElementById('dashboardAdmin').classList.toggle('hidden');}

/* Meus loadouts simulados */
const meusLoadouts={BR:["AK-47","Fennec","BY15"],MJ:["M4","QQ9","HDR"],DMZ:["RPD","SKS"]};
function mostrarMeusLoadouts(modo){
  const listaDiv=document.getElementById('listaMeusLoadouts');
  listaDiv.innerHTML='';
  meusLoadouts[modo].forEach(nome=>{
    const div=document.createElement('div');
    div.className='arma';
    const load=loadouts[nome]||{};
    let detalhe='';
    for(const key in load){
      if(load[key]!=="-") detalhe+=`<div class="loadoutDetalhe" onmouseover="showTooltip(event,'${accessories[load[key]]||''}')" onmouseout="hideTooltip()">${key}: ${load[key]}</div>`;
    }
    div.innerHTML=`<strong>${nome}</strong>${detalhe}`;
    listaDiv.appendChild(div);
  });
}
</script>
</body>
</html>
