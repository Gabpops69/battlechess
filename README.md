<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Chess Arena</title>

    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
        }

        body {
            min-height: 100vh;
            font-family: Arial, sans-serif;
            background: #151515;
            color: white;
            display: flex;
            justify-content: center;
            align-items: center;
            padding: 20px;
        }

        .game {
            width: 100%;
            max-width: 1100px;
            display: flex;
            gap: 30px;
            align-items: center;
            justify-content: center;
        }

        /* =========================
           PANNEAU
        ========================= */

        .panel {
            width: 260px;
            background: #222;
            border-radius: 16px;
            padding: 24px;
            box-shadow: 0 10px 40px rgba(0,0,0,.35);
        }

        .logo {
            text-align: center;
            font-size: 28px;
            font-weight: bold;
            margin-bottom: 25px;
        }

        .player {
            background: #2d2d2d;
            padding: 14px;
            border-radius: 10px;
            margin-bottom: 12px;
        }

        .player-name {
            font-size: 15px;
            margin-bottom: 8px;
        }

        .timer {
            font-size: 28px;
            font-weight: bold;
            font-family: monospace;
        }

        .status {
            text-align: center;
            min-height: 45px;
            margin: 20px 0;
            font-size: 16px;
            color: #ddd;
        }

        button {
            width: 100%;
            border: none;
            border-radius: 10px;
            padding: 13px;
            background: #3b82f6;
            color: white;
            font-size: 15px;
            font-weight: bold;
            cursor: pointer;
            transition: .2s;
        }

        button:hover {
            background: #2563eb;
            transform: translateY(-1px);
        }

        /* =========================
           ECHIQUIER
        ========================= */

        .board-wrapper {
            width: min(80vw, 720px);
            height: min(80vw, 720px);
            max-width: 720px;
            max-height: 720px;
        }

        .board {
            width: 100%;
            height: 100%;
            display: grid;
            grid-template-columns: repeat(8, 1fr);
            grid-template-rows: repeat(8, 1fr);
            border: 8px solid #111;
            border-radius: 8px;
            overflow: hidden;
            box-shadow: 0 15px 50px rgba(0,0,0,.5);
        }

        .square {
            position: relative;
            display: flex;
            align-items: center;
            justify-content: center;
            cursor: pointer;
            user-select: none;
        }

        .light {
            background: #f0d9b5;
        }

        .dark {
            background: #b58863;
        }

        .piece {
            font-size: clamp(30px, 7vw, 65px);
            line-height: 1;
            cursor: grab;
            filter: drop-shadow(0 2px 1px rgba(0,0,0,.25));
            transition: transform .1s;
        }

        .piece:hover {
            transform: scale(1.08);
        }

        .selected {
            box-shadow: inset 0 0 0 5px #facc15;
        }

        .possible::after {
            content: "";
            width: 25%;
            height: 25%;
            border-radius: 50%;
            background: rgba(0,0,0,.25);
            position: absolute;
        }

        .capture::after {
            content: "";
            position: absolute;
            inset: 7px;
            border: 5px solid rgba(200, 40, 40, .65);
            border-radius: 50%;
        }

        .check {
            background: #d64545 !important;
        }

        /* =========================
           RESPONSIVE
        ========================= */

        @media (max-width: 800px) {
            body {
                align-items: flex-start;
            }

            .game {
                flex-directio
```
