<script>
  export let p1;
  export let p2;
  export let alpha;
  export let power;

  function validateProportion(value) {
    if (value < 0) return 0;
    if (value > 1) return 1;
    return value;
  }

  $: p1 = validateProportion(p1);
  $: p2 = validateProportion(p2);

  function cohensD(p1, p2) {
    const difference = p1 - p2;
    const pooledSD = Math.sqrt(((p1 * (1 - p1)) + (p2 * (1 - p2))) / 2);
    return difference / pooledSD;
  }

  function zValue(p) {
    return Math.sqrt(2) * erfinv(2 * p - 1);
  }

  function erfinv(x) {
      const a = 0.147;
      const ln = Math.log(1 - x * x);
      const part1 = 2 / (Math.PI * a) + ln / 2;
      const part2 = ln / a;
      const sign = x < 0 ? -1 : 1;
      return sign * Math.sqrt(Math.sqrt(part1 * part1 - part2) - part1);
  }

  function calculateSampleSize2(p1, p2, alpha, power, alternative = 'two-sided') {
    const zAlpha = zValue(1 - alpha / (alternative === 'two-sided' ? 2 : 1));
    const zBeta = zValue(power);
    const p = (p1 + p2) / 2;
    const numerator = (zAlpha * Math.sqrt(2 * p * (1 - p)) + zBeta * Math.sqrt(p1 * (1 - p1) + p2 * (1 - p2)));
    const sampleSize = Math.pow(numerator, 2) / Math.pow(p1 - p2, 2);
    return Math.ceil(sampleSize);
  }
</script>

<style>
  .container {
    max-width: 500px;
    margin: 0 auto;
    padding: 20px;
    background-color: #f9f9f9;
    border-radius: 10px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    font-family: Arial, sans-serif;
  }

  h1 {
    text-align: center;
    color: #333;
  }

  .input-group {
    margin-bottom: 20px;
  }

  label {
    display: block;
    font-weight: bold;
    color: #555;
    margin-bottom: 8px;
  }

  input[type="number"],
  input[type="range"] {
    width: 100%;
    padding: 8px;
    margin-top: 4px;
    border-radius: 5px;
    border: 1px solid #ddd;
  }

  input[type="range"] {
    padding: 0;
    margin-top: 0;
  }

  .output {
    background-color: #eef;
    padding: 15px;
    border-radius: 10px;
    text-align: center;
    color: #333;
    font-size: 1.2em;
  }

  .range-group {
    display: flex;
    align-items: center;
  }

  .range-group span {
    margin-left: 10px;
    font-size: 1em;
    color: #333;
  }
</style>

<div class="container">
  <h1>Sample size calculator</h1>
  <p>Also known as the Power Calculator. How many visitors are needed for an A/B test?</p>

  <div class="input-group">
    <label for="p1">Baseline conversion rate (%)</label>
    <input id="p1" type="number" bind:value={p1} min="0" max="1" step="0.01" />
  </div>

  <div class="input-group">
    <label for="p2">Detectable conversion rate of variant (%)</label>
    <input id="p2" type="number" bind:value={p2} min="0" max="1" step="0.01" />
  </div>

  <div class="input-group">
    <label for="alpha"><dfn title="Percent of the time a difference will be detected, assuming one does not exist">Signifiance level (α)</dfn></label>
    <div class="range-group">
      <input id="alpha" type="range" bind:value={alpha} min="0" max="1" step="0.01" />
      <span>{alpha}</span>
    </div>
  </div>

  <div class="input-group">
    <label for="power" ><dfn title="Percent of the time the minimum effect size will be detected, assuming it exists">Statistical power (1 - β)</dfn></label>
    <div class="range-group">
      <input id="power" type="range" bind:value={power} min="0" max="1" step="0.01" />
      <span>{power}</span>
    </div>
  </div>

  <div class="output">
    <p>Cohen's d: {cohensD(p1, p2).toFixed(5)}</p>
    <p>Required Sample Size (per variant): {calculateSampleSize2(p1, p2, alpha, power)}</p>
  </div>
</div>
