<body>
    <div class="container">
      <div class="hex central">
        <div>
         <h2>What we do on a...</h2>
        <h1>Site Survey</h2>
        </div>
      </div>
        <div class="hex top-left"></div>
        <div class="hex top-right"></div>
        <div class="hex middle-left"></div>
        <div class="hex middle-right"></div>
        <div class="hex bottom-left"></div>
        <div class="hex bottom-right"></div>
    </div>
</body>

body {
    display: flex;
    justify-content: center;
    align-items: center;
    margin: 0;
    background-color: #f0f0f0;
}

.container {
    position: relative;
}

.hex {
    position: absolute;
    width: 300px;
    height: 320px;
    background-color: #ffcc00;
    clip-path: polygon(50% 0%, 100% 25%, 100% 75%, 50% 100%, 0% 75%, 0% 25%);
    transform-origin: center;
     text-align: center; /* Center text horizontally */
     display: flex;
      justify-content: center;
     align-items: center;
}

.central {
    background-color: red;
    top: 0;
    left: 0;
    transform: translate(0, 0);
}

.top-left {
    top: -300px;
    left: -200px;
    transform: translate(0, 0);
}

.top-right {
    top: -300px;
    left: 200px;
    transform: translate(0, 0);
}

.middle-left {
    top: 0px;
    left: -360px;
    transform: translate(0, 0);
}

.middle-right {
    top: 0px;
    left: 360px;
    transform: translate(0, 0);
}

.bottom-left {
    top: 300px;
    left: -200px;
    transform: translate(0, 0);
}

.bottom-right {
    top: 300px;
    left: 200px;
    transform: translate(0, 0);
}
