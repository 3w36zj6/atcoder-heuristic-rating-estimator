<script lang="ts">
  import { Chart } from "chart.js"
  import { onMount } from "svelte"
  let performancesTextArea: string = ""
  const s: number = 724.4744301
  const r: number = 0.8271973364

  let rate: number = 0

  let chartCanvas
  let chart

  const colorRating = (rating) => {
    if (rating < 400) return [128, 128, 128]
    else if (rating < 800) return [128, 64, 0]
    else if (rating < 1200) return [0, 128, 0]
    else if (rating < 1600) return [0, 192, 192]
    else if (rating < 2000) return [0, 0, 255]
    else if (rating < 2400) return [192, 192, 0]
    else if (rating < 2800) return [255, 128, 0]
    return [255, 0, 0]
  }

  const calculatePerformance = (newPerformance?: number) => {
    const performances: number[] = []
    for (const p of performancesTextArea.split("\n")) {
      if (!isNaN(parseInt(p))) {
        performances.push(parseInt(p))
      }
    }
    if (newPerformance != undefined) {
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

  const drawChart = (data) => {
    if (chart) {
      chart.destroy()
    }
    const chartData = {
      labels: [...Array(32).keys()].map((i) => {
        return i * 100
      }),
      datasets: [
        {
          label: "New Rate",
          backgroundColor: "rgb(128, 128, 128, 0)",
          borderColor: "rgb(128, 128, 128)",
          data: data,
        },
      ],
    }

    const drawBackground = (target) => {
      const xScale = target.scales["x-axis-0"]
      const yScale = target.scales["y-axis-0"]

      for (const rate of [...Array(8).keys()].map((i) => {
        return i * 400
      })) {
        chartCanvas.getContext("2d").fillStyle = `rgba(${colorRating(rate).join(
          ", "
        )}, 0.2)`
        chartCanvas
          .getContext("2d")
          .fillRect(
            xScale.left,
            Math.min(yScale.getPixelForValue(rate + 400), yScale.bottom),
            xScale.width,
            Math.min(yScale.getPixelForValue(rate), yScale.bottom) -
              Math.min(yScale.getPixelForValue(rate + 400), yScale.bottom)
          )
        chartCanvas.getContext("2d").fillStyle = `rgba(${colorRating(rate).join(
          ", "
        )}, 0.5)`
        chartCanvas
          .getContext("2d")
          .fillRect(
            xScale.getPixelForValue(rate),
            yScale.bottom,
            Math.min(xScale.getPixelForValue(rate + 400), xScale.right) -
              Math.min(xScale.getPixelForValue(rate), xScale.right),
            5
          )
      }
    }

    chart = new Chart(chartCanvas, {
      type: "line",
      data: chartData,
      options: {
        responsive: true,
        legend: { display: false },
        scales: {
          xAxes: [
            {
              scaleLabel: {
                display: true,
                labelString: "Performance",
              },
            },
          ],
          yAxes: [
            {
              scaleLabel: {
                display: true,
                labelString: "New Rate",
              },
            },
          ],
        },
      },
      plugins: [
        {
          beforeDraw: drawBackground,
        },
      ],
    } as any)
  }

  const calculateButton = () => {
    rate = calculatePerformance()
    let newRates: number[] = []
    for (const i of [...Array(32).keys()]) {
      newRates.push(calculatePerformance(i * 100))
    }
    drawChart(newRates)
  }

  onMount(calculateButton)
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
  <p>{rate || "未参加"}</p>

  <h2>次回のレート</h2>
  <p><canvas width={1280} height={720} bind:this={chartCanvas} /></p>

  <h2>View on GitHub</h2>
  <p>
    <a href="https://github.com/3w36zj6/atcoder-heuristic-rating-estimator"
      ><img
        alt="atcoder-heuristic-rating-estimator"
        src="https://gh-card.dev/repos/3w36zj6/atcoder-heuristic-rating-estimator.svg?fullname="
      /></a
    >
  </p>

  <h2>Links</h2>
  <ul>
    <li>
      <a href="https://www.dropbox.com/s/ne358pdixfafppm/AHC_rating.pdf"
        >AHC Rating System</a
      >
    </li>
  </ul>
</main>

<style>
  textarea {
    width: 150px;
    height: 200px;
  }
</style>
