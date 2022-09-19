<script>
  import { createEventDispatcher } from 'svelte'

  const dispatch = createEventDispatcher()
  let selected = false

  const select = category => {
    selected = true
    dispatch('select', { category })
  }

  const categories = [
    {
      slug: 'actors',
      label: 'Actors',
    },
    {
      slug: 'athletes',
      label: 'Athletes',
    },
    {
      slug: 'comedians',
      label: 'Comedians',
    },
    {
      slug: 'creators',
      label: 'Creators',
    },
    {
      slug: 'models',
      label: 'Models',
    },
    {
      slug: 'musicians',
      label: 'Musicians',
    },
    {
      slug: 'reality-tv',
      label: 'Reality TV',
    },
  ]
</script>

<header>
  <h1>
    CameoP<span class="logo">a</span>rison
  </h1>

  <p>
    On <a href="https://cameo.com">cameo.com</a>, you can buy personalised video
    clips from everyone from Lindsay Lohan to Ice T.
  </p>

  <p>But who commands the highest price?</p>
</header>

<p>Pick a category to play a game:</p>

<div class="categories">
  {#each categories as category}
    <button on:click={() => select(category)} disabled={selected}
      >{category.label}</button
    >
  {/each}
</div>

<style>
  h1 {
    color: #ff3e00;
    font-size: min(12vw, 4em);
    font-weight: 100;
    margin: 0 0 0.5em 0;
    text-transform: uppercase;
  }

  p {
    margin: 0 auto 1em auto;
    max-width: 24em;
  }

  .logo {
    background: url(/icons/compare.svg) 50% 50% no-repeat;
    background-size: 100% 100%;
    display: inline-block;
    left: 0.02em;
    text-indent: -9999px;
    top: 0.05em;
    transform: scale(1.4);
    width: 0.8em;
  }

  .categories {
    margin: 0 auto;
    max-width: 26em;
    width: 100%;
  }

  button {
    padding: 1em;
    display: block;
    margin: 0 0 0.2em 0;
    width: 100%;
  }

  @media (min-width: 640px) {
    button {
      display: inline;
      margin: 0 0.2em 0.4em 0.2em;
      width: auto;
    }
  }
</style>
