<script>
  // Svelte 5 Props with Runes
  let { volume = $bindable(0) } = $props();

  let length = $state(0);
  let width = $state(0);
  let depth = $state(0);

  // Unit multipliers
  let unitL = $state(1); // 1=m, 0.01=cm
  let unitW = $state(1);
  let unitD = $state(0.001); // Default mm for depth

  // Reactive calculation
  let calculatedVolume = $derived((length * unitL) * (width * unitW) * (depth * unitD));

  // Sync back to parent
  $effect(() => {
    volume = calculatedVolume;
  });
</script>

<div class="card bg-base-100 shadow-xl mb-4">
  <div class="card-body">
    <h2 class="card-title text-primary">1. Volume Calculator</h2>
    
    <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
      <div class="form-control">
        <label class="label"><span class="label-text">Length</span></label>
        <div class="input-group">
          <input type="number" bind:value={length} class="input input-bordered w-full" step="0.1" />
          <select bind:value={unitL} class="select select-bordered">
            <option value={1}>m</option>
            <option value={0.01}>cm</option>
            <option value={0.001}>mm</option>
          </select>
        </div>
      </div>

      <div class="form-control">
        <label class="label"><span class="label-text">Width</span></label>
        <div class="input-group">
          <input type="number" bind:value={width} class="input input-bordered w-full" step="0.1" />
          <select bind:value={unitW} class="select select-bordered">
            <option value={1}>m</option>
            <option value={0.01}>cm</option>
            <option value={0.001}>mm</option>
          </select>
        </div>
      </div>

      <div class="form-control">
        <label class="label"><span class="label-text">Depth/Height</span></label>
        <div class="input-group">
          <input type="number" bind:value={depth} class="input input-bordered w-full" step="10" />
          <select bind:value={unitD} class="select select-bordered">
            <option value={0.001}>mm</option>
            <option value={0.01}>cm</option>
            <option value={1}>m</option>
          </select>
        </div>
      </div>
    </div>

    <div class="alert alert-info mt-4 shadow-lg">
      <div class="flex items-center gap-2">
        <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" class="stroke-current flex-shrink-0 w-6 h-6"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"></path></svg>
        <span>Volume: <strong>{calculatedVolume.toFixed(3)} m³</strong></span>
      </div>
    </div>
  </div>
</div>