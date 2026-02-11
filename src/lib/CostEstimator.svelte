<script>
  import { products } from './products.js';

  // Props
  let { inputVolume = 0 } = $props();

  // State
  let selectedProductIndex = $state(0);
  let useManual = $state(false);
  
  // Manual Input State
  let manualInputMode = $state('volume'); // 'volume' or 'weight'
  let manualValue = $state(0);

  // Derived: Current Product
  let product = $derived(products[selectedProductIndex]);
  
  // Derived: Calculate Effective Volume (m3)
  // This is our "source of truth" for the scoop calculations
  let effectiveVolume = $derived.by(() => {
    // 1. If using the Volume Calculator module input
    if (!useManual) return inputVolume;
    
    // 2. If Manual Mode is 'Volume' (m3)
    if (manualInputMode === 'volume') return manualValue;
    
    // 3. If Manual Mode is 'Weight' (Tonnes) -> Convert to Volume
    // Formula: Volume = Mass / Density
    if (manualInputMode === 'weight' && product.density) {
      return manualValue / product.density;
    }
    
    return 0;
  });

  // Derived: Calculate Weight
  let weight = $derived.by(() => {
    // If we are manually typing the weight, just return that value
    if (useManual && manualInputMode === 'weight') return manualValue;
    
    // Otherwise calculate it from the effective volume
    return product.density ? (effectiveVolume * product.density) : 0;
  });

  // Derived: Scoops & Costs
  let scoops = $derived(effectiveVolume / 0.4); // 1 scoop = 0.4m3
  let costByScoop = $derived(product.price_scoop ? (scoops * product.price_scoop) : 0);
  let costByTon = $derived(product.price_ton ? (weight * product.price_ton) : 0);
</script>

<div class="card bg-base-100 shadow-xl mb-4 border border-base-200">
  <div class="card-body">
    <h2 class="card-title text-success">Converter & Cost Estimator</h2>
    
    <div class="form-control bg-base-200 p-2 rounded-lg">
      <label class="label cursor-pointer justify-start gap-4">
        <input type="checkbox" class="toggle toggle-secondary" bind:checked={useManual} />
        <span class="label-text font-semibold">
          {useManual ? 'Manual Input Mode' : `Using Calculator Result (${inputVolume.toFixed(2)} m³)`}
        </span>
      </label>
    </div>

    {#if useManual}
      <div class="mt-2 p-4 bg-base-200 rounded-lg animate-in fade-in zoom-in duration-200">
        
        <div class="tabs tabs-boxed mb-4 bg-base-100">
          <button 
            class="tab flex-1 {manualInputMode === 'volume' ? 'tab-active' : ''}" 
            onclick={() => manualInputMode = 'volume'}
          >
            Enter Cubic Meters ($m^3$)
          </button>
          <button 
            class="tab flex-1 {manualInputMode === 'weight' ? 'tab-active' : ''}" 
            disabled={!product.density}
            onclick={() => manualInputMode = 'weight'}
          >
            Enter Tonnes (T)
            {#if !product.density}<span class="text-xs ml-1 opacity-50">(No Density)</span>{/if}
          </button>
        </div>

        <div class="form-control w-full">
          <label class="label">
            <span class="label-text">
              Enter {manualInputMode === 'volume' ? 'Volume' : 'Weight'}
            </span>
          </label>
          <div class="input-group">
            <input 
              type="number" 
              bind:value={manualValue} 
              class="input input-bordered w-full" 
              step="0.1" 
              placeholder="0.00"
            />
            <span class="bg-base-300">
              {manualInputMode === 'volume' ? 'm³' : 'Tonnes'}
            </span>
          </div>
        </div>
      </div>
    {/if}

    <div class="divider"></div>

    <div class="form-control w-full">
      <label class="label"><span class="label-text">Select Product</span></label>
      <select class="select select-bordered select-success w-full" bind:value={selectedProductIndex}>
        {#each products as p, i}
          <option value={i}>
            {p.name} 
            {p.density ? `(${p.density} T/m³)` : ''}
          </option>
        {/each}
      </select>
    </div>

    <div class="grid grid-cols-1 md:grid-cols-2 gap-4 mt-4">
      
      <div class="stats shadow border border-base-200">
        <div class="stat">
          <div class="stat-title">Calculated Weight</div>
          <div class="stat-value text-primary text-3xl">
            {#if product.density}
              {weight.toFixed(2)} <span class="text-lg">T</span>
            {:else}
              <span class="text-gray-300 text-lg">N/A</span>
            {/if}
          </div>
          <div class="stat-desc">Based on {effectiveVolume.toFixed(2)} $m^3$</div>
        </div>
      </div>

      <div class="stats shadow border border-base-200">
        <div class="stat">
          <div class="stat-title">Scoops Required</div>
          <div class="stat-value text-primary text-3xl">{scoops.toFixed(1)}</div>
          <div class="stat-desc">Standard 0.4$m^3$ Scoop</div>
        </div>
      </div>
    </div>

    <div class="overflow-x-auto mt-6 border rounded-lg">
      <table class="table w-full">
        <thead class="bg-base-200">
          <tr>
            <th>Pricing Method</th>
            <th>Unit Price</th>
            <th class="text-right">Est. Cost</th>
          </tr>
        </thead>
        <tbody>
          <tr class={product.price_scoop ? '' : 'opacity-50'}>
            <td>
              <div class="font-bold">Retail (Scoop)</div>
              <div class="text-xs opacity-70">Best for trailers/small loads</div>
            </td>
            <td>
              {product.price_scoop ? `$${product.price_scoop.toFixed(2)}` : 'N/A'}
            </td>
            <td class="text-right font-bold text-lg">
              {product.price_scoop ? `$${costByScoop.toFixed(2)}` : '-'}
            </td>
          </tr>

          <tr class={product.price_ton && product.density ? '' : 'opacity-50'}>
            <td>
              <div class="font-bold">Bulk (Weight)</div>
              <div class="text-xs opacity-70">Best for trucks</div>
            </td>
            <td>
              {product.price_ton ? `$${product.price_ton.toFixed(2)}` : 'N/A'}
            </td>
            <td class="text-right font-bold text-lg text-success">
              {(product.price_ton && product.density) ? `$${costByTon.toFixed(2)}` : '-'}
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</div>