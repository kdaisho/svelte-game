<script>
  import { createEventDispatcher } from 'svelte'
  import { scale } from 'svelte/transition'
  import { elasticOut } from 'svelte/easing'

  export let celeb
  export let showprice
  export let winner

  const dispatch = createEventDispatcher()
</script>

<div class="card-outer">
  <button
    class="card-inner"
    on:click={() => dispatch('select')}
    style="background-image: url({celeb.image})"
    disabled={showprice}
  >
    <div class="details">
      <h2>
        <a target="_blank" href="https://cameo.com/{celeb.id}">{celeb.name}</a>
      </h2>
      <p class="type">{celeb.type}</p>
    </div>

    {#if showprice}
      <div class="price" class:large={winner}>
        <span in:scale={{ easing: elasticOut, duration: 600 }}
          >${celeb.price}</span
        >
      </div>
    {/if}
  </button>
</div>

<style>
  .card-outer {
    height: 100%;
    width: 100%;
  }

  .card-inner {
    background: 50% 0 no-repeat;
    background-size: cover;
    border-radius: var(--border-radius);
    box-shadow: 2px 4px 6px rgba(0, 0, 0, 0.2);
    height: 100%;
    left: 0;
    overflow: hidden;
    padding: 0;
    position: absolute;
    text-align: center;
    top: 0;
    width: 100%;
  }

  .details {
    background: linear-gradient(
      to bottom,
      transparent,
      rgba(0, 0, 0, 0.6) 35%,
      rgba(0, 0, 0, 0.9)
    );
    bottom: 0;
    color: #fff;
    padding: 1em 0.5em 0.5em 0.5em;
    position: absolute;
    width: 100%;
  }

  h2 {
    margin: 0 0 0.2em 0;
  }

  .type {
    margin: 0;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  .price {
    align-items: center;
    background: rgba(0, 0, 0, 0.3);
    color: #fff;
    display: flex;
    font-size: 4em;
    font-weight: 700;
    height: 100%;
    justify-content: center;
    left: 0;
    position: absolute;
    top: 0;
    width: 100%;
  }

  .price.large {
    font-size: 6em;
  }

  @media (min-width: 640px) {
    .card-outer {
      height: 0;
      padding: 0 0 100% 0;
    }
  }
</style>
