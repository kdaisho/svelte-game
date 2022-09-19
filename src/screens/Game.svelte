<script>
  import { createEventDispatcher } from 'svelte'
  import { fly, scale, crossfade } from 'svelte/transition'
  import * as eases from 'svelte/easing'
  import Card from '../components/Card.svelte'
  import { load_image, pick_random, sleep } from '../utils'

  export let selection

  const dispatch = createEventDispatcher()

  const [send, receive] = crossfade({
    easing: eases.cubicOut,
    duration: 300,
  })

  const load_details = async celeb => {
    const res = await fetch(
      `https://cameo-explorer.netlify.app/celebs/${celeb.id}.json`
    )
    const details = await res.json()
    await load_image(details.image)
    return details
  }

  const promises = selection.map(round =>
    Promise.all([load_details(round.a), load_details(round.b)])
  )

  const results = Array(selection.length)

  let i = 0
  let last_result
  let done = false
  let ready = true

  $: score = results.filter(x => x === 'right').length

  const pick_message = p => {
    switch (true) {
      case p < 0.5:
        return pick_random([`Ouch`, `That wasn't very good`, `Must try harder`])
      case p <= 0.8:
        return pick_random([`Not bad`, `Keep practicing`])
      case p < 1:
        return pick_random([`So close!`, `Almost there!`])
      default:
        return pick_random([`You rock!`, `Flawless victory!`])
    }
  }

  const submit = async (a, b, sign) => {
    last_result = Math.sign(a.price - b.price) === sign ? 'right' : 'wrong'

    await sleep(1500)

    results[i] = last_result
    console.log({ results })
    last_result = null

    await sleep(0)

    if (i < selection.length - 1) {
      i += 1
    } else {
      done = true
    }
  }
</script>

<header>
  <p>
    Tap on the more monetisable celebrity's face, or tap 'same price' if society
    values them equally.
  </p>
</header>

<div class="game-container">
  {#if done}
    <div
      class="done"
      in:scale={{ delay: 200, duration: 800, easing: eases.elasticOut }}
    >
      <strong>{score}/{results.length}</strong>
      <p>{pick_message(score / results.length)}</p>
      <button on:click={() => dispatch('restart')}>Back to main screen</button>
    </div>
  {:else if ready}
    {#await promises[i] then [a, b]}
      <div
        class="game"
        in:fly={{ duration: 200, y: 20 }}
        out:fly={{ duration: 200, y: -20 }}
        on:outrostart={() => (ready = false)}
        on:outroend={() => (ready = true)}
      >
        <div class="card-container">
          <Card
            on:select={() => submit(a, b, 1)}
            celeb={a}
            showprice={!!last_result}
            winner={a.price >= b.price}
          />
        </div>

        <div>
          <button class="same" on:click={() => submit(a, b, 0)}>
            Same price
          </button>
        </div>

        <div class="card-container">
          <Card
            on:select={() => submit(a, b, -1)}
            celeb={b}
            showprice={!!last_result}
            winner={b.price >= a.price}
          />
        </div>
      </div>
    {:catch}
      <p class="error">Oops! Failed to load data</p>
    {/await}
  {/if}
</div>

{#if last_result}
  <img
    in:fly={{ x: 100, duration: 200 }}
    out:send={{ key: i }}
    class="giant-result"
    src="/icons/{last_result}.svg"
    alt="{last_result} answer"
  />
{/if}

<div
  class="results"
  style="grid-template-columns: repeat({results.length}, 1fr)"
>
  {#each results as result, i}
    <span class="result">
      {#if result}
        <img
          in:receive={{ key: i }}
          src="/icons/{result}.svg"
          alt="{result} answer"
        />
      {/if}
    </span>
  {/each}
</div>

<style>
  .game-container {
    flex: 1;
  }

  .game {
    display: grid;
    grid-template-rows: 1fr 2em 1fr;
    gap: 1em;
    width: 100%;
    height: 100%;
    max-width: min(100%, 40vh);
    margin: 0 auto;
  }

  .game > div {
    display: flex;
    align-items: center;
  }

  .game button {
    width: 100%;
  }

  .error {
    color: red;
  }

  .giant-result {
    height: 50vmin;
    left: calc(50vw - 25vmin);
    opacity: 0.5;
    position: fixed;
    top: calc(50vh - 25vmin);
    width: 50vmin;
  }

  .results {
    display: grid;
    gap: 0.2em;
    margin: 1em auto 0 auto;
    max-width: 320px;
    width: 100%;
  }

  .result {
    background: rgba(255, 255, 255, 0.1);
    border-radius: 50%;
    padding: 0 0 100% 0;
    transition: background 0.2s;
    transition-delay: 0.2s;
  }

  .result img {
    height: 100%;
    left: 0;
    position: absolute;
    top: 0;
    width: 100%;
  }

  .done {
    align-items: center;
    display: flex;
    flex-flow: column nowrap;
    height: 100%;
    justify-content: center;
    left: 0;
    position: absolute;
    top: 0;
    width: 100%;
  }

  .done strong {
    font-size: 6em;
    font-weight: 700;
  }

  @media (min-width: 640px) {
    .game {
      max-width: 100%;
      grid-template-rows: none;
      grid-template-columns: 1fr 8em 1fr;

      /* work around weird Safari bug */
      max-height: calc(100vh - 6em);
    }

    .same {
      height: 8em;
    }
  }
</style>
