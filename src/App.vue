<script setup>
// Din <script setup> sektion er uændret og perfekt.
import { ref, watch, nextTick, onMounted, onUnmounted } from 'vue';
import gsap from 'gsap';

const isMenuOpen = ref(false);
const menuRef = ref(null);
const circleOverlayRef = ref(null);
const menuLinksRef = ref([]);

let timeline = null;

const toggleMenu = () => {
  isMenuOpen.value = !isMenuOpen.value;
};

onMounted(() => {
  timeline = gsap.timeline({
    paused: true,
    onReverseComplete: () => {
      gsap.set(circleOverlayRef.value, { display: 'none' });
      gsap.set(menuRef.value, { autoAlpha: 0, pointerEvents: 'none', display: 'none' });
      gsap.set(menuLinksRef.value, { y: '100%', autoAlpha: 0 });
    }
  });

  timeline
    .to(circleOverlayRef.value, {
      clipPath: 'circle(150% at top right)',
      duration: 0.6,
      ease: 'power3.inOut',
      onStart: () => {
        gsap.set(circleOverlayRef.value, { display: 'block' });
      }
    })
    .fromTo(menuRef.value,
      { y: '10%', autoAlpha: 0, pointerEvents: 'none', display: 'none' },
      { y: '0%', autoAlpha: 1, duration: 0.5, ease: 'power3.out', pointerEvents: 'auto', display: 'flex' },
      "-=0.2"
    )
    .from(menuLinksRef.value, {
      y: '100%',
      autoAlpha: 0,
      duration: 0.6,
      ease: 'power3.out',
      stagger: 0.08,
    }, "-=0.4");
});

onUnmounted(() => {
  if (timeline) {
    timeline.kill();
  }
});

watch(isMenuOpen, (newValue) => {
  if (timeline) {
    if (newValue) {
      nextTick(() => {
        timeline.play();
      });
    } else {
      timeline.reverse();
    }
  }
});
</script>

<template>
  <!-- Alle elementer er nu direkte børn af grid-container -->
  <div class="grid-container">
    <header class="header">
      <nav class="nav">
        <button
          class="hamburger hamburger--squeeze"
          :class="{ 'is-active': isMenuOpen }"
          type="button"
          @click="toggleMenu"
          aria-label="Toggle navigation"
        >
          <span class="hamburger-box">
            <span class="hamburger-inner"></span>
          </span>
        </button>
        <div class="circle-overlay" ref="circleOverlayRef"></div>
        <ul class="nav-menu" ref="menuRef">
          <li v-for="(item, index) in [
            { id: 'calculate', text: 'Calculate the legal rent' },
            { id: 'deposit', text: 'Deposit' },
            { id: 'rentmap', text: 'RentMap' },
            { id: 'ruling', text: 'Already got a ruling?' },
            { id: 'housingcourt', text: 'Get help in housing court' },
            { id: 'videos', text: 'Videos' }
          ]" :key="item.id" :ref="el => { if (el) menuLinksRef[index] = el }">
            <a :href="'#' + item.id" @click="toggleMenu">{{ item.text }}</a>
          </li>
        </ul>
      </nav>
    </header>

    <div class="vidbox">
      <video autoplay muted loop playsinline class="bg-video">
        <source src="/video/cph.mp4" class="video" type="video/mp4" />
        Din browser understøtter ikke video-tagget.
      </video>
    </div>

    <img src="/pics/black friday.png" alt="#" class="pic">
    <h1 class="overskrift">RENTHERO</h1>
    <h2 class="undertitel">EXPERTS IN RENT CONTROL</h2>
    <button class="knap cta-knap">FIND OUT IF YOU'RE OVERPAYING</button>
  </div>
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=PT+Sans:ital,wght@0,400;0,700;1,400;1,700&display=swap');
@import 'hamburgers/dist/hamburgers.min.css';

html, body, #app {
  margin: 0;
  min-height: 100vh;
  background-color: rgb(0, 0, 0);
  color: white;
  font-family: "PT Sans", sans-serif;
  overflow-x: hidden;
}

.grid-container {
  display: grid;
  grid-template-columns: 1fr repeat(4, minmax(0, 1fr)) 1fr; /* 6 kolonner, uændret */
  align-content: start; /* Fordeler rækkerne vertikalt */
  min-height: 100vh;
  margin: 0 1rem;
  text-align: center;
}

/* HEADER */
.header {
  grid-column: 6 / 6;
  grid-row: 1;
  position: relative;
  z-index: 40;
  display: flex;
  justify-content: center;
  width: 2.5rem;
  height: 3.5rem;
  padding-top: 0.5rem;
  left: 1.2rem;
}

.nav {
  position: relative;
  padding: 0rem;
}

/* VIDBOX - Nu som et grid item */
.vidbox {
  grid-column: 1 / 7;
  grid-row: 2 / 5;
  z-index: 1;
  border-radius: 0.9rem;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 35rem;
}

.bg-video {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

/* OVERSKRIFTER OG KNAPPER - Nu som grid items */
.overskrift {
  grid-column: 1 / 6;
  grid-row: 1;
  align-self: start;
  justify-self: start;
  z-index: 2;
  font-size: 1.7rem;
  font-weight: 700;
  color: white;
  margin-bottom: 2rem;
}
.undertitel {
  grid-column: 2 / 6;
  grid-row: 5;
  align-self: start;
  z-index: 2;
  font-size: 1.2rem;
  margin-top: 2rem;
}

.cta-knap {
  grid-column: 2 / 6;
  grid-row: 6;
  align-self: center;
  justify-self: center;
  z-index: 2;
  width: 22.5rem;
  height: 2.5rem;
  background-color: #F7DA09;
  color: black;
  border: none;
  border-radius: 5px;
  margin-top: 1rem;
}

.pic {
  grid-column: 1 / 3;
  grid-row: 2;
  z-index: 2;
  left: -3rem;
  top: -2rem;
  position: relative;
  height: 10rem;
  transform: rotate(-10deg);
}

.circle-overlay {
  position: fixed;
  top: -2rem;
  right: -2rem;
  margin: 0;
  width: 120%;
  height: 110vh;
  background-color: #F7DA09;
  z-index: 20;
  clip-path: circle(0% at top right);
  display: none;
}

/* OPDATERET NAV-MENU REGEL */
.nav-menu {
  list-style: none;
  margin: 0;
  padding: 0;
  flex-direction: column;
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100vh;
  overflow-y: auto;
  z-index: 30;
  display: none;
  justify-content: center;
  align-items: flex-start;  /* RETTET */
  padding-left: 1rem;      /* TILFØJET */
  box-sizing: border-box;  /* TILFØJET */
  opacity: 0;
  transform: translateY(10%);
  pointer-events: none;
}

.nav li {
  width: 100%;
  text-align: left;
  padding: 1rem 0;
}

.nav li:last-child {
  border-bottom: none;
}

.knap {
  font-family: "PT Sans", sans-serif;
  font-weight: 700;
  font-size: 0.9rem;
}

.nav a {
  text-decoration: none;
  font-weight: 700;
  color: #000000;
  transition: color 0.2s ease;
  display: block;
  font-size: 1.9rem;
  padding: 0.5rem 0;
}

.nav a:hover {
  color: #FFD700;
}

.hamburger {
  z-index: 35;
  position: relative;
}

.hamburger-inner, .hamburger-inner::before, .hamburger-inner::after {
  background-color: #ffffff !important;
}

.hamburger.is-active .hamburger-inner, .hamburger.is-active .hamburger-inner::before, .hamburger.is-active .hamburger-inner::after {
  background-color: #ffffff !important;
}
</style>