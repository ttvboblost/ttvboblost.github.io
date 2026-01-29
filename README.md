# ttvboblost.github.io
Site de armas metas para o codm
<!DOCTYPE html><html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>CODM Meta BR</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0"><!-- Firebase CDN --><script type="module">
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js";
import { getAuth, GoogleAuthProvider, signInWithPopup, onAuthStateChanged, signOut } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-auth.js";
import { getFirestore, doc, setDoc, getDoc, addDoc, collection, onSnapshot, updateDoc, serverTimestamp, query, where } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore.js";

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

// ================= USER STATE =================
let currentUser = null;
let currentRole = "guest"; // guest | user | admin | owner

// ================= LOGIN =================
window.googleLogin = async function(){
  const result = await signInWithPopup(auth, provider);
  const user = result.user;
  const ref = doc(db, "users", user.uid);
  const snap = await getDoc(ref);

  if(!snap.exists()){
    let role = "user";
    if(user.email === "itzbuxa@gmail.com") role = "owner";

    await setDoc(ref,{
      uid:user.uid,
      email:user.email,
      nome:user.displayName,
      foto:user.photoURL,
      role:role,
      criadoEm:serverTimestamp()
    });
  }
};

// ================= AUTH STATE =================
onAuthStateChanged(auth, async(user)=>{
  if(user){
    currentUser = user;
    const ref = doc(db, "users", user.uid);
    const snap = await getDoc(ref);
    if(snap.exists()){
      currentRole = snap.data().role;
    }
    document.getElementById("loginBtn").style.display="none";
    document.getElementById("logoutBtn").style.display="block";
    document.getElementById("perfilNome").innerText = user.displayName;
    document.getElementById("perfilBox").style.display="flex";
    if(currentRole === "admin" || currentRole === "owner"){
      document.getElementById("adminPanel").style.display="block";
    }
    loadForum();
    loadChat();
  }else{
    currentUser=null;
    currentRole="guest";
  }
});

// ================= LOGOUT =================
window.logout = function(){ signOut(auth); }

// ================= LOADOUTS =================
window.createLoadout = async function(){
  const nome = document.getElementById("loadoutNome").value;
  if(!currentUser) return alert("Faça login");

  const ref = await addDoc(collection(db,"loadouts"),{
    nome,
    autor:currentUser.uid,
    autorNome:currentUser.displayName,
    likes:0,
    dislikes:0,
    criadoEm:serverTimestamp(),
    publico:true
  });

  alert("Loadout criado! Link público:\n"+location.origin+"/index.html?loadout="+ref.id);
}

// ================= FORUM =================
window.createPost = async function(){
  const text = document.getElementById("forumText").value;
  if(!currentUser) return;
  await addDoc(collection(db,"forum"),{
    text,
    autor:currentUser.displayName,
    uid:currentUser.uid,
    criadoEm:serverTimestamp()
  });
}

function loadForum(){
  const q = query(collection(db,"forum"));
  onSnapshot(q,(snap)=>{
    const box = document.getElementById("forumPosts");
    box.innerHTML="";
    snap.forEach(docu=>{
      const d = docu.data();
      box.innerHTML += `<div class='card'><b>${d.autor}</b><p>${d.text}</p></div>`;
    });
  });
}

// ================= CHAT =================
window.sendMsg = async function(){
  const msg = document.getElementById("chatInput").value;
  if(!currentUser) return;
  await addDoc(collection(db,"chat"),{
    msg,
    autor:currentUser.displayName,
    uid:currentUser.uid,
    criadoEm:serverTimestamp()
  });
}

function loadChat(){
  const q = query(collection(db,"chat"));
  onSnapshot(q,(snap)=>{
    const box = document.getElementById("chatBox");
    box.innerHTML="";
    snap.forEach(docu=>{
      const d = docu.data();
      box.innerHTML += `<div><b>${d.autor}:</b> ${d.msg}</div>`;
    });
  });
}
</script><style>
body{margin:0;font-family:Arial;background:#0e0e0e;color:white}
header{background:#111;padding:10px;display:flex;justify-content:space-between;align-items:center}
button{background:#00ffea;border:none;padding:8px;border-radius:5px;font-weight:bold;cursor:pointer}
.section{padding:15px}
.card{background:#1a1a1a;padding:10px;margin:5px;border-radius:6px}
#chatBox{height:200px;overflow-y:auto;background:#111;padding:10px}
#perfilBox{display:none;align-items:center;gap:10px}
</style></head>
<body><header>
  <h2>CODM Meta BR</h2>
  <div id="perfilBox">
    <span id="perfilNome"></span>
    <button id="logoutBtn" onclick="logout()" style="display:none">Sair</button>
  </div>
  <button id="loginBtn" onclick="googleLogin()">Login Google</button>
</header><div id="adminPanel" class="section" style="display:none">
<h3>Admin Panel</h3>
<p>Gerenciar loadouts, armas, usuários</p>
</div><div class="section">
<h3>Criar Loadout</h3>
<input id="loadoutNome" placeholder="Nome do loadout">
<button onclick="createLoadout()">Criar</button>
</div><div class="section">
<h3>Fórum</h3>
<textarea id="forumText"></textarea>
<button onclick="createPost()">Postar</button>
<div id="forumPosts"></div>
</div><div class="section">
<h3>Chat em tempo real</h3>
<div id="chatBox"></div>
<input id="chatInput" placeholder="Digite...">
<button onclick="sendMsg()">Enviar</button>
</div></body>
</html>
