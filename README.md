<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Raspberry Pi 5 & D500 LiDAR SLAM System Infographic</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700;900&family=Noto+Sans+KR:wght@400;700;900&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', 'Noto Sans KR', sans-serif;
            background-color: #f8fafc;
        }
        .chart-container {
            position: relative;
            width: 100%;
            max-width: 600px;
            margin-left: auto;
            margin-right: auto;
            height: 300px;
            max-height: 400px;
        }
        @media (min-width: 768px) {
            .chart-container {
                height: 350px;
            }
        }
        .kpi-card {
            background-color: white;
            border-radius: 0.75rem;
            padding: 1.5rem;
            box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1);
            text-align: center;
            border-left: 4px solid #ffa600;
        }
        .flowchart-node {
            border: 2px solid #374c80;
            background-color: #ffffff;
            color: #003f5c;
        }
        .flowchart-arrow {
            color: #7a5195;
        }
    </style>
</head>
<body class="text-slate-800">
    <div class="max-w-7xl mx-auto p-4 sm:p-6 lg:p-8">

        <header class="text-center mb-16">
            <h1 class="text-4xl md:text-5xl font-black text-transparent bg-clip-text bg-gradient-to-r from-[#003f5c] to-[#7a5195] mb-4">Orchestrating Autonomy</h1>
            <p class="text-lg md:text-xl text-slate-600 max-w-3xl mx-auto">An inside look at the fully containerized, headless SLAM system for the Raspberry Pi 5 and D500 LiDAR.</p>
        </header>

        <section class="mb-20">
            <h2 class="text-3xl font-bold text-center mb-2 text-[#003f5c]">System Architecture</h2>
            <p class="text-center text-slate-500 mb-12 max-w-2xl mx-auto">A layered approach ensures modularity and performance, from bare metal to intelligent application.</p>
            <div class="flex flex-col md:flex-row items-center justify-center space-y-4 md:space-y-0 md:space-x-4">
                <div class="w-full md:w-1/4 flowchart-node font-bold p-4 rounded-lg shadow-md text-center">Hardware Layer<br>(Raspberry Pi 5 & D500)</div>
                <div class="text-4xl flowchart-arrow font-light transform md:-rotate-0 rotate-90">→</div>
                <div class="w-full md:w-1/4 flowchart-node font-bold p-4 rounded-lg shadow-md text-center">Containerization Layer<br>(Docker Engine)</div>
                <div class="text-4xl flowchart-arrow font-light transform md:-rotate-0 rotate-90">→</div>
                <div class="w-full md:w-1/4 flowchart-node font-bold p-4 rounded-lg shadow-md text-center">Middleware Layer<br>(ROS2 Humble)</div>
                <div class="text-4xl flowchart-arrow font-light transform md:-rotate-0 rotate-90">→</div>
                <div class="w-full md:w-1/4 flowchart-node font-bold p-4 rounded-lg shadow-md text-center">Application Layer<br>(SLAM Algorithms)</div>
            </div>
        </section>

        <section class="mb-20 bg-white p-8 rounded-xl shadow-lg">
            <h2 class="text-3xl font-bold text-center mb-2 text-[#003f5c]">The Build Pipeline</h2>
            <p class="text-center text-slate-500 mb-12 max-w-2xl mx-auto">A multi-stage Docker build creates a lean, optimized runtime environment, minimizing deployment size and attack surface.</p>
            <div class="relative w-full max-w-lg mx-auto">
                <div class="absolute left-1/2 -translate-x-1/2 top-0 bottom-0 w-1 bg-slate-200 rounded-full"></div>
                <div class="relative mb-8">
                    <div class="absolute left-1/2 -translate-x-1/2 -top-1 w-4 h-4 rounded-full bg-[#7a5195]"></div>
                    <div class="ml-auto md:ml-[55%] bg-slate-50 p-4 rounded-lg shadow-md w-full md:w-[45%]">
                        <h3 class="font-bold text-[#7a5195]">Stage 1: ROS2 Base</h3>
                        <p class="text-sm text-slate-600">Installs core ROS2 Humble and essential system packages.</p>
                    </div>
                </div>
                <div class="relative mb-8">
                    <div class="absolute left-1/2 -translate-x-1/2 -top-1 w-4 h-4 rounded-full bg-[#bc5090]"></div>
                    <div class="mr-auto md:mr-[55%] bg-slate-50 p-4 rounded-lg shadow-md w-full md:w-[45%]">
                        <h3 class="font-bold text-[#bc5090]">Stage 2: SLAM Libraries</h3>
                        <p class="text-sm text-slate-600">Builds Hector, GMapping, and Cartographer SLAM packages.</p>
                    </div>
                </div>
                <div class="relative mb-8">
                    <div class="absolute left-1/2 -translate-x-1/2 -top-1 w-4 h-4 rounded-full bg-[#ef5675]"></div>
                    <div class="ml-auto md:ml-[55%] bg-slate-50 p-4 rounded-lg shadow-md w-full md:w-[45%]">
                        <h3 class="font-bold text-[#ef5675]">Stage 3: LiDAR Driver</h3>
                        <p class="text-sm text-slate-600">Compiles the D500 LiDAR driver for hardware integration.</p>
                    </div>
                </div>
                <div class="relative">
                    <div class="absolute left-1/2 -translate-x-1/2 -top-1 w-4 h-4 rounded-full bg-[#ff764a]"></div>
                    <div class="mr-auto md:mr-[55%] bg-slate-50 p-4 rounded-lg shadow-md w-full md:w-[45%]">
                        <h3 class="font-bold text-[#ff764a]">Stage 4: Final Runtime</h3>
                        <p class="text-sm text-slate-600">Copies only necessary binaries and configs for a minimal final image.</p>
                    </div>
                </div>
            </div>
        </section>
        
        <section class="mb-20">
            <h2 class="text-3xl font-bold text-center mb-2 text-[#003f5c]">Core Component Specifications</h2>
            <p class="text-center text-slate-500 mb-12 max-w-2xl mx-auto">Key parameters define the system's performance, from hardware capabilities to software resource allocation.</p>
            <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8">
                <div class="kpi-card">
                    <h3 class="text-xl font-bold text-[#003f5c] mb-2">D500 LiDAR Performance</h3>
                    <p class="text-slate-500 mb-4 text-sm">The sensor's range and scan rate are critical for accurate mapping.</p>
                    <div class="chart-container h-64 max-h-64">
                        <canvas id="lidarChart"></canvas>
                    </div>
                </div>
                <div class="kpi-card">
                    <h3 class="text-xl font-bold text-[#003f5c] mb-2">Container Resources</h3>
                    <p class="text-slate-500 mb-4 text-sm">Resource limits in `docker-compose.yml` ensure stable operation on the Raspberry Pi 5.</p>
                    <div class="space-y-4 pt-8">
                        <div>
                            <p class="text-5xl font-black text-[#ff764a]">6 GB</p>
                            <p class="text-slate-500">Memory Limit</p>
                        </div>
                        <div>
                            <p class="text-5xl font-black text-[#ffa600]">3.5</p>
                            <p class="text-slate-500">CPU Core Limit</p>
                        </div>
                    </div>
                </div>
                <div class="kpi-card md:col-span-2 lg:col-span-1">
                    <h3 class="text-xl font-bold text-[#003f5c] mb-2">Network Optimization</h3>
                    <p class="text-slate-500 mb-4 text-sm">CycloneDDS is configured for low-latency, real-time communication over WiFi.</p>
                    <div class="text-left space-y-4 pt-4">
                        <div class="bg-slate-100 p-3 rounded-md">
                            <p class="font-mono text-sm text-slate-500">Priority Class</p>
                            <p class="font-bold text-lg text-[#374c80]">RealTime</p>
                        </div>
                        <div class="bg-slate-100 p-3 rounded-md">
                            <p class="font-mono text-sm text-slate-500">Max Message Size</p>
                            <p class="font-bold text-lg text-[#374c80]">65536 bytes</p>
                        </div>
                         <div class="bg-slate-100 p-3 rounded-md">
                            <p class="font-mono text-sm text-slate-500">Fragment Size (MTU)</p>
                            <p class="font-bold text-lg text-[#374c80]">1400 bytes</p>
                        </div>
                    </div>
                </div>
            </div>
        </section>

        <section class="mb-20 bg-white p-8 rounded-xl shadow-lg">
            <h2 class="text-3xl font-bold text-center mb-2 text-[#003f5c]">Choosing Your SLAM Algorithm</h2>
            <p class="text-center text-slate-500 mb-12 max-w-2xl mx-auto">The system supports three distinct SLAM algorithms, each offering a different balance of performance and map quality.</p>
            <div class="chart-container h-96 max-h-96">
                <canvas id="slamComparisonChart"></canvas>
            </div>
            <div class="mt-8 grid grid-cols-1 md:grid-cols-3 gap-6 text-center">
                <div>
                    <h4 class="font-bold text-lg text-[#374c80]">Hector SLAM</h4>
                    <p class="text-sm text-slate-600">Fast and reliable. Ideal for initial testing and applications where speed is prioritized over absolute map perfection.</p>
                </div>
                <div>
                    <h4 class="font-bold text-lg text-[#bc5090]">GMapping</h4>
                    <p class="text-sm text-slate-600">A particle-filter based algorithm that provides higher accuracy. A good balance for most applications.</p>
                </div>
                <div>
                    <h4 class="font-bold text-lg text-[#ff764a]">Cartographer</h4>
                    <p class="text-sm text-slate-600">Google's graph-based SLAM provides the highest quality maps with loop closure, but demands the most CPU resources.</p>
                </div>
            </div>
        </section>

        <section class="mb-16">
            <h2 class="text-3xl font-bold text-center mb-2 text-[#003f5c]">Deployment & Operations Workflow</h2>
            <p class="text-center text-slate-500 mb-12 max-w-2xl mx-auto">A streamlined set of scripts simplifies the entire process from building the image to live monitoring.</p>
            <div class="grid grid-cols-2 md:grid-cols-4 gap-8 text-center">
                <div class="bg-white p-6 rounded-lg shadow-md">
                    <p class="text-5xl mb-3">🔨</p>
                    <h3 class="font-bold">1. Build</h3>
                    <p class="text-sm text-slate-500 font-mono">./scripts/build.sh</p>
                </div>
                <div class="bg-white p-6 rounded-lg shadow-md">
                    <p class="text-5xl mb-3">🚀</p>
                    <h3 class="font-bold">2. Deploy</h3>
                    <p class="text-sm text-slate-500 font-mono">./scripts/deploy.sh</p>
                </div>
                <div class="bg-white p-6 rounded-lg shadow-md">
                    <p class="text-5xl mb-3">📊</p>
                    <h3 class="font-bold">3. Monitor</h3>
                    <p class="text-sm text-slate-500 font-mono">./scripts/monitor.sh</p>
                </div>
                <div class="bg-white p-6 rounded-lg shadow-md">
                    <p class="text-5xl mb-3">👁️</p>
                    <h3 class="font-bold">4. Visualize</h3>
                    <p class="text-sm text-slate-500 font-mono">rviz2</p>
                </div>
            </div>
        </section>

        <footer class="text-center mt-20 pt-8 border-t border-slate-200">
            <p class="text-slate-500">This infographic visualizes the "Raspberry Pi 5 + D500 LiDAR SLAM System".</p>
            <p class="text-sm text-slate-400">Generated on Fri, 01 Aug 2025.</p>
        </footer>

    </div>

<script>
    const processLabel = (label) => {
        if (typeof label !== 'string' || label.length <= 16) {
            return label;
        }
        const words = label.split(' ');
        const lines = [];
        let currentLine = '';
        for (const word of words) {
            if ((currentLine + ' ' + word).trim().length > 16) {
                lines.push(currentLine.trim());
                currentLine = word;
            } else {
                currentLine = (currentLine + ' ' + word).trim();
            }
        }
        if (currentLine) {
            lines.push(currentLine.trim());
        }
        return lines;
    };

    const tooltipTitleCallback = (tooltipItems) => {
        const item = tooltipItems[0];
        let label = item.chart.data.labels[item.dataIndex];
        if (Array.isArray(label)) {
            return label.join(' ');
        }
        return label;
    };
    
    const commonChartOptions = {
        responsive: true,
        maintainAspectRatio: false,
        plugins: {
            legend: {
                labels: {
                    color: '#475569',
                    font: {
                        family: "'Inter', sans-serif"
                    }
                }
            },
            tooltip: {
                callbacks: {
                    title: tooltipTitleCallback
                }
            }
        },
        scales: {
            x: {
                ticks: {
                    color: '#475569',
                    font: {
                        family: "'Inter', sans-serif"
                    }
                },
                grid: {
                    color: '#e2e8f0'
                }
            },
            y: {
                ticks: {
                    color: '#475569',
                    font: {
                        family: "'Inter', sans-serif"
                    }
                },
                grid: {
                    color: '#e2e8f0'
                }
            }
        }
    };

    const lidarCtx = document.getElementById('lidarChart').getContext('2d');
    new Chart(lidarCtx, {
        type: 'doughnut',
        data: {
            labels: ['Max Range (12m)', 'Min Range (0.12m)'],
            datasets: [{
                label: 'LiDAR Range',
                data: [12, 0.12],
                backgroundColor: ['#ffa600', '#ff764a'],
                borderColor: '#ffffff',
                borderWidth: 4,
            }]
        },
        options: {
            responsive: true,
            maintainAspectRatio: false,
            plugins: {
                legend: {
                    position: 'bottom',
                    labels: {
                        color: '#475569',
                        font: { family: "'Inter', sans-serif" }
                    }
                },
                tooltip: {
                    callbacks: {
                        title: tooltipTitleCallback
                    }
                },
                title: {
                    display: true,
                    text: 'D500 Scan Frequency: 10 Hz',
                    color: '#003f5c',
                    font: { size: 16, weight: 'bold', family: "'Inter', sans-serif" }
                }
            },
            cutout: '60%',
        }
    });

    const slamCtx = document.getElementById('slamComparisonChart').getContext('2d');
    new Chart(slamCtx, {
        type: 'bar',
        data: {
            labels: ['Hector SLAM', 'GMapping', 'Cartographer'],
            datasets: [{
                label: 'Relative CPU Usage',
                data: [25, 60, 100],
                backgroundColor: '#ef5675',
                borderColor: '#bc5090',
                borderWidth: 2,
            }, {
                label: 'Estimated Map Quality',
                data: [50, 75, 100],
                backgroundColor: '#7a5195',
                borderColor: '#374c80',
                borderWidth: 2,
            }]
        },
        options: {
            ...commonChartOptions,
            indexAxis: 'y',
            scales: {
                x: {
                   ...commonChartOptions.scales.x,
                   max: 110,
                   title: {
                       display: true,
                       text: 'Relative Score',
                       color: '#003f5c'
                   }
                },
                y: {
                   ...commonChartOptions.scales.y,
                }
            }
        }
    });

</script>
</body>
</html>
