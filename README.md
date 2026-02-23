<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>매장용 원가관리</title>

<style>
body{
    margin:0;
    background:#0f0f0f;
    color:#fff;
    font-family:sans-serif;
}

.header{
    padding:15px;
    font-size:18px;
    font-weight:bold;
    background:#1c1c1c;
}

.result-bar{
    position:sticky;
    top:0;
    background:#151515;
    padding:15px;
    display:flex;
    justify-content:space-between;
    gap:10px;
    z-index:100;
    border-bottom:1px solid #333;
}

.card{
    flex:1;
    background:#1f1f1f;
    padding:12px;
    border-radius:12px;
    text-align:center;
}

.card h2{
    margin:5px 0;
    font-size:22px;
}

.green{color:#4caf50;}
.yellow{color:#ffb300;}
.red{color:#ff5252;}

.section{
    padding:15px;
}

input{
    width:100%;
    padding:14px;
    margin-bottom:12px;
    border-radius:10px;
    border:none;
    background:#222;
    color:#fff;
    font-size:16px;
}

button{
    width:100%;
    padding:16px;
    border:none;
    border-radius:12px;
    background:#e0b94f;
    font-weight:bold;
    font-size:16px;
    margin-bottom:15px;
}

.ingredient{
    background:#1a1a1a;
    padding:12px;
    border-radius:10px;
    margin-bottom:10px;
    display:flex;
    justify-content:space-between;
    align-items:center;
    font-size:14px;
}

.delete{
    color:#ff5252;
    cursor:pointer;
}
</style>
</head>
<body>

<div class="header">📊 매장 원가 관리</div>

<div class="result-bar">
    <div class="card">
        <div>원가율</div>
        <h2 id="ratio">0%</h2>
    </div>
    <div class="card">
        <div>마진</div>
        <h2 id="margin">₩0</h2>
    </div>
    <div class="card">
        <div>BEP</div>
        <h2 id="bep">0개</h2>
    </div>
</div>

<div class="section">
    <input type="number" id="price" placeholder="판매가 (원)">
    <input type="number" id="fixed" placeholder="월 고정비 (원)">
</div>

<div class="section">
    <h3>재료 추가</h3>
    <input type="text" id="ingName" placeholder="재료명">
    <input type="number" id="ingCost" placeholder="투입 원가 (원)">
    <button onclick="addIngredient()">+ 재료 추가</button>
</div>

<div class="section">
    <h3>재료 목록</h3>
    <div id="list"></div>
</div>

<script>
let ingredients = JSON.parse(localStorage.getItem("ingredients")) || [];

function save(){
    localStorage.setItem("ingredients", JSON.stringify(ingredients));
}

function addIngredient(){
    const name = document.getElementById("ingName").value;
    const cost = parseFloat(document.getElementById("ingCost").value);
    if(!name || !cost) return;

    ingredients.push({name,cost});
    save();
    render();
}

function deleteIngredient(index){
    if(confirm("삭제하시겠습니까?")){
        ingredients.splice(index,1);
        save();
        render();
    }
}

function render(){
    const list = document.getElementById("list");
    list.innerHTML = "";

    ingredients.forEach((ing,i)=>{
        list.innerHTML += `
            <div class="ingredient">
                ${ing.name} - ₩${ing.cost.toLocaleString()}
                <span class="delete" onclick="deleteIngredient(${i})">삭제</span>
            </div>
        `;
    });

    calculate();
}

function calculate(){
    const price = parseFloat(document.getElementById("price").value) || 0;
    const fixed = parseFloat(document.getElementById("fixed").value) || 0;

    const totalCost = ingredients.reduce((sum,i)=>sum+i.cost,0);

    if(price>0){
        const ratio = (totalCost/price)*100;
        const margin = price-totalCost;
        const bep = margin>0 ? Math.ceil(fixed/margin) : 0;

        const ratioEl = document.getElementById("ratio");
        ratioEl.textContent = ratio.toFixed(1)+"%";

        ratioEl.classList.remove("green","yellow","red");
        if(ratio<=30) ratioEl.classList.add("green");
        else if(ratio<=35) ratioEl.classList.add("yellow");
        else ratioEl.classList.add("red");

        document.getElementById("margin").textContent =
            "₩"+margin.toLocaleString();

        document.getElementById("bep").textContent =
            bep+"개";
    }
}

document.getElementById("price").addEventListener("input",calculate);
document.getElementById("fixed").addEventListener("input",calculate);

render();
</script>

</body>
</html>
