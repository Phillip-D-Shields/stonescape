<script>
  import VolumeCalculator from './lib/VolumeCalculator.svelte';
  import CostEstimator from './lib/CostEstimator.svelte';

  let currentVolume = $state(0);
</script>

<div class="min-h-screen bg-base-200 p-4">
  <div class="max-w-4xl mx-auto">
    <div class="navbar bg-base-100 rounded-box shadow-lg mb-6">
      <h1 class="px-4 normal-case text-3xl">Stonescape Calc</h1>
    </div>

    <VolumeCalculator bind:volume={currentVolume} />
    
    <CostEstimator inputVolume={currentVolume} />
  </div>
</div>