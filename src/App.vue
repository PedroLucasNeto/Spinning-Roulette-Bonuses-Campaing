<template>
  <div class="page">
    <img src="./assets/IMG_4795.JPG.jpeg" class="logo" alt="Alana Retratos" />
    <div class="card">
      <p class="subtitle">Gire a roleta para ganhar!</p>

      <div class="wheel-area">
        <div class="pointer"></div>

        <svg
          class="wheel"
          viewBox="0 0 300 300"
          :style="wheelStyle"
          width="360"
          height="360"
        >
          <g v-for="(item, i) in items" :key="i">
            <path
              :d="getSectorPath(i)"
              :fill="item.color"
              stroke="#ffffff"
              stroke-width="2"
            />
            <g :transform="getTextGroupTransform(i)">
              <text
                text-anchor="middle"
                font-size="9"
                font-weight="700"
                font-family="system-ui, sans-serif"
                :fill="item.textColor"
              >
                <tspan
                  v-for="(line, li) in item.lines"
                  :key="li"
                  x="0"
                  :dy="lineDy(li, item.lines.length)"
                >
                  {{ line }}
                </tspan>
              </text>
            </g>
          </g>
          <circle
            cx="150"
            cy="150"
            r="18"
            fill="#ffffff"
            stroke="#7d1b4c"
            stroke-width="4"
          />
        </svg>
      </div>

      <button class="spin-btn" @click="spin">
        {{
          spinning
            ? "Girando..."
            : alreadyPlayed
              ? "Girar novamente!"
              : "Girar a Roleta"
        }}
      </button>

      <transition name="fade">
        <div v-if="result && !spinning" class="result">
          <p class="win-text">
            Parabéns! Você ganhou: <strong>{{ result.label }}</strong>
          </p>
        </div>
      </transition>
    </div>
  </div>
</template>

<script setup lang="ts">
  import { ref, computed } from "vue";

  interface Item {
    label: string;
    lines: string[];
    weight: number;
    win: boolean;
    color: string;
    textColor: string;
  }

  const items: Item[] = [
    {
      label: "Vídeo do ensaio",
      lines: ["Vídeo do", "ensaio"],
      weight: 1,
      win: true,
      color: "#7d1b4c",
      textColor: "#ffffff",
    },
    {
      label: "Fotos reveladas 10x15",
      lines: ["Fotos", "reveladas", "10x15"],
      weight: 2,
      win: true,
      color: "#e7c989",
      textColor: "#7d1b4c",
    },
    {
      label: "5 fotos extras",
      lines: ["5 fotos", "extras"],
      weight: 2,
      win: true,
      color: "#7d1b4c",
      textColor: "#ffffff",
    },
    {
      label: "Fotos reveladas 15x21",
      lines: ["Fotos", "reveladas", "15x21"],
      weight: 1,
      win: true,
      color: "#e7c989",
      textColor: "#7d1b4c",
    },
    {
      label: "1 Foto extra",
      lines: ["1 Foto", "extra"],
      weight: 3,
      win: true,
      color: "#7d8285",
      textColor: "#ffffff",
    },
  ];

  const N = items.length;
  const ANGLE = 360 / N;
  const CX = 150;
  const CY = 150;
  const R = 138;

  const rotation = ref(0);
  const spinning = ref(false);
  const result = ref<Item | null>(null);
  const alreadyPlayed = ref(false);

  const wheelStyle = computed(() => ({
    transform: `rotate(${rotation.value}deg)`,
    transition: spinning.value
      ? "transform 4s cubic-bezier(0.17, 0.67, 0.12, 1)"
      : "none",
  }));

  function toRad(deg: number) {
    return ((deg - 90) * Math.PI) / 180;
  }

  function getSectorPath(i: number): string {
    const start = i * ANGLE;
    const end = (i + 1) * ANGLE;
    const sx = CX + R * Math.cos(toRad(start));
    const sy = CY + R * Math.sin(toRad(start));
    const ex = CX + R * Math.cos(toRad(end));
    const ey = CY + R * Math.sin(toRad(end));
    return `M ${CX} ${CY} L ${sx} ${sy} A ${R} ${R} 0 0 1 ${ex} ${ey} Z`;
  }

  function getTextGroupTransform(i: number): string {
    const mid = (i + 0.5) * ANGLE;
    const tr = R * 0.62;
    const tx = CX + tr * Math.cos(toRad(mid));
    const ty = CY + tr * Math.sin(toRad(mid));
    // Angle of the radius from horizontal (standard math convention)
    let rot = mid - 90;
    // Flip if text would be upside down (more than 90° from horizontal)
    if (rot > 90) rot -= 180;
    if (rot < -90) rot += 180;
    return `translate(${tx}, ${ty}) rotate(${rot})`;
  }

  function lineDy(li: number, total: number): string {
    const lh = 11;
    return li === 0 ? `${-((total - 1) / 2) * lh + 3}` : `${lh}`;
  }

  function weightedRandom(): Item {
    const pool = items.flatMap((item) => Array(item.weight).fill(item));
    return pool[Math.floor(Math.random() * pool.length)];
  }

  function spin() {
    // if (spinning.value || alreadyPlayed.value) return;

    spinning.value = true;
    result.value = null;

    const selected = weightedRandom();
    const index = items.indexOf(selected);

    const currentNormalized = rotation.value % 360;
    const stopAtDeg = 360 - (index + 0.5) * ANGLE;
    const delta = (stopAtDeg - currentNormalized + 360) % 360;

    rotation.value += 5 * 360 + delta;

    setTimeout(() => {
      spinning.value = false;
      result.value = selected;
      alreadyPlayed.value = true;
      localStorage.setItem("roleta-played", "true");
    }, 4000);
  }
</script>

<style scoped>
  .page {
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    gap: 40px;
    align-items: center;
    justify-content: center;
    background:
      radial-gradient(
        ellipse at top left,
        rgba(125, 27, 76, 0.12) 0%,
        transparent 55%
      ),
      radial-gradient(
        ellipse at bottom right,
        rgba(231, 201, 137, 0.25) 0%,
        transparent 55%
      ),
      radial-gradient(ellipse at center, #ffffff 0%, #fdf4f8 100%);
    padding: 20px;
    box-sizing: border-box;
  }

  .card {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 20px;
    width: 100%;
    max-width: 480px;
  }

  .logo {
    width: 400px;
    height: 100px;
    border-radius: 12px;
    object-fit: cover;
    object-position: center 50%;
    mix-blend-mode: darken;
    /* box-shadow: 0 4px 16px rgba(125, 27, 76, 0.2); */
  }

  .subtitle {
    color: #7d8285;
    margin: -12px 0 0;
    font-size: 1rem;
    font-family: system-ui, sans-serif;
  }

  .wheel-area {
    position: relative;
    width: 360px;
    height: 360px;
  }

  .pointer {
    position: absolute;
    top: -10px;
    left: 50%;
    transform: translateX(-50%);
    width: 0;
    height: 0;
    border-left: 12px solid transparent;
    border-right: 12px solid transparent;
    border-top: 24px solid #7d1b4c;
    z-index: 10;
    filter: drop-shadow(0 2px 4px rgba(125, 27, 76, 0.4));
  }

  .wheel {
    width: 360px;
    height: 360px;
    border-radius: 50%;
    box-shadow:
      0 8px 32px rgba(125, 27, 76, 0.2),
      0 0 0 4px #7d1b4c;
    display: block;
  }

  .spin-btn {
    padding: 14px 36px;
    background: #7d1b4c;
    color: #ffffff;
    border: none;
    border-radius: 50px;
    font-size: 1rem;
    font-weight: 600;
    font-family: system-ui, sans-serif;
    cursor: pointer;
    transition:
      background 0.2s,
      transform 0.1s;
    letter-spacing: 0.3px;
  }

  .spin-btn:hover:not(:disabled) {
    background: #9a2260;
    transform: translateY(-1px);
  }

  .spin-btn:disabled {
    opacity: 0.55;
    cursor: not-allowed;
  }

  .result {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 14px;
    padding: 20px 24px;
    background: #ffffff;
    border: 2px solid #7d1b4c;
    border-radius: 16px;
    width: 100%;
    box-sizing: border-box;
    text-align: center;
  }

  .win-text {
    color: #7d1b4c;
    font-size: 1.05rem;
    margin: 0;
    font-family: system-ui, sans-serif;
  }

  .fade-enter-active,
  .fade-leave-active {
    transition:
      opacity 0.4s ease,
      transform 0.4s ease;
  }

  .fade-enter-from,
  .fade-leave-to {
    opacity: 0;
    transform: translateY(10px);
  }
</style>
