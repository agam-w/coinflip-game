<script lang="ts">
  import * as PIXI from "pixi.js";
  import * as TWEEN from "@tweenjs/tween.js";
  import { onMount } from "svelte";
  import { sound, Sound } from "@pixi/sound";

  const winRate = 0.4; // winrate chance is 40%

  let sideSelected = 0; // 0.head 1.tail
  let finalSide = 0; // final side after flip completed
  let isSpin = false; // indicate coin is currently spinning
  let isSpinCompleted = false;
  let isWin = false;

  $: isWin = finalSide === sideSelected;

  let gameContainer: any;

  const headImgUrl = "head.png";
  const tailImgUrl = "tail.png";
  const headTexture = PIXI.Texture.from(headImgUrl);
  const tailTexture = PIXI.Texture.from(tailImgUrl);

  const bgm = Sound.from({
    url: "lazy-village.mp3",
    preload: true,
    autoPlay: true,
    loop: true,
    volume: 0.7,
  });

  const btnSfx = Sound.from({
    url: "click.mp3",
    preload: true,
  });

  const spinSfx = Sound.from({
    url: "spin.mp3",
    preload: true,
    loop: true,
    speed: 2,
  });

  const winSfx = Sound.from({
    url: "win.mp3",
    preload: true,
  });

  // create a new Sprite using the texture
  const character = new PIXI.Sprite(headTexture);
  // center the sprites anchor point
  character.anchor.set(0.5);

  // delclare tween for rotation
  let rot = { rotation: 0 };
  let tween = new TWEEN.Tween(rot, false).easing(TWEEN.Easing.Quadratic.InOut);

  function randomInteger(min: number, max: number) {
    return Math.floor(Math.random() * (max - min + 1)) + min;
  }

  const startFlipCoin = () => {
    if (isSpin) return;
    spinSfx.stop();
    btnSfx.play();

    isSpin = true;
    isSpinCompleted = false;
    spinSfx.play();

    // RNG
    if (Math.random() < winRate) {
      finalSide = sideSelected;
    } else {
      finalSide = sideSelected == 0 ? 1 : 0;
    }
    console.log({
      winRate,
      sideSelected,
      finalSide,
      isWin: sideSelected === finalSide,
    });

    // reset side facing to begin anim
    if (sideSelected == 0) {
      // head
      rot = { rotation: 0 };
    } else {
      // tail
      rot = { rotation: Math.PI };
    }

    tween = new TWEEN.Tween(rot, false)
      .easing(TWEEN.Easing.Quadratic.InOut)
      .to({ rotation: finalSide == 0 ? 40 * Math.PI : 41 * Math.PI }, 3000)
      .onUpdate((v: { rotation: number }) => {
        const cosAbs = Math.abs(Math.cos(v.rotation));
        character.scale.x = cosAbs;
        const cos = Math.cos(v.rotation);
        const side = cos >= 0 ? 0 : 1;
        character.texture = side === 0 ? headTexture : tailTexture;
      })
      .onComplete((v: { rotation: number }) => {
        spinSfx.stop();
        isSpin = false;
        isSpinCompleted = true;
        if (sideSelected == finalSide) {
          winSfx.play();
        }
      })
      .stop()
      .start();
  };

  const setSide = (idx: number) => {
    spinSfx.stop();
    btnSfx.play();
    isSpinCompleted = false;

    // reset side facing to begin anim
    if (sideSelected == 0) {
      // head
      rot = { rotation: 0 };
    } else {
      // tail
      rot = { rotation: Math.PI };
    }

    sideSelected = idx;

    tween = new TWEEN.Tween(rot, false)
      .easing(TWEEN.Easing.Quadratic.InOut)
      .to({ rotation: sideSelected == 0 ? 0 : Math.PI }, 500)
      .onUpdate((v: { rotation: number }) => {
        const cosAbs = Math.abs(Math.cos(v.rotation));
        character.scale.x = cosAbs;
        const cos = Math.cos(v.rotation);
        const side = cos >= 0 ? 0 : 1;
        character.texture = side === 0 ? headTexture : tailTexture;
      })
      .stop()
      .start();
  };

  let ticker = PIXI.Ticker.shared;
  ticker.autoStart = false;
  ticker.stop();

  // loops functions
  const animate = (time: any) => {
    ticker.update(time);
    tween.update(time);
    requestAnimationFrame(animate);
  };

  onMount(() => {
    gameContainer.innerHtml = ""; // clear all child

    const app = new PIXI.Application({
      backgroundColor: "#000000",
      backgroundAlpha: 0,
      resizeTo: gameContainer,
    });

    gameContainer.appendChild(app.view);

    // move the sprite to the center of the screen
    character.x = app.screen.width / 2;
    character.y = app.screen.height / 2;

    app.stage.addChild(character);

    // loop function
    animate(performance.now());
  });
</script>

<div class="bg-indigo-950 h-screen w-screen">
  <div bind:this={gameContainer} class="w-screen h-[320px]" />
  <div class="flex justify-center text-gray-200 font-medium py-4">
    <div>
      <p class="uppercase text-sm">Pick Side</p>
      <div class="flex space-x-4 py-4">
        <button
          disabled={isSpin}
          class="coin-btn {sideSelected == 0 ? 'selected' : ''}"
          on:click={() => setSide(0)}
        >
          <img
            src={headImgUrl}
            alt="coin head"
            class="h-20 w-20 object-contain"
          />
        </button>
        <button
          disabled={isSpin}
          class="coin-btn {sideSelected == 1 ? 'selected' : ''}"
          on:click={() => setSide(1)}
        >
          <img
            src={tailImgUrl}
            alt="coin head"
            class="h-20 w-20 object-contain"
          />
        </button>
      </div>
      <div class="flex">
        <button class="btn" disabled={isSpin} on:click={startFlipCoin}
          >FLIP COIN</button
        >
      </div>
      <div class="py-4">
        {#if isSpinCompleted}
          {#if isWin}
            <p class="label-result win">You Win !</p>
          {:else}
            <p class="label-result lose">You Lose !</p>
          {/if}
        {/if}
      </div>
    </div>
  </div>
</div>

<style lang="scss">
  .coin-btn {
    @apply bg-black/20 hover:bg-white/10 p-4 rounded-xl cursor-pointer transition;
    &.selected {
      @apply outline outline-2 outline-white/80 disabled:outline-white/40;
    }
  }
  .btn {
    @apply flex-1 bg-yellow-400 hover:bg-yellow-300 
    border-b-4 border-yellow-600 hover:border-yellow-500 
    disabled:bg-yellow-600
    px-6 py-3 rounded-lg text-black font-bold transition;
  }
  .label-result {
    @apply text-center text-lg;
    &.win {
      @apply text-green-500;
    }
    &.lose {
      @apply text-red-500;
    }
  }
</style>
