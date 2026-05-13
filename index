import React, { useState, useEffect } from 'react';
import { 
  Settings, 
  CheckSquare, 
  Clock, 
  LayoutDashboard, 
  Play, 
  Pause, 
  RotateCcw, 
  Plus, 
  Trash2,
  Palette,
  BookOpen,
  Sparkles,
  Loader2,
  CheckCircle2,
  XCircle
} from 'lucide-react';

export default function App() {
  // --- THEME STATE ---
  const [colors, setColors] = useState({
    primary: '#ec4899',   // Soft Magenta/Pink
    background: '#fdf2f8', // Very light pink background
    card: '#ffffff',       // White cards
    text: '#1e293b'        // Slate text
  });

  // App automatically starts on the Home Screen (Dashboard)
  const [activeTab, setActiveTab] = useState('dashboard');
  
  // Tasks State
  const [tasks, setTasks] = useState([]);
  const [newTaskText, setNewTaskText] = useState('');

  // Timer State
  const [timerMode, setTimerMode] = useState('work'); // 'work' or 'break'
  const [durations, setDurations] = useState({
    work: 25 * 60,
    break: 5 * 60
  });
  const [timeLeft, setTimeLeft] = useState(25 * 60);       // Current ticking time
  const [isRunning, setIsRunning] = useState(false);

  // Notes & Quiz State
  const [notesText, setNotesText] = useState('');
  const [isGenerating, setIsGenerating] = useState(false);
  const [quizData, setQuizData] = useState(null);
  const [quizError, setQuizError] = useState('');
  const [selectedAnswers, setSelectedAnswers] = useState({});

  // --- TIMER LOGIC ---
  useEffect(() => {
    let interval = null;
    if (isRunning && timeLeft > 0) {
      interval = setInterval(() => {
        setTimeLeft((prev) => prev - 1);
      }, 1000);
    } else if (timeLeft === 0) {
      setIsRunning(false);
    }
    return () => clearInterval(interval);
  }, [isRunning, timeLeft]);

  const switchTimerMode = (mode) => {
    setIsRunning(false);
    setTimerMode(mode);
    setTimeLeft(durations[mode]);
  };

  const handleSliderChange = (e) => {
    setIsRunning(false);
    const newTime = parseInt(e.target.value, 10) * 60;
    setDurations(prev => ({ ...prev, [timerMode]: newTime }));
    setTimeLeft(newTime);
  };

  const resetTimer = () => {
    setIsRunning(false);
    setTimeLeft(durations[timerMode]);
  };

  const formatTime = (seconds) => {
    const m = Math.floor(seconds / 60);
    const s = seconds % 60;
    return `${m.toString().padStart(2, '0')}:${s.toString().padStart(2, '0')}`;
  };

  // --- TASKS LOGIC ---
  const addTask = (e) => {
    e.preventDefault();
    if (!newTaskText.trim()) return;
    setTasks([...tasks, { id: Date.now(), text: newTaskText, completed: false }]);
    setNewTaskText('');
  };

  const toggleTask = (id) => {
    setTasks(tasks.map(t => t.id === id ? { ...t, completed: !t.completed } : t));
  };

  const deleteTask = (id) => {
    setTasks(tasks.filter(t => t.id !== id));
  };

  // --- AI QUIZ LOGIC ---
  const generateQuiz = async () => {
    if (!notesText.trim()) return;
    
    setIsGenerating(true);
    setQuizError('');
    setQuizData(null);
    setSelectedAnswers({});

    // Calculate word count to determine how many questions to ask
    const wordCount = notesText.trim().split(/\s+/).length;
    let questionCountStr = "exactly 5";
    
    if (wordCount > 1000) {
      questionCountStr = "between 10 and 15";
    } else if (wordCount > 500) {
      questionCountStr = "between 7 and 10";
    } else if (wordCount > 250) {
      questionCountStr = "between 5 and 7";
    } else {
      questionCountStr = "exactly 5"; // Minimum of 5
    }

    const apiKey = ""; // Environment injected API key
    const url = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`;

    const payload = {
      contents: [{ parts: [{ text: `Generate a multiple choice quiz based strictly on these notes:\n\n${notesText}` }] }],
      systemInstruction: { parts: [{ text: `You are a helpful study assistant. Create ${questionCountStr} high-quality multiple choice questions based on the provided notes. Ensure the questions test understanding, not just rote memorization. There MUST be a minimum of 5 questions.` }] },
      generationConfig: {
        responseMimeType: "application/json",
        responseSchema: {
          type: "OBJECT",
          properties: {
            questions: {
              type: "ARRAY",
              items: {
                type: "OBJECT",
                properties: {
                  question: { type: "STRING" },
                  options: { type: "ARRAY", items: { type: "STRING" } },
                  correctAnswerIndex: { type: "INTEGER" },
                  explanation: { type: "STRING" }
                }
              }
            }
          }
        }
      }
    };

    const delay = (ms) => new Promise(res => setTimeout(res, ms));
    const retries = [1000, 2000, 4000, 8000, 16000];

    for (let i = 0; i <= retries.length; i++) {
      try {
        const response = await fetch(url, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(payload)
        });
        
        if (!response.ok) throw new Error(`HTTP ${response.status}`);
        
        const data = await response.json();
        const resultText = data.candidates?.[0]?.content?.parts?.[0]?.text;
        
        if (resultText) {
          const parsed = JSON.parse(resultText);
          if (parsed.questions && parsed.questions.length > 0) {
            setQuizData(parsed.questions);
            setIsGenerating(false);
            return;
          } else {
             throw new Error("Invalid response format");
          }
        }
      } catch (err) {
        if (i === retries.length) {
          setQuizError("Failed to generate quiz. Please check your connection or try adjusting your notes.");
          setIsGenerating(false);
          return;
        }
        await delay(retries[i]);
      }
    }
  };

  const handleAnswerSelect = (qIndex, optionIndex) => {
    if (selectedAnswers[qIndex] !== undefined) return;
    setSelectedAnswers(prev => ({ ...prev, [qIndex]: optionIndex }));
  };

  const resetQuiz = () => {
    setQuizData(null);
    setSelectedAnswers({});
    setNotesText('');
    setQuizError('');
  };

  // --- STYLING HELPERS ---
  const hexToRgba = (hex, alpha) => {
    try {
      const r = parseInt(hex.slice(1, 3), 16);
      const g = parseInt(hex.slice(3, 5), 16);
      const b = parseInt(hex.slice(5, 7), 16);
      if (isNaN(r) || isNaN(g) || isNaN(b)) return 'transparent';
      return `rgba(${r}, ${g}, ${b}, ${alpha})`;
    } catch (e) {
      return 'transparent';
    }
  };

  const updateColor = (key, value) => {
    setColors(prev => ({ ...prev, [key]: value }));
  };

  const borderStyle = { borderColor: hexToRgba(colors.text, 0.15) };
  const cardStyle = { backgroundColor: colors.card, borderColor: hexToRgba(colors.text, 0.1) };

  // --- COMPONENTS ---

  const NavItem = ({ id, icon: Icon, label }) => {
    const isActive = activeTab === id;
    return (
      <button
        onClick={() => setActiveTab(id)}
        className="flex flex-col md:flex-row items-center p-3 md:p-4 w-full md:rounded-xl transition-all duration-200"
        style={{
          color: isActive ? colors.primary : hexToRgba(colors.text, 0.6),
          backgroundColor: isActive && window.innerWidth > 768 ? hexToRgba(colors.primary, 0.1) : 'transparent',
        }}
      >
        <Icon size={24} className="mb-1 md:mb-0 md:mr-3" />
        <span className="text-[10px] md:text-base font-medium">{label}</span>
      </button>
    );
  };

  const renderDashboard = () => (
    <div className="space-y-6 animate-in fade-in slide-in-from-bottom-4 duration-500">
      <h1 className="text-3xl font-bold">Overview</h1>
      
      <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
        {/* Tasks Completed Card */}
        <div className="p-6 rounded-2xl shadow-sm border flex items-center space-x-4 transition-colors" style={cardStyle}>
          <div className="p-4 rounded-full" style={{ backgroundColor: hexToRgba(colors.primary, 0.1), color: colors.primary }}>
            <CheckSquare size={32} />
          </div>
          <div>
            <p className="text-sm font-medium" style={{ color: hexToRgba(colors.text, 0.6) }}>Tasks Completed</p>
            <p className="text-3xl font-bold">
              {tasks.filter(t => t.completed).length} <span className="text-lg opacity-50">/ {tasks.length || 0}</span>
            </p>
          </div>
        </div>
      </div>
    </div>
  );

  const renderTimer = () => (
    <div className="flex flex-col items-center justify-center h-full space-y-6 animate-in fade-in zoom-in-95 duration-500 pb-10">
      
      {/* Mode Toggle Switch */}
      <div className="flex space-x-2 p-1 rounded-full border shadow-sm transition-colors mb-2" style={{ backgroundColor: hexToRgba(colors.text, 0.05), ...borderStyle }}>
        <button
          onClick={() => switchTimerMode('work')}
          className="px-6 py-2 rounded-full text-sm font-semibold tracking-wide transition-all"
          style={timerMode === 'work' 
            ? { backgroundColor: colors.primary, color: '#fff', boxShadow: '0 2px 4px rgba(0,0,0,0.1)' } 
            : { color: hexToRgba(colors.text, 0.7) }}
        >
          WORK
        </button>
        <button
          onClick={() => switchTimerMode('break')}
          className="px-6 py-2 rounded-full text-sm font-semibold tracking-wide transition-all"
          style={timerMode === 'break' 
            ? { backgroundColor: colors.primary, color: '#fff', boxShadow: '0 2px 4px rgba(0,0,0,0.1)' } 
            : { color: hexToRgba(colors.text, 0.7) }}
        >
          BREAK
        </button>
      </div>

      {/* Circular Timer Display */}
      <div className="relative flex items-center justify-center w-64 h-64 md:w-80 md:h-80 rounded-full shadow-xl border-4 transition-colors" 
           style={{ backgroundColor: colors.card, borderColor: hexToRgba(colors.primary, 0.2) }}>
        <div className="absolute w-[90%] h-[90%] rounded-full border-2 border-dashed animate-spin-slow" 
             style={{ borderColor: hexToRgba(colors.primary, 0.5), animationDuration: '30s' }}></div>
        <span className="text-6xl md:text-7xl font-bold font-mono tracking-tight">
          {formatTime(timeLeft)}
        </span>
      </div>

      {/* Time Slider Feature */}
      <div className="w-full max-w-xs mt-4 flex items-center space-x-4">
        <span className="text-sm font-medium" style={{ color: hexToRgba(colors.text, 0.6) }}>1m</span>
        <input
          type="range"
          min="1"
          max="90"
          value={Math.max(1, Math.ceil(durations[timerMode] / 60))} // Dynamically grabs work or break time
          onChange={handleSliderChange}
          className="flex-1 h-2 rounded-lg appearance-none cursor-pointer"
          style={{ 
            backgroundColor: hexToRgba(colors.text, 0.1),
            accentColor: colors.primary, 
          }}
        />
        <span className="text-sm font-medium" style={{ color: hexToRgba(colors.text, 0.6) }}>90m</span>
      </div>

      {/* Timer Controls */}
      <div className="flex space-x-6 mt-4">
        <button
          onClick={() => setIsRunning(!isRunning)}
          className="flex items-center justify-center w-16 h-16 rounded-full text-white shadow-lg transform transition hover:scale-105 active:scale-95"
          style={{ backgroundColor: colors.primary }}
        >
          {isRunning ? <Pause size={28} fill="currentColor" /> : <Play size={28} fill="currentColor" className="ml-1" />}
        </button>
        <button
          onClick={resetTimer}
          className="flex items-center justify-center w-16 h-16 rounded-full shadow border transform transition hover:scale-105 active:scale-95 hover:opacity-80"
          style={{ backgroundColor: colors.card, color: colors.text, borderColor: hexToRgba(colors.text, 0.15) }}
        >
          <RotateCcw size={24} />
        </button>
      </div>
    </div>
  );

  const renderTasks = () => (
    <div className="max-w-3xl mx-auto w-full space-y-6 animate-in fade-in slide-in-from-right-4 duration-500">
      <h1 className="text-3xl font-bold">Tasks</h1>
      
      <form onSubmit={addTask} className="flex gap-2">
        <input
          type="text"
          value={newTaskText}
          onChange={(e) => setNewTaskText(e.target.value)}
          placeholder="Add a new task..."
          className="flex-1 p-4 rounded-xl border shadow-sm focus:outline-none transition-colors"
          style={{ 
            backgroundColor: colors.card, 
            color: colors.text, 
            borderColor: hexToRgba(colors.text, 0.2),
            boxShadow: `0 0 0 0 ${colors.primary}` 
          }}
        />
        <button
          type="submit"
          className="px-6 rounded-xl text-white font-medium shadow-sm transition-transform hover:scale-105 active:scale-95 flex items-center justify-center"
          style={{ backgroundColor: colors.primary }}
        >
          <Plus size={24} />
        </button>
      </form>

      <div className="space-y-3 mt-8">
        {tasks.length === 0 ? (
          <div className="text-center p-8 rounded-xl border border-dashed transition-colors" style={{ backgroundColor: colors.card, borderColor: hexToRgba(colors.text, 0.3), color: hexToRgba(colors.text, 0.6) }}>
            No tasks yet. Add one above!
          </div>
        ) : (
          tasks.map(task => (
            <div 
              key={task.id} 
              className="flex items-center justify-between p-4 rounded-xl shadow-sm border transition-all"
              style={{
                backgroundColor: colors.card,
                borderColor: hexToRgba(colors.text, task.completed ? 0.05 : 0.15),
                opacity: task.completed ? 0.7 : 1
              }}
            >
              <div className="flex items-center space-x-4 flex-1 cursor-pointer" onClick={() => toggleTask(task.id)}>
                <div 
                  className="w-6 h-6 rounded-md flex items-center justify-center border-2 transition-colors"
                  style={{
                    backgroundColor: task.completed ? colors.primary : 'transparent',
                    borderColor: task.completed ? colors.primary : hexToRgba(colors.text, 0.3)
                  }}
                >
                  {task.completed && <CheckSquare size={16} className="text-white" />}
                </div>
                <span className="text-lg transition-all" style={{ 
                  textDecoration: task.completed ? 'line-through' : 'none',
                  color: task.completed ? hexToRgba(colors.text, 0.5) : colors.text
                }}>
                  {task.text}
                </span>
              </div>
              <button 
                onClick={() => deleteTask(task.id)}
                className="p-2 rounded-lg transition-colors hover:opacity-70"
                style={{ color: hexToRgba(colors.text, 0.5) }}
              >
                <Trash2 size={20} />
              </button>
            </div>
          ))
        )}
      </div>
    </div>
  );

  const renderNotesAndQuiz = () => (
    <div className="max-w-3xl mx-auto w-full space-y-6 animate-in fade-in slide-in-from-bottom-4 duration-500">
      <div className="flex justify-between items-center mb-6">
        <h1 className="text-3xl font-bold">Learn & Quiz</h1>
        {quizData && (
          <button 
            onClick={resetQuiz}
            className="text-sm font-medium px-4 py-2 rounded-lg transition"
            style={{ backgroundColor: hexToRgba(colors.text, 0.1), color: colors.text }}
          >
            Start Over
          </button>
        )}
      </div>

      {!quizData ? (
        <div className="p-6 rounded-2xl shadow-sm border flex flex-col h-[60vh] md:h-[70vh] transition-colors" style={cardStyle}>
          <div className="mb-4">
            <h2 className="text-lg font-semibold flex items-center">
              <BookOpen className="mr-2" size={20} style={{ color: colors.primary }} />
              Study Notes
            </h2>
            <p className="text-sm mt-1" style={{ color: hexToRgba(colors.text, 0.6) }}>
              Write or paste the subject matter you learned today. The app will generate a custom quiz to test your knowledge!
            </p>
          </div>
          
          <textarea
            value={notesText}
            onChange={(e) => setNotesText(e.target.value)}
            placeholder="E.g., Photosynthesis is the process by which plants use sunlight, water, and carbon dioxide..."
            className="flex-1 w-full p-4 border rounded-xl resize-none focus:outline-none focus:ring-2 transition-all mb-4"
            style={{ 
              backgroundColor: hexToRgba(colors.text, 0.02), 
              color: colors.text,
              borderColor: hexToRgba(colors.text, 0.15),
              outlineColor: hexToRgba(colors.primary, 0.5) 
            }}
          />

          {quizError && (
             <div className="mb-4 p-3 rounded-lg text-sm border" style={{ backgroundColor: 'rgba(239, 68, 68, 0.1)', color: '#b91c1c', borderColor: 'rgba(239, 68, 68, 0.2)' }}>
               {quizError}
             </div>
          )}

          <button
            onClick={generateQuiz}
            disabled={isGenerating || !notesText.trim()}
            className="w-full py-4 rounded-xl text-white font-semibold shadow-md transition-all flex items-center justify-center disabled:opacity-50 disabled:cursor-not-allowed"
            style={{ backgroundColor: colors.primary }}
          >
            {isGenerating ? (
              <>
                <Loader2 className="animate-spin mr-2" size={20} />
                Generating Quiz...
              </>
            ) : (
              <>
                <Sparkles className="mr-2" size={20} />
                Generate Practice Quiz
              </>
            )}
          </button>
        </div>
      ) : (
        <div className="space-y-6 pb-8">
          <div className="border p-4 rounded-xl mb-6" style={{ backgroundColor: hexToRgba(colors.primary, 0.1), borderColor: hexToRgba(colors.primary, 0.2) }}>
            <p className="font-medium flex items-center" style={{ color: colors.primary }}>
               <Sparkles className="mr-2" size={20} />
               Quiz Generated! Test your knowledge below.
            </p>
          </div>

          {quizData.map((q, qIndex) => {
            const hasAnswered = selectedAnswers[qIndex] !== undefined;
            const isCorrect = selectedAnswers[qIndex] === q.correctAnswerIndex;

            return (
              <div key={qIndex} className="p-6 rounded-2xl shadow-sm border transition-colors" style={cardStyle}>
                <h3 className="text-lg font-semibold mb-4">
                  <span className="mr-2 opacity-50">{qIndex + 1}.</span>
                  {q.question}
                </h3>
                
                <div className="space-y-3">
                  {q.options.map((option, optIndex) => {
                    let bg = 'transparent';
                    let textC = colors.text;
                    let borderC = hexToRgba(colors.text, 0.2);
                    let opacity = 1;

                    if (hasAnswered) {
                      if (optIndex === q.correctAnswerIndex) {
                        bg = 'rgba(34, 197, 94, 0.1)'; // green bg
                        textC = '#15803d'; // green text
                        borderC = '#22c55e'; // green border
                      } else if (optIndex === selectedAnswers[qIndex]) {
                        bg = 'rgba(239, 68, 68, 0.1)'; // red bg
                        textC = '#b91c1c'; // red text
                        borderC = '#ef4444'; // red border
                      } else {
                        opacity = 0.5;
                      }
                    }

                    return (
                      <button
                        key={optIndex}
                        onClick={() => handleAnswerSelect(qIndex, optIndex)}
                        disabled={hasAnswered}
                        className="w-full text-left p-4 rounded-xl border-2 transition-all flex justify-between items-center"
                        style={{ backgroundColor: bg, color: textC, borderColor: borderC, opacity }}
                      >
                        <span>{option}</span>
                        {hasAnswered && optIndex === q.correctAnswerIndex && (
                          <CheckCircle2 size={20} color="#16a34a" />
                        )}
                        {hasAnswered && optIndex === selectedAnswers[qIndex] && optIndex !== q.correctAnswerIndex && (
                          <XCircle size={20} color="#dc2626" />
                        )}
                      </button>
                    );
                  })}
                </div>

                {hasAnswered && (
                  <div className="mt-4 p-4 rounded-xl text-sm border" 
                       style={{ 
                         backgroundColor: isCorrect ? 'rgba(34, 197, 94, 0.05)' : hexToRgba(colors.text, 0.05),
                         borderColor: isCorrect ? 'rgba(34, 197, 94, 0.2)' : hexToRgba(colors.text, 0.1),
                         color: isCorrect ? '#15803d' : colors.text
                       }}>
                    <span className="font-semibold block mb-1">
                      {isCorrect ? "Spot on!" : "Not quite."}
                    </span>
                    <span style={{ opacity: 0.8 }}>{q.explanation}</span>
                  </div>
                )}
              </div>
            );
          })}
        </div>
      )}
    </div>
  );

  const renderColorPickerRow = (label, description, colorKey) => (
    <div className="flex flex-col md:flex-row items-start md:items-center justify-between gap-4 py-4 border-b last:border-b-0" style={{ borderColor: hexToRgba(colors.text, 0.1) }}>
      <div>
        <h3 className="font-medium">{label}</h3>
        <p className="text-sm opacity-70">{description}</p>
      </div>
      <div className="flex items-center space-x-4">
        {/* Native Color Input wrapper simulating a Color Wheel trigger */}
        <div 
          className="w-12 h-12 rounded-full shadow-inner border-2 overflow-hidden relative cursor-pointer flex items-center justify-center transition-transform hover:scale-105 active:scale-95"
          style={{ borderColor: hexToRgba(colors.text, 0.2) }}
          title="Click to open color wheel"
        >
          {/* Conic gradient behind input to give it a "color wheel" aesthetic before picking */}
          <div className="absolute inset-0 rounded-full" 
               style={{ background: 'conic-gradient(red, yellow, lime, aqua, blue, magenta, red)', opacity: 0.2 }}></div>
          <input
            type="color"
            value={colors[colorKey]}
            onChange={(e) => updateColor(colorKey, e.target.value)}
            className="absolute inset-[-10px] w-20 h-20 cursor-pointer opacity-0 z-10"
          />
          <div className="w-[85%] h-[85%] rounded-full absolute z-0 shadow-sm border border-black/10" style={{ backgroundColor: colors[colorKey] }}></div>
        </div>
        <span className="font-mono text-sm opacity-70 uppercase w-20 text-right">{colors[colorKey]}</span>
      </div>
    </div>
  );

  const renderSettings = () => (
    <div className="max-w-3xl mx-auto w-full space-y-8 animate-in fade-in slide-in-from-left-4 duration-500">
      <h1 className="text-3xl font-bold">Settings</h1>
      
      <div className="p-6 rounded-2xl shadow-sm border transition-colors" style={cardStyle}>
        <h2 className="text-xl font-semibold flex items-center mb-2">
          <Palette className="mr-2" size={24} style={{ color: colors.primary }} />
          Theme Customization
        </h2>
        <p className="text-sm opacity-70 mb-6">Fully personalize your study environment. Click the circles below to open your device's color wheel.</p>
        
        <div className="flex flex-col">
          {renderColorPickerRow('Primary Accent', 'Buttons, icons, highlights, and active states.', 'primary')}
          {renderColorPickerRow('App Background', 'The main background color of the application.', 'background')}
          {renderColorPickerRow('Card Background', 'Background for panels, navigation, and input areas.', 'card')}
          {renderColorPickerRow('Text Color', 'Primary color for headers and reading text.', 'text')}
        </div>
      </div>
    </div>
  );

  // --- RENDER ---
  return (
    <div 
      className="min-h-screen flex flex-col md:flex-row font-sans transition-colors duration-300"
      style={{ backgroundColor: colors.background, color: colors.text }}
    >
      
      {/* Desktop Sidebar */}
      <nav className="hidden md:flex flex-col w-64 border-r h-screen sticky top-0 p-4 transition-colors duration-300 z-10"
           style={{ backgroundColor: colors.card, borderColor: hexToRgba(colors.text, 0.1) }}>
        <div className="flex items-center mb-10 px-4 py-2">
           <div className="w-8 h-8 rounded-lg mr-3 flex items-center justify-center text-white font-bold text-xl shadow-sm" style={{ backgroundColor: colors.primary }}>
             S
           </div>
           <span className="text-xl font-bold tracking-tight">StudySpace</span>
        </div>
        
        <div className="space-y-2 flex-1">
          <NavItem id="dashboard" icon={LayoutDashboard} label="Dashboard" />
          <NavItem id="timer" icon={Clock} label="Timer" />
          <NavItem id="tasks" icon={CheckSquare} label="Tasks" />
          <NavItem id="notes" icon={BookOpen} label="Learn & Quiz" />
        </div>

        <div className="pt-4 border-t mt-auto" style={{ borderColor: hexToRgba(colors.text, 0.1) }}>
          <NavItem id="settings" icon={Settings} label="Settings" />
        </div>
      </nav>

      {/* Main Content Area */}
      <main className="flex-1 h-screen overflow-y-auto pb-24 md:pb-0 relative">
        {/* Mobile Header */}
        <header className="md:hidden px-4 py-4 border-b sticky top-0 z-10 flex items-center transition-colors shadow-sm"
                style={{ backgroundColor: colors.card, borderColor: hexToRgba(colors.text, 0.1) }}>
           <div className="w-8 h-8 rounded-lg mr-3 flex items-center justify-center text-white font-bold text-xl shadow-sm" style={{ backgroundColor: colors.primary }}>
             S
           </div>
           <span className="text-xl font-bold tracking-tight">StudySpace</span>
        </header>

        <div className="p-4 md:p-8 h-full max-w-6xl mx-auto">
          {activeTab === 'dashboard' && renderDashboard()}
          {activeTab === 'timer' && renderTimer()}
          {activeTab === 'tasks' && renderTasks()}
          {activeTab === 'notes' && renderNotesAndQuiz()}
          {activeTab === 'settings' && renderSettings()}
        </div>
      </main>

      {/* Mobile Bottom Navigation */}
      <nav className="md:hidden fixed bottom-0 w-full border-t flex justify-between px-2 py-2 pb-safe z-20 shadow-[0_-4px_6px_-1px_rgba(0,0,0,0.05)] transition-colors"
           style={{ backgroundColor: colors.card, borderColor: hexToRgba(colors.text, 0.1) }}>
        <NavItem id="dashboard" icon={LayoutDashboard} label="Home" />
        <NavItem id="timer" icon={Clock} label="Timer" />
        <NavItem id="tasks" icon={CheckSquare} label="Tasks" />
        <NavItem id="notes" icon={BookOpen} label="Quiz" />
        <NavItem id="settings" icon={Settings} label="Settings" />
      </nav>

    </div>
  );
}
