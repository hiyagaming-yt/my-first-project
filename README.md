<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>2P Fighting Game</title>
  <style>
    body { margin:0; background:#111; }
    canvas { display:block; margin:20px auto; background:#333; border:3px solid white; }
  </style>
</head>
<body>
  <canvas id="game" width="900" height="450"></canvas>

  <script>
    const canvas = document.getElementById("game");
    const ctx = canvas.getContext("2d");

    const gravity = 0.8;

    class Fighter {
      constructor(x, color, controls) {
        this.x = x;
        this.y = 300;
        this.w = 50;
        this.h = 100;
        this.color = color;
        this.hp = 10;
        this.velX = 0;
        this.velY = 0;
        this.onGround = false;
        this.controls = controls;
        this.state = "idle"; // idle, punch, kick, block, jump
      }

      draw() {
        ctx.fillStyle = this.color;
        ctx.fillRect(this.x, this.y, this.w, this.h);

        // health bar
        ctx.fillStyle = "red";
        ctx.fillRect(this.x, this.y - 15, this.hp * 15, 10);

        // attack box
        if (this.state === "punch") {
          ctx.fillStyle = "yellow";
          ctx.fillRect(this.x + (this.color === "blue" ? this.w : -20), this.y+30, 20, 20);
        } else if (this.state === "kick") {
          ctx.fillStyle = "orange";
          ctx.fillRect(this.x + (this.color === "blue" ? this.w : -30), this.y+70, 30, 20);
        }
      }

      update(keys) {
        // horizontal move
        if (keys[this.controls.left]) this.velX = -5;
        else if (keys[this.controls.right]) this.velX = 5;
        else this.velX = 0;

        // jumping
        if (keys[this.controls.jump] && this.onGround) {
          this.velY = -15;
          this.onGround = false;
        }

        // attacks
        if (keys[this.controls.punch]) this.state = "punch";
        else if (keys[this.controls.kick]) this.state = "kick";
        else if (keys[this.controls.block]) this.state = "block";
        else this.state = "idle";

        // physics
        this.x += this.velX;
        this.y += this.velY;
        this.velY += gravity;

        // floor
        if (this.y + this.h >= canvas.height) {
          this.y = canvas.height - this.h;
          this.velY = 0;
          this.onGround = true;
        }
      }

      getAttackBox() {
        if (this.state === "punch") {
          return {x: this.x + (this.color === "blue" ? this.w : -20), y:this.y+30, w:20, h:20, dmg:1};
        }
        if (this.state === "kick") {
          return {x: this.x + (this.color === "blue" ? this.w : -30), y:this.y+70, w:30, h:20, dmg:2};
        }
        return null;
      }
    }

    // Controls
    const player1Controls = { left:"a", right:"d", jump:"w", punch:"t", kick:"y", block:"g" };
    const player2Controls = { left:"ArrowLeft", right:"ArrowRight", jump:"ArrowUp", punch:"/", kick:".", block:"Shift" };

    const p1 = new Fighter(200, "blue", player1Controls);
    const p2 = new Fighter(600, "green", player2Controls);

    const keys = {};
    document.addEventListener("keydown", e => keys[e.key] = true);
    document.addEventListener("keyup", e => keys[e.key] = false);

    function checkCollision(a, b) {
      return (a.x < b.x+b.w && a.x+a.w > b.x && a.y < b.y+b.h && a.y+a.h > b.y);
    }

    function handleAttacks(attacker, defender) {
      const atk = attacker.getAttackBox();
      if (atk && checkCollision(atk, defender)) {
        if (defender.state === "block") return; // blocked
        defender.hp = Math.max(0, defender.hp - atk.dmg);
      }
    }

    function update() {
      ctx.clearRect(0,0,canvas.width,canvas.height);

      p1.update(keys);
      p2.update(keys);

      handleAttacks(p1, p2);
      handleAttacks(p2, p1);

      p1.draw();
      p2.draw();

      if (p1.hp <= 0 || p2.hp <= 0) {
        ctx.fillStyle = "white";
        ctx.font = "40px Arial";
        ctx.fillText(p1.hp <= 0 ? "Player 2 Wins!" : "Player 1 Wins!", 320, 200);
        return;
      }

      requestAnimationFrame(update);
    }

    update();
  </script>
</body>
</html>
