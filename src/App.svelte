<script>
  import { onDestroy, tick } from 'svelte';

  // State
  let isFlipped = $state(false);
  let step = $state('part1'); // 'part1' | 'password' | 'part2'
  let password = $state('');
  let showError = $state(false);
  let isLightTheme = $state(false);

  // Background decoration states
  /** @type {Array<{ id: number; emoji: string; left: number; top: number; size: number }>} */
  let flowers = $state([]);

  /** @type {Array<{ id: number; emoji: string; left: number; size: number; duration: number }>} */
  let hearts = $state([]);

  /** @type {HTMLDivElement | undefined} */
  let msgContainer = $state();

  /** @type {HTMLInputElement | undefined} */
  let pwdInput = $state();

  /** @type {ReturnType<typeof setInterval> | null} */
  let heartInterval = null;
  let nextHeartId = 0;

  // Sync light-theme to body tag
  $effect(() => {
    if (typeof document !== 'undefined') {
      if (isLightTheme) {
        document.body.classList.add('light-theme');
      } else {
        document.body.classList.remove('light-theme');
      }
    }
  });

  function openCard() {
    isFlipped = true;
  }

  /** @param {MouseEvent} e */
  function flipBack(e) {
    e.stopPropagation();
    isFlipped = false;
  }

  async function nextStep() {
    step = 'password';
    await tick();
    if (pwdInput) {
      pwdInput.focus();
    }
  }

  function triggerRomanceEffects() {
    const flowerEmojis = ['🌸', '🌺', '🌻', '🌷', '💮'];
    const heartEmojis = ['💖', '💗', '💕', '🫶', '✨'];

    // 1. Bloom 22 flowers randomly across the screen
    for (let i = 0; i < 22; i++) {
      const emoji = flowerEmojis[Math.floor(Math.random() * flowerEmojis.length)];
      const left = Math.random() * 90;
      const top = Math.random() * 90;
      const size = Math.random() * 3 + 2; // 2rem to 5rem
      const delay = Math.random() * 1500;

      setTimeout(() => {
        flowers = [
          ...flowers,
          {
            id: i,
            emoji,
            left,
            top,
            size,
          }
        ];
      }, delay);
    }

    // 2. Continuously spawn floating hearts
    heartInterval = setInterval(() => {
      const id = nextHeartId++;
      const emoji = heartEmojis[Math.floor(Math.random() * heartEmojis.length)];
      const left = Math.random() * 100;
      const size = Math.random() * 1.5 + 1; // 1rem to 2.5rem
      const duration = Math.random() * 4 + 5; // 5s to 9s

      hearts = [
        ...hearts,
        {
          id,
          emoji,
          left,
          size,
          duration,
        }
      ];

      // Clean up heart after it finishes floating
      setTimeout(() => {
        hearts = hearts.filter((h) => h.id !== id);
      }, (duration + 0.5) * 1000);
    }, 500);
  }

  async function checkPassword() {
    const userInput = password.trim().toLowerCase();
    if (userInput === 'hanabi') {
      step = 'part2';
      showError = false;
      isLightTheme = true;

      triggerRomanceEffects();

      await tick();
      if (msgContainer) {
        msgContainer.scrollTop = 0;
      }
    } else {
      showError = true;
      password = '';
      if (pwdInput) {
        pwdInput.focus();
      }
    }
  }

  /** @param {KeyboardEvent} e */
  function handleKeydown(e) {
    if (e.key === 'Enter') {
      checkPassword();
    }
  }

  onDestroy(() => {
    if (heartInterval) {
      clearInterval(heartInterval);
    }
  });
</script>

<svelte:head>
  <title>A Note For You</title>
</svelte:head>

<!-- Background Decorations Layer -->
<div class="decorations-container" id="decorations" aria-hidden="true">
  {#each flowers as flower (flower.id)}
    <div
      class="flower"
      style="left: {flower.left}vw; top: {flower.top}vh; font-size: {flower.size}rem;"
    >
      {flower.emoji}
    </div>
  {/each}

  {#each hearts as heart (heart.id)}
    <div
      class="heart"
      style="left: {heart.left}vw; font-size: {heart.size}rem; animation-duration: {heart.duration}s;"
    >
      {heart.emoji}
    </div>
  {/each}
</div>

<!-- 3D Scene Setup -->
<main class="scene">
  <div
    class="card"
    id="confessionCard"
    class:is-flipped={isFlipped}
  >
    <!-- FRONT COVER -->
    <div
      class="card-face card-front"
      class:light-theme={isLightTheme}
      id="frontCover"
      role="button"
      tabindex="0"
      onclick={openCard}
      onkeydown={(e) => (e.key === 'Enter' || e.key === ' ') && openCard()}
      title="Click to open letter"
    >
      <div class="envelope-icon">💌</div>
      <div class="front-title">A Little Note</div>
      <div class="front-subtitle">(Tap to open)</div>
    </div>

    <!-- BACK COVER -->
    <div
      class="card-face card-back"
      class:light-theme={isLightTheme}
      id="cardBack"
    >
      <!-- Optional flip back button -->
      <button
        class="flip-back-btn"
        onclick={flipBack}
        type="button"
        title="Close letter"
      >
        <span class="back-arrow">‹</span> Close note
      </button>

      <div
        class="message-container"
        class:light-theme={isLightTheme}
        id="msgContainer"
        bind:this={msgContainer}
      >
        <!-- PART 1 -->
        {#if step === 'part1'}
          <div id="part1" class="fade-in">
            <p>Hello po,</p>
            <p>
              First, patawad po kung ang cold ko or parang wala akong emotion pag naka duty or pag nasa labas tas feeling mo nai-ignore o iniiwasan kita minsan kada duty wala naman akong sama ng loob HAHAHA, tapos sorry din if mas sumasama or tumatabi pa ako kila Ivan HAHAHAHA. Hindi ko lang talaga alam paano kita ia-approach nang hindi nagmumukhang weird, wala ehh.
            </p>
            <p>
              Second, patawad din po kung medyo nangungulit o nakukulitan ka na sa mga chat ko tuwing madaling araw para mag ml HAHAHA.
            </p>
            <p>
              Ang totoo niyan, nag-oobserve lang talaga ako sa'yo kapag magkasama tayo sa duty, nagfo-foodtrip, or kada magkikita tayo, and patawad po talaga if ang cold ko, di ko lang talaga alam gagawin ko kaya ganun. Tas pansin ko lang din before pag nagkikita tayo sa duty or nakikita moko before tas ikaw yung nag iinitiate na mag greet, tas yung greet mo is ang sarap sa feeling dahil parang ang saya mo masyado mag greet compare recently na parang ang simple nalang HAHAHAHA.
            </p>
            <p>
              Tas patawad po if madalas nabo-bored ka o na-o-OP sa amin nila Ivan na parang feeling ko na na-feeling mo nagiging invisible ka although hindi ko din gusto na ganun nga yung ma feel mo if ever kasi may mga instances na ang tahimik mo lang din.
            </p>
            <button class="btn" id="nextBtn" type="button" onclick={nextStep}>
              May kasunod pa...
            </button>
          </div>
        {/if}

        <!-- PASSWORD SECTION -->
        {#if step === 'password'}
          <div id="passwordSection" class="input-group fade-in">
            <p>Ano ang favorite ml hero mo?</p>
            <input
              type="text"
              id="pwdInput"
              class="password-input"
              placeholder="Enter hero name..."
              bind:this={pwdInput}
              bind:value={password}
              onkeydown={handleKeydown}
              oninput={() => (showError = false)}
            />
            {#if showError}
              <span id="errorMsg" class="error-msg">Mali eh, try again!</span>
            {/if}
            <button class="btn" id="submitPwdBtn" type="button" onclick={checkPassword}>
              Unlock Message
            </button>
          </div>
        {/if}

        <!-- PART 2 -->
        {#if step === 'part2'}
          <div id="part2" class="fade-in">
            <p>
              So ayun na nga, Around August ko kasi napansin sa sarili ko na parang may nagbago. Naalala mo yung nag inuman tayo kila Rick tas nagtatanong ka sakin kinabukasan bat tingin ng tingin sayo si Ivan HAHAHA, sakanya talaga ako unang umamin ng nararamdaman ko sayo HAHAHA.
            </p>
            <p>
              Siguro hindi nyo lang halata dahil sa mga kilos ko magaling lang siguro ako magtago ng nararamdaman HAHAHA, pero unti-unti kong na-realize na nagkakagusto na pala ako. Unexpected talaga kasi, akala ko wala lang to or matatapos ako sa mcdo ng ganun nalang pero suddenly bigla nga kayo dumating nila Ivan, ikaw tas sila Drick tas Angel. Pero ayun nga habang tumatagal, napapansin ko sa sarili ko na sobrang saya ko deep inside kapag magkasama tayo and feeling ko hindi lang talaga halata sa mga kilos ko HAHAHAHA.
            </p>
            <p>
              Nag-observe din ako sa sarili ko akala ko parang gutom lang HAHAHAH, tas may mga instances pa na nakikipag swap ako ng duty or nanghihingi ako or kinikulit ko si mam grace before para makaduty kita HAHAHAHA, tas dapat magpapa mc talaga ako tas biglang hindi nalang kasi kaduty kita HAHAHAHA.
            </p>
            <p>
              So ayun na nga nag-aalangan talaga akong magsabi, kasi baka iwasan mo ako, mailang ka, o maging weird tayo sa isat isa kapag nasa duty o food trips. Wala naman akong expectations na suklian mo yung nararamdaman ko kung wala naman talaga, pero sana wala lang sanang mangyaring iwasan or ilangan sa pagitan natin after this, kasi deep inside talagang ang saya ko ngayon pag magkakasama tayo HAHAHAH, so yun ginawa ko lang to para maging totoo sa sarili ko at ma-express tong nararamdaman ko hirap din kasi kung itatago ko lang to.
            </p>
            <p>
              Wish ko lang talaga sana wag ka pa ring umiwas o mailang sa akin or mailang na kausapin ako. Kung hanggang tropa/workmates lang talaga edi yun na yun willing din naman akong tanggapin yun, sasarilinin ko na lang 'tong nararamdaman ko HAHAHAHA.
            </p>
            <p>
              Kasalanan talaga to ni kuya neil HAHAHAHAH.
            </p>
            <p>
              Thank you teh sa pagbabasa kahit mahaba HAHAHA. Ingat palagi at sana napangiti kita kahit paano! bye po sikrittt.
            </p>
          </div>
        {/if}
      </div>
    </div>
  </div>
</main>

<style>
  /* Background Decorations Layer */
  .decorations-container {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    pointer-events: none;
    z-index: 1; /* Behind the card */
    overflow: hidden;
  }

  /* 3D Scene Setup */
  .scene {
    width: 100%;
    max-width: 400px;
    height: 560px;
    perspective: 1100px;
    z-index: 10; /* Above decorations */
    position: relative;
  }

  /* Card Container */
  .card {
    width: 100%;
    height: 100%;
    position: relative;
    transition: transform 0.85s cubic-bezier(0.4, 0.2, 0.2, 1);
    transform-style: preserve-3d;
  }

  .card.is-flipped {
    transform: rotateY(180deg);
  }

  /* Front and Back Faces */
  .card-face {
    position: absolute;
    width: 100%;
    height: 100%;
    backface-visibility: hidden;
    -webkit-backface-visibility: hidden;
    border-radius: 26px;
    /* Dark Theme Glass */
    background: rgba(255, 255, 255, 0.04);
    backdrop-filter: blur(20px);
    -webkit-backdrop-filter: blur(20px);
    border: 1px solid rgba(255, 255, 255, 0.08);
    box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.55),
                0 0 0 1px rgba(255, 255, 255, 0.05) inset;
    display: flex;
    flex-direction: column;
    overflow: hidden;
    transition: all 1.5s ease;
  }

  /* Light Theme Glass */
  .card-face.light-theme {
    background: rgba(255, 255, 255, 0.45);
    backdrop-filter: blur(8px);
    -webkit-backdrop-filter: blur(8px);
    border: 1px solid rgba(255, 255, 255, 0.7);
    box-shadow: 0 20px 45px -10px rgba(0, 0, 0, 0.12),
                0 0 30px rgba(255, 255, 255, 0.5) inset;
  }

  /* Front Face */
  .card-front {
    justify-content: center;
    align-items: center;
    text-align: center;
    cursor: pointer;
    user-select: none;
    padding: 30px;
    transition: transform 0.3s ease, border-color 0.3s ease, box-shadow 0.3s ease;
  }

  .card-front:hover {
    border-color: rgba(255, 255, 255, 0.2);
    box-shadow: 0 30px 60px -15px rgba(0, 0, 0, 0.65), 0 0 25px rgba(255, 255, 255, 0.06);
  }

  .card-front:hover .envelope-icon {
    transform: scale(1.1) translateY(-6px);
  }

  .envelope-icon {
    font-size: 4.5rem;
    margin-bottom: 1.2rem;
    opacity: 0.9;
    filter: drop-shadow(0 4px 10px rgba(0, 0, 0, 0.3));
    transition: transform 0.4s cubic-bezier(0.34, 1.56, 0.64, 1);
  }

  .front-title {
    font-size: 1.65rem;
    font-weight: 600;
    letter-spacing: 0.5px;
    color: #f8fafc;
    transition: color 1.5s ease;
  }

  .card-front.light-theme .front-title {
    color: #0f172a;
  }

  .front-subtitle {
    font-size: 0.95rem;
    color: #94a3b8;
    margin-top: 10px;
    font-weight: 400;
    transition: color 1.5s ease;
  }

  .card-front.light-theme .front-subtitle {
    color: #64748b;
  }

  /* Back Face */
  .card-back {
    transform: rotateY(180deg);
    padding: 32px 24px 26px;
    display: flex;
    flex-direction: column;
    position: relative;
  }

  .flip-back-btn {
    position: absolute;
    top: 10px;
    left: 14px;
    background: transparent;
    border: none;
    color: #94a3b8;
    cursor: pointer;
    font-size: 0.82rem;
    display: inline-flex;
    align-items: center;
    gap: 4px;
    padding: 4px 10px;
    border-radius: 999px;
    transition: all 0.25s ease;
    z-index: 20;
    font-family: inherit;
  }

  .flip-back-btn .back-arrow {
    font-size: 1.1rem;
    line-height: 1;
  }

  .flip-back-btn:hover {
    color: #ffffff;
    background: rgba(255, 255, 255, 0.1);
  }

  .card-back.light-theme .flip-back-btn {
    color: #475569;
  }

  .card-back.light-theme .flip-back-btn:hover {
    color: #0f172a;
    background: rgba(0, 0, 0, 0.07);
  }

  .message-container {
    width: 100%;
    height: 100%;
    overflow-y: auto;
    padding-right: 10px;
    padding-top: 12px;
    font-size: 1.02rem;
    line-height: 1.75;
    color: #cbd5e1;
    text-align: justify;
    transition: all 1.5s ease;
  }

  /* Light Theme Text */
  .message-container.light-theme {
    color: #0f172a;
    font-weight: 500;
    text-shadow: 0px 0px 8px rgba(255, 255, 255, 1),
                 0px 0px 16px rgba(255, 255, 255, 0.9);
  }

  .message-container p {
    margin-bottom: 20px;
  }

  /* Scrollbar */
  .message-container::-webkit-scrollbar {
    width: 6px;
  }
  .message-container::-webkit-scrollbar-track {
    background: rgba(255, 255, 255, 0.02);
    border-radius: 8px;
  }
  .message-container::-webkit-scrollbar-thumb {
    background: rgba(255, 255, 255, 0.18);
    border-radius: 8px;
  }

  .message-container.light-theme::-webkit-scrollbar-track {
    background: rgba(255, 255, 255, 0.3);
  }
  .message-container.light-theme::-webkit-scrollbar-thumb {
    background: rgba(0, 0, 0, 0.25);
  }

  /* Buttons & Inputs */
  .btn {
    background: rgba(255, 255, 255, 0.12);
    color: white;
    border: 1px solid rgba(255, 255, 255, 0.25);
    padding: 12px 20px;
    border-radius: 14px;
    font-size: 1rem;
    font-weight: 500;
    cursor: pointer;
    width: 100%;
    margin-top: 12px;
    margin-bottom: 6px;
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    backdrop-filter: blur(8px);
    font-family: inherit;
    box-shadow: 0 4px 15px rgba(0, 0, 0, 0.15);
  }

  .btn:hover {
    background: rgba(255, 255, 255, 0.22);
    border-color: rgba(255, 255, 255, 0.4);
    transform: translateY(-1px);
    box-shadow: 0 6px 20px rgba(0, 0, 0, 0.25);
  }

  .btn:active {
    transform: translateY(1px);
  }

  .input-group {
    display: flex;
    flex-direction: column;
    gap: 12px;
    justify-content: center;
    height: 100%;
    align-items: center;
    text-align: center;
    padding: 20px 8px;
  }

  .input-group p {
    font-size: 1.18rem;
    font-weight: 500;
    margin-bottom: 6px;
    color: #f1f5f9;
  }

  .password-input {
    width: 100%;
    padding: 14px 18px;
    border-radius: 14px;
    border: 1px solid rgba(255, 255, 255, 0.22);
    background: rgba(0, 0, 0, 0.25);
    color: white;
    font-size: 1.05rem;
    outline: none;
    text-align: center;
    transition: all 0.25s ease;
    font-family: inherit;
  }

  .password-input:focus {
    border-color: rgba(255, 255, 255, 0.65);
    box-shadow: 0 0 0 3px rgba(255, 255, 255, 0.15);
    background: rgba(0, 0, 0, 0.35);
  }

  .password-input::placeholder {
    color: rgba(255, 255, 255, 0.4);
  }

  .error-msg {
    color: #f87171;
    font-size: 0.92rem;
    margin-top: 2px;
    animation: shake 0.4s ease;
    font-weight: 500;
  }

  @keyframes shake {
    0%, 100% { transform: translateX(0); }
    20%, 60% { transform: translateX(-6px); }
    40%, 80% { transform: translateX(6px); }
  }

  /* Fade in helper */
  .fade-in {
    animation: fadeIn 0.45s ease-out forwards;
  }

  @keyframes fadeIn {
    from {
      opacity: 0;
      transform: translateY(6px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }

  /* Animation: Blooming Flowers */
  .flower {
    position: absolute;
    opacity: 0;
    transform: scale(0) rotate(-45deg);
    animation: bloom 2s cubic-bezier(0.175, 0.885, 0.32, 1.275) forwards;
    user-select: none;
    pointer-events: none;
  }

  @keyframes bloom {
    to {
      opacity: 0.88;
      transform: scale(1) rotate(0deg);
    }
  }

  /* Animation: Floating Hearts */
  .heart {
    position: absolute;
    bottom: -10vh;
    opacity: 0;
    animation: floatUp linear forwards;
    user-select: none;
    pointer-events: none;
  }

  @keyframes floatUp {
    0% {
      transform: translateY(0) scale(0.5);
      opacity: 0;
    }
    20% {
      opacity: 0.75;
    }
    100% {
      transform: translateY(-115vh) scale(1.2);
      opacity: 0;
    }
  }

  /* Mobile responsiveness */
  @media (max-width: 480px) {
    .scene {
      max-width: 92vw;
      height: 82vh;
    }
    .card-face {
      border-radius: 20px;
    }
    .card-back {
      padding: 28px 16px 20px;
    }
    .message-container {
      font-size: 0.96rem;
      line-height: 1.68;
    }
    .front-title {
      font-size: 1.45rem;
    }
  }
</style>
