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
#listaArmas, #meusLoadouts, #chat { padding:10px; }
.arma { background:#222; margin:5px 0; padding:5px; border-radius:5px; }
.loadoutDetalhe { font-size:0.9em; margin-left:10px; cursor:pointer; }
.abasLoadouts button { margin-right:5px; }
.hidden { display:none; }
#dashboardAdmin { background:#333; padding:10px; margin:10px 0; border-radius:5px; }
input, select, textarea { padding:5px; margin:2px; border-radius:4px; border:1px solid #00ffea; width:95%; }
#perfil { position:fixed; top:10px; right:10px; }
.tooltip { background:#333; color:#00ffea; padding:5px; border-radius:4px; display:inline-block; position:absolute; z-index:10; font-size:0.85em; }
#chat { background:#222; border-radius:5px; max-height:300px; overflow-y:auto; }
.chat-message { border-bottom:1px solid #00ffea; padding:5px 0; }
</style>
</head>
<body>

<header>
<h1>CODM Meta BR</h1>
<div id="perfil">
<button onclick="togglePerfilMenu()">Perfil/Login</button>
<div id="perfilMenu" class="hidden">
<p id="nomeUsuario">Usuário: Visitante</p>
<button id="btnDashboard" class="hidden" onclick="toggleDashboard()">Dashboard Admin</button>
<button id="loginADM" onclick="admLogin()">Login Criador (ADM)</button>
<button id="logoutBtn" class="hidden" onclick="logout()">Logout</button>
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
<p>Editar armas, loadouts e perks:</p>
<textarea id="editLoadout" placeholder="Exemplo JSON: {'arma':'AK-47','BOCA':'Supressor'}"></textarea>
<button onclick="salvarLoadout()">Salvar</button>
</div>

<div id="chat">
<h3>Chat (usuários logados)</h3>
<div id="chatMensagens"></div>
<textarea id="msgInput" placeholder="Digite sua mensagem..."></textarea>
<button onclick="enviarMsg()">Enviar</button>
</div>

<div id="tooltip" class="tooltip hidden"></div>

<script type="module">
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js";
import { getAuth, GoogleAuthProvider, signInWithPopup, onAuthStateChanged, signOut } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-auth.js";
import { getFirestore, doc, setDoc, getDoc, collection, addDoc, getDocs, onSnapshot } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore.js";

// ===============================
// Firebase Config
// ===============================
const firebaseConfig = {
  apiKey: "AIzaSyBjmdiptqFy-MnYnn9gvNIcw09YyGPY_RU",
  authDomain: "codm-meta-br-1f451.firebaseapp.com",
  projectId: "codm-meta-br-1f451",
  storageBucket: "codm-meta-br-1f451.firebasestorage.app",
  messagingSenderId: "943745991439",
  appId: "1:943745991439:web:b9774a578b6af5feed747a"
};
const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getFirestore(app);
const provider = new GoogleAuthProvider();

// ===============================
// Usuário e ADM
// ===============================
let usuario = {nome:"Visitante", tipo:"visitante", uid:""};

// ===============================
// Login ADM fixo
// ===============================
function admLogin() {
  const email = prompt("E-mail ADM:");
  const senha = prompt("Senha:");
  if(email==="itzbuxa@gmail.com" && senha==="100%lost"){
    usuario = {nome:"Bob Lost", tipo:"criador", uid:"adm123"};
    alert("Login ADM efetuado!");
    document.getElementById("nomeUsuario").innerText=`Usuário: ${usuario.nome}`;
    document.getElementById("btnDashboard").classList.remove("hidden");
    document.getElementById("logoutBtn").classList.remove("hidden");
    document.getElementById("loginADM").classList.add("hidden");
  } else {
    alert("Credenciais inválidas!");
  }
}

// ===============================
// Logout
// ===============================
function logout() {
  usuario={nome:"Visitante", tipo:"visitante", uid:""};
  document.getElementById("nomeUsuario").innerText=`Usuário: ${usuario.nome}`;
  document.getElementById("btnDashboard").classList.add("hidden");
  document.getElementById("logoutBtn").classList.add("hidden");
  document.getElementById("loginADM").classList.remove("hidden");
}

// ===============================
// Tooltip
// ===============================
const tooltip=document.getElementById('tooltip');
function showTooltip(event,text){
  tooltip.textContent=text;
  tooltip.style.left=event.pageX+10+'px';
  tooltip.style.top=event.pageY+10+'px';
  tooltip.classList.remove('hidden');
}
function hideTooltip(){tooltip.classList.add('hidden');}

// ===============================
// Funções Perfil
// ===============================
function togglePerfilMenu(){
  document.getElementById('perfilMenu').classList.toggle('hidden');
}

// ===============================
// Armas e Loadouts
// ===============================
const armas={
  AR:["AK-47","AK117","ASM10","AS VAL","BP50","BK57","DR-H","EM2","FFAR 1","FR .556","Grau 5.56","HVK-30","ICR-1","Kilo 141","Krig 6","LK24","M4","M13","Maddox","Oden","Ram-7","Swordfish","Type 19","Vargo-S"],
  SMG:["AGR 556","CBR4","Chicom","Cordite","CX-9","Fennec","GKS","HG40","ISO","KSP 45","LAPA","MAC-10","MX9","OTs 9","PDW-57","Pharo","PP19 Bizon","PPSh-41","QXR","RUS-79U","Razorback","USS 9","VMP","LC10","TEC-9","MSMC"],
  LMG:["Chopper","Dingo","Holger 26","Hades","M4LMG","PKM","RPD","S36","UL736","MG42"],
  SNIPER:["Arctic .50","DL Q33","HDR","Koshka","Locus","LW3-Tundra","M21 EBR","NA-45","Outlaw","Rytec AMR","SVD","XPR-50","ZRG 20mm"],
  MARKSMAN:["SKS","Type 63","SO-14 Marksman Rifle","Kilo Bolt-Action"],
  SHOTGUN:["BY15","HS0405","HS2126","JAK-12","KRM-262","R9-0","Striker","Echo","Expedite 12"]
};

const loadouts={};
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

function mostrarClasse(classe){
  const listaDiv=document.getElementById('listaArmas');
  listaDiv.innerHTML='';
  armas[classe].forEach(nome=>{
    const div=document.createElement('div');
    div.className='arma';
    const load=loadouts[nome];
    let detalhe='';
    for(const key in load){
      detalhe+=`<div class="loadoutDetalhe" onmouseover="showTooltip(event,'${accessories[load[key]]||''}')" onmouseout="hideTooltip()">${key}: ${load[key]}</div>`;
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
        const load=loadouts[nome];
        let detalhe='';
        for(const key in load){
          detalhe+=`<div class="loadoutDetalhe" onmouseover="showTooltip(event,'${accessories[load[key]]||''}')" onmouseout="hideTooltip()">${key}: ${load[key]}</div>`;
        }
        div.innerHTML=`<strong>${nome}</strong>${detalhe}`;
        listaDiv.appendChild(div);
      }
    });
  }
}

// ===============================
// Meus Loadouts
// ===============================
const meusLoadouts={BR:["AK-47","Fennec","BY15"],MJ:["M4","QQ9","HDR"],DMZ:["RPD","SKS"]};
function mostrarMeusLoadouts(modo){
  const listaDiv=document.getElementById('listaMeusLoadouts');
  listaDiv.innerHTML='';
  meusLoadouts[modo].forEach(nome=>{
    const div=document.createElement('div');
    div.className='arma';
    const load=loadouts[nome];
    let detalhe='';
    for(const key in load){
      detalhe+=`<div class="loadoutDetalhe" onmouseover="showTooltip(event,'${accessories[load[key]]||''}')" onmouseout="hideTooltip()">${key}: ${load[key]}</div>`;
    }
    div.innerHTML=`<strong>${nome}</strong>${detalhe}`;
    listaDiv.appendChild(div);
  });
}

// ===============================
// Dashboard Admin Editar Loadout
// ===============================
function salvarLoadout(){
  const txt=document.getElementById('editLoadout').value;
  try{
    const obj=JSON.parse(txt);
    if(obj.arma && loadouts[obj.arma]){
      loadouts[obj.arma]={...loadouts[obj.arma], ...obj};
      alert('Loadout atualizado!');
      mostrarClasse(Object.keys(armas).find(c=>armas[c].includes(obj.arma)));
    } else { alert('Arma inválida'); }
  } catch(e){ alert('JSON inválido'); }
}

// ===============================
// Chat Firebase (usuários logados)
// ===============================
function enviarMsg(){
  if(usuario.tipo==="visitante"){ alert("Faça login para enviar mensagens"); return; }
  const msg=document.getElementById('msgInput').value.trim();
  if(msg==="") return;
  addDoc(collection(db,'chat'),{
    uid: usuario.uid,
    nome: usuario.nome,
    msg: msg,
    timestamp: new Date()
  });
  document.getElementById('msgInput').value='';
}

onSnapshot(collection(db,'chat'),(snap)=>{
  const div=document.getElementById('chatMensagens');
  div.innerHTML='';
  snap.docs.forEach(d=>{
    const m=d.data();
    const msgDiv=document.createElement('div');
    msgDiv.className='chat-message';
    msgDiv.textContent=`[${m.nome}]: ${m.msg}`;
    div.appendChild(msgDiv);
    div.scrollTop=div.scrollHeight;
  });
});

// ===============================
// Inicialização
// ===============================
mostrarClasse('AR');
mostrarMeusLoadouts('BR');

</script>
</body>
</html>
