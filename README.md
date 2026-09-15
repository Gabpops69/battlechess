<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Battle Chess</title>

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
                flex-direction: column;
                gap: 15px;
            }

            .panel {
                width: min(90vw, 600px);
                order: 2;
            }

            .board-wrapper {
                width: min(94vw, 600px);
                height: min(94vw, 600px);
            }

            .logo {
                margin-bottom: 12px;
            }

            .status {
                margin: 12px 0;
            }

            .player {
                display: inline-block;
                width: 48%;
                margin-right: 1%;
            }
        }
    </style>
</head>

<body>

<div class="game">

    <div class="board-wrapper">
        <div id="board" class="board"></div>
    </div>

    <div class="panel">

        <div class="logo">
            ♔ Chess Arena
        </div>

        <div class="player">
            <div class="player-name">⚪ Blancs</div>
            <div id="whiteTimer" class="timer">10:00</div>
        </div>

        <div class="player">
            <div class="player-name">⚫ Noirs</div>
            <div id="blackTimer" class="timer">10:00</div>
        </div>

        <div id="status" class="status">
            Tour des blancs
        </div>

        <button id="restart">
            Nouvelle partie
        </button>

    </div>

</div>

<script>

/* =====================================================
   PIECES
===================================================== */

const pieces = {
    wK: "♔",
    wQ: "♕",
    wR: "♖",
    wB: "♗",
    wN: "♘",
    wP: "♙",

    bK: "♚",
    bQ: "♛",
    bR: "♜",
    bB: "♝",
    bN: "♞",
    bP: "♟"
};


/* =====================================================
   POSITION INITIALE
===================================================== */

const initialBoard = [
    ["bR","bN","bB","bQ","bK","bB","bN","bR"],
    ["bP","bP","bP","bP","bP","bP","bP","bP"],
    [null,null,null,null,null,null,null,null],
    [null,null,null,null,null,null,null,null],
    [null,null,null,null,null,null,null,null],
    [null,null,null,null,null,null,null,null],
    ["wP","wP","wP","wP","wP","wP","wP","wP"],
    ["wR","wN","wB","wQ","wK","wB","wN","wR"]
];

let board = copyBoard(initialBoard);

let turn = "w";
let selected = null;
let gameOver = false;

let enPassant = null;

let castling = {
    wK: true,
    wQ: true,
    bK: true,
    bQ: true
};


/* =====================================================
   CHRONOMETRE
===================================================== */

let whiteTime = 600;
let blackTime = 600;

setInterval(() => {

    if (gameOver) return;

    if (turn === "w") {
        whiteTime--;
    } else {
        blackTime--;
    }

    updateTimers();

    if (whiteTime <= 0) {
        endGame("Les noirs gagnent au temps !");
    }

    if (blackTime <= 0) {
        endGame("Les blancs gagnent au temps !");
    }

}, 1000);


/* =====================================================
   AFFICHAGE
===================================================== */

const boardElement = document.getElementById("board");
const statusElement = document.getElementById("status");

function renderBoard() {

    boardElement.innerHTML = "";

    for (let r = 0; r < 8; r++) {

        for (let c = 0; c < 8; c++) {

            const square = document.createElement("div");

            square.className =
                "square " +
                ((r + c) % 2 === 0 ? "light" : "dark");

            square.dataset.row = r;
            square.dataset.col = c;

            const piece = board[r][c];

            if (piece) {

                const pieceElement = document.createElement("div");

                pieceElement.className = "piece";
                pieceElement.textContent = pieces[piece];

                square.appendChild(pieceElement);
            }

            square.addEventListener("click", () => {
                handleSquareClick(r, c);
            });

            boardElement.appendChild(square);
        }
    }

    highlightSelection();

    highlightKingInCheck();
}


/* =====================================================
   CLIC SUR CASE
===================================================== */

function handleSquareClick(r, c) {

    if (gameOver) return;

    const piece = board[r][c];

    // Rien n'est sélectionné
    if (!selected) {

        if (piece && colorOf(piece) === turn) {

            selected = { r, c };

            renderBoard();
        }

        return;
    }


    // Cliquer à nouveau sur la même pièce
    if (selected.r === r && selected.c === c) {

        selected = null;

        renderBoard();

        return;
    }


    // Changer de pièce
    if (piece && colorOf(piece) === turn) {

        selected = { r, c };

        renderBoard();

        return;
    }


    // Essayer de déplacer
    if (tryMove(selected.r, selected.c, r, c)) {

        selected = null;

        renderBoard();

        updateGameStatus();
    }
}


/* =====================================================
   DEPLACEMENT
===================================================== */

function tryMove(fr, fc, tr, tc) {

    const piece = board[fr][fc];

    if (!piece) return false;

    const color = colorOf(piece);

    if (color !== turn) return false;

    const legalMoves = getLegalMoves(fr, fc);

    const valid = legalMoves.some(
        move => move.r === tr && move.c === tc
    );

    if (!valid) return false;

    const captured = board[tr][tc];

    // Déplacement
    board[tr][tc] = piece;
    board[fr][fc] = null;

    // En passant
    if (
        piece[1] === "P" &&
        enPassant &&
        tr === enPassant.r &&
        tc === enPassant.c &&
        !captured
    ) {

        const capturedRow =
            color === "w" ? tr + 1 : tr - 1;

        board[capturedRow][tc] = null;
    }

    // Roque
    if (piece[1] === "K" && Math.abs(tc - fc) === 2) {

        if (tc === 6) {

            board[tr][5] = board[tr][7];
            board[tr][7] = null;

        } else if (tc === 2) {

            board[tr][3] = board[tr][0];
            board[tr][0] = null;
        }
    }

    // Mise à jour roque
    updateCastlingRights(piece, fr, fc);

    if (captured) {
        updateCastlingRights(captured, tr, tc);
    }

    // Promotion
    if (
        piece[1] === "P" &&
        (tr === 0 || tr === 7)
    ) {

        promotePawn(tr, tc, color);
    }

    // Nouveau en passant
    enPassant = null;

    if (
        piece[1] === "P" &&
        Math.abs(tr - fr) === 2
    ) {

        enPassant = {
            r: (tr + fr) / 2,
            c: fc
        };
    }

    turn = opposite(turn);

    return true;
}


/* =====================================================
   COUPS LEGAUX
===================================================== */

function getLegalMoves(r, c) {

    const piece = board[r][c];

    if (!piece) return [];

    const color = colorOf(piece);

    const pseudoMoves = getPseudoMoves(r, c);

    const legalMoves = [];

    for (const move of pseudoMoves) {

        const copy = copyBoard(board);

        const target = copy[move.r][move.c];

        copy[move.r][move.c] = piece;
        copy[r][c] = null;

        // En passant
        if (
            piece[1] === "P" &&
            enPassant &&
            move.r === enPassant.r &&
            move.c === enPassant.c &&
            !target
        ) {

            const capturedRow =
                color === "w"
                    ? move.r + 1
                    : move.r - 1;

            copy[capturedRow][move.c] = null;
        }

        if (!isKingInCheck(copy, color)) {
            legalMoves.push(move);
        }
    }

    return legalMoves;
}


/* =====================================================
   COUPS POSSIBLES
===================================================== */

function getPseudoMoves(r, c) {

    const piece = board[r][c];

    if (!piece) return [];

    const type = piece[1];
    const color = colorOf(piece);

    const moves = [];

    function add(r2, c2) {

        if (r2 < 0 || r2 > 7 || c2 < 0 || c2 > 7) {
            return;
        }

        const target = board[r2][c2];

        if (!target) {

            moves.push({r:r2,c:c2});

        } else if (colorOf(target) !== color) {

            if (target[1] !== "K") {
                moves.push({r:r2,c:c2});
            }
        }
    }


    /* PION */

    if (type === "P") {

        const direction = color === "w" ? -1 : 1;

        const startRow = color === "w" ? 6 : 1;

        if (
            r + direction >= 0 &&
            r + direction <= 7 &&
            !board[r + direction][c]
        ) {

            add(r + direction, c);

            if (
                r === startRow &&
                !board[r + direction * 2][c]
            ) {

                add(r + direction * 2, c);
            }
        }

        // Captures
        for (const dc of [-1,1]) {

            const nr = r + direction;
            const nc = c + dc;

            if (
                nr >= 0 &&
                nr <= 7 &&
                nc >= 0 &&
                nc <= 7
            ) {

                const target = board[nr][nc];

                if (
                    target &&
                    colorOf(target) !== color &&
                    target[1] !== "K"
                ) {
                    moves.push({r:nr,c:nc});
                }

                if (
                    enPassant &&
                    enPassant.r === nr &&
                    enPassant.c === nc
                ) {
                    moves.push({r:nr,c:nc});
                }
            }
        }
    }


    /* CAVALIER */

    if (type === "N") {

        const jumps = [
            [-2,-1],[-2,1],
            [-1,-2],[-1,2],
            [1,-2],[1,2],
            [2,-1],[2,1]
        ];

        for (const [dr,dc] of jumps) {
            add(r+dr,c+dc);
        }
    }


    /* ROI */

    if (type === "K") {

        for (let dr=-1; dr<=1; dr++) {

            for (let dc=-1; dc<=1; dc++) {

                if (dr !== 0 || dc !== 0) {
                    add(r+dr,c+dc);
                }
            }
        }

        // Roque
        if (!isKingInCheck(board, color)) {

            if (
                color === "w" &&
                r === 7 &&
                c === 4
            ) {

                if (
                    castling.wK &&
                    board[7][5] === null &&
                    board[7][6] === null &&
                    board[7][7] === "wR" &&
                    !squareAttacked(board,7,5,"b") &&
                    !squareAttacked(board,7,6,"b")
                ) {

                    moves.push({r:7,c:6});
                }

                if (
                    castling.wQ &&
                    board[7][1] === null &&
                    board[7][2] === null &&
                    board[7][3] === null &&
                    board[7][0] === "wR" &&
                    !squareAttacked(board,7,3,"b") &&
                    !squareAttacked(board,7,2,"b")
                ) {

                    moves.push({r:7,c:2});
                }
            }

            if (
                color === "b" &&
                r === 0 &&
                c === 4
            ) {

                if (
                    castling.bK &&
                    board[0][5] === null &&
                    board[0][6] === null &&
                    board[0][7] === "bR" &&
                    !squareAttacked(board,0,5,"w") &&
                    !squareAttacked(board,0,6,"w")
                ) {

                    moves.push({r:0,c:6});
                }

                if (
                    castling.bQ &&
                    board[0][1] === null &&
                    board[0][2] === null &&
                    board[0][3] === null &&
                    board[0][0] === "bR" &&
                    !squareAttacked(board,0,3,"w") &&
                    !squareAttacked(board,0,2,"w")
                ) {

                    moves.push({r:0,c:2});
                }
            }
        }
    }


    /* TOURS */

    if (type === "R") {

        slide(
            r,c,
            [[1,0],[-1,0],[0,1],[0,-1]],
            moves,
            color
        );
    }


    /* FOU */

    if (type === "B") {

        slide(
            r,c,
            [[1,1],[1,-1],[-1,1],[-1,-1]],
            moves,
            color
        );
    }


    /* DAME */

    if (type === "Q") {

        slide(
            r,c,
            [
                [1,0],[-1,0],[0,1],[0,-1],
                [1,1],[1,-1],[-1,1],[-1,-1]
            ],
            moves,
            color
        );
    }

    return moves;
}


/* =====================================================
   DEPLACEMENTS LINEAIRES
===================================================== */

function slide(r,c,directions,moves,color) {

    for (const [dr,dc] of directions) {

        let nr = r + dr;
        let nc = c + dc;

        while (
            nr >= 0 &&
            nr <= 7 &&
            nc >= 0 &&
            nc <= 7
        ) {

            const target = board[nr][nc];

            if (!target) {

                moves.push({r:nr,c:nc});

            } else {

                if (
                    colorOf(target) !== color &&
                    target[1] !== "K"
                ) {

                    moves.push({r:nr,c:nc});
                }

                break;
            }

            nr += dr;
            nc += dc;
        }
    }
}


/* =====================================================
   ECHEC
===================================================== */

function isKingInCheck(position, color) {

    let king = null;

    for (let r=0; r<8; r++) {

        for (let c=0; c<8; c++) {

            if (position[r][c] === color + "K") {
                king = {r,c};
            }
        }
    }

    if (!king) return true;

    return squareAttacked(
        position,
        king.r,
        king.c,
        opposite(color)
    );
}


function squareAttacked(position,r,c,attacker) {

    for (let rr=0; rr<8; rr++) {

        for (let cc=0; cc<8; cc++) {

            const piece = position[rr][cc];

            if (!piece || colorOf(piece) !== attacker) {
                continue;
            }

            const type = piece[1];

            // Pions
            if (type === "P") {

                const direction =
                    attacker === "w" ? -1 : 1;

                if (
                    rr + direction === r &&
                    Math.abs(cc-c) === 1
                ) {
                    return true;
                }
            }

            // Cavaliers
            if (type === "N") {

                const jumps = [
                    [-2,-1],[-2,1],
                    [-1,-2],[-1,2],
                    [1,-2],[1,2],
                    [2,-1],[2,1]
                ];

                if (
                    jumps.some(
                        ([dr,dc]) =>
                        rr+dr === r &&
                        cc+dc === c
                    )
                ) {
                    return true;
                }
            }

            // Roi
            if (type === "K") {

                if (
                    Math.max(
                        Math.abs(rr-r),
                        Math.abs(cc-c)
                    ) === 1
                ) {
                    return true;
                }
            }

            // Tours / Dames
            if (type === "R" || type === "Q") {

                if (
                    rr === r ||
                    cc === c
                ) {

                    if (clearPath(position,rr,cc,r,c)) {
                        return true;
                    }
                }
            }

            // Fous / Dames
            if (type === "B" || type === "Q") {

                if (
                    Math.abs(rr-r) ===
                    Math.abs(cc-c)
                ) {

                    if (clearPath(position,rr,cc,r,c)) {
                        return true;
                    }
                }
            }
        }
    }

    return false;
}


function clearPath(position,r1,c1,r2,c2) {

    const dr = Math.sign(r2-r1);
    const dc = Math.sign(c2-c1);

    let r = r1 + dr;
    let c = c1 + dc;

    while (r !== r2 || c !== c2) {

        if (position[r][c]) {
            return false;
        }

        r += dr;
        c += dc;
    }

    return true;
}


/* =====================================================
   MAT / NULLE
===================================================== */

function updateGameStatus() {

    const inCheck = isKingInCheck(board, turn);

    let hasMoves = false;

    for (let r=0; r<8; r++) {

        for (let c=0; c<8; c++) {

            const piece = board[r][c];

            if (
                piece &&
                colorOf(piece) === turn &&
                getLegalMoves(r,c).length > 0
            ) {

                hasMoves = true;
                break;
            }
        }

        if (hasMoves) break;
    }

    if (!hasMoves) {

        if (inCheck) {

            endGame(
                turn === "w"
                    ? "Échec et mat ! Les noirs gagnent 👑"
                    : "Échec et mat ! Les blancs gagnent 👑"
            );

        } else {

            endGame("Pat ! Partie nulle.");
        }

        return;
    }

    if (inCheck) {

        statusElement.textContent =
            "⚠️ Échec ! Tour des " +
            (turn === "w" ? "blancs" : "noirs");

    } else {

        statusElement.textContent =
            "Tour des " +
            (turn === "w" ? "blancs ⚪" : "noirs ⚫");
    }
}


/* =====================================================
   PROMOTION
===================================================== */

function promotePawn(r,c,color) {

    let choice = prompt(
        "Promotion : choisis une pièce\n" +
        "Q = Dame\n" +
        "R = Tour\n" +
        "B = Fou\n" +
        "N = Cavalier",
        "Q"
    );

    choice = choice
        ? choice.toUpperCase()
        : "Q";

    if (!["Q","R","B","N"].includes(choice)) {
        choice = "Q";
    }

    board[r][c] = color + choice;
}


/* =====================================================
   ROQUE
===================================================== */

function updateCastlingRights(piece,r,c) {

    if (piece === "wK") {
        castling.wK = false;
        castling.wQ = false;
    }

    if (piece === "bK") {
        castling.bK = false;
        castling.bQ = false;
    }

    if (piece === "wR") {

        if (r === 7 && c === 0) {
            castling.wQ = false;
        }

        if (r === 7 && c === 7) {
            castling.wK = false;
        }
    }

    if (piece === "bR") {

        if (r === 0 && c === 0) {
            castling.bQ = false;
        }

        if (r === 0 && c === 7) {
            castling.bK = false;
        }
    }
}


/* =====================================================
   VISUEL
===================================================== */

function highlightSelection() {

    if (!selected) return;

    const squares =
        document.querySelectorAll(".square");

    const selectedIndex =
        selected.r * 8 + selected.c;

    squares[selectedIndex].classList.add("selected");

    const moves =
        getLegalMoves(selected.r, selected.c);

    for (const move of moves) {

        const index =
            move.r * 8 + move.c;

        const square = squares[index];

        square.classList.add("possible");

        if (board[move.r][move.c]) {
            square.classList.add("capture");
        }
    }
}


function highlightKingInCheck() {

    const squares =
        document.querySelectorAll(".square");

    for (let r=0; r<8; r++) {

        for (let c=0; c<8; c++) {

            const piece = board[r][c];

            if (
                piece &&
                piece[1] === "K" &&
                isKingInCheck(board,colorOf(piece))
            ) {

                squares[r*8+c].classList.add("check");
            }
        }
    }
}


/* =====================================================
   UTILITAIRES
===================================================== */

function colorOf(piece) {
    return piece[0];
}

function opposite(color) {
    return color === "w" ? "b" : "w";
}

function copyBoard(board) {
    return board.map(row => [...row]);
}

function endGame(message) {

    gameOver = true;

    statusElement.textContent = message;
}


/* =====================================================
   CHRONOMETRES
===================================================== */

function formatTime(seconds) {

    seconds = Math.max(0, seconds);

    const minutes =
        Math.floor(seconds / 60);

    const secs =
        seconds % 60;

    return (
        String(minutes).padStart(2,"0") +
        ":" +
        String(secs).padStart(2,"0")
    );
}


function updateTimers() {

    document.getElementById("whiteTimer")
        .textContent = formatTime(whiteTime);

    document.getElementById("blackTimer")
        .textContent = formatTime(blackTime);
}


/* =====================================================
   NOUVELLE PARTIE
===================================================== */

document.getElementById("restart")
    .addEventListener("click", () => {

        board = copyBoard(initialBoard);

        turn = "w";

        selected = null;

        gameOver = false;

        enPassant = null;

        castling = {
            wK: true,
            wQ: true,
            bK: true,
            bQ: true
        };

        whiteTime = 600;
        blackTime = 600;

        updateTimers();

        statusElement.textContent =
            "Tour des blancs";

        renderBoard();
    });


/* =====================================================
   DEMARRAGE
===================================================== */

renderBoard();
updateTimers();

</script>

</body>
</html>
```
