<!DOCTYPE html>
<html lang="vi">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Karaoke với sao & nhạc nền</title>
    <link href="https://fonts.googleapis.com/css2?family=Quicksand:wght@400;600&display=swap" rel="stylesheet">
    <style>
        body {
            margin: 0;
            font-family: 'Quicksand', sans-serif;
            background-color: #000;
            overflow: hidden;
            height: 100vh;
        }

        #starCanvas {
            position: fixed;
            top: 0;
            left: 0;
            width: 100vw;
            height: 100vh;
            z-index: 0;
        }

        #scrollWrapper {
            position: fixed;
            top: 0;
            left: 0;
            width: 100vw;
            height: 100vh;
            overflow-y: scroll;
            z-index: 2;
        }

        #lyricsContainer {
            min-height: 100vh;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: flex-start;
            text-align: center;
            padding: 80px 20px;
            box-sizing: border-box;
        }

        .line {
            display: block;
            margin: 25px 0;
            opacity: 0;
            transform: translateY(20px);
            transition: opacity 0.6s ease, transform 0.6s ease;
            font-size: 1.6rem;
            line-height: 2.2rem;
            color: #fff;
        }

        .line.show {
            opacity: 1;
            transform: translateY(0);
        }

        .cursor {
            display: inline-block;
            width: 3px;
            height: 1.2em;
            background-color: #ffc0cb;
            margin-left: 2px;
            animation: blink 0.8s infinite;
            vertical-align: bottom;
        }

        @keyframes blink {

            0%,
            50%,
            100% {
                opacity: 1;
            }

            25%,
            75% {
                opacity: 0;
            }
        }

        .typing-char {
            display: inline-block;
            margin-right: 2px;
            /* giữ khoảng cách chữ */
            white-space: pre;
            animation: bounceChar 0.4s ease forwards;
        }

        @keyframes bounceChar {
            0% {
                transform: translateY(0);
            }

            25% {
                transform: translateY(-10px);
            }

            50% {
                transform: translateY(0);
            }

            75% {
                transform: translateY(-5px);
            }

            100% {
                transform: translateY(0);
            }
        }

        #startBtn {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            padding: 12px 25px;
            font-size: 1.3rem;
            cursor: pointer;
            border: none;
            border-radius: 10px;
            background-color: #a215d5;
            color: #000;
            z-index: 3;
        }

        #welcomeOverlay {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.6);
            display: flex;
            justify-content: center;
            align-items: center;
            z-index: 10;
            pointer-events: none;
            opacity: 0;
            transition: opacity 1s ease;
        }

        #welcomeText {
            display: inline-block;
            width: 70%;
            text-align: center;
            font-family: 'Quicksand', sans-serif;
            font-size: 3rem;
            color: #000;
            background: rgb(180, 105, 255);
            padding: 20px 0;
            border-radius: 15px;
            text-shadow: 0 0 20px #ff69b4, 0 0 40px #ff69b4;
            transform: scale(0.5);
            transition: transform 1s ease, text-shadow 1s ease;
        }

        .active-char {
            color: #ff69b4;
            /* nổi bật */
            opacity: 1;
        }
    </style>
</head>

<body>
    <div id="welcomeOverlay"><span id="welcomeText">- Created by Mosey -</span></div>
    <canvas id="starCanvas"></canvas>
    <button id="startBtn">Start</button>

    <div id="scrollWrapper">
        <div id="lyricsContainer"></div>
    </div>

    <audio id="bgMusic" src="only u (reup).mp3"></audio>

    <script>
        // ========== STAR BACKGROUND ==========
        const canvas = document.getElementById("starCanvas");
        const ctx = canvas.getContext("2d");
        let W = canvas.width = window.innerWidth;
        let H = canvas.height = window.innerHeight;
        window.addEventListener("resize", () => {
            W = canvas.width = window.innerWidth;
            H = canvas.height = window.innerHeight;
        });
        class Star {
            constructor() {
                this.reset();
                this.tail = [];
                this.twinkleSpeed = Math.random() * 0.05 + 0.02; // tốc độ nhấp nháy khác nhau
                this.twinklePhase = Math.random() * Math.PI * 2; // phase random
            }

            reset() {
                this.x = Math.random() * W;
                this.y = Math.random() * H;
                this.vx = (Math.random() * 2 - 1) * 1.2;
                this.vy = (Math.random() * 2 - 1) * 1.2;
                this.r = Math.random() * 2 + 1;
                this.baseAlpha = Math.random() * 0.7 + 0.3;
                this.tail = [];
            }

            update() {
                this.x += this.vx;
                this.y += this.vy;
                if (this.x < 0 || this.x > W) this.vx *= -1;
                if (this.y < 0 || this.y > H) this.vy *= -1;

                this.tail.push({ x: this.x, y: this.y, alpha: this.baseAlpha });
                if (this.tail.length > 5) this.tail.shift();

                // alpha ngẫu nhiên theo sin, twinkle độc lập từng star
                const time = Date.now();
                this.twinkleAlpha = this.baseAlpha + Math.sin(time * this.twinkleSpeed + this.twinklePhase) * 0.3;
                if (this.twinkleAlpha > 1) this.twinkleAlpha = 1;
                if (this.twinkleAlpha < 0.1) this.twinkleAlpha = 0.1;
            }

            draw() {
                // vẽ tail
                for (let i = 0; i < this.tail.length; i++) {
                    const t = this.tail[i];
                    ctx.beginPath();
                    ctx.arc(t.x, t.y, this.r * (i / this.tail.length), 0, 2 * Math.PI);
                    ctx.fillStyle = `rgba(255,255,255,${t.alpha * (i / this.tail.length)})`;
                    ctx.fill();
                }

                // vẽ star chính với twinkle
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.r, 0, 2 * Math.PI);
                ctx.fillStyle = `rgba(255,255,255,${this.twinkleAlpha})`;
                ctx.shadowBlur = 5;
                ctx.shadowColor = "#fff";
                ctx.fill();
            }
        }

        // tạo mảng stars
        const stars = Array.from({ length: 80 }, () => new Star());

        // animate
        (function animateStars() {
            ctx.clearRect(0, 0, W, H);
            stars.forEach(s => { s.update(); s.draw(); });
            requestAnimationFrame(animateStars);
        })();



        // ========== LYRICS ==========
        const rawLyrics = `
 only u (reup) - micah
...
Anh nhớ những ngày mà mình còn bên cạnh nhau
Thật đẹp biết mấy khoảnh khắc yêu thương này
Thật đẹp biết mấy những lúc ta vừa quen nhau
Thật đẹp biết mấy lúc ta mới yêu nhau
Khi đôi ta mới yêu nhau thường hay mộng mơ lắm
Mình từng hẹn ước sẽ mãi luôn bên nhau
Mình từng đã hứa sẽ mãi không bao giờ xa cách rời
Mà nay giờ đây hai đứa hai nơi
...
Có bao giờ em nhớ, nhớ về anh như ngày xưa.
Có bao giờ em nghĩ, nghĩ về anh bây giờ...
Có bao giờ em thấy thật tiếc cho tình yêu của đôi mình.
Và khi em gặp người mới chắc gì đã hơn anh.
Có bao giờ em nghĩ không ai hiểu em bằng anh.
Lúc em cần nũng nĩu trên bờ vai anh này
Lúc em cần ai đó ôm em thật lâu từ đằng sau lưng em này
...
`;
        const lyricsList = rawLyrics.split('\n').filter(l => l.trim() !== '');
        const lyricsContainer = document.getElementById('lyricsContainer');
        const scrollWrapper = document.getElementById('scrollWrapper');
        let lineIndex = 0;
        let cursor;
        const lineDelays = [400, 12000, 900, 900, 900, 900, 900, 900, 900, 500, 3000, 800, 800, 1200, 900, 900, 900, 300];

        function typeWordsEffect(text, lineDiv, callback) {
            if (!cursor) {
                cursor = document.createElement('span');
                cursor.classList.add('cursor');
            }
            lineDiv.appendChild(cursor);

            let i = 0;
            function showChar() {
                if (i < text.length) {
                    const charSpan = document.createElement('span');
                    charSpan.textContent = text[i];
                    charSpan.classList.add('typing-char', 'active-char');
                    lineDiv.insertBefore(charSpan, cursor);

                    // update opacity các chữ trước
                    Array.from(lineDiv.querySelectorAll('.typing-char')).forEach(c => {
                        if (c !== charSpan) {
                            c.style.transition = 'opacity 1s ease';
                            c.style.opacity = 0.5;
                            c.classList.remove('active-char');
                        }
                    });

                    const delay = text[i] === ' ' ? 20 : Math.random() * 40 + 40;
                    i++;
                    setTimeout(showChar, delay);
                } else {
                    // Khi dòng kết thúc, xóa class active-char của tất cả để không còn chữ sáng
                    Array.from(lineDiv.querySelectorAll('.typing-char.active-char')).forEach(c => {
                        c.style.transition = 'opacity 1s ease';
                        c.style.opacity = 0.5;
                        c.classList.remove('active-char');
                    });
                    callback && callback();
                }
            }
            showChar();
        }




        function scrollToCurrentLine(lineDiv) {
            scrollWrapper.scrollTo({
                top: lineDiv.offsetTop - window.innerHeight / 2 + lineDiv.offsetHeight / 2,
                behavior: 'smooth'
            });
        }

        function showNextLine() {
            if (lineIndex < lyricsList.length) {
                const lineDiv = document.createElement('div');
                lineDiv.classList.add('line');
                lyricsContainer.appendChild(lineDiv);
                requestAnimationFrame(() => lineDiv.classList.add('show'));

                typeWordsEffect(lyricsList[lineIndex], lineDiv, () => {
                    scrollToCurrentLine(lineDiv);
                    const delay = lineDelays[lineIndex] || 800;
                    lineIndex++;
                    setTimeout(showNextLine, delay);
                });
            }
        }

        // ========== START BUTTON ==========
        document.getElementById('startBtn').addEventListener('click', () => {
            document.getElementById('startBtn').style.display = 'none';
            const bgMusic = document.getElementById('bgMusic');
            bgMusic.removeAttribute('loop');
            bgMusic.play().catch(() => console.log('Autoplay bị chặn'));
            showNextLine();
        });

        // ========== WELCOME ==========
        window.addEventListener('load', () => {
            const overlay = document.getElementById('welcomeOverlay');
            const text = document.getElementById('welcomeText');
            setTimeout(() => {
                overlay.style.opacity = '1';
                text.style.transform = 'scale(1)';
            }, 100);
            setTimeout(() => {
                overlay.style.opacity = '0';
                text.style.transform = 'scale(0.5)';
                setTimeout(() => overlay.remove(), 1000);
            }, 2100);
        });
    </script>
</body>

</html>
