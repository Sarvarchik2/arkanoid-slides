<template>
  <div class="mini-ark">
    <canvas
      ref="cv"
      :width="W"
      :height="H"
      @click="toggle"
    />
    <div class="hud">
      Score: {{ score }} ·
      <span v-if="state === 'run'">Click to pause · mouse moves the paddle</span>
      <span v-else-if="state === 'win'">You won! Click to play again</span>
      <span v-else-if="state === 'lose'">Ball lost. Click to play again</span>
      <span v-else>Click to start</span>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'

const W = 480
const H = 300
const cv = ref(null)
const score = ref(0)
const state = ref('idle') // idle | run | pause | win | lose

let ctx = null
let raf = null
const paddle = { w: 80, h: 10, x: (W - 80) / 2 }
const ball = { x: W / 2, y: 200, vx: 3, vy: -3, r: 6 }
let bricks = []
let trail = []
let particles = []

function spawnBurst(x, y, color) {
  for (let i = 0; i < 8; i += 1) {
    const angle = Math.random() * Math.PI * 2
    const speed = 1 + Math.random() * 2.5
    particles.push({
      x,
      y,
      vx: Math.cos(angle) * speed,
      vy: Math.sin(angle) * speed - 1,
      life: 18 + Math.random() * 10,
      color,
    })
  }
}

function resetBricks() {
  bricks = []
  const cols = 8
  const rows = 4
  const bw = W / cols
  const bh = 18
  const colors = ['#ff5252', '#ffb142', '#2ed573', '#1e90ff']
  for (let r = 0; r < rows; r += 1) {
    for (let c = 0; c < cols; c += 1) {
      bricks.push({ x: c * bw + 2, y: 30 + r * bh, w: bw - 4, h: bh - 3, color: colors[r] })
    }
  }
}

function reset() {
  score.value = 0
  ball.x = W / 2
  ball.y = 200
  ball.vx = 3
  ball.vy = -3
  trail = []
  particles = []
  resetBricks()
}

function step() {
  trail.push({ x: ball.x, y: ball.y })
  if (trail.length > 7) trail.shift()

  for (let i = particles.length - 1; i >= 0; i -= 1) {
    const p = particles[i]
    p.x += p.vx
    p.y += p.vy
    p.vy += 0.12
    p.life -= 1
    if (p.life <= 0) particles.splice(i, 1)
  }

  ball.x += ball.vx
  ball.y += ball.vy

  if (ball.x < ball.r) {
    ball.x = ball.r
    ball.vx = Math.abs(ball.vx)
  } else if (ball.x > W - ball.r) {
    ball.x = W - ball.r
    ball.vx = -Math.abs(ball.vx)
  }
  if (ball.y < ball.r) {
    ball.y = ball.r
    ball.vy = Math.abs(ball.vy)
  }

  const py = H - 24
  if (
    ball.vy > 0 &&
    ball.y + ball.r >= py &&
    ball.y < py + paddle.h &&
    ball.x > paddle.x - ball.r &&
    ball.x < paddle.x + paddle.w + ball.r
  ) {
    ball.vy = -Math.abs(ball.vy)
    ball.vx = ((ball.x - (paddle.x + paddle.w / 2)) / (paddle.w / 2)) * 4
    if (Math.abs(ball.vx) < 0.8) ball.vx = ball.vx >= 0 ? 0.8 : -0.8
  }

  if (ball.y > H + 30) state.value = 'lose'

  for (let i = bricks.length - 1; i >= 0; i -= 1) {
    const b = bricks[i]
    if (
      ball.x > b.x - ball.r &&
      ball.x < b.x + b.w + ball.r &&
      ball.y > b.y - ball.r &&
      ball.y < b.y + b.h + ball.r
    ) {
      spawnBurst(b.x + b.w / 2, b.y + b.h / 2, b.color)
      bricks.splice(i, 1)
      ball.vy *= -1
      score.value += 10
      break
    }
  }

  if (!bricks.length) state.value = 'win'
}

function draw() {
  ctx.fillStyle = '#0b1120'
  ctx.fillRect(0, 0, W, H)

  // Ball trail (fading, like the real game's TRAIL_LENGTH feature)
  for (let i = 0; i < trail.length; i += 1) {
    const fade = (i + 1) / (trail.length + 1)
    ctx.fillStyle = `rgba(255, 255, 255, ${0.28 * fade})`
    ctx.beginPath()
    ctx.arc(trail[i].x, trail[i].y, ball.r * (0.5 + fade * 0.5), 0, Math.PI * 2)
    ctx.fill()
  }

  // Neon bricks
  ctx.save()
  ctx.shadowBlur = 8
  for (const b of bricks) {
    ctx.shadowColor = b.color
    ctx.fillStyle = b.color
    ctx.fillRect(b.x, b.y, b.w, b.h)
  }
  ctx.restore()

  // Particle bursts (like the real game's particles.py)
  for (const p of particles) {
    ctx.globalAlpha = Math.max(p.life / 24, 0)
    ctx.fillStyle = p.color
    ctx.fillRect(p.x - 2, p.y - 2, 4, 4)
  }
  ctx.globalAlpha = 1

  ctx.fillStyle = '#00e5ff'
  if (ctx.roundRect) {
    ctx.beginPath()
    ctx.roundRect(paddle.x, H - 24, paddle.w, paddle.h, 5)
    ctx.fill()
  } else {
    ctx.fillRect(paddle.x, H - 24, paddle.w, paddle.h)
  }

  ctx.fillStyle = '#ffffff'
  ctx.beginPath()
  ctx.arc(ball.x, ball.y, ball.r, 0, Math.PI * 2)
  ctx.fill()

  if (state.value !== 'run') {
    ctx.fillStyle = 'rgba(0, 0, 0, 0.55)'
    ctx.fillRect(0, 0, W, H)
    ctx.fillStyle = '#00e5ff'
    ctx.font = '28px "VT323", monospace'
    ctx.textAlign = 'center'
    ctx.shadowColor = '#00e5ff'
    ctx.shadowBlur = 12
    const msg = {
      idle: 'CLICK TO START',
      pause: 'PAUSED',
      win: 'YOU WON!',
      lose: 'BALL LOST',
    }[state.value]
    ctx.fillText(msg, W / 2, H / 2)
    ctx.shadowBlur = 0
  }
}

function loop() {
  if (state.value === 'run') step()
  draw()
  raf = requestAnimationFrame(loop)
}

function toggle() {
  if (state.value === 'run') {
    state.value = 'pause'
  } else {
    if (state.value === 'win' || state.value === 'lose') reset()
    state.value = 'run'
  }
}

function onMove(e) {
  const rect = cv.value.getBoundingClientRect()
  const x = ((e.clientX - rect.left) / rect.width) * W - paddle.w / 2
  paddle.x = Math.min(Math.max(x, 0), W - paddle.w)
}

onMounted(() => {
  ctx = cv.value.getContext('2d')
  reset()
  window.addEventListener('mousemove', onMove)
  loop()
})

onBeforeUnmount(() => {
  if (raf) cancelAnimationFrame(raf)
  window.removeEventListener('mousemove', onMove)
})
</script>

<style scoped>
.mini-ark {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
}
.mini-ark canvas {
  border: 2px solid rgba(0, 229, 255, 0.5);
  border-radius: 8px;
  box-shadow: 0 0 18px rgba(0, 229, 255, 0.25);
  cursor: crosshair;
}
.hud {
  font-family: 'VT323', monospace;
  font-size: 1.15em;
  letter-spacing: 0.06em;
  color: rgba(0, 229, 255, 0.85);
}
</style>
