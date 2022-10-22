<script>
  export let price;
  export let answer;
  export let category;
  export let img;
  export let disabled;
  import { createEventDispatcher } from 'svelte';
  const dispatch = createEventDispatcher();
  
  function registerClick() {
    if (!disabled) {
      dispatch('click', [price, answer, img, category]);
    }
  }

</script>

<div class={disabled ? "q disabled" : "q"} on:click={registerClick} on:keypress={registerClick}>
  ${price}
</div>

<style>
  .q {
    border-radius: 10px;
    background-color: #ffffff;
    padding: 10px 20px;
    border: 8px solid black;
    cursor: pointer;
  }

  .disabled {
    background-color: #888888;
    cursor: not-allowed;
  }
</style>