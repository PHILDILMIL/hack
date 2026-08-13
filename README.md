<!DOCTYPE html>
<html lang="de">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>System Diagnostics</title>

<style>
* {
    box-sizing: border-box;
}

html, body {
    margin: 0;
    width: 100%;
    height: 100%;
    overflow: hidden;
    background: #020202;
    font-family: Consolas, monospace;
}

/* Startbildschirm */
#consent {
    position: fixed;
    inset: 0;
    background: #050505;
    color: white;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-direction: column;
    text-align: center;
    z-index: 100;
}

#consent button {
    background: #111;
    color: white;
    border: 1px solid #666;
    padding: 15px 35px;
    font: 18px Consolas, monospace;
    cursor: pointer;
}

/* Scan */
#terminal {
    width: 100vw;
    height: 100vh;
    padding: 40px;
    background: #030303;
    color: #00ff55;
}

.header {
    color: #777;
    border-bottom: 1px solid #222;
    padding-bottom: 15px;
}

.warning {
    color: #ff3333;
    font-size: 32px;
    font-weight: bold;
    text-align: center;
    margin: 45px 0;
    animation: blink .7s infinite;
}

@keyframes blink {
    50% {
        opacity: .25;
    }
}

#output {
    font-size: 18px;
    line-height: 1.8;
}

.progress-container {
    position: fixed;
    left: 40px;
    right: 40px;
    bottom: 70px;
    height: 25px;
    border: 1px solid #00ff55;
}

#progress {
    width: 0%;
    height: 100%;
    background: #00ff55;
    box-shadow: 0 0 20px #00ff55;
}

#percent {
    position: fixed;
    right: 40px;
    bottom: 35px;
    color: #00ff55;
}

/* Frage */
#question {
    display: none;
    position: fixed;
    inset: 0;
    background: #050505;
    color: white;
    align-items: center;
    justify-content: center;
    flex-direction: column;
    font-family: Arial, sans-serif;
    z-index: 200;
}

#question h1 {
    font-size: 32px;
    font-weight: normal;
    margin-bottom: 35px;
}

.buttons {
    display: flex;
    gap: 20px;
}

.buttons button {
    min-width: 120px;
    padding: 14px 30px;
    font-size: 18px;
    cursor: pointer;
    border: 1px solid #777;
    background: #151515;
    color: white;
}

.buttons button:hover {
    background: #292929;
}

/* Neustart */
#restart {
    display: none;
    position: fixed;
    inset: 0;
    background: #050505;
    color: white;
    align-items: center;
    justify-content: center;
    flex-direction: column;
    font-family: Arial, sans-serif;
    z-index: 300;
}

.spinner {
    width: 60px;
    height: 60px;
    border: 5px solid #333;
    border-top-color: white;
    border-radius: 50%;
    animation: spin 1s linear infinite;
    margin-bottom: 30px;
}

@keyframes spin {
    to {
        transform: rotate(360deg);
    }
}

.restart-text {
    font-size: 30px;
}

.subtext {
    margin-top: 15px;
    color: #999;
}

/* Schwarzer Bildschirm */
#black {
    display: none;
    position: fixed;
    inset: 0;
    width: 100vw;
    height: 100vh;
    background: black;
    z-index: 999999;
}

/* Mauszeiger im schwarzen Bildschirm ausblenden */
#black.active {
    cursor: none;
}
</style>
</head>

<body>

<!-- Akzeptieren -->
<div id="consent">
    <h1>System Diagnostics</h1>
    <p>Bitte akzeptieren Sie, um die Systemdiagnose zu starten.</p>

    <button id="accept">
        Akzeptieren & Starten
    </button>
</div>

<!-- Scan -->
<div id="terminal">

    <div class="header">
        SYSTEM DIAGNOSTICS // SECURE MODE
    </div>

    <div class="warning">
        ⚠ KRITISCHER SYSTEMSCAN ⚠
    </div>

    <div id="output"></div>

    <div class="progress-container">
        <div id="progress"></div>
    </div>

    <div id="percent">0%</div>

</div>

<!-- Neustart-Frage -->
<div id="question">

    <h1>Wollen Sie den PC neu starten?</h1>

    <div class="buttons">
        <button id="yes">Ja</button>
        <button id="no">Nein</button>
    </div>

</div>

<!-- Neustart -->
<div id="restart">

    <div class="spinner"></div>

    <div class="restart-text">
        Der PC wird neu gestartet
    </div>

    <div class="subtext" id="countdown">
        Neustart wird vorbereitet...
    </div>

</div>

<!-- Schwarzer Bildschirm -->
<div id="black"></div>


<script>

const accept = document.getElementById("accept");


/* Akzeptieren + Vollbild + Scan starten */
accept.addEventListener("click", async () => {

    try {
        await document.documentElement.requestFullscreen();
    } catch (error) {
        console.log("Vollbild konnte nicht aktiviert werden.");
    }

    document.getElementById("consent").style.display = "none";

    startScan();
});


/* Scan */
function startScan() {

    const output = document.getElementById("output");
    const progress = document.getElementById("progress");
    const percent = document.getElementById("percent");

    const messages = [
        "[SYSTEM] Initialisiere Systemanalyse...",
        "[OK] Arbeitsspeicher überprüft",
        "[OK] Systemprozesse analysiert",
        "[SCAN] Überprüfe Systemdateien...",
        "[SCAN] Überprüfe Sicherheitsstatus...",
        "[WARNING] Ungewöhnliche Systemaktivität erkannt",
        "[SCAN] Reparaturprozess gestartet...",
        "[OK] Systemdateien verarbeitet",
        "[OK] Sicherheitsprüfung abgeschlossen",
        "[SYSTEM] Vorbereitung des Neustarts..."
    ];

    let value = 0;
    let message = 0;

    const timer = setInterval(() => {

        value += Math.floor(Math.random() * 4) + 1;

        if (value > 100) {
            value = 100;
        }

        progress.style.width = value + "%";
        percent.textContent = value + "%";

        if (
            message < messages.length &&
            value >= (message + 1) * 9
        ) {

            const line = document.createElement("div");

            line.textContent = messages[message];

            output.appendChild(line);

            message++;

            if (output.children.length > 12) {
                output.removeChild(output.firstChild);
            }
        }

        if (value >= 100) {

            clearInterval(timer);

            setTimeout(() => {

                document.getElementById("terminal").style.display = "none";

                document.getElementById("question").style.display = "flex";

            }, 1500);
        }

    }, 450);
}


/* Beide Buttons führen zum Fake-Neustart */
document.getElementById("yes").onclick = fakeRestart;
document.getElementById("no").onclick = fakeRestart;


function fakeRestart() {

    document.getElementById("question").style.display = "none";

    document.getElementById("restart").style.display = "flex";

    let seconds = 20;

    const countdown = document.getElementById("countdown");

    countdown.textContent =
        "Neustart wird vorbereitet... " + seconds + " Sekunden";


    const timer = setInterval(() => {

        seconds--;

        countdown.textContent =
            "Neustart wird vorbereitet... " + seconds + " Sekunden";


        if (seconds <= 0) {

            clearInterval(timer);

            document.getElementById("restart").style.display = "none";

            const black = document.getElementById("black");

            black.style.display = "block";

            /*
             * Mauszeiger verschwindet nur innerhalb
             * des schwarzen Browser-Bereichs.
             */
            black.classList.add("active");

            document.body.style.cursor = "none";

        }

    }, 1000);
}

</script>

</body>
</html>
::: <style>
#black {
    display: none;
    position: fixed;
    inset: 0;
    width: 100vw;
    height: 100vh;
    background: #000;
    z-index: 999999;
    cursor: none;
}
</style>

<script>
function showBlackScreen() {
    const black = document.getElementById("black");

    black.style.display = "block";

    // Mauszeiger innerhalb der Seite unsichtbar
    document.documentElement.style.cursor = "none";
    document.body.style.cursor = "none";
    black.style.cursor = "none";
}
</script>
:::
