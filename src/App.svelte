<script>
  import { onMount } from 'svelte'
  import Welcome from './screens/Welcome.svelte'
  import Game from './screens/Game.svelte'
  import { select } from './select'
  import { load_image } from './utils'

  let celebs_promise

  let state = 'welcome'
  let selection

  const start = async event => {
    const { celebs, lookup } = await celebs_promise
    selection = select(celebs, lookup, event.detail.category.slug)
    state = 'playing'
  }

  const load_celebs = async () => {
    const res = await fetch('https://cameo-explorer.netlify.app/celebs.json')
    const data = await res.json()

    const lookup = new Map()
    data.forEach(celeb => {
      lookup.set(celeb.id, celeb)
    })

    const subset = new Set()
    data.forEach(celeb => {
      if (celeb.reviews >= 50) {
        subset.add(celeb)
        celeb.similar.forEach(id => {
          subset.add(lookup.get(id))
        })
      }
    })

    return {
      celebs: Array.from(subset),
      lookup,
    }
  }

  onMount(() => {
    celebs_promise = load_celebs()
    load_image('/icons/right.svg')
    load_image('/icons/wrong.svg')
  })
</script>

<main>
  {#if state === 'welcome'}
    <Welcome on:select={start} />
  {:else if state === 'playing'}
    <Game {selection} on:restart={() => (state = 'welcome')} />
  {/if}
</main>

<style>
  main {
    display: flex;
    flex-flow: column nowrap;
    justify-content: center;
    margin: 0 auto;
    max-width: 800px;
    height: 100%;
    padding: 1rem;
    text-align: center;
  }
</style>
