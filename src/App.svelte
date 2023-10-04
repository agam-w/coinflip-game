<script lang="ts">
  import * as PIXI from "pixi.js";
  import * as TWEEN from "@tweenjs/tween.js";
  import { onMount } from "svelte";

  let gameContainer: any;

  const headImgUrl = "https://pixijs.com/assets/flowerTop.png";
  const tailImgUrl = "https://pixijs.com/assets/eggHead.png";
  const headTexture = PIXI.Texture.from(headImgUrl);
  const tailTexture = PIXI.Texture.from(tailImgUrl);

  let sideSelected = 0; // 0.head 1.tail

  // create a new Sprite using the texture
  const character = new PIXI.Sprite(headTexture);
  // center the sprites anchor point
  character.anchor.set(0.5);

  // delclare tween for rotation
  let rot = { rotation: 0 };

  const tween = new TWEEN.Tween(rot, false)
    .easing(TWEEN.Easing.Quadratic.InOut)
    .onUpdate((v: { rotation: number }) => {
      character.rotation = v.rotation;
      // todo: random rotation with side
      character.texture =
        Math.round(v.rotation % 2) === 0 ? headTexture : tailTexture;
    })

  const startFlipCoin = () => {
    // todo: random rotation
    tween.to({ rotation: 100 }, 3000).start();
  };

  const setSide = (idx: number) => {
    sideSelected = idx;
    character.texture = sideSelected === 0 ? headTexture : tailTexture;
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
        <button class="btn" on:click={startFlipCoin}>FLIP COIN</button>
      </div>
    </div>
  </div>
</div>

<style lang="scss">
  .coin-btn {
    @apply bg-black/20 hover:bg-white/10 p-4 rounded-xl cursor-pointer transition;
    &.selected {
      @apply outline outline-2 outline-white/80;
    }
  }
  .btn {
    @apply flex-1 bg-yellow-400 hover:bg-yellow-300 
    border-b-4 border-yellow-600 hover:border-yellow-500 
    px-6 py-3 rounded-lg text-black font-bold transition;
  }
</style>
