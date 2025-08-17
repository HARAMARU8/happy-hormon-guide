<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>インタラクティブ幸せホルモンガイド</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+JP:wght@400;500;700&display=swap" rel="stylesheet">
    <!-- Chosen Palette: Calm Harmony (Warm neutrals with soft, complementary accents for each hormone) -->
    <!-- Application Structure Plan: A single-page app with a main visual navigation panel for the four hormones. Selecting a hormone dynamically updates a central content area with its details and highlights its profile on an interactive radar chart. This card-based, interactive dashboard structure was chosen to break down the report's text into engaging, user-driven segments, promoting exploration and comparison over linear reading. -->
    <!-- Visualization & Content Choices: Report Info: Hormone roles & characteristics -> Goal: Compare hormone profiles -> Viz: Radar Chart (Chart.js/Canvas) -> Interaction: Highlighting the selected hormone's data series on the chart. Report Info: "How to Boost" methods -> Goal: Present actionable advice clearly -> Presentation: Grid of icon+text cards -> Interaction: Hover effects. This approach turns static lists into scannable, visually appealing elements. CONFIRMATION: NO SVG/Mermaid used. -->
    <!-- CONFIRMATION: NO SVG graphics used. NO Mermaid JS used. -->
    <style>
        body {
            font-family: 'Noto Sans JP', sans-serif;
            background-color: #FDFBF8;
            color: #4A4A4A;
        }
        .hormone-nav-card {
            transition: transform 0.3s ease, box-shadow 0.3s ease;
        }
        .hormone-nav-card:hover {
            transform: translateY(-5px);
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
        }
        .hormone-nav-card.active {
            box-shadow: 0 0 0 3px var(--tw-shadow-color);
        }
        .fade-in {
            animation: fadeIn 0.5s ease-in-out;
        }
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(10px); }
            to { opacity: 1; transform: translateY(0); }
        }
        .chart-container {
            position: relative;
            width: 100%;
            max-width: 500px;
            margin-left: auto;
            margin-right: auto;
            height: 300px;
            max-height: 400px;
        }
        @media (min-width: 768px) {
            .chart-container {
                height: 400px;
            }
        }
    </style>
</head>
<body class="antialiased">

    <div class="container mx-auto px-4 py-8 md:py-12">
        
        <header class="text-center mb-8 md:mb-12">
            <h1 class="text-3xl md:text-4xl font-bold text-stone-800 mb-2">インタラクティブ幸せホルモンガイド</h1>
            <p class="text-stone-600">4つの鍵で解き明かす、心と体の満たし方</p>
        </header>

        <section class="mb-8 md:mb-12 text-center max-w-3xl mx-auto">
            <p class="text-stone-700 leading-relaxed">
                私たちの幸福感は、脳内で働く4つの主要な化学物質、通称「幸せホルモン」に大きく影響されています。このガイドでは、それぞれのホルモンの役割や、日常生活で分泌を促す方法をインタラクティブに探求できます。下のカードから気になるホルモンを選んで、あなたのウェルビーイングを高めるヒントを見つけましょう。
            </p>
        </section>

        <nav id="hormone-nav" class="grid grid-cols-2 md:grid-cols-4 gap-4 md:gap-6 mb-8 md:mb-12">
        </nav>

        <main class="grid grid-cols-1 lg:grid-cols-5 gap-8">
            <div id="hormone-details" class="lg:col-span-3 order-2 lg:order-1">
            </div>
            <aside class="lg:col-span-2 order-1 lg:order-2">
                <div class="bg-white rounded-2xl shadow-lg p-4 md:p-6 sticky top-8">
                    <h3 class="text-xl font-bold text-stone-800 mb-4 text-center">ホルモンプロファイル比較</h3>
                    <div class="chart-container">
                        <canvas id="hormoneChart"></canvas>
                    </div>
                    <p class="text-xs text-stone-500 mt-4 text-center">各ホルモンがどの感情や感覚に強く影響するかを視覚的に比較します。</p>
                </div>
            </aside>
        </main>
        
        <footer class="text-center mt-12 text-stone-500 text-sm">
            <p>&copy; 2025 Interactive Hormone Guide. All rights reserved.</p>
        </footer>

    </div>

    <script>
        const hormoneData = {
            dopamine: {
                name: "ドーパミン",
                nickname: "やる気ホルモン",
                color: "#FFD700",
                shadowColor: "rgba(255, 215, 0, 0.5)",
                icon: "🚀",
                description: "目標達成時の喜びや快感を生み出し、モチベーションを高める。",
                details: {
                    role: {
                        title: "主な役割",
                        content: "意欲、達成感、快感、注意力、記憶力を司る「脳の報酬系」を活性化させます。何かを成し遂げた時の高揚感は、ドーパミンの働きによるものです。次の行動への強い原動力となります。"
                    },
                    deficiency: {
                        title: "不足すると…",
                        content: "意欲の低下、無気力、無感動、無関心といった状態に陥りやすくなります。ワクワク感や感動が薄れたと感じる場合、分泌量が低下している可能性があります。"
                    },
                    boost: {
                        title: "分泌を促す方法",
                        methods: [
                            { icon: "🎯", text: "小さな目標達成" },
                            { icon: "🎁", text: "自分へのご褒美" },
                            { icon: "💡", text: "新しいことへの挑戦" },
                            { icon: "💖", text: "楽しい活動に没頭" }
                        ]
                    }
                },
                chartData: [5, 2, 1, 3]
            },
            serotonin: {
                name: "セロトニン",
                nickname: "幸せホルモン",
                color: "#34D399",
                shadowColor: "rgba(52, 211, 153, 0.5)",
                icon: "😌",
                description: "精神を安定させ、穏やかな気持ちをもたらす心の司令塔。",
                details: {
                    role: {
                        title: "主な役割",
                        content: "精神を安定させ、感情の波をコントロールします。自律神経のバランスを整え、夜には睡眠ホルモン「メラトニン」の原料にもなるため、良質な睡眠にも不可欠です。"
                    },
                    deficiency: {
                        title: "不足すると…",
                        content: "精神的に不安定になり、イライラや不安感、気分の落ち込み、不眠などを引き起こしやすくなります。慢性的な疲労感を感じることもあります。"
                    },
                    boost: {
                        title: "分泌を促す方法",
                        methods: [
                            { icon: "☀️", text: "日光浴（特に朝日）" },
                            { icon: "🚶", text: "リズム運動" },
                            { icon: "🍌", text: "トリプトファン摂取" },
                            { icon: "🧘", text: "深呼吸や瞑想" }
                        ]
                    }
                },
                chartData: [2, 5, 3, 1]
            },
            oxytocin: {
                name: "オキシトシン",
                nickname: "愛情ホルモン",
                color: "#F472B6",
                shadowColor: "rgba(244, 114, 182, 0.5)",
                icon: "❤️",
                description: "人との繋がりや信頼感を深め、ストレスを和らげる。",
                details: {
                    role: {
                        title: "主な役割",
                        content: "他者への信頼感や共感性を高め、社会的な絆を深めます。「愛情ホルモン」とも呼ばれ、不安や恐怖を和らげ、ストレスを軽減する効果があります。"
                    },
                    deficiency: {
                        title: "不足すると…",
                        content: "不安感や緊張感、焦燥感が強くなることがあります。社会的なつながりが希薄だと感じやすくなる可能性も指摘されています。"
                    },
                    boost: {
                        title: "分泌を促す方法",
                        methods: [
                            { icon: "🤗", text: "スキンシップ" },
                            { icon: "🤝", text: "親しい人との交流" },
                            { icon: "🎁", text: "人に親切にする" },
                            { icon: "🐾", text: "ペットとの触れ合い" }
                        ]
                    }
                },
                chartData: [1, 4, 5, 2]
            },
            endorphin: {
                name: "β-エンドルフィン",
                nickname: "脳内麻薬",
                color: "#60A5FA",
                shadowColor: "rgba(96, 165, 250, 0.5)",
                icon: " euphoric",
                description: "強い多幸感や高揚感をもたらし、痛みを和らげる。",
                details: {
                    role: {
                        title: "主な役割",
                        content: "モルヒネの数倍とも言われる強力な鎮痛作用を持ち、気分を高揚させ幸福感や陶酔感をもたらします。「ランナーズハイ」はこのホルモンの働きによるものです。"
                    },
                    deficiency: {
                        title: "不足すると…",
                        content: "幸福感を感じにくくなったり、身体的・精神的な痛みに対して敏感になったりする可能性があります。ストレスへの耐性が低下することも考えられます。"
                    },
                    boost: {
                        title: "分泌を促す方法",
                        methods: [
                            { icon: "🏃", text: "適度な運動" },
                            { icon: "🌶️", text: "美味しい食事" },
                            { icon: "🎶", text: "好きな音楽を聴く" },
                            { icon: "🧖", text: "サウナや入浴" }
                        ]
                    }
                },
                chartData: [3, 1, 1, 5]
            }
        };

        const navContainer = document.getElementById('hormone-nav');
        const detailsContainer = document.getElementById('hormone-details');
        let activeHormone = 'dopamine';
        let hormoneChart;

        function renderNav() {
            let navHTML = '';
            for (const key in hormoneData) {
                const hormone = hormoneData[key];
                navHTML += `
                    <button 
                        class="hormone-nav-card text-left p-4 rounded-2xl cursor-pointer border-2 border-transparent" 
                        data-hormone="${key}"
                        style="background-color: ${hormone.color}20; --tw-shadow-color: ${hormone.color};"
                    >
                        <div class="text-4xl mb-2">${hormone.icon}</div>
                        <h3 class="font-bold text-lg text-stone-800">${hormone.name}</h3>
                        <p class="text-sm text-stone-600">${hormone.nickname}</p>
                    </button>
                `;
            }
            navContainer.innerHTML = navHTML;
        }

        function renderDetails(hormoneKey) {
            const hormone = hormoneData[hormoneKey];
            let boostMethodsHTML = '';
            hormone.details.boost.methods.forEach(method => {
                boostMethodsHTML += `
                    <div class="bg-stone-100 rounded-lg p-3 flex items-center">
                        <span class="text-2xl mr-3">${method.icon}</span>
                        <span class="text-stone-700">${method.text}</span>
                    </div>
                `;
            });

            const detailsHTML = `
                <div class="fade-in bg-white rounded-2xl shadow-lg p-6 md:p-8">
                    <div class="flex items-center mb-4">
                        <span class="text-5xl mr-4">${hormone.icon}</span>
                        <div>
                            <h2 class="text-3xl font-bold" style="color: ${hormone.color};">${hormone.name}</h2>
                            <p class="text-stone-600 font-semibold">${hormone.nickname}</p>
                        </div>
                    </div>
                    <p class="text-stone-700 mb-6">${hormone.description}</p>
                    
                    <div class="space-y-6">
                        <div>
                            <h4 class="font-bold text-lg text-stone-800 mb-2">${hormone.details.role.title}</h4>
                            <p class="text-stone-600 leading-relaxed">${hormone.details.role.content}</p>
                        </div>
                        <div>
                            <h4 class="font-bold text-lg text-stone-800 mb-2">${hormone.details.deficiency.title}</h4>
                            <p class="text-stone-600 leading-relaxed">${hormone.details.deficiency.content}</p>
                        </div>
                        <div>
                            <h4 class="font-bold text-lg text-stone-800 mb-2">${hormone.details.boost.title}</h4>
                            <div class="grid grid-cols-2 gap-3">
                                ${boostMethodsHTML}
                            </div>
                        </div>
                    </div>
                </div>
            `;
            detailsContainer.innerHTML = detailsHTML;
        }

        function updateActiveNav(hormoneKey) {
            document.querySelectorAll('.hormone-nav-card').forEach(card => {
                if (card.dataset.hormone === hormoneKey) {
                    card.classList.add('active');
                } else {
                    card.classList.remove('active');
                }
            });
        }
        
        function updateChart(hormoneKey) {
            if (!hormoneChart) return;
            
            hormoneChart.data.datasets.forEach((dataset, index) => {
                const key = Object.keys(hormoneData)[index];
                if (key === hormoneKey) {
                    dataset.borderWidth = 4;
                    dataset.pointRadius = 5;
                    dataset.pointBackgroundColor = hormoneData[key].color;
                } else {
                    dataset.borderWidth = 2;
                    dataset.pointRadius = 3;
                    dataset.pointBackgroundColor = '#ffffff';
                }
            });
            hormoneChart.update();
        }

        function handleNavClick(e) {
            const card = e.target.closest('.hormone-nav-card');
            if (card) {
                const hormoneKey = card.dataset.hormone;
                activeHormone = hormoneKey;
                renderDetails(hormoneKey);
                updateActiveNav(hormoneKey);
                updateChart(hormoneKey);
            }
        }

        function createChart() {
            const ctx = document.getElementById('hormoneChart').getContext('2d');
            const datasets = Object.keys(hormoneData).map(key => {
                const hormone = hormoneData[key];
                return {
                    label: hormone.name,
                    data: hormone.chartData,
                    borderColor: hormone.color,
                    backgroundColor: `${hormone.color}33`,
                    borderWidth: 2,
                    pointRadius: 3,
                    pointBackgroundColor: '#ffffff',
                    pointBorderColor: hormone.color,
                    pointHoverRadius: 7,
                };
            });

            hormoneChart = new Chart(ctx, {
                type: 'radar',
                data: {
                    labels: ['やる気', '癒し', '絆', '高揚感'],
                    datasets: datasets
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        legend: {
                            position: 'bottom',
                            labels: {
                                usePointStyle: true,
                                boxWidth: 8
                            }
                        },
                        tooltip: {
                            callbacks: {
                                label: function(context) {
                                    let label = context.dataset.label || '';
                                    if (label) {
                                        label += ': ';
                                    }
                                    if (context.parsed.r !== null) {
                                        label += context.parsed.r;
                                    }
                                    return label;
                                }
                            }
                        }
                    },
                    scales: {
                        r: {
                            angleLines: {
                                color: '#E5E7EB'
                            },
                            grid: {
                                color: '#E5E7EB'
                            },
                            pointLabels: {
                                font: {
                                    size: 12,
                                    weight: 'bold'
                                },
                                color: '#6B7280'
                            },
                            ticks: {
                                backdropColor: 'rgba(255, 255, 255, 0.75)',
                                stepSize: 1,
                                display: false
                            },
                            suggestedMin: 0,
                            suggestedMax: 5
                        }
                    }
                }
            });
        }

        document.addEventListener('DOMContentLoaded', () => {
            renderNav();
            createChart();
            
            renderDetails(activeHormone);
            updateActiveNav(activeHormone);
            updateChart(activeHormone);

            navContainer.addEventListener('click', handleNavClick);
        });
    </script>
</body>
</html>
