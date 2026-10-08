<!DOCTYPE html>
<html lang="en" class="h-full">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>QuizMaster Pro — Timed Quiz System</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/lucide@latest"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&family=JetBrains+Mono:wght@400;500;600&display=swap" rel="stylesheet">
    <script>
        tailwind.config = {
            darkMode: 'class',
            theme: {
                extend: {
                    fontFamily: {
                        sans: ['Inter', 'sans-serif'],
                        mono: ['JetBrains Mono', 'monospace'],
                    },
                    colors: {
                        brand: {
                            50: '#eef2ff',
                            100: '#e0e7ff',
                            500: '#6366f1',
                            600: '#4f46e5',
                            700: '#4338ca',
                            900: '#312e81',
                        }
                    }
                }
            }
        }
    </script>
    <style>
        ::-webkit-scrollbar {
            width: 6px;
            height: 6px;
        }
        ::-webkit-scrollbar-track {
            background: transparent;
        }
        ::-webkit-scrollbar-thumb {
            background: #cbd5e1;
            border-radius: 9999px;
        }
        .dark ::-webkit-scrollbar-thumb {
            background: #475569;
        }
        .pulse-warning {
            animation: pulse-red 1.2s infinite;
        }
        @keyframes pulse-red {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.5; }
        }
    </style>
</head>
<body class="bg-slate-50 dark:bg-slate-900 text-slate-800 dark:text-slate-100 font-sans antialiased h-full transition-colors duration-200">
    <div id="app" class="h-full flex flex-col">
        <!-- TOP NAVIGATION BAR -->
        <header class="h-16 border-b border-slate-200 dark:border-slate-800 bg-white/80 dark:bg-slate-900/80 backdrop-blur-md px-6 flex items-center justify-between z-30 sticky top-0">
            <div class="flex items-center gap-3">
                <div class="bg-indigo-600 text-white p-2 rounded-xl shadow-lg shadow-indigo-500/30 flex items-center justify-center">
                    <i data-lucide="brain-circuit" class="w-6 h-6"></i>
                </div>
                <div>
                    <span class="text-lg font-bold bg-gradient-to-r from-indigo-600 to-violet-600 dark:from-indigo-400 dark:to-violet-400 bg-clip-text text-transparent">QuizMaster Pro</span>
                    <span class="hidden sm:inline-block text-xs px-2 py-0.5 rounded-full bg-slate-100 dark:bg-slate-800 text-slate-500 ml-2 font-mono">v2.4.0</span>
                </div>
            </div>

            <!-- Header Right Actions -->
            <div class="flex items-center gap-4">
                <button id="themeToggleBtn" onclick="toggleTheme()" class="p-2.5 rounded-xl hover:bg-slate-100 dark:hover:bg-slate-800 text-slate-600 dark:text-slate-300 transition-colors">
                    <i data-lucide="sun" class="w-5 h-5 hidden dark:block"></i>
                    <i data-lucide="moon" class="w-5 h-5 block dark:hidden"></i>
                </button>

                <button onclick="toggleDbConsole()" class="flex items-center gap-2 px-3 py-1.5 rounded-xl bg-slate-100 dark:bg-slate-800 text-slate-700 dark:text-slate-300 hover:bg-slate-200 dark:hover:bg-slate-700 transition-colors text-xs font-mono border border-slate-200 dark:border-slate-700">
                    <i data-lucide="database" class="w-4 h-4 text-indigo-500"></i>
                    <span class="hidden md:inline">JDBC Log</span>
                    <span id="dbBadge" class="bg-indigo-600 text-white text-[10px] px-1.5 py-0.2 rounded-full font-sans font-semibold">0</span>
                </button>

                <div id="userHeaderProfile" class="flex items-center gap-3 border-l border-slate-200 dark:border-slate-800 pl-4">
                </div>
            </div>
        </header>

        <!-- MAIN LAYOUT WRAPPER -->
        <div class="flex-1 flex overflow-hidden relative">
            <!-- SIDEBAR -->
            <aside id="sidebar" class="w-64 border-r border-slate-200 dark:border-slate-800 bg-white dark:bg-slate-900 flex flex-col shrink-0 transition-all duration-300">
                <div class="p-4 flex flex-col gap-1 flex-1">
                    <div class="text-xs font-semibold text-slate-400 uppercase tracking-wider px-3 mb-2 font-mono">Navigation</div>
                    
                    <button onclick="switchTab('dashboard')" id="nav-dashboard" class="nav-item flex items-center gap-3 px-3 py-2.5 rounded-xl text-sm font-medium transition-colors text-indigo-600 bg-indigo-50 dark:bg-indigo-950/50 dark:text-indigo-400">
                        <i data-lucide="layout-dashboard" class="w-4 h-4"></i>
                        <span>Dashboard</span>
                    </button>

                    <button onclick="switchTab('exams')" id="nav-exams" class="nav-item flex items-center gap-3 px-3 py-2.5 rounded-xl text-sm font-medium text-slate-600 dark:text-slate-400 hover:bg-slate-100 dark:hover:bg-slate-800 transition-colors">
                        <i data-lucide="file-text" class="w-4 h-4"></i>
                        <span id="navExamsText">Available Exams</span>
                    </button>

                    <div id="teacherOnlyNav" class="hidden flex-col gap-1 mt-2">
                        <div class="text-xs font-semibold text-slate-400 uppercase tracking-wider px-3 my-2 font-mono">Teacher Tools</div>
                        <button onclick="switchTab('question-bank')" id="nav-question-bank" class="nav-item flex items-center gap-3 px-3 py-2.5 rounded-xl text-sm font-medium text-slate-600 dark:text-slate-400 hover:bg-slate-100 dark:hover:bg-slate-800 transition-colors">
                            <i data-lucide="database-zap" class="w-4 h-4"></i>
                            <span>Question Bank</span>
                        </button>
                        <button onclick="switchTab('analytics')" id="nav-analytics" class="nav-item flex items-center gap-3 px-3 py-2.5 rounded-xl text-sm font-medium text-slate-600 dark:text-slate-400 hover:bg-slate-100 dark:hover:bg-slate-800 transition-colors">
                            <i data-lucide="bar-chart-3" class="w-4 h-4"></i>
                            <span>Student Analytics</span>
                        </button>
                    </div>

                    <div id="studentOnlyNav" class="flex flex-col gap-1 mt-2">
                        <button onclick="switchTab('history')" id="nav-history" class="nav-item flex items-center gap-3 px-3 py-2.5 rounded-xl text-sm font-medium text-slate-600 dark:text-slate-400 hover:bg-slate-100 dark:hover:bg-slate-800 transition-colors">
                            <i data-lucide="history" class="w-4 h-4"></i>
                            <span>My History</span>
                        </button>
                    </div>
                </div>

                <div class="p-4 border-t border-slate-200 dark:border-slate-800">
                    <div class="p-3 bg-slate-100 dark:bg-slate-800/80 rounded-xl flex items-center justify-between">
                        <div class="flex items-center gap-2">
                            <span id="roleBadge" class="w-2.5 h-2.5 rounded-full bg-emerald-500"></span>
                            <span id="roleText" class="text-xs font-semibold text-slate-700 dark:text-slate-300">Student Mode</span>
                        </div>
                        <button onclick="toggleRoleSwitch()" class="text-xs text-indigo-600 dark:text-indigo-400 font-medium hover:underline">Switch</button>
                    </div>
                </div>
            </aside>

            <main class="flex-1 overflow-y-auto p-6 bg-slate-50 dark:bg-slate-900/50" id="mainContainer">
                <!-- VIEW 1: DASHBOARD HUB -->
                <div id="view-dashboard" class="space-y-6 max-w-7xl mx-auto">
                    <div class="bg-gradient-to-r from-indigo-600 via-indigo-700 to-violet-700 rounded-3xl p-8 text-white relative overflow-hidden shadow-xl">
                        <div class="absolute right-0 top-0 bottom-0 w-1/3 opacity-10 bg-[radial-gradient(#fff_1px,transparent_1px)] [background-size:16px_16px]"></div>
                        <div class="relative z-10 max-w-2xl">
                            <span class="px-3 py-1 bg-white/20 rounded-full text-xs font-semibold tracking-wide uppercase backdrop-blur-md">Welcome Back</span>
                            <h1 id="welcomeHeading" class="text-3xl sm:text-4xl font-extrabold mt-3 tracking-tight">Ready for today's challenge?</h1>
                            <p class="text-indigo-100 mt-2 text-sm sm:text-base leading-relaxed">
                                Experience real-time adaptive exams with auto-submission timers and deep analytics. Select a scheduled exam below to begin.
                            </p>
                        </div>
                    </div>

                    <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4">
                        <div class="bg-white dark:bg-slate-800 p-5 rounded-2xl border border-slate-200 dark:border-slate-700/60 shadow-sm flex items-center justify-between">
                            <div>
                                <p class="text-xs font-medium text-slate-500 dark:text-slate-400">Total Exams Taken</p>
                                <h3 id="statTotalExams" class="text-2xl font-bold mt-1 text-slate-800 dark:text-slate-100">12</h3>
                            </div>
                            <div class="p-3 bg-indigo-50 dark:bg-indigo-950/50 text-indigo-600 dark:text-indigo-400 rounded-xl">
                                <i data-lucide="award" class="w-6 h-6"></i>
                            </div>
                        </div>
                        <div class="bg-white dark:bg-slate-800 p-5 rounded-2xl border border-slate-200 dark:border-slate-700/60 shadow-sm flex items-center justify-between">
                            <div>
                                <p class="text-xs font-medium text-slate-500 dark:text-slate-400">Average Accuracy</p>
                                <h3 id="statAvgScore" class="text-2xl font-bold mt-1 text-slate-800 dark:text-slate-100">84.5%</h3>
                            </div>
                            <div class="p-3 bg-emerald-50 dark:bg-emerald-950/50 text-emerald-600 dark:text-emerald-400 rounded-xl">
                                <i data-lucide="trending-up" class="w-6 h-6"></i>
                            </div>
                        </div>
                        <div class="bg-white dark:bg-slate-800 p-5 rounded-2xl border border-slate-200 dark:border-slate-700/60 shadow-sm flex items-center justify-between">
                            <div>
                                <p class="text-xs font-medium text-slate-500 dark:text-slate-400">Question Bank Size</p>
                                <h3 id="statBankCount" class="text-2xl font-bold mt-1 text-slate-800 dark:text-slate-100">45</h3>
                            </div>
                            <div class="p-3 bg-amber-50 dark:bg-amber-950/50 text-amber-600 dark:text-amber-400 rounded-xl">
                                <i data-lucide="database" class="w-6 h-6"></i>
                            </div>
                        </div>
                        <div class="bg-white dark:bg-slate-800 p-5 rounded-2xl border border-slate-200 dark:border-slate-700/60 shadow-sm flex items-center justify-between">
                            <div>
                                <p class="text-xs font-medium text-slate-500 dark:text-slate-400">Avg Time / Question</p>
                                <h3 id="statAvgTime" class="text-2xl font-bold mt-1 text-slate-800 dark:text-slate-100">42s</h3>
                            </div>
                            <div class="p-3 bg-purple-50 dark:bg-purple-950/50 text-purple-600 dark:text-purple-400 rounded-xl">
                                <i data-lucide="clock" class="w-6 h-6"></i>
                            </div>
                        </div>
                    </div>

                    <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                        <div class="lg:col-span-2 bg-white dark:bg-slate-800 p-6 rounded-2xl border border-slate-200 dark:border-slate-700/60 shadow-sm">
                            <div class="flex items-center justify-between mb-4">
                                <h3 class="font-bold text-slate-800 dark:text-slate-100 flex items-center gap-2">
                                    <i data-lucide="activity" class="w-4 h-4 text-indigo-500"></i> Performance Trends
                                </h3>
                                <span class="text-xs font-mono text-slate-400">Last 5 Attempts</span>
                            </div>
                            <div class="h-64 relative">
                                <canvas id="performanceChart"></canvas>
                            </div>
                        </div>

                        <div class="bg-white dark:bg-slate-800 p-6 rounded-2xl border border-slate-200 dark:border-slate-700/60 shadow-sm flex flex-col">
                            <div class="flex items-center justify-between mb-4">
                                <h3 class="font-bold text-slate-800 dark:text-slate-100 flex items-center gap-2">
                                    <i data-lucide="zap" class="w-4 h-4 text-amber-500"></i> Active Quizzes
                                </h3>
                                <button onclick="switchTab('exams')" class="text-xs text-indigo-600 dark:text-indigo-400 font-semibold hover:underline">View All</button>
                            </div>
                            <div id="quickExamList" class="space-y-3 flex-1 overflow-y-auto pr-1"></div>
                        </div>
                    </div>
                </div>

                <!-- VIEW 2: EXAMS LIST -->
                <div id="view-exams" class="hidden space-y-6 max-w-7xl mx-auto">
                    <div class="flex flex-col sm:flex-row sm:items-center justify-between gap-4">
                        <div>
                            <h2 class="text-2xl font-bold text-slate-800 dark:text-slate-100">Available Quizzes & Exams</h2>
                            <p class="text-slate-500 dark:text-slate-400 text-sm">Select an exam to start or schedule a new one as an instructor.</p>
                        </div>
                        <button id="btnCreateExam" onclick="openCreateExamModal()" class="hidden px-4 py-2.5 bg-indigo-600 hover:bg-indigo-700 text-white font-medium rounded-xl shadow-lg shadow-indigo-500/20 text-sm flex items-center gap-2 transition-all">
                            <i data-lucide="plus-circle" class="w-4 h-4"></i> Create New Exam
                        </button>
                    </div>

                    <div id="examsGrid" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6"></div>
                </div>

                <!-- VIEW 3: QUESTION BANK MANAGER -->
                <div id="view-question-bank" class="hidden space-y-6 max-w-7xl mx-auto">
                    <div class="flex flex-col sm:flex-row sm:items-center justify-between gap-4">
                        <div>
                            <h2 class="text-2xl font-bold text-slate-800 dark:text-slate-100">Question Bank Manager</h2>
                            <p class="text-slate-500 dark:text-slate-400 text-sm">Add, filter, edit, and organize exam questions by topic & difficulty.</p>
                        </div>
                        <button onclick="openQuestionModal()" class="px-4 py-2.5 bg-indigo-600 hover:bg-indigo-700 text-white font-medium rounded-xl shadow-lg shadow-indigo-500/20 text-sm flex items-center gap-2 transition-all">
                            <i data-lucide="plus" class="w-4 h-4"></i> Add New Question
                        </button>
                    </div>

                    <div class="bg-white dark:bg-slate-800 p-4 rounded-2xl border border-slate-200 dark:border-slate-700/60 shadow-sm flex flex-wrap gap-4 items-center justify-between">
                        <div class="flex items-center gap-3 flex-1 min-w-[240px]">
                            <i data-lucide="search" class="w-4 h-4 text-slate-400"></i>
                            <input type="text" id="qbSearchInput" oninput="renderQuestionBank()" placeholder="Search question text or options..." class="w-full bg-slate-50 dark:bg-slate-900 text-sm border-0 focus:ring-2 focus:ring-indigo-500 rounded-xl px-3 py-2">
                        </div>
                        <div class="flex items-center gap-3">
                            <select id="qbFilterTopic" onchange="renderQuestionBank()" class="bg-slate-50 dark:bg-slate-900 text-sm border-0 focus:ring-2 focus:ring-indigo-500 rounded-xl px-3 py-2">
                                <option value="ALL">All Topics</option>
                                <option value="Computer Science">Computer Science</option>
                                <option value="Mathematics">Mathematics</option>
                                <option value="General Knowledge">General Knowledge</option>
                            </select>
                            <select id="qbFilterDifficulty" onchange="renderQuestionBank()" class="bg-slate-50 dark:bg-slate-900 text-sm border-0 focus:ring-2 focus:ring-indigo-500 rounded-xl px-3 py-2">
                                <option value="ALL">All Difficulties</option>
                                <option value="Easy">Easy</option>
                                <option value="Medium">Medium</option>
                                <option value="Hard">Hard</option>
                            </select>
                        </div>
                    </div>

                    <div id="questionBankList" class="space-y-4"></div>
                </div>

                <!-- VIEW 4: STUDENT ANALYTICS -->
                <div id="view-analytics" class="hidden space-y-6 max-w-7xl mx-auto">
                    <div>
                        <h2 class="text-2xl font-bold text-slate-800 dark:text-slate-100">Student Class Performance</h2>
                        <p class="text-slate-500 dark:text-slate-400 text-sm">Aggregated results across all test takers.</p>
                    </div>

                    <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                        <div class="bg-white dark:bg-slate-800 p-6 rounded-2xl border border-slate-200 dark:border-slate-700/60 shadow-sm">
                            <h3 class="font-bold text-slate-800 dark:text-slate-100 mb-4">Topic Mastery Breakdown</h3>
                            <div class="h-64">
                                <canvas id="topicMasteryChart"></canvas>
                            </div>
                        </div>
                        <div class="bg-white dark:bg-slate-800 p-6 rounded-2xl border border-slate-200 dark:border-slate-700/60 shadow-sm">
                            <h3 class="font-bold text-slate-800 dark:text-slate-100 mb-4">Pass / Fail Ratio</h3>
                            <div class="h-64 flex items-center justify-center">
                                <canvas id="passFailChart"></canvas>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- VIEW 5: HISTORY -->
                <div id="view-history" class="hidden space-y-6 max-w-7xl mx-auto">
                    <div>
                        <h2 class="text-2xl font-bold text-slate-800 dark:text-slate-100">Attempt History</h2>
                        <p class="text-slate-500 dark:text-slate-400 text-sm">Review past quiz attempts, detailed explanations, and timestamps.</p>
                    </div>

                    <div id="historyTableContainer" class="bg-white dark:bg-slate-800 rounded-2xl border border-slate-200 dark:border-slate-700/60 overflow-hidden shadow-sm"></div>
                </div>

                <!-- VIEW 6: LIVE TIMED QUIZ ENGINE -->
                <div id="view-quiz-engine" class="hidden max-w-6xl mx-auto h-[calc(100vh-7rem)] flex flex-col gap-4">
                    <div class="bg-white dark:bg-slate-800 p-4 rounded-2xl border border-slate-200 dark:border-slate-700/60 shadow-sm flex flex-col sm:flex-row items-center justify-between gap-4">
                        <div class="flex items-center gap-4">
                            <div>
                                <h3 id="quizTitle" class="font-bold text-slate-800 dark:text-slate-100 text-base">Computer Science Fundamentals</h3>
                                <span id="quizQuestionCountBadge" class="text-xs font-mono text-slate-400">Question 1 of 10</span>
                            </div>
                        </div>

                        <div id="timerContainer" class="flex items-center gap-3 px-5 py-2.5 bg-slate-100 dark:bg-slate-900 border border-slate-200 dark:border-slate-700 rounded-2xl transition-colors">
                            <i data-lucide="clock" id="timerIcon" class="w-5 h-5 text-indigo-600 dark:text-indigo-400"></i>
                            <div class="flex flex-col">
                                <span class="text-[10px] font-semibold uppercase tracking-wider text-slate-400">Time Remaining</span>
                                <span id="timerDisplay" class="font-mono text-xl font-extrabold text-slate-800 dark:text-slate-100 tracking-wider">10:00</span>
                            </div>
                        </div>

                        <button onclick="confirmSubmitQuiz()" class="px-5 py-2.5 bg-emerald-600 hover:bg-emerald-700 text-white font-medium text-sm rounded-xl shadow-lg shadow-emerald-500/20 flex items-center gap-2 transition-all">
                            <i data-lucide="check-circle" class="w-4 h-4"></i> Submit Exam
                        </button>
                    </div>

                    <div class="flex-1 grid grid-cols-1 lg:grid-cols-4 gap-4 overflow-hidden">
                        <div class="lg:col-span-3 bg-white dark:bg-slate-800 rounded-2xl border border-slate-200 dark:border-slate-700/60 p-6 flex flex-col justify-between overflow-y-auto">
                            <div>
                                <div class="flex items-center justify-between mb-4">
                                    <div class="flex items-center gap-2">
                                        <span id="questionCategoryTag" class="px-2.5 py-1 bg-indigo-50 dark:bg-indigo-950/60 text-indigo-600 dark:text-indigo-400 text-xs font-semibold rounded-lg">Computer Science</span>
                                        <span id="questionDifficultyTag" class="px-2.5 py-1 bg-amber-50 dark:bg-amber-950/60 text-amber-600 dark:text-amber-400 text-xs font-semibold rounded-lg">Medium</span>
                                    </div>
                                    <button id="flagBtn" onclick="toggleFlagCurrentQuestion()" class="flex items-center gap-1.5 text-xs font-medium px-3 py-1.5 rounded-lg border border-slate-200 dark:border-slate-700 text-slate-600 dark:text-slate-400 hover:bg-amber-50 hover:text-amber-600 dark:hover:bg-amber-950/40 transition-colors">
                                        <i data-lucide="bookmark" class="w-3.5 h-3.5"></i>
                                        <span id="flagText">Flag for Review</span>
                                    </button>
                                </div>

                                <h2 id="questionText" class="text-lg sm:text-xl font-bold text-slate-800 dark:text-slate-100 mb-6 leading-relaxed">Loading...</h2>
                                <div id="optionsContainer" class="space-y-3"></div>
                            </div>

                            <div class="flex items-center justify-between pt-6 mt-6 border-t border-slate-100 dark:border-slate-700/50">
                                <button id="btnPrevQuestion" onclick="navigateQuestion(-1)" class="px-4 py-2 bg-slate-100 dark:bg-slate-700 text-slate-700 dark:text-slate-200 text-sm font-medium rounded-xl hover:bg-slate-200 dark:hover:bg-slate-600 transition-colors flex items-center gap-2">
                                    <i data-lucide="chevron-left" class="w-4 h-4"></i> Previous
                                </button>
                                <button onclick="clearCurrentResponse()" class="text-xs text-slate-400 hover:text-slate-600 dark:hover:text-slate-200 underline">Clear Choice</button>
                                <button id="btnNextQuestion" onclick="navigateQuestion(1)" class="px-5 py-2 bg-indigo-600 text-white text-sm font-medium rounded-xl hover:bg-indigo-700 transition-colors flex items-center gap-2">
                                    Next <i data-lucide="chevron-right" class="w-4 h-4"></i>
                                </button>
                            </div>
                        </div>

                        <div class="bg-white dark:bg-slate-800 rounded-2xl border border-slate-200 dark:border-slate-700/60 p-5 flex flex-col justify-between">
                            <div>
                                <h4 class="font-bold text-sm text-slate-800 dark:text-slate-100 mb-3 flex items-center gap-2">
                                    <i data-lucide="grid" class="w-4 h-4 text-indigo-500"></i> Question Palette
                                </h4>
                                
                                <div class="grid grid-cols-2 gap-2 text-[11px] text-slate-500 dark:text-slate-400 mb-4 pb-3 border-b border-slate-100 dark:border-slate-700/60">
                                    <div class="flex items-center gap-1.5"><span class="w-2.5 h-2.5 rounded-full bg-emerald-500"></span> Answered</div>
                                    <div class="flex items-center gap-1.5"><span class="w-2.5 h-2.5 rounded-full bg-amber-500"></span> Flagged</div>
                                    <div class="flex items-center gap-1.5"><span class="w-2.5 h-2.5 rounded-full bg-indigo-600"></span> Current</div>
                                    <div class="flex items-center gap-1.5"><span class="w-2.5 h-2.5 rounded-full bg-slate-200 dark:bg-slate-700"></span> Unvisited</div>
                                </div>

                                <div id="paletteGrid" class="grid grid-cols-4 gap-2 max-h-56 overflow-y-auto pr-1"></div>
                            </div>

                            <div class="mt-4 pt-3 border-t border-slate-100 dark:border-slate-700/60 text-xs text-slate-500 space-y-1 font-mono">
                                <div class="flex justify-between"><span>Answered:</span><span id="palAnsweredCount" class="font-semibold text-slate-800 dark:text-slate-200">0</span></div>
                                <div class="flex justify-between"><span>Flagged:</span><span id="palFlaggedCount" class="font-semibold text-slate-800 dark:text-slate-200">0</span></div>
                                <div class="flex justify-between"><span>Remaining:</span><span id="palUnansweredCount" class="font-semibold text-slate-800 dark:text-slate-200">0</span></div>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- VIEW 7: RESULTS & BREAKDOWN ENGINE -->
                <div id="view-results" class="hidden max-w-5xl mx-auto space-y-6">
                    <div id="resultBanner" class="bg-white dark:bg-slate-800 p-8 rounded-3xl border border-slate-200 dark:border-slate-700/60 shadow-xl text-center relative overflow-hidden">
                        <div id="resultStatusBadge" class="inline-flex items-center gap-2 px-4 py-1.5 rounded-full text-xs font-bold uppercase tracking-wider mb-4"></div>

                        <h2 class="text-3xl font-extrabold text-slate-800 dark:text-slate-100" id="resultTitle">Quiz Completed!</h2>
                        <p id="resultSubtitle" class="text-slate-500 dark:text-slate-400 text-sm mt-1">Here is your detailed diagnostic analysis.</p>

                        <div class="grid grid-cols-2 md:grid-cols-4 gap-4 mt-6 pt-6 border-t border-slate-100 dark:border-slate-700/60">
                            <div>
                                <span class="text-xs text-slate-400 font-medium">Final Score</span>
                                <h3 id="resScoreText" class="text-3xl font-extrabold text-indigo-600 dark:text-indigo-400 mt-1">80%</h3>
                            </div>
                            <div>
                                <span class="text-xs text-slate-400 font-medium">Correct Answers</span>
                                <h3 id="resCorrectText" class="text-3xl font-extrabold text-emerald-500 mt-1">8 / 10</h3>
                            </div>
                            <div>
                                <span class="text-xs text-slate-400 font-medium">Time Taken</span>
                                <h3 id="resTimeText" class="text-3xl font-extrabold text-amber-500 mt-1">04:22</h3>
                            </div>
                            <div>
                                <span class="text-xs text-slate-400 font-medium">Accuracy</span>
                                <h3 id="resAccuracyText" class="text-3xl font-extrabold text-purple-500 mt-1">80.0%</h3>
                            </div>
                        </div>
                    </div>

                    <div class="bg-white dark:bg-slate-800 p-6 rounded-2xl border border-slate-200 dark:border-slate-700/60 shadow-sm">
                        <h3 class="font-bold text-lg text-slate-800 dark:text-slate-100 mb-4 flex items-center gap-2">
                            <i data-lucide="list-checks" class="w-5 h-5 text-indigo-500"></i> Per-Question Review
                        </h3>
                        <div id="resultBreakdownList" class="space-y-4"></div>
                    </div>

                    <div class="flex justify-end gap-3">
                        <button onclick="switchTab('dashboard')" class="px-6 py-2.5 bg-indigo-600 hover:bg-indigo-700 text-white font-semibold text-sm rounded-xl shadow-lg shadow-indigo-500/20 transition-all">
                            Back to Dashboard
                        </button>
                    </div>
                </div>
            </main>
        </div>
    </div>

    <!-- JDBC SIMULATION LOG DRAWER -->
    <div id="dbConsoleDrawer" class="fixed inset-y-0 right-0 w-full max-w-md bg-slate-950 text-slate-200 shadow-2xl z-50 transform translate-x-full transition-transform duration-300 ease-in-out border-l border-slate-800 flex flex-col font-mono text-xs">
        <div class="p-4 bg-slate-900 border-b border-slate-800 flex items-center justify-between">
            <div class="flex items-center gap-2">
                <i data-lucide="database" class="w-4 h-4 text-emerald-400"></i>
                <span class="font-bold text-slate-100">JDBC Executed Queries Log</span>
            </div>
            <button onclick="toggleDbConsole()" class="text-slate-400 hover:text-white"><i data-lucide="x" class="w-5 h-5"></i></button>
        </div>
        <div id="dbLogContainer" class="flex-1 p-4 overflow-y-auto space-y-3"></div>
        <div class="p-3 bg-slate-900 border-t border-slate-800 flex items-center justify-between text-[11px] text-slate-400">
            <span>Database: <strong class="text-emerald-400">QuizMaster_ProdDB</strong></span>
            <button onclick="clearDbLogs()" class="text-rose-400 hover:underline">Clear Logs</button>
        </div>
    </div>

    <!-- INSTRUCTION MODAL -->
    <div id="instructionModal" class="fixed inset-0 bg-slate-900/60 backdrop-blur-sm z-50 hidden flex items-center justify-center p-4">
        <div class="bg-white dark:bg-slate-800 rounded-3xl max-w-lg w-full p-6 shadow-2xl border border-slate-200 dark:border-slate-700">
            <div class="flex items-center gap-3 text-indigo-600 dark:text-indigo-400 mb-3">
                <i data-lucide="shield-alert" class="w-6 h-6"></i>
                <h3 class="text-xl font-bold text-slate-800 dark:text-slate-100">Exam Instructions</h3>
            </div>
            <p id="modalExamTitle" class="text-sm font-semibold text-slate-600 dark:text-slate-300 mb-4">Exam Session</p>

            <div class="space-y-3 text-xs text-slate-600 dark:text-slate-300 bg-slate-50 dark:bg-slate-900/80 p-4 rounded-xl border border-slate-200 dark:border-slate-700/60">
                <div class="flex items-start gap-2">
                    <i data-lucide="clock" class="w-4 h-4 text-indigo-500 shrink-0 mt-0.5"></i>
                    <span><strong>Timer Rule:</strong> Upon expiration, your answers will be <strong>automatically submitted</strong>.</span>
                </div>
                <div class="flex items-start gap-2">
                    <i data-lucide="shuffle" class="w-4 h-4 text-indigo-500 shrink-0 mt-0.5"></i>
                    <span><strong>Randomization:</strong> Questions and options are randomly shuffled.</span>
                </div>
            </div>

            <div class="mt-6 flex justify-end gap-3">
                <button onclick="closeInstructionModal()" class="px-4 py-2 text-sm text-slate-600 dark:text-slate-300 hover:bg-slate-100 dark:hover:bg-slate-700 rounded-xl">Cancel</button>
                <button onclick="launchQuizConfirmed()" class="px-5 py-2 text-sm bg-indigo-600 hover:bg-indigo-700 text-white font-semibold rounded-xl shadow-lg flex items-center gap-2">
                    Start Test Now <i data-lucide="arrow-right" class="w-4 h-4"></i>
                </button>
            </div>
        </div>
    </div>

    <!-- CREATE/EDIT QUESTION MODAL -->
    <div id="questionModal" class="fixed inset-0 bg-slate-900/60 backdrop-blur-sm z-50 hidden flex items-center justify-center p-4">
        <div class="bg-white dark:bg-slate-800 rounded-3xl max-w-xl w-full p-6 shadow-2xl border border-slate-200 dark:border-slate-700 max-h-[90vh] overflow-y-auto">
            <h3 id="questionModalTitle" class="text-xl font-bold text-slate-800 dark:text-slate-100 mb-4">Add Question to Bank</h3>
            <form id="questionForm" onsubmit="handleSaveQuestion(event)" class="space-y-4 text-sm">
                <input type="hidden" id="qEditId">
                <div>
                    <label class="block text-xs font-semibold text-slate-600 dark:text-slate-300 mb-1">Question Statement</label>
                    <textarea id="qInputText" required rows="3" class="w-full bg-slate-50 dark:bg-slate-900 border border-slate-200 dark:border-slate-700 rounded-xl p-3 focus:ring-2 focus:ring-indigo-500"></textarea>
                </div>

                <div class="grid grid-cols-2 gap-4">
                    <div>
                        <label class="block text-xs font-semibold text-slate-600 dark:text-slate-300 mb-1">Topic Tag</label>
                        <select id="qInputTopic" class="w-full bg-slate-50 dark:bg-slate-900 border border-slate-200 dark:border-slate-700 rounded-xl p-2.5">
                            <option value="Computer Science">Computer Science</option>
                            <option value="Mathematics">Mathematics</option>
                            <option value="General Knowledge">General Knowledge</option>
                        </select>
                    </div>
                    <div>
                        <label class="block text-xs font-semibold text-slate-600 dark:text-slate-300 mb-1">Difficulty</label>
                        <select id="qInputDifficulty" class="w-full bg-slate-50 dark:bg-slate-900 border border-slate-200 dark:border-slate-700 rounded-xl p-2.5">
                            <option value="Easy">Easy</option>
                            <option value="Medium">Medium</option>
                            <option value="Hard">Hard</option>
                        </select>
                    </div>
                </div>

                <div class="space-y-2">
                    <label class="block text-xs font-semibold text-slate-600 dark:text-slate-300">Answer Options & Correct Selection</label>
                    <div class="flex items-center gap-2"><input type="radio" name="correctOpt" value="0" checked><input type="text" id="opt0" required placeholder="Option 1" class="flex-1 bg-slate-50 dark:bg-slate-900 border p-2 rounded-xl text-xs"></div>
                    <div class="flex items-center gap-2"><input type="radio" name="correctOpt" value="1"><input type="text" id="opt1" required placeholder="Option 2" class="flex-1 bg-slate-50 dark:bg-slate-900 border p-2 rounded-xl text-xs"></div>
                    <div class="flex items-center gap-2"><input type="radio" name="correctOpt" value="2"><input type="text" id="opt2" required placeholder="Option 3" class="flex-1 bg-slate-50 dark:bg-slate-900 border p-2 rounded-xl text-xs"></div>
                    <div class="flex items-center gap-2"><input type="radio" name="correctOpt" value="3"><input type="text" id="opt3" required placeholder="Option 4" class="flex-1 bg-slate-50 dark:bg-slate-900 border p-2 rounded-xl text-xs"></div>
                </div>

                <div>
                    <label class="block text-xs font-semibold text-slate-600 dark:text-slate-300 mb-1">Explanation</label>
                    <textarea id="qInputExplanation" rows="2" placeholder="Explanation for correct answer..." class="w-full bg-slate-50 dark:bg-slate-900 border border-slate-200 dark:border-slate-700 rounded-xl p-3 focus:ring-2 focus:ring-indigo-500"></textarea>
                </div>

                <div class="flex justify-end gap-3 pt-2">
                    <button type="button" onclick="closeQuestionModal()" class="px-4 py-2 text-slate-500 hover:bg-slate-100 rounded-xl">Cancel</button>
                    <button type="submit" class="px-5 py-2 bg-indigo-600 hover:bg-indigo-700 text-white font-medium rounded-xl shadow-lg">Save Question</button>
                </div>
            </form>
        </div>
    </div>

    <script>
        const state = {
            currentUser: { id: 'STU-102', name: 'Alex Johnson', role: 'STUDENT', avatar: 'https://images.unsplash.com/photo-1534528741775-53994a69daeb?w=100&auto=format&fit=crop&q=80' },
            currentTab: 'dashboard',
            isDarkMode: false,
            dbLogs: [],
            questionBank: [
                { id: 'Q101', text: 'Which data structure follows the First In First Out (FIFO) principle?', topic: 'Computer Science', difficulty: 'Easy', options: ['Stack', 'Queue', 'Binary Tree', 'Graph'], correct: 1, explanation: 'Queue follows FIFO order.' },
                { id: 'Q102', text: 'What is the time complexity of searching an element in a balanced Binary Search Tree?', topic: 'Computer Science', difficulty: 'Medium', options: ['O(1)', 'O(n)', 'O(log n)', 'O(n log n)'], correct: 2, explanation: 'In balanced BST, tree height is log n.' },
                { id: 'Q103', text: 'Solve for x: 2x + 5 = 15', topic: 'Mathematics', difficulty: 'Easy', options: ['x = 3', 'x = 5', 'x = 10', 'x = 7'], correct: 1, explanation: '2x = 10 => x = 5.' },
                { id: 'Q104', text: 'Which SQL keyword is used to retrieve distinct values only?', topic: 'Computer Science', difficulty: 'Easy', options: ['DIFFERENT', 'UNIQUE', 'DISTINCT', 'FILTER'], correct: 2, explanation: 'SELECT DISTINCT filters unique rows.' }
            ],
            exams: [
                { id: 'EXAM-01', title: 'CS Fundamentals & Data Structures', durationMinutes: 5, totalQuestions: 3, topic: 'Computer Science', difficulty: 'Medium', questionIds: ['Q101', 'Q102', 'Q104'] },
                { id: 'EXAM-02', title: 'Algebra Speed Test', durationMinutes: 3, totalQuestions: 1, topic: 'Mathematics', difficulty: 'Easy', questionIds: ['Q103'] }
            ],
            history: [
                { attemptId: 'ATT-8821', examTitle: 'CS Fundamentals & Data Structures', date: '2026-10-02', scorePercent: 100, timeTaken: '02:15', passed: true }
            ],
            activeQuiz: null
        };

        window.onload = function() {
            renderHeaderUser();
            logDbQuery("SELECT * FROM users WHERE user_id = 'STU-102';");
            renderDashboard();
            renderExamsList();
            renderQuestionBank();
            renderHistoryTable();
            lucide.createIcons();
        };

        function toggleTheme() {
            document.documentElement.classList.toggle('dark');
            state.isDarkMode = !state.isDarkMode;
        }

        function toggleRoleSwitch() {
            state.currentUser.role = state.currentUser.role === 'STUDENT' ? 'TEACHER' : 'STUDENT';
            state.currentUser.name = state.currentUser.role === 'TEACHER' ? 'Dr. Robert Ford' : 'Alex Johnson';
            renderHeaderUser();
            updateRoleNavUI();
            logDbQuery(`UPDATE users SET active_role = '${state.currentUser.role}' WHERE user_id = '${state.currentUser.id}';`);
        }

        function renderHeaderUser() {
            document.getElementById('userHeaderProfile').innerHTML = `
                <img src="${state.currentUser.avatar}" class="w-8 h-8 rounded-full border border-indigo-500 object-cover">
                <div class="hidden sm:block text-left">
                    <p class="text-xs font-semibold text-slate-800 dark:text-slate-100">${state.currentUser.name}</p>
                    <p class="text-[10px] text-slate-400 font-mono">${state.currentUser.role}</p>
                </div>
            `;
        }

        function updateRoleNavUI() {
            const isTeacher = state.currentUser.role === 'TEACHER';
            document.getElementById('teacherOnlyNav').style.display = isTeacher ? 'flex' : 'none';
            document.getElementById('studentOnlyNav').style.display = isTeacher ? 'none' : 'flex';
            document.getElementById('roleText').innerText = isTeacher ? 'Teacher / Admin' : 'Student Mode';
            document.getElementById('roleBadge').className = isTeacher ? 'w-2.5 h-2.5 rounded-full bg-indigo-500' : 'w-2.5 h-2.5 rounded-full bg-emerald-500';
        }

        function switchTab(tabId) {
            ['dashboard', 'exams', 'question-bank', 'analytics', 'history', 'quiz-engine', 'results'].forEach(v => {
                const el = document.getElementById(`view-${v}`);
                if (el) el.classList.add('hidden');
            });

            const activeView = document.getElementById(`view-${tabId}`);
            if (activeView) activeView.classList.remove('hidden');

            document.querySelectorAll('.nav-item').forEach(btn => btn.className = 'nav-item flex items-center gap-3 px-3 py-2.5 rounded-xl text-sm font-medium text-slate-600 dark:text-slate-400 hover:bg-slate-100 dark:hover:bg-slate-800 transition-colors');
            const activeNav = document.getElementById(`nav-${tabId}`);
            if (activeNav) activeNav.className = 'nav-item flex items-center gap-3 px-3 py-2.5 rounded-xl text-sm font-medium text-indigo-600 bg-indigo-50 dark:bg-indigo-950/50 dark:text-indigo-400';

            if (tabId === 'dashboard') initCharts();
            if (tabId === 'analytics') initTeacherCharts();
        }

        function logDbQuery(sqlString) {
            const timestamp = new Date().toLocaleTimeString();
            state.dbLogs.unshift({ timestamp, sql: sqlString });
            document.getElementById('dbBadge').innerText = state.dbLogs.length;

            const container = document.getElementById('dbLogContainer');
            if (container) {
                container.innerHTML = state.dbLogs.map(log => `
                    <div class="p-2.5 rounded bg-slate-900 border border-slate-800 text-[11px] font-mono">
                        <span class="text-slate-500">[${log.timestamp}] JDBC:</span>
                        <div class="text-emerald-400 mt-1 break-all">${escapeHtml(log.sql)}</div>
                    </div>
                `).join('');
            }
        }

        function toggleDbConsole() { document.getElementById('dbConsoleDrawer').classList.toggle('translate-x-full'); }
        function clearDbLogs() { state.dbLogs = []; document.getElementById('dbBadge').innerText = '0'; document.getElementById('dbLogContainer').innerHTML = ''; }
        function escapeHtml(str) { return str.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;"); }

        function renderDashboard() {
            document.getElementById('quickExamList').innerHTML = state.exams.map(exam => `
                <div class="p-3.5 bg-slate-50 dark:bg-slate-900/60 rounded-xl border border-slate-200 dark:border-slate-700/60 flex items-center justify-between">
                    <div>
                        <h4 class="font-bold text-xs text-slate-800 dark:text-slate-200">${exam.title}</h4>
                        <span class="text-[10px] text-slate-400">${exam.durationMinutes} mins • ${exam.totalQuestions} Questions</span>
                    </div>
                    <button onclick="promptStartExam('${exam.id}')" class="px-3 py-1.5 bg-indigo-600 text-white rounded-lg text-xs font-semibold">Start</button>
                </div>
            `).join('');
            initCharts();
            lucide.createIcons();
        }

        let perfChartInstance = null;
        function initCharts() {
            const ctx = document.getElementById('performanceChart');
            if (!ctx) return;
            if (perfChartInstance) perfChartInstance.destroy();
            perfChartInstance = new Chart(ctx, {
                type: 'line',
                data: {
                    labels: ['Attempt 1', 'Attempt 2', 'Attempt 3', 'Attempt 4', 'Attempt 5'],
                    datasets: [{ label: 'Score %', data: [70, 80, 75, 90, 100], borderColor: '#6366f1', fill: true, backgroundColor: 'rgba(99, 102, 241, 0.1)', tension: 0.4 }]
                },
                options: { responsive: true, maintainAspectRatio: false, plugins: { legend: { display: false } } }
            });
        }

        let topicChartInstance = null, passFailChartInstance = null;
        function initTeacherCharts() {
            const ctx1 = document.getElementById('topicMasteryChart');
            if (ctx1) {
                if (topicChartInstance) topicChartInstance.destroy();
                topicChartInstance = new Chart(ctx1, { type: 'bar', data: { labels: ['CS', 'Math', 'GK'], datasets: [{ data: [85, 70, 95], backgroundColor: ['#6366f1', '#f59e0b', '#10b981'] }] }, options: { responsive: true, maintainAspectRatio: false } });
            }
            const ctx2 = document.getElementById('passFailChart');
            if (ctx2) {
                if (passFailChartInstance) passFailChartInstance.destroy();
                passFailChartInstance = new Chart(ctx2, { type: 'doughnut', data: { labels: ['Passed', 'Failed'], datasets: [{ data: [90, 10], backgroundColor: ['#10b981', '#f43f5e'] }] }, options: { responsive: true, maintainAspectRatio: false } });
            }
        }

        function renderExamsList() {
            document.getElementById('examsGrid').innerHTML = state.exams.map(exam => `
                <div class="bg-white dark:bg-slate-800 p-6 rounded-2xl border border-slate-200 dark:border-slate-700/60 shadow-sm flex flex-col justify-between">
                    <div>
                        <div class="flex justify-between items-start mb-3"><span class="px-2.5 py-1 bg-indigo-50 dark:bg-indigo-950/60 text-indigo-600 text-xs font-semibold rounded-lg">${exam.topic}</span></div>
                        <h3 class="font-bold text-lg text-slate-800 dark:text-slate-100 mb-2">${exam.title}</h3>
                        <div class="space-y-1.5 text-xs text-slate-500 mb-6 font-mono">
                            <div>Duration: ${exam.durationMinutes} Minutes</div>
                            <div>Questions: ${exam.totalQuestions}</div>
                        </div>
                    </div>
                    <button onclick="promptStartExam('${exam.id}')" class="w-full py-2.5 bg-indigo-600 text-white font-medium text-sm rounded-xl shadow-lg flex items-center justify-center gap-2">
                        Start Examination
                    </button>
                </div>
            `).join('');
            lucide.createIcons();
        }

        function renderQuestionBank() {
            const list = document.getElementById('questionBankList');
            const search = document.getElementById('qbSearchInput').value.toLowerCase();
            const topic = document.getElementById('qbFilterTopic').value;
            const diff = document.getElementById('qbFilterDifficulty').value;

            const filtered = state.questionBank.filter(q => {
                const matchesSearch = q.text.toLowerCase().includes(search);
                const matchesTopic = topic === 'ALL' || q.topic === topic;
                const matchesDiff = diff === 'ALL' || q.difficulty === diff;
                return matchesSearch && matchesTopic && matchesDiff;
            });

            list.innerHTML = filtered.map(q => `
                <div class="bg-white dark:bg-slate-800 p-5 rounded-2xl border border-slate-200 dark:border-slate-700/60 space-y-3">
                    <div class="flex justify-between">
                        <div>
                            <span class="text-xs text-indigo-600 font-semibold">${q.topic} (${q.difficulty})</span>
                            <h4 class="font-semibold text-slate-800 dark:text-slate-100 text-sm">${escapeHtml(q.text)}</h4>
                        </div>
                        <button onclick="deleteQuestion('${q.id}')" class="text-rose-500"><i data-lucide="trash-2" class="w-4 h-4"></i></button>
                    </div>
                </div>
            `).join('');
            document.getElementById('statBankCount').innerText = state.questionBank.length;
            lucide.createIcons();
        }

        function openQuestionModal() { document.getElementById('questionModal').classList.remove('hidden'); }
        function closeQuestionModal() { document.getElementById('questionModal').classList.add('hidden'); }

        function handleSaveQuestion(e) {
            e.preventDefault();
            const text = document.getElementById('qInputText').value;
            const topic = document.getElementById('qInputTopic').value;
            const difficulty = document.getElementById('qInputDifficulty').value;
            const options = [document.getElementById('opt0').value, document.getElementById('opt1').value, document.getElementById('opt2').value, document.getElementById('opt3').value];
            const correct = parseInt(document.querySelector('input[name="correctOpt"]:checked').value);
            const explanation = document.getElementById('qInputExplanation').value;

            const newId = 'Q' + (100 + state.questionBank.length + 1);
            state.questionBank.push({ id: newId, text, topic, difficulty, options, correct, explanation });
            logDbQuery(`INSERT INTO question_bank VALUES ('${newId}', '${topic}', '${difficulty}', '${text.replace(/'/g, "''")}');`);
            closeQuestionModal();
            renderQuestionBank();
        }

        function deleteQuestion(qId) {
            state.questionBank = state.questionBank.filter(q => q.id !== qId);
            logDbQuery(`DELETE FROM question_bank WHERE question_id = '${qId}';`);
            renderQuestionBank();
        }

        let pendingExamId = null;
        function promptStartExam(examId) {
            pendingExamId = examId;
            document.getElementById('instructionModal').classList.remove('hidden');
        }
        function closeInstructionModal() { document.getElementById('instructionModal').classList.add('hidden'); }

        function launchQuizConfirmed() {
            closeInstructionModal();
            const exam = state.exams.find(e => e.id === pendingExamId);
            let questions = state.questionBank.filter(q => exam.questionIds.includes(q.id));

            state.activeQuiz = {
                examId: exam.id,
                title: exam.title,
                totalTimeSeconds: exam.durationMinutes * 60,
                timeRemainingSeconds: exam.durationMinutes * 60,
                questions: questions.map(q => ({ ...q, userChoice: null, flagged: false })),
                currentIndex: 0,
                timerInterval: null
            };

            switchTab('quiz-engine');
            renderQuizQuestion();
            startQuizTimer();
        }

        function startQuizTimer() {
            if (state.activeQuiz.timerInterval) clearInterval(state.activeQuiz.timerInterval);
            state.activeQuiz.timerInterval = setInterval(() => {
                state.activeQuiz.timeRemainingSeconds--;
                const mins = Math.floor(state.activeQuiz.timeRemainingSeconds / 60);
                const secs = state.activeQuiz.timeRemainingSeconds % 60;
                document.getElementById('timerDisplay').innerText = `${String(mins).padStart(2, '0')}:${String(secs).padStart(2, '0')}`;

                if (state.activeQuiz.timeRemainingSeconds <= 0) {
                    clearInterval(state.activeQuiz.timerInterval);
                    alert("⏰ Time limit reached! Automatically submitting test.");
                    submitExamProcess();
                }
            }, 1000);
        }

        function renderQuizQuestion() {
            const q = state.activeQuiz.questions[state.activeQuiz.currentIndex];
            document.getElementById('quizTitle').innerText = state.activeQuiz.title;
            document.getElementById('quizQuestionCountBadge').innerText = `Question ${state.activeQuiz.currentIndex + 1} of ${state.activeQuiz.questions.length}`;
            document.getElementById('questionText').innerText = q.text;

            document.getElementById('optionsContainer').innerHTML = q.options.map((opt, idx) => `
                <button onclick="selectOption(${idx})" class="w-full text-left p-4 rounded-xl border ${q.userChoice === idx ? 'border-indigo-600 bg-indigo-50 dark:bg-indigo-950/50' : 'border-slate-200 dark:border-slate-700'}">
                    ${String.fromCharCode(65 + idx)}. ${escapeHtml(opt)}
                </button>
            `).join('');

            renderQuestionPalette();
            lucide.createIcons();
        }

        function selectOption(idx) {
            state.activeQuiz.questions[state.activeQuiz.currentIndex].userChoice = idx;
            renderQuizQuestion();
        }

        function toggleFlagCurrentQuestion() {
            const q = state.activeQuiz.questions[state.activeQuiz.currentIndex];
            q.flagged = !q.flagged;
            renderQuizQuestion();
        }

        function navigateQuestion(step) {
            const newIdx = state.activeQuiz.currentIndex + step;
            if (newIdx >= 0 && newIdx < state.activeQuiz.questions.length) {
                state.activeQuiz.currentIndex = newIdx;
                renderQuizQuestion();
            }
        }

        function renderQuestionPalette() {
            document.getElementById('paletteGrid').innerHTML = state.activeQuiz.questions.map((q, idx) => `
                <button onclick="state.activeQuiz.currentIndex=${idx}; renderQuizQuestion();" class="h-9 rounded-lg border text-xs ${q.userChoice !== null ? 'bg-emerald-500 text-white' : q.flagged ? 'bg-amber-500 text-white' : 'bg-slate-100 dark:bg-slate-800'} flex items-center justify-center">
                    ${idx + 1}
                </button>
            `).join('');
        }

        function confirmSubmitQuiz() {
            if (confirm("Submit exam now?")) submitExamProcess();
        }

        function submitExamProcess() {
            if (state.activeQuiz.timerInterval) clearInterval(state.activeQuiz.timerInterval);
            let correctCount = 0;
            state.activeQuiz.questions.forEach(q => { if (q.userChoice === q.correct) correctCount++; });

            const scorePercent = Math.round((correctCount / state.activeQuiz.questions.length) * 100);
            const passed = scorePercent >= 60;

            logDbQuery(`INSERT INTO exam_results VALUES ('ATT-${Math.floor(Math.random()*1000)}', '${state.currentUser.id}', ${scorePercent}, ${passed});`);

            document.getElementById('resScoreText').innerText = `${scorePercent}%`;
            document.getElementById('resCorrectText').innerText = `${correctCount} / ${state.activeQuiz.questions.length}`;
            switchTab('results');
        }

        function renderHistoryTable() {
            document.getElementById('historyTableContainer').innerHTML = `
                <table class="w-full text-left text-xs font-sans">
                    <thead class="bg-slate-50 dark:bg-slate-900 border-b">
                        <tr><th class="p-3">ID</th><th class="p-3">Exam</th><th class="p-3">Score</th><th class="p-3">Status</th></tr>
                    </thead>
                    <tbody>
                        ${state.history.map(h => `
                            <tr><td class="p-3 font-mono">${h.attemptId}</td><td class="p-3 font-semibold">${h.examTitle}</td><td class="p-3">${h.scorePercent}%</td><td class="p-3"><span class="px-2 py-1 rounded-full text-[10px] bg-emerald-100 text-emerald-800">PASSED</span></td></tr>
                        `).join('')}
                    </tbody>
                </table>
            `;
        }
    </script>
</body>
</html>