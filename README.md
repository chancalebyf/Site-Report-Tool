# Site-Report-Tool

<!DOCTYPE html>
<html lang="zh-HK">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>E&M Project Report Generator</title>
    
    <!-- React and Tailwind -->
    <script src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    <script src="https://cdn.tailwindcss.com"></script>

    <!-- Print Styles -->
    <style>
        @media print {
            @page { size: A4; margin: 0mm; }
            body { background: white; margin: 0; padding: 0; -webkit-print-color-adjust: exact; print-color-adjust: exact; }
            .print-hidden { display: none !important; }
            .page-break-after-always { page-break-after: always; }
            .break-inside-avoid { break-inside: avoid; }
            input { border: none !important; }
            /* Fix input text color during print */
            input[type="text"], textarea { color: #000 !important; }
            /* Hide placeholder texts in print */
            ::placeholder { color: transparent; }
        }
        /* Custom Scrollbar for better UX */
        ::-webkit-scrollbar { width: 8px; height: 8px; }
        ::-webkit-scrollbar-track { background: #f1f1f1; }
        ::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 4px; }
        ::-webkit-scrollbar-thumb:hover { background: #94a3b8; }
    </style>
</head>
<body class="bg-gray-100 min-h-screen">
    <div id="root"></div>

    <script type="text/babel">
        const { useState, useRef, useEffect } = React;

        // --- Simple Icon Components ---
        const IconWrapper = ({ children, size = 20, className = "", onClick }) => (
            <svg xmlns="http://www.w3.org/2000/svg" onClick={onClick} width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={className}>{children}</svg>
        );
        const Icons = {
            Camera: (p) => <IconWrapper {...p}><path d="M14.5 4h-5L7 7H4a2 2 0 0 0-2 2v9a2 2 0 0 0 2 2h16a2 2 0 0 0 2-2V9a2 2 0 0 0-2-2h-3z"/><circle cx="12" cy="13" r="3"/></IconWrapper>,
            Printer: (p) => <IconWrapper {...p}><polyline points="6 9 6 2 18 2 18 9"/><path d="M6 18H4a2 2 0 0 1-2-2v-5a2 2 0 0 1 2-2h16a2 2 0 0 1 2 2v5a2 2 0 0 1-2 2h-2"/><rect x="6" y="14" width="12" height="8"/></IconWrapper>,
            Plus: (p) => <IconWrapper {...p}><line x1="12" y1="5" x2="12" y2="19"/><line x1="5" y1="12" x2="19" y2="12"/></IconWrapper>,
            Trash2: (p) => <IconWrapper {...p}><polyline points="3 6 5 6 21 6"/><path d="M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2"/><line x1="10" y1="11" x2="10" y2="17"/><line x1="14" y1="11" x2="14" y2="17"/></IconWrapper>,
            XCircle: (p) => <IconWrapper {...p}><circle cx="12" cy="12" r="10"/><line x1="15" y1="9" x2="9" y2="15"/><line x1="9" y1="9" x2="15" y2="15"/></IconWrapper>,
            Upload: (p) => <IconWrapper {...p}><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/><polyline points="17 8 12 3 7 8"/><line x1="12" y1="3" x2="12" y2="15"/></IconWrapper>,
            User: (p) => <IconWrapper {...p}><path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"/><circle cx="12" cy="7" r="4"/></IconWrapper>,
            Calendar: (p) => <IconWrapper {...p}><rect x="3" y="4" width="18" height="18" rx="2" ry="2"/><line x1="16" y1="2" x2="16" y2="6"/><line x1="8" y1="2" x2="8" y2="6"/><line x1="3" y1="10" x2="21" y2="10"/></IconWrapper>,
            ChevronDown: (p) => <IconWrapper {...p}><polyline points="6 9 12 15 18 9"/></IconWrapper>,
            ChevronRight: (p) => <IconWrapper {...p}><polyline points="9 18 15 12 9 6"/></IconWrapper>,
            Image: (p) => <IconWrapper {...p}><rect x="3" y="3" width="18" height="18" rx="2" ry="2"/><circle cx="8.5" cy="8.5" r="1.5"/><polyline points="21 15 16 10 5 21"/></IconWrapper>,
            RefreshCw: (p) => <IconWrapper {...p}><polyline points="23 4 23 10 17 10"/><polyline points="1 20 1 14 7 14"/><path d="M3.51 9a9 9 0 0 1 14.85-3.36L23 10M1 14l4.64 4.36A9 9 0 0 0 20.49 15"/></IconWrapper>,
            Move: (p) => <IconWrapper {...p}><polyline points="5 9 2 12 5 15"/><polyline points="9 5 12 2 15 5"/><polyline points="15 19 12 22 9 19"/><polyline points="19 9 22 12 19 15"/><line x1="2" y1="12" x2="22" y2="12"/><line x1="12" y1="2" x2="12" y2="22"/></IconWrapper>,
            Copy: (p) => <IconWrapper {...p}><rect x="9" y="9" width="13" height="13" rx="2" ry="2"/><path d="M5 15H4a2 2 0 0 1-2-2V4a2 2 0 0 1 2-2h9a2 2 0 0 1 2 2v1"/></IconWrapper>,
            Edit: (p) => <IconWrapper {...p}><path d="M11 4H4a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2v-7"/><path d="M18.5 2.5a2.121 2.121 0 0 1 3 3L12 15l-4 1 1-4 9.5-9.5z"/></IconWrapper>
        };

        // Default templates
        const DEFAULT_PHOTO_TYPES = [
            'Power Analyzer (Overview)', 'Amp (L1)', 'Amp (L2)', 'Amp (L3)',
            'Volt (L1)', 'Volt (L2)', 'Volt (L3)',
            'KW (Real Power)', 'KVA (Apparent Power)', 'KVAR (Reactive Power)'
        ];

        const DEFAULT_SWITCH_NAMES = ['Q7', 'Q8', 'Q15', 'Q16'];

        const App = () => {
            const [projectInfo, setProjectInfo] = useState({
                title: 'T&C Report - Power Analysis',
                subTitle: '電力系統測試報告',
                projectName: '',
                contractNo: '',
                engineer: '',
                date: new Date().toISOString().split('T')[0],
                footerText: 'Generated by AutoReport System'
            });

            const [machines, setMachines] = useState([]);
            const [expandedMachine, setExpandedMachine] = useState(null);
            
            // Ref & State
            const dragItem = useRef(null);
            const dragOverItem = useRef(null);
            const singleInputRef = useRef(null);
            const [uploadTarget, setUploadTarget] = useState(null);

            // --- Intelligent "Add Machine" Logic ---
            const addMachine = () => {
                const newMachineId = Date.now();
                let templateStructure = {};

                // 1. If machines exist, CLONE the LAST machine's structure
                if (machines.length > 0) {
                    const lastMachine = machines[machines.length - 1];
                    templateStructure = {
                        switches: lastMachine.switches.map((s, i) => ({
                            id: `${newMachineId}_sw_${i}`,
                            name: s.name,
                            photos: s.photos.map(p => ({
                                type: p.type,
                                url: null,
                                caption: p.caption,
                                timestamp: ''
                            }))
                        }))
                    };
                } else {
                    // 2. If no machines, use DEFAULT structure
                    templateStructure = {
                        switches: DEFAULT_SWITCH_NAMES.map((name, i) => ({
                            id: `${newMachineId}_sw_${i}`,
                            name: name,
                            photos: DEFAULT_PHOTO_TYPES.map(type => ({
                                type: type,
                                url: null,
                                caption: type,
                                timestamp: ''
                            }))
                        }))
                    };
                }

                const newMachine = {
                    id: newMachineId,
                    name: `Machine ${machines.length + 1}`,
                    serialPhoto: { url: null, caption: '機身編號 (Machine Serial No.)', timestamp: '' },
                    switches: templateStructure.switches
                };

                setMachines([...machines, newMachine]);
                setExpandedMachine(newMachineId);
                
                setTimeout(() => {
                    const el = document.getElementById(`machine-${newMachineId}`);
                    if(el) el.scrollIntoView({ behavior: 'smooth', block: 'start' });
                }, 100);
            };

            const duplicateMachine = (machineId, e) => {
                e.stopPropagation();
                const targetMachine = machines.find(m => m.id === machineId);
                if (!targetMachine) return;

                const newMachineId = Date.now();
                const newMachine = {
                    ...JSON.parse(JSON.stringify(targetMachine)),
                    id: newMachineId,
                    name: `${targetMachine.name} (Copy)`,
                    serialPhoto: { ...targetMachine.serialPhoto, url: null, timestamp: '' },
                    switches: targetMachine.switches.map((s, i) => ({
                        ...s,
                        id: `${newMachineId}_sw_${i}`,
                        photos: s.photos.map(p => ({ ...p, url: null, timestamp: '' }))
                    }))
                };

                const index = machines.findIndex(m => m.id === machineId);
                const newMachines = [...machines];
                newMachines.splice(index + 1, 0, newMachine);
                setMachines(newMachines);
                setExpandedMachine(newMachineId);
            };

            // --- Dynamic Structure Editing ---

            const updateSwitchName = (machineId, switchId, newName) => {
                setMachines(prev => prev.map(m => {
                    if (m.id !== machineId) return m;
                    return {
                        ...m,
                        switches: m.switches.map(s => s.id === switchId ? { ...s, name: newName } : s)
                    };
                }));
            };

            const addSwitch = (machineId) => {
                setMachines(prev => prev.map(m => {
                    if (m.id !== machineId) return m;
                    const newSwitchId = Date.now();
                    const newSwitch = {
                        id: `${m.id}_sw_${newSwitchId}`,
                        name: `Switch ${m.switches.length + 1}`,
                        photos: Array(4).fill(null).map((_, i) => ({
                            type: `Photo ${i+1}`,
                            url: null,
                            caption: `Photo ${i+1}`,
                            timestamp: ''
                        }))
                    };
                    return { ...m, switches: [...m.switches, newSwitch] };
                }));
            };

            const removeSwitch = (machineId, switchId) => {
                // Removed window.confirm to avoid blocking issues
                setMachines(prev => prev.map(m => {
                    if (m.id !== machineId) return m;
                    return { ...m, switches: m.switches.filter(s => s.id !== switchId) };
                }));
            };

            const addPhotoSlot = (machineId, switchId) => {
                setMachines(prev => prev.map(m => {
                    if (m.id !== machineId) return m;
                    return {
                        ...m,
                        switches: m.switches.map(s => {
                            if (s.id !== switchId) return s;
                            const newIdx = s.photos.length + 1;
                            return {
                                ...s,
                                photos: [...s.photos, {
                                    type: `Photo ${newIdx}`,
                                    url: null,
                                    caption: `Photo ${newIdx}`,
                                    timestamp: ''
                                }]
                            };
                        })
                    };
                }));
            };

            const removePhotoSlot = (machineId, switchId, photoIdx) => {
                // Removed window.confirm to avoid blocking issues
                setMachines(prev => prev.map(m => {
                    if (m.id !== machineId) return m;
                    return {
                        ...m,
                        switches: m.switches.map(s => {
                            if (s.id !== switchId) return s;
                            const newPhotos = [...s.photos];
                            newPhotos.splice(photoIdx, 1);
                            return { ...s, photos: newPhotos };
                        })
                    };
                }));
            };

            // --- Upload & Drag/Drop Logic ---
            const handleSwitchUpload = (e, machineId, switchId) => {
                const files = Array.from(e.target.files);
                files.sort((a, b) => a.name.localeCompare(b.name));

                setMachines(prev => prev.map(m => {
                    if (m.id !== machineId) return m;
                    return {
                        ...m,
                        switches: m.switches.map(s => {
                            if (s.id !== switchId) return s;
                            const newPhotos = [...s.photos];
                            let fileIdx = 0;
                            // Fill slots linearly
                            for (let i = 0; i < newPhotos.length && fileIdx < files.length; i++) {
                                newPhotos[i] = {
                                    ...newPhotos[i],
                                    url: URL.createObjectURL(files[fileIdx]),
                                    timestamp: new Date().toLocaleString()
                                };
                                fileIdx++;
                            }
                            // Auto-add slots if files > slots
                            while(fileIdx < files.length) {
                                newPhotos.push({
                                    type: `Photo ${newPhotos.length + 1}`,
                                    url: URL.createObjectURL(files[fileIdx]),
                                    caption: `Photo ${newPhotos.length + 1}`,
                                    timestamp: new Date().toLocaleString()
                                });
                                fileIdx++;
                            }
                            return { ...s, photos: newPhotos };
                        })
                    };
                }));
                e.target.value = '';
            };

            const triggerSingleUpload = (machineId, switchId, photoIndex) => {
                setUploadTarget({ machineId, switchId, photoIndex });
                singleInputRef.current.click();
            };

            const handleSingleFileChange = (e) => {
                if (!e.target.files || e.target.files.length === 0 || !uploadTarget) return;
                const file = e.target.files[0];
                setMachines(prev => prev.map(m => {
                    if (m.id !== uploadTarget.machineId) return m;
                    return {
                        ...m,
                        switches: m.switches.map(s => {
                            if (s.id !== uploadTarget.switchId) return s;
                            const newPhotos = [...s.photos];
                            newPhotos[uploadTarget.photoIndex] = {
                                ...newPhotos[uploadTarget.photoIndex],
                                url: URL.createObjectURL(file),
                                timestamp: new Date().toLocaleString()
                            };
                            return { ...s, photos: newPhotos };
                        })
                    };
                }));
                e.target.value = '';
                setUploadTarget(null);
            };

            const handleDragStart = (e, machineId, switchId, index) => {
                dragItem.current = { machineId, switchId, index };
                e.dataTransfer.effectAllowed = "move";
            };
            const handleDragEnter = (e, machineId, switchId, index) => {
                dragOverItem.current = { machineId, switchId, index };
                e.preventDefault();
            };
            const handleDrop = (e) => {
                e.preventDefault();
                const source = dragItem.current;
                const destination = dragOverItem.current;
                if (!source || !destination || source.machineId !== destination.machineId || source.switchId !== destination.switchId || source.index === destination.index) {
                    dragItem.current = null; dragOverItem.current = null; return;
                }
                setMachines(prev => prev.map(m => {
                    if (m.id !== source.machineId) return m;
                    return {
                        ...m,
                        switches: m.switches.map(s => {
                            if (s.id !== source.switchId) return s;
                            const newPhotos = [...s.photos];
                            const tempUrl = newPhotos[source.index].url;
                            const tempTime = newPhotos[source.index].timestamp;
                            newPhotos[source.index] = { ...newPhotos[source.index], url: newPhotos[destination.index].url, timestamp: newPhotos[destination.index].timestamp };
                            newPhotos[destination.index] = { ...newPhotos[destination.index], url: tempUrl, timestamp: tempTime };
                            return { ...s, photos: newPhotos };
                        })
                    };
                }));
                dragItem.current = null; dragOverItem.current = null;
            };

            const handleSerialPhotoUpload = (e, machineId) => {
                const file = e.target.files[0];
                if (!file) return;
                setMachines(prev => prev.map(m => {
                    if (m.id !== machineId) return m;
                    return { ...m, serialPhoto: { ...m.serialPhoto, url: URL.createObjectURL(file), timestamp: new Date().toLocaleString() } };
                }));
                e.target.value = '';
            };

            const clearPhoto = (machineId, switchId, photoIndex) => {
                setMachines(prev => prev.map(m => {
                    if (m.id !== machineId) return m;
                    return {
                        ...m,
                        switches: m.switches.map(s => {
                            if (s.id !== switchId) return s;
                            const newPhotos = [...s.photos];
                            newPhotos[photoIndex] = { ...newPhotos[photoIndex], url: null, timestamp: '' };
                            return { ...s, photos: newPhotos };
                        })
                    };
                }));
            };
            const removeSerialPhoto = (machineId) => {
                setMachines(prev => prev.map(m => {
                    if (m.id !== machineId) return m;
                    return { ...m, serialPhoto: { ...m.serialPhoto, url: null, timestamp: '' } };
                }));
            };
            const updateCaption = (machineId, switchId, photoIndex, text) => {
                setMachines(prev => prev.map(m => {
                    if (m.id !== machineId) return m;
                    return {
                        ...m,
                        switches: m.switches.map(s => {
                            if (s.id !== switchId) return s;
                            const newPhotos = [...s.photos];
                            newPhotos[photoIndex] = { ...newPhotos[photoIndex], caption: text };
                            return { ...s, photos: newPhotos };
                        })
                    };
                }));
            };
            const toggleMachine = (id) => setExpandedMachine(expandedMachine === id ? null : id);
            const deleteMachine = (id) => { 
                // Removed window.confirm to avoid blocking issues
                setMachines(machines.filter(m => m.id !== id)); 
            };

            return (
                <div className="flex flex-col items-center p-4 sm:p-8 font-sans">
                    <input type="file" ref={singleInputRef} onChange={handleSingleFileChange} className="hidden" accept="image/*" />

                    {/* 控制台 (Control Panel) */}
                    <div className="w-full max-w-5xl bg-white rounded-xl shadow-lg p-6 mb-8 print-hidden">
                        <div className="flex flex-col md:flex-row justify-between items-start md:items-end mb-6 border-b pb-4 gap-4">
                            <div className="flex-grow w-full">
                                <label className="text-xs font-bold text-gray-400 uppercase mb-1 block">Report Title</label>
                                <input className="text-3xl font-bold text-slate-800 w-full border-none focus:ring-0 p-0 placeholder-gray-300 bg-transparent outline-none" value={projectInfo.title} onChange={e => setProjectInfo({...projectInfo, title: e.target.value})} placeholder="輸入報告標題..." />
                                <input className="text-lg text-gray-500 w-full border-none focus:ring-0 p-0 mt-1 placeholder-gray-300 bg-transparent outline-none" value={projectInfo.subTitle} onChange={e => setProjectInfo({...projectInfo, subTitle: e.target.value})} placeholder="輸入副標題..." />
                            </div>
                            <button onClick={() => window.print()} className="flex items-center gap-2 bg-slate-800 hover:bg-slate-900 text-white px-4 py-3 rounded-lg font-semibold shadow-md h-fit whitespace-nowrap">
                                <Icons.Printer size={18} /> Export PDF
                            </button>
                        </div>

                        {/* Project Info */}
                        <div className="grid grid-cols-1 md:grid-cols-4 gap-4 mb-8 bg-slate-50 p-4 rounded-lg">
                            <div className="md:col-span-2">
                                <label className="text-xs font-bold text-gray-500 uppercase">Project Name</label>
                                <input className="w-full p-2 border rounded" value={projectInfo.projectName} onChange={e => setProjectInfo({...projectInfo, projectName: e.target.value})} placeholder="e.g. Data Center Phase 1" />
                            </div>
                            <div>
                                <label className="text-xs font-bold text-gray-500 uppercase">Contract No.</label>
                                <input className="w-full p-2 border rounded" value={projectInfo.contractNo} onChange={e => setProjectInfo({...projectInfo, contractNo: e.target.value})} />
                            </div>
                            <div>
                                <label className="text-xs font-bold text-gray-500 uppercase">Date</label>
                                <input type="date" className="w-full p-2 border rounded" value={projectInfo.date} onChange={e => setProjectInfo({...projectInfo, date: e.target.value})} />
                            </div>
                             <div className="md:col-span-4 mt-2">
                                <label className="text-xs font-bold text-gray-500 uppercase flex items-center gap-2"><Icons.Edit size={12}/> Report Footer Text</label>
                                <input className="w-full p-2 border rounded mt-1 text-sm bg-white" value={projectInfo.footerText} onChange={e => setProjectInfo({...projectInfo, footerText: e.target.value})} placeholder="e.g. Generated by My Company" />
                            </div>
                        </div>

                        {/* Machine List */}
                        <div className="space-y-4">
                            {machines.map((machine, mIdx) => (
                                <div key={machine.id} id={`machine-${machine.id}`} className="border rounded-lg overflow-hidden bg-white shadow-sm transition-shadow hover:shadow-md">
                                    <div className="flex items-center justify-between p-4 bg-gray-50 cursor-pointer hover:bg-gray-100" onClick={() => toggleMachine(machine.id)}>
                                        <div className="flex items-center gap-3 overflow-hidden">
                                            {expandedMachine === machine.id ? <Icons.ChevronDown /> : <Icons.ChevronRight />}
                                            <div className="flex-grow min-w-0">
                                                <input 
                                                    className="font-bold text-lg bg-transparent border-b border-transparent hover:border-gray-300 outline-none w-full truncate" 
                                                    value={machine.name} 
                                                    onClick={(e) => e.stopPropagation()} 
                                                    onChange={(e) => { 
                                                        setMachines(prev => prev.map((m) => m.id === machine.id ? { ...m, name: e.target.value } : m)); 
                                                    }} 
                                                />
                                            </div>
                                            <span className="text-xs bg-blue-100 text-blue-800 px-2 py-1 rounded-full whitespace-nowrap">{machine.switches.reduce((acc, s) => acc + s.photos.filter(p => p.url).length, 0)} Photos</span>
                                        </div>
                                        <div className="flex items-center gap-1">
                                            <button onClick={(e) => duplicateMachine(machine.id, e)} className="text-gray-400 hover:text-blue-600 p-2" title="Duplicate Machine Structure"><Icons.Copy size={18} /></button>
                                            <button onClick={(e) => { e.stopPropagation(); deleteMachine(machine.id); }} className="text-gray-400 hover:text-red-500 p-2"><Icons.Trash2 size={18} /></button>
                                        </div>
                                    </div>

                                    {expandedMachine === machine.id && (
                                        <div className="p-4 bg-white border-t space-y-8 animate-fade-in">
                                            {/* Serial Number Section */}
                                            <div className="bg-slate-50 p-4 rounded-lg border border-slate-200">
                                                <h3 className="font-bold text-slate-700 mb-3 flex items-center gap-2"><Icons.Image size={18}/> 機身編號 / Serial No.</h3>
                                                <div className="relative w-48 h-36 border-2 border-dashed border-gray-300 hover:border-blue-400 transition-colors rounded bg-white flex flex-col items-center justify-center text-center overflow-hidden group">
                                                    {machine.serialPhoto.url ? (
                                                        <>
                                                            <img src={machine.serialPhoto.url} className="w-full h-full object-cover" />
                                                            <div className="absolute inset-0 bg-black/50 opacity-0 group-hover:opacity-100 transition-opacity flex items-center justify-center">
                                                                <button onClick={() => removeSerialPhoto(machine.id)} className="text-white bg-red-500 p-2 rounded-full"><Icons.Trash2 size={16} /></button>
                                                            </div>
                                                        </>
                                                    ) : (
                                                        <div className="text-gray-400 flex flex-col items-center cursor-pointer w-full h-full justify-center hover:text-blue-500" onClick={() => document.getElementById(`serial-upload-${machine.id}`).click()}>
                                                            <Icons.Camera size={24} /><span className="text-xs mt-1">點擊上傳</span>
                                                        </div>
                                                    )}
                                                    <input id={`serial-upload-${machine.id}`} type="file" className="hidden" accept="image/*" onChange={(e) => handleSerialPhotoUpload(e, machine.id)} />
                                                </div>
                                            </div>

                                            {/* Switches */}
                                            <div className="space-y-6">
                                                {machine.switches.map((sw, sIdx) => (
                                                    <div key={sw.id} className="border rounded-lg p-4 bg-white shadow-sm hover:border-blue-300 transition-colors relative group/switch">
                                                        <div className="flex flex-wrap justify-between items-start mb-4 gap-4">
                                                            <div className="flex items-center gap-3 flex-grow">
                                                                <span className="w-8 h-8 rounded-full bg-slate-800 text-white flex items-center justify-center font-bold text-sm shrink-0">{sIdx + 1}</span>
                                                                <div className="flex-grow">
                                                                    <label className="text-[10px] uppercase text-gray-400 font-bold block mb-1">Switch Name (Editable)</label>
                                                                    <input 
                                                                        className="font-bold text-lg text-slate-800 bg-gray-50 border-b border-gray-300 focus:border-blue-500 outline-none w-full px-1 hover:bg-white transition-colors" 
                                                                        value={sw.name}
                                                                        onChange={(e) => updateSwitchName(machine.id, sw.id, e.target.value)}
                                                                        placeholder="Enter Switch Name (e.g. Q7)"
                                                                    />
                                                                </div>
                                                            </div>
                                                            
                                                            <div className="flex items-center gap-2">
                                                                <div className="relative">
                                                                    <input type="file" id={`file-${sw.id}`} multiple accept="image/*" className="hidden" onChange={(e) => handleSwitchUpload(e, machine.id, sw.id)} />
                                                                    <label htmlFor={`file-${sw.id}`} className="flex items-center gap-2 bg-blue-50 hover:bg-blue-100 text-blue-700 text-xs font-bold px-3 py-2 rounded cursor-pointer border border-blue-200"><Icons.Upload size={14} /> Batch Upload</label>
                                                                </div>
                                                                <button onClick={() => removeSwitch(machine.id, sw.id)} className="text-red-300 hover:text-red-500 p-2" title="Remove Switch"><Icons.Trash2 size={16} /></button>
                                                            </div>
                                                        </div>
                                                        
                                                        <div className="grid grid-cols-2 sm:grid-cols-4 md:grid-cols-5 lg:grid-cols-6 gap-3">
                                                            {sw.photos.map((photo, pIdx) => (
                                                                <div key={pIdx} className={`relative aspect-square border rounded-md bg-gray-50 flex flex-col items-center justify-center text-center p-1 group/photo transition-all duration-200 ${photo.url ? 'hover:shadow-md border-blue-200 bg-white' : 'border-dashed border-gray-300'}`}
                                                                    draggable={!!photo.url}
                                                                    onDragStart={(e) => handleDragStart(e, machine.id, sw.id, pIdx)}
                                                                    onDragEnter={(e) => handleDragEnter(e, machine.id, sw.id, pIdx)}
                                                                    onDragOver={(e) => e.preventDefault()}
                                                                    onDrop={handleDrop}
                                                                    onClick={() => !photo.url && triggerSingleUpload(machine.id, sw.id, pIdx)}
                                                                >
                                                                    {photo.url ? (
                                                                        <>
                                                                            <img src={photo.url} className="w-full h-full object-cover rounded pointer-events-none" />
                                                                            <div className="absolute inset-0 bg-black/60 opacity-0 group-hover/photo:opacity-100 transition-opacity flex items-center justify-center rounded gap-2 z-10">
                                                                                <button onClick={(e) => { e.stopPropagation(); triggerSingleUpload(machine.id, sw.id, pIdx); }} className="text-white bg-blue-500 hover:bg-blue-600 p-1.5 rounded-full" title="Replace"><Icons.RefreshCw size={14} /></button>
                                                                                <button onClick={(e) => { e.stopPropagation(); clearPhoto(machine.id, sw.id, pIdx); }} className="text-white bg-orange-500 hover:bg-orange-600 p-1.5 rounded-full" title="Clear Image"><Icons.Trash2 size={14} /></button>
                                                                            </div>
                                                                            <div className="absolute top-1 right-1 text-white/70 cursor-grab z-10 opacity-0 group-hover/photo:opacity-100"><Icons.Move size={12} /></div>
                                                                            <div className="absolute bottom-0 left-0 right-0 bg-white/90 text-slate-800 text-[10px] py-1 px-1 border-t truncate font-medium">{photo.caption}</div>
                                                                        </>
                                                                    ) : (
                                                                        <div className="text-gray-300 flex flex-col items-center cursor-pointer hover:text-blue-500 w-full h-full justify-center">
                                                                            <span className="text-[10px] font-bold text-gray-400 absolute top-1 left-2">{pIdx + 1}</span>
                                                                            <Icons.Plus size={18} />
                                                                            <input 
                                                                                className="text-[10px] mt-1 text-center bg-transparent border-none w-full text-gray-500 placeholder-gray-300 focus:text-blue-600 focus:ring-0 p-0" 
                                                                                value={photo.caption} 
                                                                                onClick={(e) => e.stopPropagation()}
                                                                                onChange={(e) => updateCaption(machine.id, sw.id, pIdx, e.target.value)}
                                                                                placeholder="Caption"
                                                                            />
                                                                        </div>
                                                                    )}
                                                                    {/* Delete Slot Button - Top Right Corner */}
                                                                    <button 
                                                                        onClick={(e) => { e.stopPropagation(); removePhotoSlot(machine.id, sw.id, pIdx); }}
                                                                        className="absolute -top-2 -right-2 bg-red-100 text-red-500 rounded-full p-0.5 opacity-0 group-hover/photo:opacity-100 transition-opacity hover:bg-red-200 shadow-sm z-20"
                                                                        title="Remove Slot"
                                                                    >
                                                                        <Icons.XCircle size={16} />
                                                                    </button>
                                                                </div>
                                                            ))}
                                                            
                                                            {/* Add Photo Slot Button */}
                                                            <button 
                                                                onClick={() => addPhotoSlot(machine.id, sw.id)}
                                                                className="aspect-square border-2 border-dashed border-gray-200 rounded-md flex flex-col items-center justify-center text-gray-400 hover:border-green-400 hover:text-green-500 hover:bg-green-50 transition-all"
                                                                title="Add Photo Slot"
                                                            >
                                                                <Icons.Plus size={24} />
                                                                <span className="text-[10px] font-bold mt-1">Add Slot</span>
                                                            </button>
                                                        </div>
                                                    </div>
                                                ))}
                                                <button onClick={() => addSwitch(machine.id)} className="w-full py-2 border-2 border-dashed border-slate-200 rounded-lg text-slate-400 hover:border-slate-400 hover:text-slate-600 transition-colors text-sm font-bold flex items-center justify-center gap-2">
                                                    <Icons.Plus size={16} /> Add Another Switch Group
                                                </button>
                                            </div>
                                        </div>
                                    )}
                                </div>
                            ))}
                            
                            {/* Main Add Button */}
                            <button onClick={addMachine} className="w-full py-6 border-2 border-dashed border-blue-300 bg-blue-50 text-blue-600 rounded-xl hover:bg-blue-100 hover:border-blue-400 transition-all flex flex-col items-center justify-center gap-2 shadow-sm group">
                                <div className="bg-white p-2 rounded-full shadow-sm group-hover:scale-110 transition-transform"><Icons.Plus size={24} /></div>
                                <span className="font-bold text-lg">
                                    {machines.length === 0 ? "Start: Add First Machine" : "Clone & Add New Machine"}
                                </span>
                                {machines.length > 0 && <span className="text-xs text-blue-400">Copies structure (switches & photos) from the last machine</span>}
                            </button>
                        </div>
                    </div>

                    {/* PDF Preview / Print Area */}
                    <div id="report-content" className="w-full max-w-[210mm] bg-white shadow-2xl print:shadow-none print:w-full">
                        {/* Cover Page */}
                        <div className="p-[20mm] h-[297mm] flex flex-col justify-between border-b print:border-none relative page-break-after-always">
                            <div className="text-right border-b-4 border-blue-900 pb-4">
                                <h2 className="text-4xl font-black text-blue-900 tracking-wider break-words">{projectInfo.title}</h2>
                                <h3 className="text-2xl text-gray-500 break-words">{projectInfo.subTitle}</h3>
                            </div>
                            <div className="flex-grow flex flex-col justify-center py-20">
                                <div className="space-y-8">
                                    <div><span className="text-sm uppercase tracking-widest text-gray-400 block mb-1 font-bold">Project Name</span><div className="text-3xl font-bold border-l-4 border-blue-600 pl-4">{projectInfo.projectName || '---'}</div></div>
                                    <div><span className="text-sm uppercase tracking-widest text-gray-400 block mb-1 font-bold">Contract No.</span><div className="text-xl font-semibold">{projectInfo.contractNo || '---'}</div></div>
                                    <div><span className="text-sm uppercase tracking-widest text-gray-400 block mb-1 font-bold">Total Scope</span><div className="text-xl font-semibold">{machines.length} Machines</div></div>
                                    <div className="grid grid-cols-2 gap-10 mt-12">
                                        <div><span className="text-sm uppercase tracking-widest text-gray-400 block mb-1 font-bold">Engineer</span><div className="text-lg font-medium flex items-center gap-2"><Icons.User size={18}/> {projectInfo.engineer || '---'}</div></div>
                                        <div><span className="text-sm uppercase tracking-widest text-gray-400 block mb-1 font-bold">Date</span><div className="text-lg font-medium flex items-center gap-2"><Icons.Calendar size={18}/> {projectInfo.date}</div></div>
                                    </div>
                                </div>
                            </div>
                            <div className="mt-auto pt-10 border-t flex justify-between items-end italic text-gray-400 text-sm">
                                <div>{projectInfo.footerText}</div><div>Page 1</div>
                            </div>
                        </div>

                        {/* Content Pages */}
                        <div className="p-[15mm]">
                            {machines.map((machine, mIdx) => (
                                <div key={machine.id}>
                                    <div className="mb-6 border-b-2 border-black pb-2 mt-4 break-inside-avoid">
                                        <h2 className="text-2xl font-bold text-slate-800 uppercase flex justify-between items-end">
                                            {machine.name} <span className="text-sm font-normal text-gray-500 normal-case">Machine {mIdx + 1} of {machines.length}</span>
                                        </h2>
                                    </div>

                                    {machine.serialPhoto.url && (
                                        <div className="mb-8 break-inside-avoid">
                                            <div className="flex items-start gap-4 p-4 bg-slate-50 border rounded-lg">
                                                <div className="w-1/3 aspect-[4/3] border bg-white"><img src={machine.serialPhoto.url} className="w-full h-full object-contain" /></div>
                                                <div className="flex-grow pt-2"><h3 className="font-bold text-lg text-slate-800 border-l-4 border-blue-600 pl-3">機身編號 / Serial No.</h3><p className="text-gray-500 mt-2 pl-4 text-sm">Recorded: {machine.serialPhoto.timestamp}</p></div>
                                            </div>
                                        </div>
                                    )}

                                    {machine.switches.map((sw, sIdx) => (
                                        <div key={sw.id} className="mb-8 break-inside-avoid">
                                            <div className="bg-slate-100 p-2 mb-4 rounded flex justify-between items-center border-l-4 border-blue-600">
                                                <h3 className="font-bold text-lg text-slate-700 pl-2">{sw.name}</h3>
                                                <span className="text-xs text-gray-500 px-2">{sw.photos.filter(p=>p.url).length} Photos</span>
                                            </div>
                                            <div className="grid grid-cols-2 gap-x-4 gap-y-6">
                                                {sw.photos.map((photo, pIdx) => (
                                                    photo.url && (
                                                        <div key={pIdx} className="flex flex-col break-inside-avoid">
                                                            <div className="aspect-[4/3] w-full border border-gray-300 bg-gray-50 mb-1 overflow-hidden">
                                                                <img src={photo.url} className="w-full h-full object-contain" />
                                                            </div>
                                                            <div className="flex gap-2 items-start">
                                                                <div className="bg-slate-800 text-white text-[10px] font-bold px-1.5 py-1 rounded min-w-[20px] text-center">{pIdx + 1}</div>
                                                                <div className="flex-grow"><input className="w-full text-sm font-semibold text-slate-800 bg-transparent border-none p-0 focus:ring-0" value={photo.caption} onChange={(e) => updateCaption(machine.id, sw.id, pIdx, e.target.value)} /></div>
                                                            </div>
                                                        </div>
                                                    )
                                                ))}
                                            </div>
                                        </div>
                                    ))}
                                    <div className="page-break-after-always"></div>
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
