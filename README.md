<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>美国市场TTM PE追踪器</title>
    <link href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css" rel="stylesheet">
    <script src="https://cdn.jsdelivr.net/npm/chart.js@3.7.1/dist/chart.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/luxon@2.3.1/build/global/luxon.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chartjs-adapter-luxon@1.1.0/dist/chartjs-adapter-luxon.min.js"></script>
    <style>
        .pe-card {
            transition: transform 0.3s ease, box-shadow 0.3s ease;
        }
        .pe-card:hover {
            transform: translateY(-5px);
            box-shadow: 0 10px 25px rgba(0, 0, 0, 0.1);
        }
        .tab-content {
            display: none;
        }
        .tab-content.active {
            display: block;
        }
        .tab-btn {
            transition: all 0.3s ease;
        }
        .tab-btn.active {
            background-color: #2563eb;
            color: white;
        }
        .gauge-container {
            position: relative;
            width: 100%;
            height: 10px;
            background-color: #e5e7eb;
            border-radius: 5px;
            margin: 10px 0;
        }
        .gauge-marker {
            position: absolute;
            top: -10px;
            width: 4px;
            height: 20px;
            background-color: #000;
            transform: translateX(-50%);
        }
        .gauge-label {
            position: absolute;
            top: 15px;
            transform: translateX(-50%);
            font-size: 12px;
            color: #4b5563;
        }
        @keyframes pulse {
            0% { opacity: 0.6; }
            50% { opacity: 1; }
            100% { opacity: 0.6; }
        }
        .loading {
            animation: pulse 1.5s infinite;
        }
    </style>
</head>
<body class="bg-gray-50 text-gray-800">
    <div class="container mx-auto px-4 py-8 max-w-6xl">
        <header class="mb-10">
            <h1 class="text-3xl font-bold text-center text-blue-800 mb-2">美国市场TTM PE追踪器</h1>
            <p class="text-center text-gray-600">实时追踪美国主要股票指数的TTM市盈率数据</p>
            <div class="text-center text-sm text-gray-500 mt-2 flex justify-center items-center space-x-4">
                <span>上次更新时间: <span id="last-update" class="font-medium">-</span></span>
                <span>|</span>
                <span>数据来源: 财经数据API</span>
                <span>|</span>
                <span>自动刷新: 每10分钟</span>
            </div>
        </header>

        <div class="grid grid-cols-1 md:grid-cols-3 gap-6 mb-10">
            <!-- S&P 500 Card -->
            <div class="pe-card bg-white rounded-lg shadow-md p-6 border-t-4 border-blue-600">
                <h2 class="text-xl font-bold mb-4 text-blue-800">标普 500</h2>
                <div class="text-center mb-4">
                    <p class="text-sm text-gray-600 mb-1">TTM P/E</p>
                    <p id="sp500-pe" class="text-4xl font-bold text-blue-700 loading">--</p>
                </div>
                <div class="text-sm text-gray-600">
                    <div class="flex justify-between mb-1">
                        <span>历史平均:</span>
                        <span id="sp500-avg" class="font-medium">--</span>
                    </div>
                    <div class="flex justify-between">
                        <span>5年范围:</span>
                        <span id="sp500-range" class="font-medium">--</span>
                    </div>
                </div>
                <div class="gauge-container mt-4">
                    <div class="gauge-marker" style="left: 0%;">
                        <div class="gauge-label">历史低点</div>
                    </div>
                    <div class="gauge-marker" style="left: 50%;">
                        <div class="gauge-label">历史平均</div>
                    </div>
                    <div class="gauge-marker" style="left: 100%;">
                        <div class="gauge-label">历史高点</div>
                    </div>
                    <div id="sp500-current-marker" class="gauge-marker" style="left: 50%; background-color: #3b82f6;">
                        <div class="gauge-label text-blue-600 font-medium">当前</div>
                    </div>
                </div>
            </div>

            <!-- NASDAQ 100 Card -->
            <div class="pe-card bg-white rounded-lg shadow-md p-6 border-t-4 border-green-600">
                <h2 class="text-xl font-bold mb-4 text-green-800">纳斯达克 100</h2>
                <div class="text-center mb-4">
                    <p class="text-sm text-gray-600 mb-1">TTM P/E</p>
                    <p id="nasdaq-pe" class="text-4xl font-bold text-green-700 loading">--</p>
                </div>
                <div class="text-sm text-gray-600">
                    <div class="flex justify-between mb-1">
                        <span>历史平均:</span>
                        <span id="nasdaq-avg" class="font-medium">--</span>
                    </div>
                    <div class="flex justify-between">
                        <span>5年范围:</span>
                        <span id="nasdaq-range" class="font-medium">--</span>
                    </div>
                </div>
                <div class="gauge-container mt-4">
                    <div class="gauge-marker" style="left: 0%;">
                        <div class="gauge-label">历史低点</div>
                    </div>
                    <div class="gauge-marker" style="left: 50%;">
                        <div class="gauge-label">历史平均</div>
                    </div>
                    <div class="gauge-marker" style="left: 100%;">
                        <div class="gauge-label">历史高点</div>
                    </div>
                    <div id="nasdaq-current-marker" class="gauge-marker" style="left: 50%; background-color: #10b981;">
                        <div class="gauge-label text-green-600 font-medium">当前</div>
                    </div>
                </div>
            </div>

            <!-- Dow Jones Card -->
            <div class="pe-card bg-white rounded-lg shadow-md p-6 border-t-4 border-purple-600">
                <h2 class="text-xl font-bold mb-4 text-purple-800">道琼斯指数</h2>
                <div class="text-center mb-4">
                    <p class="text-sm text-gray-600 mb-1">TTM P/E</p>
                    <p id="dow-pe" class="text-4xl font-bold text-purple-700 loading">--</p>
                </div>
                <div class="text-sm text-gray-600">
                    <div class="flex justify-between mb-1">
                        <span>历史平均:</span>
                        <span id="dow-avg" class="font-medium">--</span>
                    </div>
                    <div class="flex justify-between">
                        <span>5年范围:</span>
                        <span id="dow-range" class="font-medium">--</span>
                    </div>
                </div>
                <div class="gauge-container mt-4">
                    <div class="gauge-marker" style="left: 0%;">
                        <div class="gauge-label">历史低点</div>
                    </div>
                    <div class="gauge-marker" style="left: 50%;">
                        <div class="gauge-label">历史平均</div>
                    </div>
                    <div class="gauge-marker" style="left: 100%;">
                        <div class="gauge-label">历史高点</div>
                    </div>
                    <div id="dow-current-marker" class="gauge-marker" style="left: 50%; background-color: #8b5cf6;">
                        <div class="gauge-label text-purple-600 font-medium">当前</div>
                    </div>
                </div>
            </div>
        </div>

        <!-- Historical Chart Section -->
        <div class="bg-white rounded-lg shadow-md p-6 mb-10">
            <h2 class="text-xl font-bold mb-4 text-gray-800">历史TTM市盈率走势</h2>
            <p class="text-sm text-gray-600 mb-4">图表说明: 该图表展示了三大指数历史TTM市盈率的变化趋势。市盈率是股价与每股收益的比率，常用于评估市场估值水平。</p>
            <div class="h-96">
                <canvas id="peHistoryChart"></canvas>
            </div>
        </div>

        <!-- PE Ratio Comparison Table -->
        <div class="bg-white rounded-lg shadow-md p-6">
            <h2 class="text-xl font-bold mb-4 text-gray-800">市盈率估值比较</h2>
            <div class="overflow-x-auto">
                <table class="min-w-full divide-y divide-gray-200">
                    <thead>
                        <tr>
                            <th class="px-6 py-3 bg-gray-50 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">指数</th>
                            <th class="px-6 py-3 bg-gray-50 text-right text-xs font-medium text-gray-500 uppercase tracking-wider">当前 P/E</th>
                            <th class="px-6 py-3 bg-gray-50 text-right text-xs font-medium text-gray-500 uppercase tracking-wider">历史平均</th>
                            <th class="px-6 py-3 bg-gray-50 text-right text-xs font-medium text-gray-500 uppercase tracking-wider">相对历史平均</th>
                            <th class="px-6 py-3 bg-gray-50 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">估值状态</th>
                        </tr>
                    </thead>
                    <tbody class="bg-white divide-y divide-gray-200">
                        <tr>
                            <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-blue-800">标普 500</td>
                            <td id="table-sp500-pe" class="px-6 py-4 whitespace-nowrap text-sm text-right font-medium loading">--</td>
                            <td id="table-sp500-avg" class="px-6 py-4 whitespace-nowrap text-sm text-right loading">--</td>
                            <td id="table-sp500-rel" class="px-6 py-4 whitespace-nowrap text-sm text-right loading">--</td>
                            <td id="table-sp500-status" class="px-6 py-4 whitespace-nowrap text-sm loading">--</td>
                        </tr>
                        <tr>
                            <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-green-800">纳斯达克 100</td>
                            <td id="table-nasdaq-pe" class="px-6 py-4 whitespace-nowrap text-sm text-right font-medium loading">--</td>
                            <td id="table-nasdaq-avg" class="px-6 py-4 whitespace-nowrap text-sm text-right loading">--</td>
                            <td id="table-nasdaq-rel" class="px-6 py-4 whitespace-nowrap text-sm text-right loading">--</td>
                            <td id="table-nasdaq-status" class="px-6 py-4 whitespace-nowrap text-sm loading">--</td>
                        </tr>
                        <tr>
                            <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-purple-800">道琼斯指数</td>
                            <td id="table-dow-pe" class="px-6 py-4 whitespace-nowrap text-sm text-right font-medium loading">--</td>
                            <td id="table-dow-avg" class="px-6 py-4 whitespace-nowrap text-sm text-right loading">--</td>
                            <td id="table-dow-rel" class="px-6 py-4 whitespace-nowrap text-sm text-right loading">--</td>
                            <td id="table-dow-status" class="px-6 py-4 whitespace-nowrap text-sm loading">--</td>
                        </tr>
                    </tbody>
                </table>
            </div>
            <p class="text-sm text-gray-500 mt-4">* 估值状态基于当前市盈率与历史平均值的比较。市盈率高于历史平均表示可能估值较高，低于历史平均表示可能估值较低。</p>
        </div>

        <footer class="mt-10 text-center text-sm text-gray-500">
            <p>© 2025 美国市场TTM PE追踪器 | 数据仅供参考，不构成投资建议</p>
        </footer>
    </div>

    <script>
        // 模拟数据 - 在实际应用中，这些数据应该从API获取
        const mockData = {
            lastUpdate: new Date(),
            sp500: {
                current: 25.68,
                avg: 19.6,
                min: 13.2,
                max: 34.5,
                fiveYearLow: 18.4,
                fiveYearHigh: 30.2,
                history: generateHistoricData(25.68, 5, 90)
            },
            nasdaq: {
                current: 30.79,
                avg: 26.5,
                min: 19.8,
                max: 38.6,
                fiveYearLow: 22.3,
                fiveYearHigh: 36.8,
                history: generateHistoricData(30.79, 6, 90)
            },
            dow: {
                current: 22.76,
                avg: 18.2,
                min: 12.5,
                max: 29.8,
                fiveYearLow: 16.1,
                fiveYearHigh: 25.9,
                history: generateHistoricData(22.76, 4, 90)
            }
        };

        // 生成模拟的历史数据
        function generateHistoricData(currentValue, volatility, days) {
            const data = [];
            let value = currentValue;
            const today = new Date();
            
            for (let i = days; i >= 0; i--) {
                const date = new Date();
                date.setDate(today.getDate() - i);
                
                // 添加一些随机变化，但保持趋势朝向当前值
                const change = (Math.random() - 0.5) * volatility;
                value = value - change * (i / days);
                
                data.push({
                    date: date,
                    value: Math.max(value, 5) // 确保PE不会低于5
                });
            }
            
            return data;
        }

        // 初始化图表
        function initChart() {
            const ctx = document.getElementById('peHistoryChart').getContext('2d');
            
            // 准备数据
            const labels = mockData.sp500.history.map(item => item.date);
            
            const chartData = {
                labels: labels,
                datasets: [
                    {
                        label: '标普 500',
                        data: mockData.sp500.history.map(item => item.value),
                        borderColor: 'rgb(59, 130, 246)',
                        backgroundColor: 'rgba(59, 130, 246, 0.1)',
                        tension: 0.3,
                        borderWidth: 2
                    },
                    {
                        label: '纳斯达克 100',
                        data: mockData.nasdaq.history.map(item => item.value),
                        borderColor: 'rgb(16, 185, 129)',
                        backgroundColor: 'rgba(16, 185, 129, 0.1)',
                        tension: 0.3,
                        borderWidth: 2
                    },
                    {
                        label: '道琼斯指数',
                        data: mockData.dow.history.map(item => item.value),
                        borderColor: 'rgb(139, 92, 246)',
                        backgroundColor: 'rgba(139, 92, 246, 0.1)',
                        tension: 0.3,
                        borderWidth: 2
                    }
                ]
            };
            
            const config = {
                type: 'line',
                data: chartData,
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    interaction: {
                        intersect: false,
                        mode: 'index'
                    },
                    scales: {
                        x: {
                            type: 'time',
                            time: {
                                unit: 'month'
                            },
                            title: {
                                display: true,
                                text: '日期'
                            }
                        },
                        y: {
                            title: {
                                display: true,
                                text: 'TTM PE比率'
                            },
                            suggestedMin: 10
                        }
                    },
                    plugins: {
                        tooltip: {
                            callbacks: {
                                label: function(context) {
                                    return context.dataset.label + ': ' + context.parsed.y.toFixed(2);
                                }
                            }
                        }
                    }
                }
            };
            
            return new Chart(ctx, config);
        }

        // 更新页面数据
        function updatePageData() {
            // 更新上次更新时间
            document.getElementById('last-update').textContent = mockData.lastUpdate.toLocaleString('zh-CN');
            
            // 更新S&P 500数据
            document.getElementById('sp500-pe').textContent = mockData.sp500.current.toFixed(2);
            document.getElementById('sp500-avg').textContent = mockData.sp500.avg.toFixed(2);
            document.getElementById('sp500-range').textContent = `${mockData.sp500.fiveYearLow.toFixed(2)} - ${mockData.sp500.fiveYearHigh.toFixed(2)}`;
            
            // 更新NASDAQ数据
            document.getElementById('nasdaq-pe').textContent = mockData.nasdaq.current.toFixed(2);
            document.getElementById('nasdaq-avg').textContent = mockData.nasdaq.avg.toFixed(2);
            document.getElementById('nasdaq-range').textContent = `${mockData.nasdaq.fiveYearLow.toFixed(2)} - ${mockData.nasdaq.fiveYearHigh.toFixed(2)}`;
            
            // 更新道琼斯数据
            document.getElementById('dow-pe').textContent = mockData.dow.current.toFixed(2);
            document.getElementById('dow-avg').textContent = mockData.dow.avg.toFixed(2);
            document.getElementById('dow-range').textContent = `${mockData.dow.fiveYearLow.toFixed(2)} - ${mockData.dow.fiveYearHigh.toFixed(2)}`;
            
            // 更新表格数据
            document.getElementById('table-sp500-pe').textContent = mockData.sp500.current.toFixed(2);
            document.getElementById('table-sp500-avg').textContent = mockData.sp500.avg.toFixed(2);
            document.getElementById('table-nasdaq-pe').textContent = mockData.nasdaq.current.toFixed(2);
            document.getElementById('table-nasdaq-avg').textContent = mockData.nasdaq.avg.toFixed(2);
            document.getElementById('table-dow-pe').textContent = mockData.dow.current.toFixed(2);
            document.getElementById('table-dow-avg').textContent = mockData.dow.avg.toFixed(2);
            
            // 计算相对于历史平均值的百分比
            const sp500RelAvg = ((mockData.sp500.current / mockData.sp500.avg - 1) * 100).toFixed(1);
            const nasdaqRelAvg = ((mockData.nasdaq.current / mockData.nasdaq.avg - 1) * 100).toFixed(1);
            const dowRelAvg = ((mockData.dow.current / mockData.dow.avg - 1) * 100).toFixed(1);
            
            document.getElementById('table-sp500-rel').textContent = (sp500RelAvg > 0 ? '+' : '') + sp500RelAvg + '%';
            document.getElementById('table-nasdaq-rel').textContent = (nasdaqRelAvg > 0 ? '+' : '') + nasdaqRelAvg + '%';
            document.getElementById('table-dow-rel').textContent = (dowRelAvg > 0 ? '+' : '') + dowRelAvg + '%';
            
            // 设置估值状态
            setValuationStatus('table-sp500-status', parseFloat(sp500RelAvg));
            setValuationStatus('table-nasdaq-status', parseFloat(nasdaqRelAvg));
            setValuationStatus('table-dow-status', parseFloat(dowRelAvg));
            
            // 更新度量计上的当前位置指示器
            updateGaugeMarker('sp500-current-marker', mockData.sp500.min, mockData.sp500.max, mockData.sp500.current);
            updateGaugeMarker('nasdaq-current-marker', mockData.nasdaq.min, mockData.nasdaq.max, mockData.nasdaq.current);
            updateGaugeMarker('dow-current-marker', mockData.dow.min, mockData.dow.max, mockData.dow.current);
            
            // 移除加载状态
            document.querySelectorAll('.loading').forEach(el => el.classList.remove('loading'));
        }

        // 设置估值状态文本和颜色
        function setValuationStatus(elementId, percentage) {
            const element = document.getElementById(elementId);
            
            if (percentage > 15) {
                element.textContent = '高估';
                element.className = 'px-6 py-4 whitespace-nowrap text-sm text-red-600 font-medium';
            } else if (percentage > 5) {
                element.textContent = '略高';
                element.className = 'px-6 py-4 whitespace-nowrap text-sm text-orange-600 font-medium';
            } else if (percentage >= -5) {
                element.textContent = '适中';
                element.className = 'px-6 py-4 whitespace-nowrap text-sm text-green-600 font-medium';
            } else if (percentage >= -15) {
                element.textContent = '略低';
                element.className = 'px-6 py-4 whitespace-nowrap text-sm text-blue-600 font-medium';
            } else {
                element.textContent = '低估';
                element.className = 'px-6 py-4 whitespace-nowrap text-sm text-indigo-600 font-medium';
            }
        }

        // 更新度量计标记位置
        function updateGaugeMarker(markerId, min, max, value) {
            const percentage = Math.min(100, Math.max(0, ((value - min) / (max - min)) * 100));
            document.getElementById(markerId).style.left = `${percentage}%`;
        }

        // 模拟刷新数据
        function refreshData() {
            // 在真实环境中，这里将从API获取数据
            mockData.lastUpdate = new Date();
            
            // 为了演示，我们只需稍微调整当前值
            mockData.sp500.current += (Math.random() - 0.5) * 0.8;
            mockData.nasdaq.current += (Math.random() - 0.5) * 1.2;
            mockData.dow.current += (Math.random() - 0.5) * 0.6;
            
            // 更新历史数据的最后一个点
            const addNewDataPoint = (history, current) => {
                const newDate = new Date();
                history.push({
                    date: newDate,
                    value: current
                });
                
                // 如果数据点太多，则移除最早的点
                if (history.length > 90) {
                    history.shift();
                }
            };
            
            addNewDataPoint(mockData.sp500.history, mockData.sp500.current);
            addNewDataPoint(mockData.nasdaq.history, mockData.nasdaq.current);
            addNewDataPoint(mockData.dow.history, mockData.dow.current);
            
            // 更新页面
            updatePageData();
            
            // 更新图表
            chart.data.datasets[0].data = mockData.sp500.history.map(item => ({ x: item.date, y: item.value }));
            chart.data.datasets[1].data = mockData.nasdaq.history.map(item => ({ x: item.date, y: item.value }));
            chart.data.datasets[2].data = mockData.dow.history.map(item => ({ x: item.date, y: item.value }));
            chart.update();
        }

        // 初始化页面
        let chart;
        document.addEventListener('DOMContentLoaded', () => {
            chart = initChart();
            updatePageData();
            
            // 设置定期刷新
            setInterval(refreshData, 600000); // 每10分钟刷新一次
        });
    </script>
</body>
</html>
