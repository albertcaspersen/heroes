<script setup>
import { ref, watch, nextTick, onMounted, onUnmounted } from 'vue';
import gsap from 'gsap';

const isMenuOpen = ref(false);
const menuRef = ref(null);
const circleOverlayRef = ref(null);
const menuLinksRef = ref([]); // Til at gemme referencer til dine links

// --- GSAP Tidslinje ---
let timeline = null;

const toggleMenu = () => {
  isMenuOpen.value = !isMenuOpen.value;
};

onMounted(() => {
  timeline = gsap.timeline({
    paused: true,
    onReverseComplete: () => {
      // Skjuler overlayet og menuen fuldstændigt, når animationen er færdig med at køre baglæns
      gsap.set(circleOverlayRef.value, { display: 'none' });
      gsap.set(menuRef.value, { autoAlpha: 0, pointerEvents: 'none', display: 'none' });
      // Nulstil linkenes position og opacitet, så de er klar til næste animation
      gsap.set(menuLinksRef.value, { y: '100%', autoAlpha: 0 });
    }
  });

  timeline
    .to(circleOverlayRef.value, {
      clipPath: 'circle(150% at top right)', // Udvider cirklen
      duration: 0.6,
      ease: 'power3.inOut',
      onStart: () => {
        gsap.set(circleOverlayRef.value, { display: 'block' }); // Vis overlayet før animation starter
      }
    })
    .fromTo(menuRef.value,
      {
        y: '10%',           // Starter lidt længere nede
        autoAlpha: 0,       // Starter helt gennemsigtig
        pointerEvents: 'none', // Ikke interagerbar i starten
        display: 'none',    // Skjult som standard
      },
      {
        y: '0%',            // Flytter til normal position
        autoAlpha: 1,       // Fader helt ind
        duration: 0.5,      // Varighed for menu-animation
        ease: 'power3.out',
        pointerEvents: 'auto', // Gør menuen interagerbar
        display: 'flex',    // Vis menuen (sætter 'flex' da det er dens CSS display type)
      },
      "-=0.2" // Starter menu-animationen lidt før cirkel-animationen er helt færdig
    )
    // Tilføj link-animationerne her
    .from(menuLinksRef.value, {
      y: '100%', // Starter 100% nede (uden for syne)
      autoAlpha: 0, // Starter helt gennemsigtig
      duration: 0.6,
      ease: 'power3.out',
      stagger: 0.08, // Forsinkelse mellem hvert link
    }, "-=0.4"); // Start link-animationerne lidt før menu-animationen er helt færdig
});

onUnmounted(() => {
  if (timeline) {
    timeline.kill();
  }
});

watch(isMenuOpen, (newValue) => {
  if (timeline) {
    if (newValue) {
      nextTick(() => { // Sørg for at DOM er opdateret og links er tilgængelige
        timeline.play();
      });
    } else {
      timeline.reverse();
    }
  }
});
</script>

<template>
  <div class="hero">
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

        <!-- Cirkel overlay element -->
        <div class="circle-overlay" ref="circleOverlayRef"></div>

        <ul class="nav-menu" ref="menuRef">
          <!-- Tilføj ref til hvert <li> element -->
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
        <source src="/video/cph.mp4" type="video/mp4" />
        Din browser understøtter ikke video-tagget.
      </video>
    </div>

    <h1 class="overskrift">RENTHERO</h1>
    <h2 style="font-size: 1.2rem; position: absolute; bottom: 4.7rem">EXPERTS IN RENT CONTROL</h2>
    <button class="knap" style="width: 22.5rem; height: 2.5rem; position: absolute; bottom: 2.4rem; background-color: #F7DA09; color: black; border: none; border-radius: 5px;  ">FIND OUT IF YOU'RE OVERPAYING</button>
    <img src="/public/pics/black friday.png" alt="#" class="pic">
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

/* HEADER */
.header {
  position: absolute;
  top: 0.3rem;
  right: 0rem;
  width: 90%;
  max-width: 400px;
  padding: 0.5rem 0.1rem;
  display: flex;
  justify-content: flex-end;
  align-items: center;
  z-index: 10; /* Header er under menu og overlay */
  border-radius: 10px;
}

.nav {
  width: 100%;
  display: flex;
  justify-content: flex-end;
  position: relative;
}

/* Styling for cirkel-overlay */
.circle-overlay {
  position: absolute;
  top: -2rem;
  right: -2rem;
  margin: 0;
  width: 120%;
  height: 110vh;
  background-color: #F7DA09;
  border-radius: 0;
  z-index: 20; /* Overlay er over header, men under menu */
  clip-path: circle(0% at top right);
  display: none; /* Skjult som standard, GSAP viser den */
}


.nav-menu {
  list-style: none;
  margin: 0;
  padding: 0;
  flex-direction: column;
  position: fixed;
  top: 0;
  right: 0;
  width: 100%;
  height: 100vh;
  overflow-y: auto;
  z-index: 30; /* Menu er øverst, over overlayet */
  display: none; /* Skjult som standard i CSS */
  justify-content: center;
  align-items: left;

  /* Initial tilstand for GSAP håndteres i JS - disse er kun for at matche */
  opacity: 0;
  transform: translateY(10%);
  pointer-events: none;
  left: 1rem;
  
}

.nav li {
  width: 100%;
  text-align: left;
  padding: 1rem 0;
  border-bottom: solid rgba(255, 255, 255, 0.1) 1px;
  
  
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
  z-index: 35; /* Sørg for at hamburgeren altid er øverst, så den kan klikkes */
}

.hamburger-inner,
.hamburger-inner::before,
.hamburger-inner::after {
  background-color: #ffffff !important;
}

.hamburger.is-active .hamburger-inner,
.hamburger.is-active .hamburger-inner::before,
.hamburger.is-active .hamburger-inner::after {
  background-color: #ffffff !important;
}


/* HERO */
.hero {
  position: relative;
  height: 100vh;
  overflow: hidden;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.vidbox {
  position: relative;
  border-radius: 0.9rem;
  width: 22.5rem;
  height: 39rem;
  top: -1rem;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
}

.bg-video {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.pic {
  position: absolute;
  height: 10rem;
  top: 2.8rem;
  left: -1rem;
  transform: rotate(-10deg);
}

.overskrift {
  position: absolute;
  margin-top: 0rem;
  text-align: center;
  font-size: 1.7rem;
  font-weight: 700;
  color: white;
  top: 1.5rem;
}
</style>