<script>
import { base } from '$app/paths';
    let maxClick = $state(2);
let cnt = $state(2);// tip: https://svelte.dev/docs/svelte/$state
    
  
    function onClick() {
  if (cnt > 0) {
    cnt -= 1;
  }
}
  </script>
  
  <h1>[Zichun su]'s VIS Site</h1>
  <img
    width="200px"
    src="{base}/IMG_0910.jpg"
  />
  <div>
    You can click up to
    <select 
        bind:value={maxClick} 
        onchange={() => (cnt = maxClick)}>
      {#each [2, 4, 6] as optionNum}
        <option value={optionNum}>
          {optionNum}
        </option>
      {/each}
    </select>
    times
  </div>
  <button onclick={onClick}> Click Me </button>

  {#if cnt > 0}
  <p id="info">Remaining Number of Clicks: {cnt}</p>
{:else}
  <p>No more clicks allowed</p>
{/if}
  
  
  <style>
    body {
      font-family: Arial, Helvetica, sans-serif;
    }
    button {
      background-color: #44aa66;
      /* background-color: blue; */
      color: white;
      font-size: xx-large;
      padding: 10px 20px;
      border: none;
      cursor: pointer;
      border-radius: 5px;
    }
  </style>
  