<script lang="ts">

// Imports --------------------------------------------------------------------

import TactileSvg from "./lib/TactileSvg.svelte";

// Handlers -------------------------------------------------------------------

let fillColor = $state("red");

function onClick(e: Event) {
  fillColor = fillColor === "red" ? "green" : "red";
  e.preventDefault(); 
}

</script>

<TactileSvg>
  <!-- svelte-ignore a11y_click_events_have_key_events -->
  <!-- svelte-ignore a11y_no_static_element_interactions -->
  <circle 
    onmouseenter={onClick} 
    onmouseout={onClick} 
    onblur={onClick}
    cx="50" 
    cy="50" 
    r="5" 
    fill={fillColor} />
  <rect x="20" y="20" width="10" height="10" fill="blue" />
</TactileSvg>