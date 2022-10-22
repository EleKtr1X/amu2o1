<script>
  export let img;
  export let price;
  export let name;
  export let info;

  import { createEventDispatcher } from 'svelte';
  const dispatch = createEventDispatcher();

  function clickHandle() {
    dispatch('click');
  }
</script>

<div class="powerup">
  <img
    src={img}
    alt={name}
    on:click={clickHandle}
    on:keypress={clickHandle}
    class="img"
  >
  <abbr title="{name} &mdash; {info}">
    <b class="text">${price}</b>
  </abbr>
</div>

<style>
  .powerup {
    display: flex;
    flex-direction: column;
    align-items: center;
  }
  
  .img {
    margin-top: 25px;
    margin-bottom: -15px;
    max-height: 75px;
    cursor: pointer;
  }

  .text {
    font-size: 20px;
  }

  .text:hover {
    cursor: help;
  }
</style>