!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Pomodoro Study Timer</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/react/18.2.0/umd/react.production.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/react-dom/18.2.0/umd/react-dom.production.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/babel-standalone/7.22.5/babel.min.js"></script>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap');
        body { font-family: 'Inter', sans-serif; }
    </style>
</head>
<body>
    <div id="root"></div>
    
    <script type="text/babel">
        const { useState, useEffect, useRef } = React;

        // Lucide React icons as simple SVG components
        const Play = () => (
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round">
                <polygon points="5,3 19,12 5,21"></polygon>
            </svg>
        );

        const Pause = () => (
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round">
                <rect x="6" y="4" width="4" height="16"></rect>
                <rect x="14" y="4" width="4" height="16"></rect>
            </svg>
        );

        const Square = () => (
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round">
                <rect x="3" y="3" width="18" height="18" rx="2"></rect>
            </svg>
        );

        const Settings = () => (
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round">
                <circle cx="12" cy="12" r="3"></circle>
                <path d="M12 1v6M12 17v6M4.22 4.22l4.24 4.24M15.54 15.54l4.24 4.24M1 12h6M17 12h6M4.22 19.78l4.24-4.24M15.54 8.46l4.24-4.24"></path>
            </svg>
        );

        const X = () => (
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round">
                <line x1="18" y1="6" x2="6" y2="18"></line>
                <line x1="6" y1="6" x2="18" y2="18"></line>
            </svg>
        );

        const PomodoroTimer = () => {
            const [timeLeft, setTimeLeft] = useState(25 * 60);
            const [isRunning, setIsRunning] = useState(false);
            const [currentSession, setCurrentSession] = useState(1);
            const [isBreak, setIsBreak] = useState(false);
            const [showSettings, setShowSettings] = useState(false);
            const [theme, setTheme] = useState(() => {
                return localStorage.getItem('pomodoro-theme') || 'blue';
            });
            const [longBreakDuration, setLongBreakDuration] = useState(() => {
                return parseInt(localStorage.getItem('pomodoro-longbreak')) || 15;
            });
            
            const intervalRef = useRef(null);
            const audioContextRef = useRef(null);
            
            const WORK_DURATION = 25 * 60;
            const SHORT_BREAK_DURATION = 5 * 60;
            
            const themes = {
                blue: {
                    primary: 'bg-blue-500',
                    secondary: 'bg-blue-100',
                    text: 'text-blue-600',
                    button: 'bg-blue-500 hover:bg-blue-600',
                    progress: 'stroke-blue-500'
                },
                green: {
                    primary: 'bg-green-500',
                    secondary: 'bg-green-100',
                    text: 'text-green-600',
                    button: 'bg-green-500 hover:bg-green-600',
                    progress: 'stroke-green-500'
                },
                purple: {
                    primary: 'bg-purple-500',
                    secondary: 'bg-purple-100',
                    text: 'text-purple-600',
                    button: 'bg-purple-500 hover:bg-purple-600',
                    progress: 'stroke-purple-500'
                },
                red: {
                    primary: 'bg-red-500',
                    secondary: 'bg-red-100',
                    text: 'text-red-600',
                    button: 'bg-red-500 hover:bg-red-600',
                    progress: 'stroke-red-500'
                }
            };

            const initAudio = () => {
                if (!audioContextRef.current) {
                    audioContextRef.current = new (window.AudioContext || window.webkitAudioContext)();
                }
                return audioContextRef.current;
            };

            const playNotificationSound = (isWorkEnd = false) => {
                try {
                    const audioContext = initAudio();
                    
                    if (isWorkEnd) {
                        const frequencies = [440, 554, 659];
                        frequencies.forEach((freq, index) => {
                            setTimeout(() => {
                                const osc = audioContext.createOscillator();
                                const gain = audioContext.createGain();
                                osc.connect(gain);
                                gain.connect(audioContext.destination);
                                osc.frequency.value = freq;
                                osc.type = 'sine';
                                gain.gain.setValueAtTime(0.3, audioContext.currentTime);
                                gain.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + 0.3);
                                osc.start(audioContext.currentTime);
                                osc.stop(audioContext.currentTime + 0.3);
                            }, index * 200);
                        });
                    } else {
                        const frequencies = [659, 523];
                        frequencies.forEach((freq, index) => {
                            setTimeout(() => {
                                const osc = audioContext.createOscillator();
                                const gain = audioContext.createGain();
                                osc.connect(gain);
                                gain.connect(audioContext.destination);
                                osc.frequency.value = freq;
                                osc.type = 'sine';
                                gain.gain.setValueAtTime(0.3, audioContext.currentTime);
                                gain.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + 0.4);
                                osc.start(audioContext.currentTime);
                                osc.stop(audioContext.currentTime + 0.4);
                            }, index * 300);
                        });
                    }
                } catch (error) {
                    console.warn('Audio not supported:', error);
                }
            };

            useEffect(() => {
                if (isRunning && timeLeft > 0) {
                    intervalRef.current = setInterval(() => {
                        setTimeLeft(prev => prev - 1);
                    }, 1000);
                } else if (timeLeft === 0) {
                    if (isBreak) {
                        playNotificationSound(false);
                        setIsBreak(false);
                        setTimeLeft(WORK_DURATION);
                        setCurrentSession(prev => prev + 1);
                    } else {
                        playNotificationSound(true);
                        setIsBreak(true);
                        
                        if (currentSession % 5 === 0) {
                            setTimeLeft(longBreakDuration * 60);
                        } else {
                            setTimeLeft(SHORT_BREAK_DURATION);
                        }
                    }
                    setIsRunning(false);
                }

                return () => {
                    if (intervalRef.current) {
                        clearInterval(intervalRef.current);
                    }
                };
            }, [isRunning, timeLeft, isBreak, currentSession, longBreakDuration]);

            const startTimer = () => {
                initAudio();
                setIsRunning(true);
            };

            const pauseTimer = () => {
                setIsRunning(false);
            };

            const resetTimer = () => {
                setIsRunning(false);
                setIsBreak(false);
                setCurrentSession(1);
                setTimeLeft(WORK_DURATION);
            };

            const saveSettings = () => {
                localStorage.setItem('pomodoro-theme', theme);
                localStorage.setItem('pomodoro-longbreak', longBreakDuration.toString());
                setShowSettings(false);
            };

            const formatTime = (seconds) => {
                const mins = Math.floor(seconds / 60);
                const secs = seconds % 60;
                return `${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`;
            };

            const getProgress = () => {
                const totalTime = isBreak 
                    ? (currentSession % 5 === 0 ? longBreakDuration * 60 : SHORT_BREAK_DURATION)
                    : WORK_DURATION;
                return ((totalTime - timeLeft) / totalTime) * 283;
            };

            const currentTheme = themes[theme];

            return (
                <div className={`min-h-screen ${currentTheme.secondary} flex items-center justify-center p-4`}>
                    <div className="bg-white rounded-2xl shadow-2xl p-8 w-full max-w-md">
                        <div className="flex justify-between items-center mb-8">
                            <h1 className={`text-2xl font-bold ${currentTheme.text}`}>
                                Pomodoro Timer
                            </h1>
                            <button
                                onClick={() => setShowSettings(true)}
                                className={`p-2 rounded-lg ${currentTheme.primary} text-white hover:opacity-90 transition-opacity`}
                            >
                                <Settings />
                            </button>
                        </div>

                        <div className="text-center mb-6">
                            <div className={`text-lg font-semibold ${currentTheme.text}`}>
                                {isBreak 
                                    ? (currentSession % 5 === 0 ? 'Long Break' : 'Short Break')
                                    : `Work Session ${currentSession}`
                                }
                            </div>
                            <div className="text-gray-500 text-sm mt-1">
                                {isBreak ? '' : `Next long break: Session ${Math.ceil(currentSession / 5) * 5}`}
                            </div>
                        </div>

                        <div className="relative mb-8 flex justify-center">
                            <svg className="w-64 h-64 transform -rotate-90" viewBox="0 0 100 100">
                                <circle
                                    cx="50"
                                    cy="50"
                                    r="45"
                                    stroke="currentColor"
                                    strokeWidth="4"
                                    fill="none"
                                    className="text-gray-200"
                                />
                                <circle
                                    cx="50"
                                    cy="50"
                                    r="45"
                                    stroke="currentColor"
                                    strokeWidth="4"
                                    fill="none"
                                    strokeDasharray="283"
                                    strokeDashoffset={283 - getProgress()}
                                    className={`${currentTheme.progress} transition-all duration-1000 ease-linear`}
                                    strokeLinecap="round"
                                />
                            </svg>
                            <div className="absolute inset-0 flex flex-col items-center justify-center">
                                <div className={`text-4xl font-mono font-bold ${currentTheme.text}`}>
                                    {formatTime(timeLeft)}
                                </div>
                                <div className="text-gray-500 text-sm mt-1">
                                    {isBreak ? 'Break Time' : 'Focus Time'}
                                </div>
                            </div>
                        </div>

                        <div className="flex justify-center space-x-4 mb-6">
                            {!isRunning ? (
                                <button
                                    onClick={startTimer}
                                    className={`${currentTheme.button} text-white px-6 py-3 rounded-lg font-semibold transition-colors flex items-center space-x-2`}
                                >
                                    <Play />
                                    <span>Start</span>
                                </button>
                            ) : (
                                <button
                                    onClick={pauseTimer}
                                    className={`${currentTheme.button} text-white px-6 py-3 rounded-lg font-semibold transition-colors flex items-center space-x-2`}
                                >
                                    <Pause />
                                    <span>Pause</span>
                                </button>
                            )}
                            <button
                                onClick={resetTimer}
                                className="bg-gray-500 hover:bg-gray-600 text-white px-6 py-3 rounded-lg font-semibold transition-colors flex items-center space-x-2"
                            >
                                <Square />
                                <span>Reset</span>
                            </button>
                        </div>

                        <div className="text-center">
                            <div className="text-gray-600 text-sm mb-2">Session Progress</div>
                            <div className="flex justify-center space-x-2">
                                {[1, 2, 3, 4, 5].map((num) => (
                                    <div
                                        key={num}
                                        className={`w-3 h-3 rounded-full ${
                                            num <= (currentSession % 5 === 0 ? 5 : currentSession % 5)
                                                ? currentTheme.primary
                                                : 'bg-gray-200'
                                        }`}
                                    />
                                ))}
                            </div>
                            <div className="text-xs text-gray-500 mt-2">
                                Sessions until long break: {5 - (currentSession % 5 === 0 ? 0 : currentSession % 5)}
                            </div>
                        </div>

                        {showSettings && (
                            <div className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4 z-50">
                                <div className="bg-white rounded-xl p-6 w-full max-w-sm">
                                    <div className="flex justify-between items-center mb-4">
                                        <h3 className="text-lg font-semibold text-gray-800">Settings</h3>
                                        <button
                                            onClick={() => setShowSettings(false)}
                                            className="text-gray-500 hover:text-gray-700"
                                        >
                                            <X />
                                        </button>
                                    </div>
                                    
                                    <div className="mb-4">
                                        <label className="block text-sm font-medium text-gray-700 mb-2">
                                            Theme
                                        </label>
                                        <div className="grid grid-cols-2 gap-2">
                                            {Object.keys(themes).map((themeName) => (
                                                <button
                                                    key={themeName}
                                                    onClick={() => setTheme(themeName)}
                                                    className={`p-3 rounded-lg border-2 transition-colors ${
                                                        theme === themeName
                                                            ? `${themes[themeName].primary} text-white border-transparent`
                                                            : 'bg-gray-100 text-gray-700 border-gray-200 hover:bg-gray-200'
                                                    }`}
                                                >
                                                    {themeName.charAt(0).toUpperCase() + themeName.slice(1)}
                                                </button>
                                            ))}
                                        </div>
                                    </div>

                                    <div className="mb-6">
                                        <label className="block text-sm font-medium text-gray-700 mb-2">
                                            Long Break Duration
                                        </label>
                                        <select
                                            value={longBreakDuration}
                                            onChange={(e) => setLongBreakDuration(Number(e.target.value))}
                                            className="w-full p-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent"
                                        >
                                            <option value={15}>15 minutes</option>
                                            <option value={20}>20 minutes</option>
                                        </select>
                                    </div>

                                    <button
                                        onClick={saveSettings}
                                        className={`w-full ${currentTheme.button} text-white py-2 rounded-lg font-semibold transition-colors`}
                                    >
                                        Save Settings
                                    </button>
                                </div>
                            </div>
                        )}
                    </div>
                </div>
            );
        };

        ReactDOM.render(<PomodoroTimer />, document.getElementById('root'));
    </script>
</body>
</html># Sintu-kumar-sahu-