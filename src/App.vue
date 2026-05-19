<template>
  <div class="game-wrapper">
    <div class="game-header">
      <span class="title">⚔️ 按住移动 · 持续格斗 ⚔️</span>
      <span class="round-indicator"
        >🏆 第 {{ currentRound }} 局 · 比分 {{ score.p1 }} : {{ score.p2 }}</span
      >
      <div class="tips-mobile">
        <span>👇 按住方向键持续移动</span><span>🛡️ 精准格挡可反伤</span>
      </div>
    </div>

    <div class="arena-container">
      <div class="arena-ground"></div>
      <div class="ground-lines"></div>

      <!-- 特效层 -->
      <div class="particles-layer">
        <span
          v-for="p in particles"
          :key="p.id"
          class="particle"
          :style="{
            left: p.x + 'px',
            top: p.y + 'px',
            width: p.size + 'px',
            height: p.size + 'px',
            background: p.color,
          }"
        ></span>
      </div>
      <div class="damage-numbers-layer">
        <span
          v-for="d in damageNumbers"
          :key="d.id"
          class="damage-number"
          :class="{ crit: d.isCrit }"
          :style="{ left: d.x + 'px', top: d.y + 'px', color: d.color }"
          >{{ d.text }}</span
        >
      </div>
      <div class="special-text-layer">
        <span
          v-for="s in specialTexts"
          :key="s.id"
          class="special-text"
          :style="{ left: s.x + 'px', top: s.y + 'px', color: s.color }"
          >{{ s.text }}</span
        >
      </div>

      <!-- HUD 玩家1 -->
      <div class="player-hud p1-hud" :style="{ left: player1.x + 'px', bottom: '180px' }">
        <span class="hud-name">⚔️ 圣骑士</span>
        <div class="hp-bar-outer">
          <div
            class="hp-bar-inner"
            :style="{ width: (player1.hp / player1.maxHp) * 100 + '%' }"
          ></div>
        </div>
        <span class="hp-text">{{ Math.max(0, player1.hp) }}/{{ player1.maxHp }}</span>
        <div class="energy-bar-outer">
          <div
            class="energy-bar-inner"
            :style="{ width: (player1.energy / player1.maxEnergy) * 100 + '%' }"
          ></div>
        </div>
        <div class="skill-indicators">
          <div class="skill-icon" :class="{ ready: player1.skill1Cd <= 0 }">
            K
            <div
              class="cooldown-overlay"
              :style="{ height: (player1.skill1Cd / player1.skill1MaxCd) * 100 + '%' }"
            ></div>
            <span class="cd-text" v-if="player1.skill1Cd > 0">{{
              player1.skill1Cd.toFixed(1)
            }}</span>
          </div>
          <div class="skill-icon" :class="{ ready: player1.skill2Cd <= 0 }">
            L
            <div
              class="cooldown-overlay"
              :style="{ height: (player1.skill2Cd / player1.skill2MaxCd) * 100 + '%' }"
            ></div>
            <span class="cd-text" v-if="player1.skill2Cd > 0">{{
              player1.skill2Cd.toFixed(1)
            }}</span>
          </div>
          <div
            class="skill-icon ultimate"
            :class="{ ready: player1.skill3Cd <= 0 && player1.energy >= player1.skill3Cost }"
          >
            I
            <div
              class="cooldown-overlay"
              :style="{ height: (player1.skill3Cd / player1.skill3MaxCd) * 100 + '%' }"
            ></div>
            <span class="cd-text" v-if="player1.skill3Cd > 0">{{
              player1.skill3Cd.toFixed(1)
            }}</span>
          </div>
        </div>
      </div>

      <!-- HUD 玩家2 -->
      <div class="player-hud p2-hud" :style="{ left: player2.x + 'px', bottom: '180px' }">
        <span class="hud-name">🗡️ 暗影武士</span>
        <div class="hp-bar-outer">
          <div
            class="hp-bar-inner"
            :style="{ width: (player2.hp / player2.maxHp) * 100 + '%' }"
          ></div>
        </div>
        <span class="hp-text">{{ Math.max(0, player2.hp) }}/{{ player2.maxHp }}</span>
        <div class="energy-bar-outer">
          <div
            class="energy-bar-inner"
            :style="{ width: (player2.energy / player2.maxEnergy) * 100 + '%' }"
          ></div>
        </div>
        <div class="skill-indicators">
          <div class="skill-icon" :class="{ ready: player2.skill1Cd <= 0 }">
            2
            <div
              class="cooldown-overlay"
              :style="{ height: (player2.skill1Cd / player2.skill1MaxCd) * 100 + '%' }"
            ></div>
            <span class="cd-text" v-if="player2.skill1Cd > 0">{{
              player2.skill1Cd.toFixed(1)
            }}</span>
          </div>
          <div class="skill-icon" :class="{ ready: player2.skill2Cd <= 0 }">
            3
            <div
              class="cooldown-overlay"
              :style="{ height: (player2.skill2Cd / player2.skill2MaxCd) * 100 + '%' }"
            ></div>
            <span class="cd-text" v-if="player2.skill2Cd > 0">{{
              player2.skill2Cd.toFixed(1)
            }}</span>
          </div>
          <div
            class="skill-icon ultimate"
            :class="{ ready: player2.skill3Cd <= 0 && player2.energy >= player2.skill3Cost }"
          >
            4
            <div
              class="cooldown-overlay"
              :style="{ height: (player2.skill3Cd / player2.skill3MaxCd) * 100 + '%' }"
            ></div>
            <span class="cd-text" v-if="player2.skill3Cd > 0">{{
              player2.skill3Cd.toFixed(1)
            }}</span>
          </div>
        </div>
      </div>

      <!-- 角色实体 -->
      <div
        class="player-character p1"
        :class="{
          attacking: player1.isAttacking,
          blocking: player1.isBlocking,
          'perfect-block': player1.isPerfectBlock,
          shaking: player1.isShaking,
        }"
        :style="{ left: player1.x - 30 + 'px' }"
      >
        <div class="character-sprite">
          <div class="char-head"></div>
          <div class="char-body"></div>
          <div class="char-weapon"></div>
          <div class="char-legs">
            <div class="char-leg"></div>
            <div class="char-leg"></div>
          </div>
        </div>
      </div>
      <div
        class="player-character p2"
        :class="{
          attacking: player2.isAttacking,
          blocking: player2.isBlocking,
          'perfect-block': player2.isPerfectBlock,
          shaking: player2.isShaking,
        }"
        :style="{ left: player2.x - 30 + 'px' }"
      >
        <div class="character-sprite">
          <div class="char-head"></div>
          <div class="char-body"></div>
          <div class="char-weapon"></div>
          <div class="char-legs">
            <div class="char-leg"></div>
            <div class="char-leg"></div>
          </div>
        </div>
      </div>

      <!-- 游戏结束遮罩 -->
      <div class="game-over-overlay" v-if="gameOver" @click.stop>
        <div :class="'game-over-text ' + (winner === 1 ? 'winner-p1' : 'winner-p2')">
          {{ winner === 1 ? '🏆 圣骑士胜利！' : '🏆 暗影武士胜利！' }}
        </div>
        <div style="color: #ccc">总比分 {{ score.p1 }} : {{ score.p2 }}</div>
        <button class="restart-btn" @click="restartGame">⚡ 再来一局</button>
      </div>
    </div>

    <!-- 触屏控制区：按住方向键持续移动 -->
    <div class="touch-controls">
      <!-- 玩家1 左侧控制区 -->
      <div class="control-panel">
        <div class="panel-title">🟦 圣骑士 (P1)</div>
        <div class="move-row">
          <button
            class="ctrl-btn"
            @touchstart.prevent="startMove('p1', 'left')"
            @touchend.prevent="stopMove('p1', 'left')"
            @touchcancel="stopMove('p1', 'left')"
            @mousedown.prevent="startMove('p1', 'left')"
            @mouseup="stopMove('p1', 'left')"
            @mouseleave="stopMove('p1', 'left')"
          >
            ◀ 左移
          </button>
          <button
            class="ctrl-btn"
            @touchstart.prevent="startMove('p1', 'right')"
            @touchend.prevent="stopMove('p1', 'right')"
            @touchcancel="stopMove('p1', 'right')"
            @mousedown.prevent="startMove('p1', 'right')"
            @mouseup="stopMove('p1', 'right')"
            @mouseleave="stopMove('p1', 'right')"
          >
            右移 ▶
          </button>
          <button
            class="ctrl-btn block-btn"
            @touchstart.prevent="startBlock('p1')"
            @touchend.prevent="stopBlock('p1')"
            @mouseup="stopBlock('p1')"
            @mouseleave="stopBlock('p1')"
          >
            🛡️ 格挡
          </button>
        </div>
        <div class="skill-row">
          <button
            class="ctrl-btn attack-btn"
            @click="doAttack('p1', 0)"
            @touchstart.prevent="doAttack('p1', 0)"
          >
            ⚔️ 普攻
          </button>
          <button
            class="ctrl-btn"
            @click="doAttack('p1', 1)"
            @touchstart.prevent="doAttack('p1', 1)"
          >
            ✨ 技能K
          </button>
          <button
            class="ctrl-btn"
            @click="doAttack('p1', 2)"
            @touchstart.prevent="doAttack('p1', 2)"
          >
            💢 技能L
          </button>
          <button
            class="ctrl-btn skill-special"
            @click="doAttack('p1', 3)"
            @touchstart.prevent="doAttack('p1', 3)"
          >
            🌪️ 必杀I
          </button>
        </div>
      </div>
      <!-- 玩家2 右侧控制区 -->
      <div class="control-panel">
        <div class="panel-title">🟥 暗影武士 (P2)</div>
        <div class="move-row">
          <button
            class="ctrl-btn"
            @touchstart.prevent="startMove('p2', 'left')"
            @touchend.prevent="stopMove('p2', 'left')"
            @touchcancel="stopMove('p2', 'left')"
            @mousedown.prevent="startMove('p2', 'left')"
            @mouseup="stopMove('p2', 'left')"
            @mouseleave="stopMove('p2', 'left')"
          >
            ◀ 左移
          </button>
          <button
            class="ctrl-btn"
            @touchstart.prevent="startMove('p2', 'right')"
            @touchend.prevent="stopMove('p2', 'right')"
            @touchcancel="stopMove('p2', 'right')"
            @mousedown.prevent="startMove('p2', 'right')"
            @mouseup="stopMove('p2', 'right')"
            @mouseleave="stopMove('p2', 'right')"
          >
            右移 ▶
          </button>
          <button
            class="ctrl-btn block-btn"
            @touchstart.prevent="startBlock('p2')"
            @touchend.prevent="stopBlock('p2')"
            @mouseup="stopBlock('p2')"
            @mouseleave="stopBlock('p2')"
          >
            🛡️ 格挡
          </button>
        </div>
        <div class="skill-row">
          <button
            class="ctrl-btn attack-btn"
            @click="doAttack('p2', 0)"
            @touchstart.prevent="doAttack('p2', 0)"
          >
            ⚔️ 普攻
          </button>
          <button
            class="ctrl-btn"
            @click="doAttack('p2', 1)"
            @touchstart.prevent="doAttack('p2', 1)"
          >
            🌀 疾风2
          </button>
          <button
            class="ctrl-btn"
            @click="doAttack('p2', 2)"
            @touchstart.prevent="doAttack('p2', 2)"
          >
            💥 裂空3
          </button>
          <button
            class="ctrl-btn skill-special"
            @click="doAttack('p2', 3)"
            @touchstart.prevent="doAttack('p2', 3)"
          >
            🔥 暗影4
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted, onUnmounted } from 'vue'

// ========== 平衡参数 & 场景边界 ==========
const ARENA_LEFT = 45
const ARENA_RIGHT = 835
const MIN_DIST = 70
const MOVE_SPEED = 280 // 像素/秒，按住持续移动的速度

// 攻击范围
const RANGES = { normal: 88, skill1: 110, skill2: 135, skill3: 240 }
const CRIT_CHANCE = 0.16
const CRIT_MULTI = 1.6
const DMG_NORMAL = [8, 14]
const DMG_SKILL1 = [16, 26]
const DMG_SKILL2 = [20, 34]
const DMG_SKILL3 = [28, 44]

// 响应式数据
const gameOver = ref(false)
const winner = ref(0)
const currentRound = ref(1)
const score = reactive({ p1: 0, p2: 0 })
const particles = ref([])
const damageNumbers = ref([])
const specialTexts = ref([])
let pid = 0,
  did = 0,
  sid = 0

// 移动标志 (按住持续移动)
const moveFlags = reactive({
  p1: { left: false, right: false },
  p2: { left: false, right: false },
})

const createPlayer = (side) => ({
  x: side === 'p1' ? 180 : 720,
  hp: 100,
  maxHp: 100,
  energy: 0,
  maxEnergy: 100,
  isAttacking: false,
  isBlocking: false,
  isPerfectBlock: false,
  isShaking: false,
  comboCount: 0,
  lastHitTime: 0,
  blockStartTime: 0,
  normalCd: 0,
  normalMaxCd: 0.45,
  skill1Cd: 0,
  skill1MaxCd: 3.2,
  skill1Cost: 18,
  skill2Cd: 0,
  skill2MaxCd: 5.5,
  skill2Cost: 30,
  skill3Cd: 0,
  skill3MaxCd: 11,
  skill3Cost: 55,
  blockHeld: false,
  stunnedUntil: 0,
})

const player1 = reactive(createPlayer('p1'))
const player2 = reactive(createPlayer('p2'))
let gameActive = true
let lastTimestamp = 0
let animFrame = null

const clamp = (v, min, max) => Math.min(max, Math.max(min, v))
const distance = () => Math.abs(player1.x - player2.x)
const isInRange = (att, def, range) => Math.abs(att.x - def.x) <= range

// 特效函数
function spawnParticles(x, y, count, color) {
  for (let i = 0; i < count; i++) {
    const ang = Math.random() * Math.PI * 2
    const spd = 30 + Math.random() * 100
    particles.value.push({
      id: ++pid,
      x,
      y,
      size: 3 + Math.random() * 7,
      color,
      dx: Math.cos(ang) * spd,
      dy: Math.sin(ang) * spd - 30,
    })
    if (particles.value.length > 60) particles.value.splice(0, 20)
  }
}

function spawnDamageNumber(x, y, amt, isCrit = false, isHeal = false) {
  damageNumbers.value.push({
    id: ++did,
    x,
    y: y - 20,
    text: isHeal ? `+${amt}` : isCrit ? `${amt} 暴击!` : `${amt}`,
    color: isHeal ? '#6eff6e' : isCrit ? '#ffd966' : '#fff',
    isCrit,
  })
  if (damageNumbers.value.length > 20) damageNumbers.value.shift()
}

function spawnSpecialText(x, y, text, color) {
  specialTexts.value.push({ id: ++sid, x, y, text, color })
  if (specialTexts.value.length > 12) specialTexts.value.shift()
}

// 伤害处理
function dealDamage(att, def, baseDmg, skillLvl = 0) {
  const nowPerf = performance.now() / 1000
  if (def.stunnedUntil > nowPerf) return false
  let range = RANGES.normal
  if (skillLvl === 1) range = RANGES.skill1
  if (skillLvl === 2) range = RANGES.skill2
  if (skillLvl === 3) range = RANGES.skill3
  if (!isInRange(att, def, range)) {
    spawnParticles(att.x, 100, 4, '#aaa')
    return false
  }

  let finalDmg = baseDmg
  const isPerfect = def.isBlocking && nowPerf - def.blockStartTime < 0.28
  if (isPerfect) {
    def.isPerfectBlock = true
    setTimeout(() => {
      if (def.isPerfectBlock) def.isPerfectBlock = false
    }, 400)
    let reflect = Math.floor(baseDmg * 0.25)
    att.hp = Math.max(0, att.hp - reflect)
    spawnDamageNumber(att.x, 80, reflect, false)
    spawnSpecialText(def.x, 70, '完美格挡!', '#ffffaa')
    spawnParticles(def.x, 100, 20, 'gold')
    att.comboCount = 0
    return false
  }
  let blocked = def.isBlocking && !isPerfect
  if (blocked) {
    finalDmg = Math.floor(baseDmg * 0.4)
    spawnSpecialText(def.x, 55, '格挡', '#88ccff')
  }

  let isCrit = false
  if (!blocked && !isPerfect && Math.random() < CRIT_CHANCE) {
    isCrit = true
    finalDmg = Math.floor(finalDmg * CRIT_MULTI)
  }
  if (att.comboCount > 1 && !blocked && !isPerfect) {
    let bonus = Math.min(att.comboCount * 0.04, 0.2)
    finalDmg = Math.floor(finalDmg * (1 + bonus))
  }
  def.hp = Math.max(0, def.hp - finalDmg)
  def.isShaking = true
  setTimeout(() => {
    def.isShaking = false
  }, 250)
  let knock = skillLvl === 3 ? 24 : skillLvl === 2 ? 12 : 6
  if (att === player1) def.x = clamp(def.x + knock, ARENA_LEFT, ARENA_RIGHT)
  else def.x = clamp(def.x - knock, ARENA_LEFT, ARENA_RIGHT)
  enforceMinDist()
  def.energy = Math.min(def.maxEnergy, def.energy + Math.floor(finalDmg * 0.3))
  spawnParticles(def.x, 90 + Math.random() * 30, isCrit ? 14 : 8, isCrit ? '#ffcc44' : '#ff8866')
  spawnDamageNumber(def.x, 70, finalDmg, isCrit)
  if (isCrit) spawnSpecialText(def.x, 50, '💥暴击!', '#ffaa33')
  let now2 = performance.now() / 1000
  if (now2 - att.lastHitTime < 1.8) att.comboCount++
  else att.comboCount = 1
  att.lastHitTime = now2
  if (att.comboCount >= 4) spawnSpecialText(att.x, 40, `${att.comboCount}连击!`, '#ff9900')
  return true
}

// 强制保持最小距离
function enforceMinDist() {
  let d = distance()
  if (d < MIN_DIST) {
    let mid = (player1.x + player2.x) / 2
    player1.x = clamp(mid - MIN_DIST / 2, ARENA_LEFT, ARENA_RIGHT)
    player2.x = clamp(mid + MIN_DIST / 2, ARENA_LEFT, ARENA_RIGHT)
  }
}

// 攻击接口
function performAttack(att, def, skill) {
  let now = performance.now() / 1000
  if (att.stunnedUntil > now) return
  let dmg = 0,
    engGain = 0,
    cdKey = '',
    maxCd = 0,
    cost = 0
  if (skill === 0) {
    if (att.normalCd > 0) return
    dmg = DMG_NORMAL[0] + Math.floor(Math.random() * (DMG_NORMAL[1] - DMG_NORMAL[0] + 1))
    engGain = 12
    cdKey = 'normalCd'
    maxCd = att.normalMaxCd
    cost = 0
  }
  if (skill === 1) {
    if (att.skill1Cd > 0 || att.energy < att.skill1Cost) return
    dmg = DMG_SKILL1[0] + Math.floor(Math.random() * (DMG_SKILL1[1] - DMG_SKILL1[0] + 1))
    engGain = 7
    cdKey = 'skill1Cd'
    maxCd = att.skill1MaxCd
    cost = att.skill1Cost
  }
  if (skill === 2) {
    if (att.skill2Cd > 0 || att.energy < att.skill2Cost) return
    dmg = DMG_SKILL2[0] + Math.floor(Math.random() * (DMG_SKILL2[1] - DMG_SKILL2[0] + 1))
    engGain = 5
    cdKey = 'skill2Cd'
    maxCd = att.skill2MaxCd
    cost = att.skill2Cost
  }
  if (skill === 3) {
    if (att.skill3Cd > 0 || att.energy < att.skill3Cost) return
    dmg = DMG_SKILL3[0] + Math.floor(Math.random() * (DMG_SKILL3[1] - DMG_SKILL3[0] + 1))
    engGain = 0
    cdKey = 'skill3Cd'
    maxCd = att.skill3MaxCd
    cost = att.skill3Cost
  }
  att.energy -= cost
  att[cdKey] = maxCd
  att.isAttacking = true
  setTimeout(() => {
    att.isAttacking = false
  }, 280)
  let hit = dealDamage(att, def, dmg, skill)
  if (hit !== false) {
    att.energy = Math.min(att.maxEnergy, att.energy + engGain)
  }
  if (skill === 3 && hit === true) {
    def.stunnedUntil = performance.now() / 1000 + 0.65
    spawnSpecialText(def.x, 60, '⚡眩晕', '#ff7700')
  }
}

// 按住持续移动的控制方法
function startMove(side, direction) {
  if (side === 'p1') {
    if (direction === 'left') moveFlags.p1.left = true
    if (direction === 'right') moveFlags.p1.right = true
  } else {
    if (direction === 'left') moveFlags.p2.left = true
    if (direction === 'right') moveFlags.p2.right = true
  }
}

function stopMove(side, direction) {
  if (side === 'p1') {
    if (direction === 'left') moveFlags.p1.left = false
    if (direction === 'right') moveFlags.p1.right = false
  } else {
    if (direction === 'left') moveFlags.p2.left = false
    if (direction === 'right') moveFlags.p2.right = false
  }
}

// 格挡
function startBlock(side) {
  let p = side === 'p1' ? player1 : player2
  if (p.stunnedUntil <= performance.now() / 1000) {
    p.blockHeld = true
    p.blockStartTime = performance.now() / 1000
  }
}

function stopBlock(side) {
  let p = side === 'p1' ? player1 : player2
  p.blockHeld = false
  p.isBlocking = false
}

function doAttack(side, skillLv) {
  let att = side === 'p1' ? player1 : player2
  let def = side === 'p1' ? player2 : player1
  performAttack(att, def, skillLv)
}

// 更新移动（基于dt）
function updateMovement(dt) {
  if (moveFlags.p1.left) player1.x = clamp(player1.x - MOVE_SPEED * dt, ARENA_LEFT, ARENA_RIGHT)
  if (moveFlags.p1.right) player1.x = clamp(player1.x + MOVE_SPEED * dt, ARENA_LEFT, ARENA_RIGHT)
  if (moveFlags.p2.left) player2.x = clamp(player2.x - MOVE_SPEED * dt, ARENA_LEFT, ARENA_RIGHT)
  if (moveFlags.p2.right) player2.x = clamp(player2.x + MOVE_SPEED * dt, ARENA_LEFT, ARENA_RIGHT)
  enforceMinDist()
}

// 游戏主循环
function gameLoop(ts) {
  if (!gameActive && !gameOver.value) {
    animFrame = requestAnimationFrame(gameLoop)
    return
  }
  let dt = Math.min(0.033, (ts - lastTimestamp) / 1000)
  if (dt <= 0.01) dt = 0.016
  lastTimestamp = ts
  const nowSec = ts / 1000

  updateMovement(dt)

  function updateGeneric(p) {
    if (p.stunnedUntil > nowSec) {
      p.isBlocking = false
      p.blockHeld = false
      return
    }
    let wasBlock = p.blockHeld
    p.isBlocking = wasBlock && p.stunnedUntil <= nowSec
    if (!p.blockHeld) {
      p.isBlocking = false
      p.isPerfectBlock = false
    }
    if (p.normalCd > 0) p.normalCd = Math.max(0, p.normalCd - dt)
    if (p.skill1Cd > 0) p.skill1Cd = Math.max(0, p.skill1Cd - dt)
    if (p.skill2Cd > 0) p.skill2Cd = Math.max(0, p.skill2Cd - dt)
    if (p.skill3Cd > 0) p.skill3Cd = Math.max(0, p.skill3Cd - dt)
    p.energy = Math.min(p.maxEnergy, p.energy + 1.8 * dt)
    if (p.lastHitTime > 0 && nowSec - p.lastHitTime > 2.0) p.comboCount = 0
    if (p.stunnedUntil < nowSec) p.stunnedUntil = 0
  }
  updateGeneric(player1)
  updateGeneric(player2)
  enforceMinDist()

  if (player1.hp <= 0 || player2.hp <= 0) {
    if (!gameOver.value) {
      gameActive = false
      gameOver.value = true
      if (player1.hp <= 0 && player2.hp <= 0) winner.value = player1.hp > player2.hp ? 1 : 2
      else winner.value = player1.hp <= 0 ? 2 : 1
      if (winner.value === 1) score.p1++
      else score.p2++
    }
  }
  animFrame = requestAnimationFrame(gameLoop)
}

function restartGame() {
  if (score.p1 >= 2 || score.p2 >= 2) {
    score.p1 = 0
    score.p2 = 0
    currentRound.value = 1
  } else currentRound.value++
  player1.hp = 100
  player2.hp = 100
  player1.energy = 0
  player2.energy = 0
  player1.x = 180
  player2.x = 720
  player1.normalCd = player1.skill1Cd = player1.skill2Cd = player1.skill3Cd = 0
  player2.normalCd = player2.skill1Cd = player2.skill2Cd = player2.skill3Cd = 0
  player1.comboCount = player2.comboCount = 0
  player1.stunnedUntil = player2.stunnedUntil = 0
  player1.blockHeld = player2.blockHeld = false
  player1.isBlocking = player2.isBlocking = false
  moveFlags.p1.left = moveFlags.p1.right = false
  moveFlags.p2.left = moveFlags.p2.right = false
  gameOver.value = false
  gameActive = true
  winner.value = 0
  particles.value = []
  damageNumbers.value = []
  specialTexts.value = []
}

onMounted(() => {
  lastTimestamp = performance.now()
  animFrame = requestAnimationFrame(gameLoop)
})

onUnmounted(() => {
  if (animFrame) cancelAnimationFrame(animFrame)
})
</script>

<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  user-select: none;
  -webkit-tap-highlight-color: transparent;
}

.game-wrapper {
  width: 100%;
  max-width: 1000px;
  margin: 0 auto;
  background: #16213e;
  border-radius: 28px;
  overflow: hidden;
  box-shadow:
    0 0 50px rgba(0, 0, 0, 0.6),
    inset 0 0 30px rgba(0, 0, 0, 0.3);
  border: 2px solid #2a2a55;
}

.game-header {
  display: flex;
  justify-content: space-between;
  align-items: baseline;
  padding: 10px 16px;
  background: rgba(0, 0, 0, 0.6);
  backdrop-filter: blur(4px);
  border-bottom: 1px solid #ffd74055;
  flex-wrap: wrap;
  gap: 8px;
}

.title {
  font-size: 1.3rem;
  font-weight: 900;
  background: linear-gradient(135deg, #4da6ff, #ff8a5c);
  -webkit-background-clip: text;
  background-clip: text;
  color: transparent;
}

.round-indicator {
  color: #ffd740;
  font-weight: bold;
  font-size: 0.85rem;
  letter-spacing: 1px;
}
.tips-mobile {
  color: #aaa;
  font-size: 0.7rem;
  display: flex;
  gap: 12px;
}

.arena-container {
  position: relative;
  width: 100%;
  height: 460px;
  background: radial-gradient(ellipse at 50% 70%, #1e1e3a, #0f0f1f);
  overflow: hidden;
  touch-action: none;
}

.arena-ground {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  height: 70px;
  background: linear-gradient(180deg, #2f2f4a, #1a1a2c);
  border-top: 3px solid #4a4a70;
}

.ground-lines {
  position: absolute;
  bottom: 68px;
  left: 5%;
  right: 5%;
  height: 2px;
  background: repeating-linear-gradient(
    90deg,
    rgba(255, 215, 0, 0.3) 0px,
    rgba(255, 215, 0, 0.3) 40px,
    transparent 40px,
    transparent 100px
  );
}

.player-character {
  position: absolute;
  bottom: 65px;
  z-index: 5;
  transition: left 0.04s ease-out;
  pointer-events: none;
  will-change: left;
}

.character-sprite {
  position: relative;
  width: 60px;
  height: 90px;
  display: flex;
  flex-direction: column;
  align-items: center;
}
.char-head {
  width: 38px;
  height: 38px;
  border-radius: 50%;
  border: 3px solid rgba(255, 255, 240, 0.5);
  box-shadow: 0 4px 8px black;
}
.p1 .char-head {
  background: linear-gradient(145deg, #5fa9ff, #2f74c0);
}
.p2 .char-head {
  background: linear-gradient(145deg, #ff6b6b, #c03434);
}
.char-body {
  width: 46px;
  height: 38px;
  border-radius: 10px;
  margin-top: -6px;
  border: 2px solid rgba(255, 255, 200, 0.4);
}
.p1 .char-body {
  background: linear-gradient(145deg, #4a8ec7, #2d6290);
}
.p2 .char-body {
  background: linear-gradient(145deg, #d94c4c, #a52929);
}
.char-weapon {
  position: absolute;
  top: 30px;
  width: 32px;
  height: 7px;
  background: silver;
  border-radius: 3px;
  box-shadow: 0 2px 5px black;
}
.p1 .char-weapon {
  right: -28px;
  transform-origin: left;
}
.p2 .char-weapon {
  left: -28px;
  transform-origin: right;
}
.char-legs {
  display: flex;
  gap: 8px;
  margin-top: -4px;
}
.char-leg {
  width: 16px;
  height: 22px;
  border-radius: 6px;
  background: #334e68;
}
.p2 .char-leg {
  background: #8b2c2c;
}

.player-character.attacking .char-weapon {
  animation: swing 0.2s ease-out;
}
@keyframes swing {
  0% {
    transform: rotate(0deg);
  }
  50% {
    transform: rotate(35deg);
  }
  100% {
    transform: rotate(0deg);
  }
}
.player-character.shaking {
  animation: shake 0.2s ease-in-out;
}
@keyframes shake {
  0%,
  100% {
    transform: translateX(0);
  }
  25% {
    transform: translateX(-6px);
  }
  75% {
    transform: translateX(6px);
  }
}
.perfect-block {
  filter: drop-shadow(0 0 12px gold) brightness(1.5);
  animation: perfectFlash 0.4s;
}
@keyframes perfectFlash {
  0% {
    filter: brightness(1);
  }
  50% {
    filter: brightness(2) drop-shadow(0 0 20px white);
  }
}

.player-hud {
  position: absolute;
  z-index: 8;
  pointer-events: none;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 3px;
  width: 110px;
  transform: translateX(-50%);
}

.hud-name {
  font-weight: bold;
  font-size: 0.7rem;
  letter-spacing: 1px;
  text-shadow: 0 0 6px black;
}
.hp-bar-outer {
  width: 100px;
  height: 9px;
  background: #2a1e1e;
  border-radius: 8px;
  border: 1px solid #555;
  overflow: hidden;
}
.hp-bar-inner {
  height: 100%;
  width: 100%;
  background: linear-gradient(90deg, #4caf50, #81c784);
  transition: width 0.2s;
}
.p2-hud .hp-bar-inner {
  background: linear-gradient(90deg, #e34d4d, #ff7b7b);
}
.energy-bar-outer {
  width: 85px;
  height: 5px;
  background: #1f1f2e;
  border-radius: 4px;
  overflow: hidden;
}
.energy-bar-inner {
  background: linear-gradient(90deg, #ffb347, #ffd966);
  height: 100%;
  transition: width 0.2s;
}
.skill-indicators {
  display: flex;
  gap: 6px;
  margin-top: 2px;
}
.skill-icon {
  width: 28px;
  height: 28px;
  background: #26263b;
  border-radius: 8px;
  border: 2px solid #6c6c8a;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  font-size: 0.75rem;
  color: #ddd;
  position: relative;
  overflow: hidden;
}
.skill-icon.ready {
  border-color: #ffd966;
  box-shadow: 0 0 12px gold;
}
.cooldown-overlay {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  background: rgba(0, 0, 0, 0.7);
  transition: height 0.2s;
}
.cd-text {
  position: absolute;
  font-size: 0.7rem;
  font-weight: bold;
  color: white;
  z-index: 2;
}

.touch-controls {
  display: flex;
  justify-content: space-between;
  background: rgba(0, 0, 0, 0.85);
  backdrop-filter: blur(12px);
  padding: 12px 8px;
  gap: 12px;
  flex-wrap: wrap;
}

.control-panel {
  flex: 1;
  min-width: 230px;
  background: rgba(20, 20, 40, 0.7);
  border-radius: 28px;
  padding: 8px 10px;
  box-shadow: 0 5px 12px rgba(0, 0, 0, 0.4);
}

.panel-title {
  font-size: 0.7rem;
  text-align: center;
  margin-bottom: 8px;
  letter-spacing: 2px;
  color: #ffd966;
}

.move-row {
  display: flex;
  justify-content: center;
  gap: 12px;
  margin-bottom: 10px;
  flex-wrap: wrap;
}

.ctrl-btn {
  background: #2c2c44;
  border: none;
  color: white;
  font-weight: bold;
  font-size: 1rem;
  padding: 10px 16px;
  border-radius: 60px;
  box-shadow: 0 3px 0 #0a0a14;
  transition: 0.05s linear;
  touch-action: manipulation;
  cursor: pointer;
  text-align: center;
  min-width: 65px;
  font-family: monospace;
  letter-spacing: 1px;
}

.ctrl-btn:active {
  transform: translateY(2px);
  box-shadow: 0 1px 0 #0a0a14;
  background: #4a4a70;
}
.block-btn {
  background: #2c5f7a;
}
.attack-btn {
  background: #9e3c3c;
}
.skill-special {
  background: #8b5a2b;
  border-color: #ffaa33;
}

.particles-layer,
.damage-numbers-layer,
.special-text-layer {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 10;
}
.damage-number {
  position: absolute;
  font-weight: 900;
  font-size: 1.5rem;
  animation: floatUp 0.8s forwards;
  text-shadow: 0 0 8px black;
}
@keyframes floatUp {
  0% {
    opacity: 1;
    transform: translateY(0);
  }
  100% {
    opacity: 0;
    transform: translateY(-55px);
  }
}
.special-text {
  position: absolute;
  font-weight: bold;
  font-size: 1.2rem;
  animation: pop 0.9s forwards;
  text-shadow: 0 0 6px cyan;
}
@keyframes pop {
  0% {
    transform: scale(0.5);
    opacity: 1;
  }
  50% {
    transform: scale(1.4);
  }
  100% {
    opacity: 0;
    transform: scale(1.8);
  }
}
.particle {
  position: absolute;
  border-radius: 50%;
  pointer-events: none;
  animation: fadeOut 0.6s forwards;
}
@keyframes fadeOut {
  to {
    opacity: 0;
    transform: scale(0.5);
  }
}

.game-over-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.85);
  z-index: 30;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  gap: 20px;
}
.restart-btn {
  background: #ffd966;
  color: #1e1e2a;
  border: none;
  padding: 12px 30px;
  border-radius: 40px;
  font-weight: bold;
  font-size: 1.2rem;
  cursor: pointer;
}

@media (max-width: 680px) {
  .ctrl-btn {
    padding: 8px 12px;
    min-width: 55px;
    font-size: 0.85rem;
  }
  .skill-row .ctrl-btn {
    padding: 6px 10px;
    font-size: 0.8rem;
  }
  .skill-icon {
    width: 22px;
    height: 22px;
    font-size: 0.6rem;
  }
  .arena-container {
    height: 400px;
  }
}
</style>
