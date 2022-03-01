<script lang="ts">
  let performancesTextArea: string = ""
  const s: number = 724.4744301
  const r: number = 0.8271973364

  let rate: number = 0

  const calculatePerformance = (newPerformance?: number) => {
    const performances: number[] = []
    for (const p of performancesTextArea.split("\n")) {
      if (!isNaN(parseInt(p))) {
        performances.push(parseInt(p))
      }
    }
    if (newPerformance) {
      performances.push(newPerformance)
    }
    let extendedPerformances: number[] = []
    for (const p of performances) {
      for (const j of [...Array(100).keys()]) {
        extendedPerformances.push(p - s * Math.log(j + 1))
      }
    }
    extendedPerformances = extendedPerformances.sort((a, b) => b - a)
    let rateDenominator: number = 0
    let rateNumerator: number = 0
    for (const i of [...Array(100).keys()]) {
      rateDenominator += extendedPerformances[i] * r ** i
      rateNumerator += r ** i
    }
    let rate: number = rateDenominator / rateNumerator
    if (rate < 400) {
      rate = 400 / Math.exp((400 - rate) / 400)
    }
    return rate
  }

  const calculateButton = () => {
    rate = calculatePerformance()
    for (const i of [...Array(400).keys()]) {
      console.log(i * 10, calculatePerformance(i * 10))
    }
  }
</script>

<main>
  <h1>AtCoder Heuristic Rating Estimator</h1>

  <h2>過去のパフォーマンス</h2>
  <p>
    改行区切りで入力してください。レート計算式よりパフォーマンスの順番は関係ありません。
  </p>
  <p>
    <textarea
      bind:value={performancesTextArea}
      placeholder="enter your performances."
    />
  </p>
  <p><button on:click={calculateButton}> 計算 </button></p>

  <h2>現在のレート</h2>
  <p>{rate || "?"}</p>
</main>

<style>
  textarea {
    width: 150px;
    height: 200px;
  }
</style>
