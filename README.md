
<title>人工衛星シミュレーション

<style>

body{
    font-family:sans-serif;
    margin:20px;
}

.container{
    display:flex;
    gap:20px;
}

.controls{
    width:320px;
}

label{
    display:block;
    margin-top:10px;
}

input{
    width:140px;
}

canvas{
    background:#000;
    border:1px solid #666;
}

.info{
    margin-top:10px;
    font-size:18px;
    font-weight:bold;
}

button{
    margin:2px;
}

</style>
</head>

<body>

<h1>人工衛星シミュレーション</h1>

<div class="container">

<div class="controls">

<label>
質量倍率（地球比）
<br>
<input
id="massFactor"
type="number"
value="1"
step="0.1">
</label>

<button onclick="doubleMass()">
2倍
</button>

<button onclick="blackHoleMass()">
7.2億倍
</button>

<p>
現在の質量<br>
<span id="massDisplay"></span>
</p>

<hr>

<label>
半径倍率（地球比）
<br>
<input
id="radiusFactor"
type="number"
value="1"
step="0.1">
</label>

<button onclick="halfRadius()">
半径1/2
</button>

<button onclick="blackHoleRadius()">
半径約8.9mm
</button>

<p>
現在の半径<br>
<span id="radiusDisplay"></span>
</p>

<hr>

<label>
発射速度 (km/s)
<br>
<input
id="velocity"
type="number"
value="7.9"
step="0.1">
</label>

<br><br>

<button onclick="lightSpeed()">
光速度モード
</button>

<button onclick="launch()">
打上げ
</button>

<button onclick="resetSimulation()">
リセット
</button>


<div class="info">
軌道：
<span id="orbitType">未判定</span>
</div>

</div>

<canvas
id="canvas"
width="900"
height="700">
</canvas>

</div>

<script>

const G = 6.674e-11;

const EARTH_MASS = 5.972e24;

const EARTH_RADIUS = 6.37e6;

const canvas =
document.getElementById("canvas");

const ctx =
canvas.getContext("2d");

let planet;

let orbitData;

let satelliteIndex = 0;

let animationId;


function updateDisplay(){

    const mf =
    Number(
        massFactor.value
    );

    const rf =
    Number(
        radiusFactor.value
    );

    const M =
    EARTH_MASS * mf;

    const R =
    EARTH_RADIUS * rf;

    massDisplay.textContent =
    M.toExponential(3)
    +" kg";

    radiusDisplay.textContent =
    R.toExponential(3)
    +" m";
}

updateDisplay();

massFactor.oninput =
updateDisplay;

radiusFactor.oninput =
updateDisplay;


function doubleMass(){

    massFactor.value =
    Number(
        massFactor.value
    ) * 2;

    updateDisplay();
}

function blackHoleMass(){

    massFactor.value =
    7.2e8;

    updateDisplay();
}

function halfRadius(){

    radiusFactor.value =
    Number(
        radiusFactor.value
    ) / 2;

    updateDisplay();
}

function blackHoleRadius(){

    radiusFactor.value =
    1.39e-9;

    updateDisplay();
}

function lightSpeed(){

    velocity.value =
    300000;
}

function calculateOrbit(
    M,
    R,
    V0
){

    const dt = 10;

    const maxSteps =
    500000;

    let x = R;

    let y = 0;

    let vx = 0;

    let vy = V0;

    const path = [];

    for(
        let i=0;
        i<maxSteps;
        i++
    ){

        path.push({
            x:x,
            y:y
        });

        const r =
        Math.sqrt(
            x*x +
            y*y
        );

        const ax =
        -G*M*x/
        Math.pow(
            r,
            3
        );

        const ay =
        -G*M*y/
        Math.pow(
            r,
            3
        );

        vx += ax*dt;
        vy += ay*dt;

        x += vx*dt;
        y += vy*dt;
    }

    return path;
}

function launch(){

    cancelAnimationFrame(
        animationId
    );

    satelliteIndex = 0;

    const M =
    EARTH_MASS *
    Number(
        massFactor.value
    );

    const R =
    EARTH_RADIUS *
    Number(
        radiusFactor.value
    );

    const V =
    Number(
        velocity.value
    ) * 1000;

    planet = {

        mass:M,

        radius:R

    };

    const V1 =
    Math.sqrt(
        G*M/R
    );

    const V2 =
    Math.sqrt(
        2*G*M/R
    );

    let type;

    if(V < V1){

        type = "失敗";
    }
    else if(
        Math.abs(
        V-V1
        )/V1 < 0.03
    ){

        type =
        "円軌道";
    }
    else if(
        V < V2
    ){

        type =
        "楕円軌道";
    }
    else{

        type =
        "脱出";
    }

    orbitData = {

        type:type,

        path:
        calculateOrbit(
            M,
            R,
            V
        )
    };

    status.textContent =
    type;

    orbitType.textContent =
    type;

    animate();
}


function drawPlanet(scale){

    ctx.fillStyle =
    "deepskyblue";

    ctx.beginPath();

    ctx.arc(
        450,
        350,
        planet.radius *
        scale,
        0,
        Math.PI*2
    );

    ctx.fill();
}

function drawOrbit(scale){

    switch(
        orbitData.type
    ){

        case "失敗":
        ctx.strokeStyle =
        "red";
        break;

        case "円軌道":
        ctx.strokeStyle =
        "lime";
        break;

        case "楕円軌道":
        ctx.strokeStyle =
        "yellow";
        break;

        case "脱出":
        ctx.strokeStyle =
        "orange";
        break;
    }

    ctx.beginPath();

    orbitData.path.forEach(

        (p,i)=>{

        const px =
        450+p.x*scale;

        const py =
        350+p.y*scale;

        if(i===0){

            ctx.moveTo(px,py);

        }else{

            ctx.lineTo(px,py);

        }
    });

    ctx.stroke();
}

function drawSatellite(x,y){

    ctx.fillStyle =
    "white";

    ctx.beginPath();

    ctx.arc(
        x,
        y,
        5,
        0,
        Math.PI*2
    );

    ctx.fill();
}

function animate(){

    ctx.clearRect(
        0,
        0,
        canvas.width,
        canvas.height
    );

    const scale =
    200/
    planet.radius;

    drawPlanet(scale);

    drawOrbit(scale);

    const point =
    orbitData.path[
        satelliteIndex
    ];

    drawSatellite(

        450+
        point.x*scale,

        350+
        point.y*scale

    );

    satelliteIndex++;

    if(
        satelliteIndex >=
        orbitData.path.length
    ){

        satelliteIndex = 0;
    }

    animationId =
    requestAnimationFrame(
        animate
    );
}

</script>
</body>
</html>
