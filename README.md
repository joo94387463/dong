# dong
var canvas = document.getElementById('canvas');
var ctx = canvas.getContext('2d');

canvas.width = window.innerWidth - 100;
canvas.height = window.innerHeight - 100;

// 배경 이미지 로드
var backgroundImage = new Image();
backgroundImage.src = 'back.png'; // 배경 이미지 경로를 설정하세요

var highScore = 0; // 최고 점수 기록용 변수

// 충돌 소리 추가
var crashSound = new Audio('y2mate.com - 효과음 마리오 죽는소리.mp3'); // 충돌 소리 파일 경로를 설정하세요

// 배경음악 추가
var backgroundMusic = new Audio('backgr.mp3'); // 배경음악 파일 경로를 설정하세요
backgroundMusic.loop = false; // 반복 재생은 수동으로 제어
backgroundMusic.volume = 0.5; // 볼륨 설정 (0.0 ~ 1.0)

// 배경음악 재생 시간 복원
var savedTime = localStorage.getItem('musicTime');
if (savedTime) {
    backgroundMusic.currentTime = parseFloat(savedTime); // 저장된 재생 시간 복원
}
backgroundMusic.play(); // 음악 재생

// 음악이 끝나면 다시 재생
backgroundMusic.addEventListener('ended', function () {
    backgroundMusic.currentTime = 0;
    backgroundMusic.play();
});

// 페이지가 닫히거나 새로고침될 때 재생 시간 저장
window.addEventListener('beforeunload', function () {
    localStorage.setItem('musicTime', backgroundMusic.currentTime); // 현재 재생 시간을 저장
});

backgroundImage.onload = function () {
    // Dino 걷는 이미지 두 개
    var dinoImage1 = new Image();
    dinoImage1.src = 'realrightdong.png'; // 첫 번째 걷는 이미지

    var dinoImage2 = new Image();
    dinoImage2.src = 'realleftdong.png'; // 두 번째 걷는 이미지

    var dino = {
        x: 100,
        y: 200,
        width: 50,
        height: 100,
        frameIndex: 0, // 현재 프레임 인덱스
        frameTimer: 0, // 프레임 교체를 위한 타이머
        draw() {
            this.frameTimer++;
            if (this.frameTimer % 5 === 0) { // 일정 주기마다 프레임 교체
                this.frameIndex = (this.frameIndex + 1) % 2;
            }
            if (this.frameIndex === 0) {
                ctx.drawImage(dinoImage1, this.x, this.y, this.width, this.height);
            } else {
                ctx.drawImage(dinoImage2, this.x, this.y, this.width, this.height);
            }
        }
    };

    var img2 = new Image();
    img2.src = 'Ohdong.png';

    class Cactus {
        constructor() {
            this.x = 400;
            this.y = 250;
            this.width = 50;
            this.height = 50;
        }
        draw() {
            ctx.drawImage(img2, this.x, this.y, this.width, this.height);
        }
    }

    var timer = 0;
    var cactuss = [];
    var jumptimer = 0;
    var stop_game;
    var whilejump = false;
    var canJump = true; // 점프 가능 여부
    var jumpSpeed = 4; // 점프 속도
    var gravitySpeed = 4; // 중력 속도
    var speed = 2; // 초기 장애물 속도
    var spawnRate = 100; // 기본 생성 주기
    var gameOver = false; // 게임 종료 상태
    var score = 0; // 현재 점수

    function move() {
        if (gameOver) return; // 게임 종료 상태에서는 move 중단

        stop_game = requestAnimationFrame(move);
        timer++;
        ctx.clearRect(0, 0, canvas.width, canvas.height);

        // 배경 이미지 그리기
        ctx.drawImage(backgroundImage, 0, 0, canvas.width, canvas.height);

        // 장애물 생성 로직
        if (timer % spawnRate === 0) {
            var cactus = new Cactus();
            cactuss.push(cactus);
        }

        cactuss.forEach((a, i, o) => {
            if (a.x < 0) {
                o.splice(i, 1); // 화면 밖으로 나가면 배열에서 제거
                score++; // 점수 증가
            }
            crush(a, dino);

            a.x -= speed; // 속도 반영
            a.draw();
        });

        // 점프 로직
        if (whilejump === true) {
            dino.y -= jumpSpeed; // 점프 속도 적용
            jumptimer++;
            if (dino.y <= 0) {
                dino.y = 0; // 창 상단을 넘어가지 않도록 제한
            }
        }
        if (whilejump === false) {
            if (dino.y < 200) {
                dino.y += gravitySpeed; // 중력 속도 적용
            } else {
                canJump = true; // 바닥에 착지하면 점프 가능
            }
        }
        if (jumptimer > 50) { // 점프 지속 시간 제한
            whilejump = false;
            jumptimer = 0;
        }

        dino.draw();

        // 점수 표시
        ctx.fillStyle = 'yellow';
        ctx.font = '20px Arial';
        ctx.textAlign = 'left';
        ctx.fillText('점수: ' + score, 10, 30); // 좌측 상단에 점수 표시

        // 최고 점수 표시
        ctx.fillText('최고 점수: ' + highScore, 10, 60);
    }

    function crush(cactus, dino) {
        var x_gap = cactus.x - (dino.x + dino.width);
        var y_gap = dino.y - (cactus.y + cactus.height);

        // 충돌 조건
        if (x_gap < 0 && cactus.x + cactus.width > dino.x &&
            y_gap < 0 && dino.y + dino.height > cactus.y) {
            gameOver = true;
            cancelAnimationFrame(stop_game); // 게임 정지

            // 충돌 소리 재생
            crashSound.play();
            backgroundMusic.pause()
            

            // 최고 점수 갱신
            if (score > highScore) {
                highScore = score;
            }

            // "게임 종료" 메시지 표시
            ctx.fillStyle = 'black';
            ctx.font = '68px Arial';
            ctx.textAlign = 'center';
            ctx.fillText('게임 종료', canvas.width / 2, canvas.height / 4);
        }
    }

    document.addEventListener('keydown', function (e) {
        if (e.code === 'Space') {
            if (gameOver) {
                resetGame(); // 게임 종료 후 스페이스바 눌렀을 때 게임 초기화
                move(); // 게임 다시 시작
            } else if (canJump) {
                whilejump = true;
                canJump = false; // 점프 중에는 다시 점프 불가
            }
        }
    });

    function resetGame() {
        // 게임 상태 초기화
        gameOver = false;
        dino.y = 200;
        dino.x = 100;
        jumpSpeed = 3;
        gravitySpeed = 4;
        speed = 2;
        spawnRate = 100;
        cactuss = [];
        timer = 0;
        jumptimer = 0;
        whilejump = false;
        canJump = true;
        score = 0; // 점수 초기화
        backgroundMusic.play();

   

    }

    // 게임 시작
    move();
};
