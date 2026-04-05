<script>
	import { onMount } from 'svelte';
  import Chart from 'chart.js/auto';

  let latencyChart;
  let chart;

  onMount(function() {
    const fontStack = "ui-monospace, monospace";
    Chart.defaults.font.family = fontStack;

    chart = new Chart(
      latencyChart,
      {
        type: 'bar',
        data: {
          labels: ['8 Clients', '16 Clients', '32 Clients'],
          datasets: [
            {
              label: 'p50',
              data: [379, 736, 1352],
              backgroundColor: 'rgba(204, 255, 0, 0.7)',
              hoverBackgroundColor: 'rgba(204, 255, 0, 1)',
              borderRadius: 0,
              borderSkipped: false,
              barPercentage: 0.85,
              categoryPercentage: 0.7
            },
            {
              label: 'p95',
              data: [446, 843, 1592],
              backgroundColor: 'rgba(34, 0, 204, 0.7)',
              hoverBackgroundColor: 'rgba(34, 0, 204, 1)',
              borderRadius: 0,
              borderSkipped: false,
              barPercentage: 0.85,
              categoryPercentage: 0.7
            },
            {
              label: 'p99',
              data: [476, 892, 1700],
              backgroundColor: 'rgba(238, 0, 102, 0.7)',
              hoverBackgroundColor: 'rgba(238, 0, 102, 1)',
              borderRadius: 0,
              borderSkipped: false,
              barPercentage: 0.85,
              categoryPercentage: 0.7
            },
          ]
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          plugins: {
            title: {
              display: true,
              text: '1M requests, 70/30 write-to-read ratio'
            },
            legend: {
              position: 'top',
              labels: {
                usePointStyle: true,
                padding: 20,
                font: {
                  size: 13,
                  weight: '600'
                }
              }
            },
            tooltip: {
              backgroundColor: 'rgba(17, 24, 39, 0.95)',
              titleColor: '#fff',
              titleFont: { size: 14, family: fontStack },
              bodyColor: '#e5e7eb',
              bodyFont: { size: 13, family: fontStack },
              padding: 12,
              cornerRadius: 8,
              displayColors: true,
              callbacks: {
                label: function(context) {
                  return ` ${context.dataset.label}: ${context.raw} µs`;
                }
              }
            }
          },
          scales: {
            x: {
              grid: {
                display: false,
                drawBorder: false
              },
              ticks: {
                font: {
                  weight: '600'
                }
              }
            },
            y: {
              title: {
                display: true,
                text: 'Latency (microseconds)',
                font: {
                  weight: 'bold',
                  size: 13
                },
                padding: { bottom: 10 }
              },
              grid: {
                color: 'rgba(0, 0, 0, 0.06)',
                drawBorder: false,
              },
              beginAtZero: true
            }
          }
        }
      }
    );

    return () => {
      chart.destroy();
    };
  });
</script>

<div class="px-8 mb-4 w-full min-h-84">
  <canvas bind:this={latencyChart}></canvas>
</div>
