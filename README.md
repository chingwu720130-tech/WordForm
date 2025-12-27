<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>字帖產生器 - 清新活潑版</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- React & Babel -->
    <script src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    <!-- PDF Libraries -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@400;700;900&family=Noto+Serif+TC:wght@400;700&display=swap');
        
        body {
            font-family: 'Noto Sans TC', sans-serif;
            background-color: #f8fafc;
        }

        .a4-page {
            background-image: linear-gradient(to bottom, #fff, #fafafa);
        }

        @media print {
            body { background: white; padding: 0; }
            .no-print { display: none !important; }
            .a4-page { 
                box-shadow: none !important; 
                margin: 0 !important; 
                border: none !important;
            }
        }

        /* 隱藏滾動條但保持功能 */
        .preview-container::-webkit-scrollbar {
            width: 8px;
        }
        .preview-container::-webkit-scrollbar-track {
            background: transparent;
        }
        .preview-container::-webkit-scrollbar-thumb {
            background: #cbd5e1;
            border-radius: 4px;
        }
    </style>
</head>
<body>
    <div id="root"></div>

    <script type="text/babel">
        const { useState, useEffect, useMemo } = React;

        // --- 內建 SVG 圖示元件 (修正 JSX 語法錯誤，使用 Fragment 包裹相鄰元素) ---
        const Icon = ({ name, size = 24, className = "" }) => {
            const icons = {
                Download: <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4m4-5 5 5 5-5m-5 5V3"/>,
                Upload: <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4m4-9 5-5 5 5m-5-5v12"/>,
                Type: (
                    <>
                        <polyline points="4 7 4 4 20 4 20 7"/>
                        <path d="M9 20h6"/>
                        <path d="M12 4v16"/>
                    </>
                ),
                RefreshCw: (
                    <>
                        <path d="M3 12a9 9 0 0 1 9-9 9.75 9.75 0 0 1 6.74 2.74L21 8"/>
                        <path d="M21 3v5h-5"/>
                        <path d="M21 12a9 9 0 0 1-9 9 9.75 9.75 0 0 1-6.74-2.74L3 16"/>
                        <path d="M3 21v-5h5"/>
                    </>
                ),
                Eye: (
                    <>
                        <path d="M1 12s4-8 11-8 11 8 11 8-4 8-11 8-11-8-11-8z"/>
                        <circle cx="12" cy="12" r="3"/>
                    </>
                ),
                Sparkles: (
                    <>
                        <path d="M9.937 15.5A2 2 0 0 0 8.5 14.063l-6.135-1.582a.5.5 0 0 1 0-.962L8.5 9.937A2 2 0 0 0 9.937 8.5l1.582-6.135a.5.5 0 0 1 .962 0L14.063 8.5A2 2 0 0 0 15.5 9.937l6.135 1.582a.5.5 0 0 1 0 .962L15.5 14.063A2 2 0 0 0 14.063 15.5L12.481 21.635a.5.5 0 0 1-.962 0L9.937 15.5ZM19 3v4"/>
                        <path d="M21 5h-4"/>
                        <path d="M3 17v4"/>
                        <path d="M5 19H1"/>
                    </>
                ),
                Brain: (
                    <>
                        <path d="M9.5 2A2.5 2.5 0 0 1 12 4.5v15a2.5 2.5 0 0 1-4.96.44 2.5 2.5 0 0 1-2.96-3.08 3 3 0 0 1-.34-5.58 2.5 2.5 0 0 1 1.32-4.24 2.5 2.5 0 0 1 4.44-2.54Z"/>
                        <path d="M14.5 2A2.5 2.5 0 0 0 12 4.5v15a2.5 2.5 0 0 0 4.96.44 2.5 2.5 0 0 0 2.96-3.08 3 3 0 0 0 .34-5.58 2.5 2.5 0 0 0-1.32-4.24 2.5 2.5 0 0 0-4.44-2.54Z"/>
                    </>
                ),
                Zap: <polygon points="13 2 3 14 12 14 11 22 21 10 12 10 13 2"/>,
                Loader2: <path d="M21 12a9 9 0 1 1-6.219-8.56"/>,
                Heart: <path d="M19 14c1.49-1.46 3-3.21 3-5.5A5.5 5.5 0 0 0 16.5 3c-1.76 0-3 .5-4.5 2-1.5-1.5-2.74-2-4.5-2A5.5 5.5 0 0 0 2 8.5c0 2.3 1.5 4.05 3 5.5l7 7Z"/>,
                Sun: (
                    <>
                        <circle cx="12" cy="12" r="4"/>
                        <path d="M12 2v2"/>
                        <path d="M12 20v2"/>
                        <path d="M4.93 4.93l1.41 1.41"/>
                        <path d="M17.66 17.66l1.41 1.41"/>
                        <path d="M2 12h2"/>
                        <path d="M20 12h2"/>
                        <path d="M4.93 19.07l1.41-1.41"/>
                        <path d="M17.66 6.34l1.41-1.41"/>
                    </>
                ),
                User: (
                    <>
                        <path d="M19 21v-2a4 4 0 0 0-4-4H9a4 4 0 0 0-4 4v2"/>
                        <circle cx="12" cy="7" r="4"/>
                    </>
                ),
                Calendar: (
                    <>
                        <rect x="3" y="4" width="18" height="18" rx="2" ry="2"/>
                        <line x1="16" y1="2" x2="16" y2="6"/>
                        <line x1="8" y1="2" x2="8" y2="6"/>
                        <line x1="3" y1="10" x2="21" y2="10"/>
                    </>
                ),
                Clock: (
                    <>
                        <circle cx="12" cy="12" r="10"/>
                        <polyline points="12 6 12 12 16 14"/>
                    </>
                )
            };

            return (
                <svg
                    xmlns="http://www.w3.org/2000/svg"
                    width={size}
                    height={size}
                    viewBox="0 0 24 24"
                    fill="none"
                    stroke="currentColor"
                    strokeWidth="2"
                    strokeLinecap="round"
                    strokeLinejoin="round"
                    className={className}
                >
                    {icons[name]}
                </svg>
            );
        };

        const App = () => {
            // --- 狀態管理 ---
            const [category, setCategory] = useState('tang_poetry');
            const [title, setTitle] = useState('新店耕莘醫院失智共照中心');
            const [logo, setLogo] = useState(null);
            const [pagesCount, setPagesCount] = useState(1);
            const [generatedPages, setGeneratedPages] = useState([]);
            const [isGenerating, setIsGenerating] = useState(false);
            
            // --- AI 相關狀態 ---
            const [isAiLoading, setIsAiLoading] = useState(false);
            const [aiPrompt, setAiPrompt] = useState('');
            const [showAiPanel, setShowAiPanel] = useState(false);

            // --- API 設定 ---
            const apiKey = ""; 

            // --- 字帖內容資料庫 ---
            const contentDB = {
                tang_poetry: [
                    "慈母手中線，遊子身上衣。臨行密密縫，意恐遲遲歸。誰言寸草心，報得三春暉。",
                    "白日依山盡，黃河入海流。欲窮千里目，更上一層樓。",
                    "春眠不覺曉，處處聞啼鳥。夜來風雨聲，花落知多少。",
                    "床前明月光，疑是地上霜。舉頭望明月，低頭思故鄉。"
                ],
                buddhist: [
                    "觀自在菩薩，行深般若波羅蜜多時，照見五蘊皆空，度一切苦厄。",
                    "舍利子，色不異空，空不異色，色即是空，空即是色。",
                    "一切有為法，如夢幻泡影，如露亦如電，應作如是觀。"
                ],
                catholic: [
                    "主是我的牧者，我實在無所缺。祂使我臥在碧綠的草地上。",
                    "凡勞苦擔重擔的人，可以到我這裡來，我就使你們得安息。"
                ],
                christian: [
                    "耶和華是我的亮光，是我的拯救，我還怕誰呢？",
                    "喜樂的心乃是良藥，憂傷的靈使骨枯乾。"
                ],
                positive: [
                    "每一天都是新的開始，給自己一個微笑，世界也會對你微笑。",
                    "心態決定高度，努力決定成敗，堅持就是勝利。"
                ],
                jingsi: [
                    "口說好話，心想好意，身行好事。",
                    "原諒別人就是善待自己。生氣是拿別人的錯誤來懲罰自己。"
                ],
                essays: [
                    "晨間的陽光穿透薄霧，灑在窗前的黃金葛上。生命的美好藏在平凡的瞬間裡。",
                    "漫步在午後的公園，微風輕拂，聽著鳥鳴，感受大地的呼吸。"
                ],
                english: [
                    "Apple (蘋果)", "Banana (香蕉)", "Cat (貓)", "Dog (狗)", "Egg (雞蛋)", 
                    "Fish (魚)", "Good (好的)", "Home (家)", "Ice (冰)", "Joy (喜悅)", 
                    "Kind (善良)", "Love (愛)", "Milk (牛奶)", "Nice (不錯)", "Open (打開)", 
                    "Pear (水梨)", "Queen (女王)", "Rice (米飯)", "Sun (太陽)", "Tea (茶)",
                    "Unit (單位)", "Visit (訪問)", "Water (水)", "Yellow (黃色)", "Zebra (斑馬)", 
                    "Bread (麵包)", "Cloud (雲朵)", "Dream (夢想)", "Earth (地球)", "Flower (花朵)",
                    "Green (綠色)", "Heart (心臟)", "Island (島嶼)", "Jump (跳躍)", "Key (鑰匙)",
                    "Leaf (樹葉)", "Moon (月亮)", "Night (夜晚)", "Orange (橘子)", "Piano (鋼琴)"
                ]
            };

            const categories = [
                { id: 'tang_poetry', name: '一、唐詩書寫練習', color: 'bg-rose-400', theme: 'rose' },
                { id: 'buddhist', name: '二、佛教經書', color: 'bg-amber-400', theme: 'amber' },
                { id: 'catholic', name: '三、天主教', color: 'bg-sky-400', theme: 'sky' },
                { id: 'christian', name: '四、基督教經書', color: 'bg-indigo-400', theme: 'indigo' },
                { id: 'positive', name: '五、正能量文章', color: 'bg-emerald-400', theme: 'emerald' },
                { id: 'jingsi', name: '六、靜思語', color: 'bg-teal-400', theme: 'teal' },
                { id: 'essays', name: '七、小品文章', color: 'bg-orange-400', theme: 'orange' },
                { id: 'english', name: '八、英文單字+中文翻譯', color: 'bg-fuchsia-400', theme: 'fuchsia' },
            ];

            // --- ✨ Gemini AI 內容生成 (含重試邏輯) ---
            const fetchWithRetry = async (url, options, retries = 5, backoff = 1000) => {
                const response = await fetch(url, options);
                if (!response.ok && retries > 0) {
                    await new Promise(resolve => setTimeout(resolve, backoff));
                    return fetchWithRetry(url, options, retries - 1, backoff * 2);
                }
                return response;
            };

            const handleAiGenerate = async () => {
                if (!aiPrompt && category !== 'english') return;
                setIsAiLoading(true);

                const systemPrompt = category === 'english' 
                    ? `您是一位失智症個案管理師。請根據主題 "${aiPrompt || '日常生活'}" 提供 18 個適合長輩練習的英文單字與中文翻譯。格式必須為 JSON 陣列，例如: ["Word (翻譯)", ...]`
                    : `您是一位字帖創作者。請根據主題 "${aiPrompt}" 創作出適合長輩書寫的中文短文或詩詞。字數請控制在 30-45 字之間。內容要正面、溫馨且朗朗上口。請直接返回文字內容。`;

                try {
                    const response = await fetchWithRetry(
                        `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`,
                        {
                            method: 'POST',
                            headers: { 'Content-Type': 'application/json' },
                            body: JSON.stringify({
                                contents: [{ parts: [{ text: systemPrompt }] }],
                                generationConfig: category === 'english' ? { responseMimeType: "application/json" } : {}
                            })
                        }
                    );

                    const result = await response.json();
                    const aiText = result.candidates?.[0]?.content?.parts?.[0]?.text;

                    if (aiText) {
                        const pages = [];
                        for (let i = 0; i < pagesCount; i++) {
                            let pageData = { id: `ai-page-${Date.now()}-${i}`, type: category, isAi: true };
                            if (category === 'english') {
                                try {
                                    pageData.items = JSON.parse(aiText).slice(0, 18);
                                } catch {
                                    pageData.items = ["Error (錯誤)", "Retry (重試)"];
                                }
                            } else {
                                pageData.content = aiText.slice(0, 42);
                            }
                            pages.push(pageData);
                        }
                        setGeneratedPages(pages);
                    }
                } catch (error) {
                    console.error("AI 生成失敗", error);
                } finally {
                    setIsAiLoading(false);
                }
            };

            const handleLogoUpload = (e) => {
                const file = e.target.files[0];
                if (file) {
                    const reader = new FileReader();
                    reader.onloadend = () => setLogo(reader.result);
                    reader.readAsDataURL(file);
                }
            };

            const generatePreview = () => {
                const pages = [];
                const pool = [...contentDB[category]];
                for (let i = 0; i < pagesCount; i++) {
                    const shuffled = [...pool].sort(() => Math.random() - 0.5);
                    let pageData = { id: `page-${Date.now()}-${i}`, type: category };
                    if (category === 'english') {
                        pageData.items = shuffled.slice(0, 18);
                    } else {
                        let selectedContent = shuffled[0];
                        if (selectedContent.length < 35 && shuffled.length > 1) {
                            selectedContent += " " + shuffled[1];
                        }
                        pageData.content = selectedContent.slice(0, 42);
                    }
                    pages.push(pageData);
                }
                setGeneratedPages(pages);
            };

            useEffect(() => {
                generatePreview();
            }, [category, pagesCount]);

            const downloadPDF = async () => {
                setIsGenerating(true);
                const { jsPDF } = window.jspdf;
                const pdf = new jsPDF('p', 'mm', 'a4');
                const pages = document.querySelectorAll('.a4-page');

                for (let i = 0; i < pages.length; i++) {
                    const canvas = await window.html2canvas(pages[i], { scale: 2.5, useCORS: true, logging: false });
                    const imgData = canvas.toDataURL('image/png');
                    if (i > 0) pdf.addPage();
                    pdf.addImage(imgData, 'PNG', 0, 0, 210, 297);
                }

                pdf.save(`字帖練習_${title}.pdf`);
                setIsGenerating(false);
            };

            // --- 中文練習格 ---
            const PracticeGrid = ({ text }) => {
                const chars = text.split('').filter(c => !/[，。？！；：、\s\(\)（）]/.test(c));
                return (
                    <div className="grid grid-cols-7 gap-x-4 gap-y-3 mt-4">
                        {chars.map((char, index) => (
                            <div key={index} className="flex flex-col items-center">
                                <div className="w-16 h-16 border-2 border-slate-100 flex items-center justify-center text-3xl text-slate-200 relative bg-slate-50/30 rounded-lg">
                                    <span className="z-10 font-serif opacity-40">{char}</span>
                                    <div className="absolute top-1/2 left-0 right-0 border-t border-dashed border-slate-200/50"></div>
                                    <div className="absolute left-1/2 top-0 bottom-0 border-l border-dashed border-slate-200/50"></div>
                                </div>
                                <div className="w-16 h-16 border-[3px] border-slate-800 mt-1 flex items-center justify-center relative bg-white rounded-lg shadow-sm">
                                    <div className="absolute top-1/2 left-0 right-0 border-t border-dotted border-slate-100"></div>
                                    <div className="absolute left-1/2 top-0 bottom-0 border-l border-dotted border-slate-100"></div>
                                </div>
                            </div>
                        ))}
                    </div>
                );
            };

            // --- 英文 18 單字 ---
            const EnglishPractice18 = ({ items = [] }) => {
                return (
                    <div className="grid grid-cols-2 gap-x-10 gap-y-6 mt-6">
                        {items.map((item, idx) => {
                            const match = item.match(/([a-zA-Z\s]+)\s\(([^)]+)\)/);
                            const english = match ? match[1].trim() : item;
                            const chinese = match ? match[2] : "";
                            return (
                                <div key={idx} className="flex flex-col gap-1.5">
                                    <div className="flex justify-between items-end border-b-2 border-slate-200 pb-1">
                                        <span className="text-2xl font-serif font-bold text-indigo-400 italic tracking-wider">{english}</span>
                                        <span className="text-base font-bold text-slate-500 bg-slate-50 px-2 py-0.5 rounded shadow-sm">{chinese}</span>
                                    </div>
                                    <div className="w-full h-8 flex flex-col justify-between pt-0.5 relative">
                                        <div className="w-full border-t border-sky-50"></div>
                                        <div className="w-full border-t border-dotted border-sky-100"></div>
                                        <div className="w-full border-t-2 border-sky-400/50"></div>
                                    </div>
                                </div>
                            );
                        })}
                    </div>
                );
            };

            const activeTheme = useMemo(() => categories.find(c => c.id === category) || categories[0], [category]);

            return (
                <div className="min-h-screen transition-colors duration-500 bg-slate-100 p-4 lg:p-8 font-sans text-slate-800">
                    <div className="max-w-7xl mx-auto">
                        
                        {/* 控制面板 */}
                        <div className="no-print bg-white/80 backdrop-blur-xl rounded-[2.5rem] shadow-2xl overflow-hidden mb-12 border border-white">
                            <div className={`${activeTheme.color} p-8 text-white flex flex-col md:flex-row justify-between items-center gap-6 transition-colors duration-500`}>
                                <div className="flex items-center gap-5">
                                    <div className="bg-white/30 p-4 rounded-3xl backdrop-blur-md">
                                        <Icon name="Sparkles" size={40} className="text-white animate-pulse" />
                                    </div>
                                    <div>
                                        <h1 className="text-3xl font-black tracking-tight drop-shadow-md">字帖產生器</h1>
                                        <p className="text-white/80 font-medium flex items-center gap-2 mt-1">
                                            <Icon name="Heart" size={16} className="fill-current" /> 設計者：吳淑菁 失智個管師
                                        </p>
                                    </div>
                                </div>
                                <div className="flex gap-4">
                                    <button 
                                        onClick={() => setShowAiPanel(!showAiPanel)}
                                        className="bg-indigo-600 hover:bg-indigo-700 text-white px-6 py-3 rounded-2xl flex items-center gap-2 transition-all font-bold shadow-lg"
                                    >
                                        <Icon name="Brain" size={20} /> ✨ AI 靈感
                                    </button>
                                    <button 
                                        onClick={downloadPDF}
                                        disabled={isGenerating}
                                        className="bg-white text-slate-900 px-8 py-3 rounded-2xl flex items-center gap-2 transition-all font-black shadow-xl hover:shadow-2xl active:scale-95 disabled:opacity-50"
                                    >
                                        {isGenerating ? 'PDF 製作中...' : <><Icon name="Download" size={20} className="text-emerald-500" /> 下載 A4 列印檔</>}
                                    </button>
                                </div>
                            </div>

                            {/* ✨ AI 生成面板 */}
                            {showAiPanel && (
                                <div className="p-8 bg-indigo-50 border-b border-indigo-100">
                                    <div className="max-w-3xl mx-auto space-y-4">
                                        <div className="flex items-center gap-3 mb-2">
                                            <div className="p-2 bg-indigo-500 rounded-lg text-white">
                                                <Icon name="Zap" size={20} />
                                            </div>
                                            <h2 className="text-xl font-black text-indigo-900">✨ AI 智慧內容助手</h2>
                                        </div>
                                        <div className="flex gap-3">
                                            <input 
                                                type="text" 
                                                value={aiPrompt}
                                                onChange={(e) => setAiPrompt(e.target.value)}
                                                placeholder={category === 'english' ? "輸入單字主題（例如：熱帶水果、家庭成員）" : "輸入字帖主題（例如：給長輩的鼓勵）"}
                                                className="flex-1 bg-white border-2 border-indigo-200 rounded-2xl px-6 py-4 outline-none focus:border-indigo-500 transition-all font-bold text-slate-700 shadow-sm"
                                            />
                                            <button 
                                                onClick={handleAiGenerate}
                                                disabled={isAiLoading}
                                                className="bg-indigo-600 hover:bg-indigo-700 text-white px-8 py-4 rounded-2xl font-black flex items-center gap-2 shadow-lg disabled:opacity-50 min-w-[160px] justify-center"
                                            >
                                                {isAiLoading ? <Icon name="Loader2" size={24} className="animate-spin" /> : <Icon name="Sparkles" size={24} />}
                                                ✨ 生成
                                            </button>
                                        </div>
                                    </div>
                                </div>
                            )}

                            <div className="p-8 grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-8 bg-gradient-to-b from-white to-slate-50/50">
                                <div className="space-y-3">
                                    <label className="text-sm font-black text-slate-400 uppercase tracking-widest ml-1">字帖類別</label>
                                    <select value={category} onChange={(e) => setCategory(e.target.value)} className="w-full bg-white border-2 border-slate-100 rounded-2xl px-5 py-4 outline-none shadow-sm focus:border-teal-400 transition-all font-bold text-slate-600">
                                        {categories.map(cat => <option key={cat.id} value={cat.id}>{cat.name}</option>)}
                                    </select>
                                </div>
                                <div className="space-y-3">
                                    <label className="text-sm font-black text-slate-400 uppercase tracking-widest ml-1">單位名稱</label>
                                    <input type="text" value={title} onChange={(e) => setTitle(e.target.value)} className="w-full bg-white border-2 border-slate-100 rounded-2xl px-5 py-4 outline-none shadow-sm focus:border-teal-400 transition-all font-bold text-slate-600" />
                                </div>
                                <div className="space-y-3">
                                    <label className="text-sm font-black text-slate-400 uppercase tracking-widest ml-1">單位 LOGO</label>
                                    <label className="flex cursor-pointer bg-white border-2 border-dashed border-slate-200 rounded-2xl px-5 py-3 hover:border-teal-400 items-center justify-center gap-3 text-slate-400 shadow-sm transition-all hover:bg-teal-50 group">
                                        <Icon name="Upload" size={20} className="group-hover:text-teal-500" />
                                        <span className="text-sm font-bold truncate">{logo ? 'Logo 已載入' : '點此上傳'}</span>
                                        <input type="file" className="hidden" accept="image/*" onChange={handleLogoUpload} />
                                    </label>
                                </div>
                                <div className="space-y-3">
                                    <label className="text-sm font-black text-slate-400 uppercase tracking-widest ml-1">產出張數</label>
                                    <div className="flex items-center gap-4 bg-white border-2 border-slate-100 rounded-2xl px-5 py-2.5">
                                        <input type="range" min="1" max="10" value={pagesCount} onChange={(e) => setPagesCount(parseInt(e.target.value) || 1)} className="flex-1 accent-teal-500" />
                                        <span className="text-xl font-black text-teal-600 min-w-[30px]">{pagesCount}</span>
                                    </div>
                                </div>
                            </div>
                        </div>

                        {/* 預覽區域 */}
                        <div className="flex flex-col items-center gap-16 pb-32 overflow-y-auto preview-container h-[calc(100vh-250px)] sm:h-auto">
                            {generatedPages.map((page, index) => (
                                <div 
                                    key={page.id}
                                    className="a4-page bg-white shadow-[0_30px_100px_rgba(0,0,0,0.08)] relative overflow-hidden"
                                    style={{ width: '210mm', minHeight: '297mm', padding: '15mm', boxSizing: 'border-box', flexShrink: 0 }}
                                >
                                    <div className="absolute inset-4 border border-slate-100 pointer-events-none rounded-sm"></div>

                                    {/* 頁首 */}
                                    <div className="flex justify-between items-center mb-6 pb-4 border-b-[6px] border-slate-900 relative">
                                        <div className="flex items-center gap-6">
                                            {logo ? <img src={logo} alt="Logo" className="h-16 w-auto object-contain max-w-[100px]" /> : <div className="h-16 w-16 bg-slate-50 rounded-2xl flex items-center justify-center text-[10px] text-slate-300 border-2 border-dashed font-bold">LOGO</div>}
                                            <div>
                                                <h2 className="text-3xl font-black text-slate-900 tracking-tight">{title}</h2>
                                                <p className="text-sm font-black text-teal-500 mt-1 uppercase tracking-widest flex items-center gap-2">
                                                    <Icon name="Sun" size={14} className="fill-current" /> 心靈與專注力訓練練習卷
                                                </p>
                                            </div>
                                        </div>
                                        <div className="text-right">
                                           <div className={`text-xs px-4 py-2 rounded-full font-black text-white shadow-lg ${page.isAi ? 'bg-indigo-500' : activeTheme.color}`}>
                                             {page.isAi ? '✨ AI 智慧生成' : activeTheme.name}
                                           </div>
                                           <p className="text-[10px] text-slate-300 mt-2 font-black tracking-tighter">PAGE {index + 1}</p>
                                        </div>
                                    </div>

                                    {/* 資訊欄 */}
                                    <div className="grid grid-cols-12 gap-10 mb-6">
                                        <div className="col-span-4 flex items-center border-b-[3px] border-slate-200 pb-2">
                                            <span className="font-black text-slate-700 text-xl mr-2">姓名：</span>
                                            <div className="flex-1"></div>
                                        </div>
                                        <div className="col-span-5 flex items-center border-b-[3px] border-slate-200 pb-2">
                                            <span className="font-black text-slate-700 text-xl mr-2">日期：</span>
                                            <span className="text-slate-200 font-black text-xl flex-1 text-center">___ 年 ___ 月 ___ 日</span>
                                        </div>
                                        <div className="col-span-3 flex items-center border-b-[3px] border-slate-200 pb-2">
                                            <span className="font-black text-slate-700 text-xl mr-2">星期：</span>
                                            <span className="text-slate-200 font-black text-xl flex-1 text-center">___</span>
                                        </div>
                                    </div>

                                    {/* 內容區 */}
                                    {page.type !== 'english' && (
                                        <div className="bg-slate-50/50 p-6 rounded-3xl border-2 border-slate-100 mb-4 shadow-inner relative overflow-hidden">
                                            <div className={`absolute top-0 left-0 w-2 h-full ${page.isAi ? 'bg-indigo-400' : 'bg-teal-400'}`}></div>
                                            <h3 className="text-2xl font-bold leading-relaxed text-slate-800 text-justify tracking-widest">
                                                {page.content}
                                            </h3>
                                        </div>
                                    )}

                                    {/* 練習區 */}
                                    <div className="mt-2">
                                        <div className="flex items-center gap-3 mb-2">
                                            <div className={`h-2.5 w-2.5 rounded-full ${page.isAi ? 'bg-indigo-400' : activeTheme.color}`}></div>
                                            <span className="font-black text-slate-700 text-lg tracking-widest">
                                                {page.type === 'english' ? '【 英文單字書寫練習 - 每頁 18 組 】' : '【 請依照上方內容在格內練習書寫 】'}
                                            </span>
                                        </div>
                                        
                                        {page.type === 'english' ? (
                                            <EnglishPractice18 items={page.items} />
                                        ) : (
                                            <PracticeGrid text={page.content} />
                                        )}
                                    </div>

                                    {/* 頁尾 */}
                                    <div className="absolute bottom-[10mm] left-[15mm] right-[15mm] pt-4 border-t-2 border-slate-100 flex justify-between items-center text-[10px] text-slate-400 font-black uppercase tracking-[0.2em]">
                                        <div className="flex items-center gap-2">
                                           <Icon name="Heart" size={10} className="text-rose-300 fill-current" />
                                           專注・靜心・維持美好認知能力
                                        </div>
                                        <span>設計者：吳淑菁 失智個管師</span>
                                    </div>
                                </div>
                            ))}
                        </div>
                    </div>
                </div>
            );
        };

        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<App />);
    </script>
</body>
</html>
