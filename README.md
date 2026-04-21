<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>EduCraft BNCC - Edição Definitiva</title>
    <script src="https://tailwindcss.com"></script>
    <style>
        @import url('https://googleapis.com');
        body { image-rendering: pixelated; font-family: 'Press Start 2P', cursive; transition: 0.3s; background-color: #1a2e1a; }
        .pixel-border { box-shadow: 4px 4px 0 #000; border: 4px solid #fff; }
        .locked { filter: grayscale(1) brightness(0.4); cursor: not-allowed; }
        .night-mode { background-color: #0c001a !important; color: #ff5555; }
        @keyframes shake { 0% {transform: translate(2px, 2px);} 50% {transform: translate(-3px, -3px);} 100% {transform: translate(2px, 2px);} }
        .damage { animation: shake 0.2s 3; }
        .btn-minecraft:active { transform: translateY(4px); box-shadow: 0px 0px 0 #000; }
    </style>
</head>
<body id="body" class="text-white text-[10px] sm:text-[12px]">

<!-- BARRA DE STATUS -->
<div class="p-4 flex justify-between items-center bg-black/60 sticky top-0 z-50 border-b-4 border-white/20">
    <div id="hearts" class="text-lg tracking-widest">❤️❤️❤️</div>
    <div class="text-yellow-400">XP: <span id="xp">0</span></div>
</div>

<!-- INVENTÁRIO DE ITENS -->
<div class="px-4 py-2 flex justify-around bg-black/40 border-b-4 border-black" id="inventory">
    <div id="item1" class="w-12 h-12 border-4 border-gray-700 flex items-center justify-center bg-gray-800 grayscale" title="Matemática">⛏️</div>
    <div id="item2" class="w-12 h-12 border-4 border-gray-700 flex items-center justify-center bg-gray-800 grayscale" title="Português">📖</div>
    <div id="item3" class="w-12 h-12 border-4 border-gray-700 flex items-center justify-center bg-gray-800 grayscale" title="Ciências">🧪</div>
    <div id="item4" class="w-12 h-12 border-4 border-gray-700 flex items-center justify-center bg-gray-800 grayscale" title="História">⚔️</div>
    <div id="item5" class="w-12 h-12 border-4 border-gray-700 flex items-center justify-center bg-gray-800 grayscale" title="Geografia">🗺️</div>
</div>

<!-- MAPA DE BIOMAS -->
<div id="map" class="grid grid-cols-1 gap-4 p-6 overflow-y-auto">
    <button onclick="start(1)" id="b1" class="btn-minecraft pixel-border bg-green-700 p-6 text-left hover:bg-green-600">🌲 FLORESTA (MATEMÁTICA)</button>
    <button onclick="start(2)" id="b2" class="btn-minecraft pixel-border bg-blue-700 p-6 text-left locked">🌊 OCEANO (PORTUGUÊS) 🔒</button>
    <button onclick="start(3)" id="b3" class="btn-minecraft pixel-border bg-yellow-800 p-6 text-left locked">🏜️ DESERTO (CIÊNCIAS) 🔒</button>
    <button onclick="start(4)" id="b4" class="btn-minecraft pixel-border bg-red-800 p-6 text-left locked">🌋 VULCÃO (HISTÓRIA) 🔒</button>
    <button onclick="start(5)" id="b5" class="btn-minecraft pixel-border bg-purple-800 p-6 text-left locked">🏔️ MONTANHA (GEOGRAFIA) 🔒</button>
    <button onclick="start('boss')" id="bb" class="btn-minecraft pixel-border bg-gray-950 p-6 text-center locked border-red-600 text-red-500">👹 BOSS FINAL: REVISÃO 🔒</button>
</div>

<!-- TELA DO JOGO (QUIZ) -->
<div id="game" class="hidden p-6">
    <div id="container" class="bg-gray-900 p-6 pixel-border min-h-[300px] flex flex-col justify-between">
        <div>
            <div class="flex justify-between text-yellow-500 mb-6 text-[8px]">
                <span id="skill-tag">BNCC</span>
                <span id="timer" class="text-red-500">20s</span>
            </div>
            <h2 id="q-text" class="leading-relaxed mb-10 text-center uppercase"></h2>
            <div id="ans-box" class="space-y-4"></div>
        </div>

        <!-- CARD DE ESTUDO (REPARAÇÃO DE MEMÓRIA) -->
        <div id="card" class="hidden mt-6 bg-blue-900 p-4 border-4 border-white">
            <p id="card-text" class="mb-4 text-center leading-loose"></p>
            <button onclick="closeCard()" class="bg-white text-black p-3 w-full font-bold">ESTUDAR E VOLTAR</button>
        </div>
    </div>
    <button onclick="back()" class="mt-6 text-gray-400 w-full text-center">SAIR DO JOGO</button>
</div>

<script>
// LÓGICA E CONTEÚDO
let unlocked = parseInt(localStorage.getItem("unlocked")) || 1;
let lives = 3;
let xp = parseInt(localStorage.getItem("xp")) || 0;
let current = 1;
let index = 0;
let time = 20;
let clock;

const db = {
    1: { n: "MAT", sk: "EF09MA01", q: [
        {q:"QUANTO É 10³?", a:["30","100","1000"], c:2, e:"10x10x10 = 1000"},
        {q:"RAIZ DE 144?", a:["12","14","10"], c:0, e:"12x12 = 144"},
        {q:"5² + 5 É?", a:["15","30","25"], c:1, e:"25 + 5 = 30"}
    ]},
    2: { n: "POR", sk: "EF69LP27", q: [
        {q:"FATO É DIFERENTE DE:", a:["NOTÍCIA","OPINIÃO","DADO"], c:1, e:"Opinião é o que alguém pensa, Fato é real."},
        {q:"CLICKBAIT SERVE PARA:", a:["INFORMAR","ENGANAR/ATRAIR","ENSINAR"], c:1, e:"Iscas de clique usam mentiras para atrair."}
    ]},
    3: { n: "CIE", sk: "EF09CI01", q: [
        {q:"O NÚCLEO DO ÁTOMO TEM:", a:["SÓ ELÉTRONS","PRÓTONS E NÊUTRONS","NADA"], c:1, e:"O núcleo é positivo e neutro."},
        {q:"CARGA DO ELÉTRON?", a:["POSITIVA","NEGATIVA","NEUTRA"], c:1, e:"Elétrons são negativos."}
    ]},
    4: { n: "HIS", sk: "EF09HI10", q: [
        {q:"GETÚLIO VARGAS CRIOU A:", a:["INTERNET","CLT (LEIS TRAB.)","NASA"], c:1, e:"A CLT garantiu direitos aos trabalhadores."},
        {q:"HITLER LIDEROU O:", a:["FASCISMO","NAZISMO","SOCIALISMO"], c:1, e:"O Nazismo foi o regime alemão."}
    ]},
    5: { n: "GEO", sk: "EF09GE01", q: [
        {q:"O QUE É GLOBALIZAÇÃO?", a:["INTEGRAÇÃO MUNDIAL","ISOLAMENTO","GUERRA"], c:0, e:"É a conexão econômica e cultural do mundo."},
        {q:"O PIB MEDE A:", a:["ÁREA DO PAÍS","RIQUEZA PRODUZIDA","POPULAÇÃO"], c:1, e:"Produto Interno Bruto é a riqueza total."}
    ]},
    'boss': { n: "BOSS", sk: "BOSS REVISÃO", q: [
        {q:"√49 + 3?", a:["10","7","13"], c:0, e:"7 + 3 = 10"},
        {q:"MOLÉCULA DA ÁGUA?", a:["CO2","H2O","O2"], c:1, e:"H2O: Hidrogênio e Oxigênio."}
    ]}
};

function start(id) {
    if(id !== 'boss' && id > unlocked) return;
    if(id === 'boss' && unlocked < 5) return;
    current = id; index = 0; lives = 3; 
    document.getElementById("map").classList.add("hidden");
    document.getElementById("game").classList.remove("hidden");
    refresh(); load();
}

function load() {
    clearInterval(clock); time = 20;
    document.getElementById("timer").innerText = time+"s";
    let q = db[current].q[index];
    document.getElementById("q-text").innerText = q.q;
    document.getElementById("skill-tag").innerText = "BNCC: " + db[current].sk;
    const box = document.getElementById("ans-box");
    box.innerHTML = "";
    q.a.forEach((a,i) => {
        let b = document.createElement("button");
        b.className = "btn-minecraft bg-gray-800 p-4 w-full pixel-border hover:bg-gray-700 text-left uppercase";
        b.innerText = a;
        b.onclick = () => check(i);
        box.appendChild(b);
    });
    clock = setInterval(() => {
        time--; document.getElementById("timer").innerText = time+"s";
        if(time <= 0) { hit("O TEMPO ESGOTOU! FOQUE MAIS!"); }
    }, 1000);
}

function check(i) {
    let q = db[current].q[index];
    if(i === q.c) {
        xp += 100; index++;
        if(index >= db[current].q.length) {
            alert("VOCÊ MINEROU ESTE CONHECIMENTO! 💎");
            if(current === unlocked && unlocked < 5) unlocked++;
            save(); back();
        } else { load(); }
    } else { hit(q.e); }
}

function hit(msg) {
    clearInterval(clock); lives--; refresh();
    document.getElementById("body").classList.add("night-mode");
    document.getElementById("container").classList.add("damage");
    document.getElementById("card-text").innerText = "DICA DO MESTRE: " + msg;
    document.getElementById("card").classList.remove("hidden");
    if(lives <= 0) { alert("VOCÊ DESMAIOU! REVISE E TENTE NOVAMENTE."); back(); }
}

function closeCard() {
    document.getElementById("card").classList.add("hidden");
    document.getElementById("body").classList.remove("night-mode");
    document.getElementById("container").classList.remove("damage");
    load();
}

function back() {
    clearInterval(clock);
    document.getElementById("game").classList.add("hidden");
    document.getElementById("map").classList.remove("hidden");
    save(); refresh();
}

function save() {
    localStorage.setItem("unlocked", unlocked);
    localStorage.setItem("xp", xp);
}

function refresh() {
    document.getElementById("hearts").innerText = "❤️".repeat(lives);
    document.getElementById("xp").innerText = xp;
    for(let i=1; i<=5; i++) {
        if(i <= unlocked) {
            document.getElementById("b"+i).classList.remove("locked");
            document.getElementById("b"+i).innerHTML = document.getElementById("b"+i).innerHTML.replace("🔒", "");
            document.getElementById("item"+i).classList.remove("grayscale");
        }
    }
    if(unlocked >= 5) document.getElementById("bb").classList.remove("locked");
}

refresh();
</script>
</body>
</html>
