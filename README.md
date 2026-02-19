<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>EDF J7 Pro - Sistema Escolar</title>
    <!-- Bibliotecas Essenciais -->
    <script crossorigin src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
    <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@400;600;800&display=swap');
        
        * { 
            margin: 0; 
            padding: 0; 
            box-sizing: border-box; 
            -webkit-tap-highlight-color: transparent;
        }

        body { 
            font-family: 'Plus Jakarta Sans', sans-serif; 
            overflow: hidden;
            background-color: #f0fdf4;
        }

        /* Custom Scrollbar para o WebView */
        ::-webkit-scrollbar {
            width: 6px;
        }
        ::-webkit-scrollbar-track {
            background: #f1f1f1;
        }
        ::-webkit-scrollbar-thumb {
            background: #166534;
            border-radius: 10px;
        }

        .sidebar-transition {
            transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
        }
    </style>
</head>
<body>
    <div id="root"></div>

    <script type="text/babel">
        const { useState, useMemo, useRef, useEffect } = React;

        // Ícones em SVG para evitar dependências de fontes externas
        const Icon = ({ name, className = "w-5 h-5" }) => {
            const icons = {
                dashboard: <svg fill="none" viewBox="0 0 24 24" stroke="currentColor"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M4 6a2 2 0 012-2h2a2 2 0 012 2v2a2 2 0 01-2 2H6a2 2 0 01-2-2V6zM14 6a2 2 0 012-2h2a2 2 0 012 2v2a2 2 0 01-2 2h-2a2 2 0 01-2-2V6zM4 16a2 2 0 012-2h2a2 2 0 012 2v2a2 2 0 01-2 2H6a2 2 0 01-2-2v-2zM14 16a2 2 0 012-2h2a2 2 0 012 2v2a2 2 0 01-2 2h-2a2 2 0 01-2-2v-2z" /></svg>,
                users: <svg fill="none" viewBox="0 0 24 24" stroke="currentColor"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M12 4.354a4 4 0 110 5.292M15 21H3v-1a6 6 0 0112 0v1zm0 0h6v-1a6 6 0 00-9-5.197M13 7a4 4 0 11-8 0 4 4 0 018 0z" /></svg>,
                plus: <svg fill="none" viewBox="0 0 24 24" stroke="currentColor"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M12 4v16m8-8H4" /></svg>,
                trash: <svg fill="none" viewBox="0 0 24 24" stroke="currentColor"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" /></svg>,
                camera: <svg fill="none" viewBox="0 0 24 24" stroke="currentColor"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M3 9a2 2 0 012-2h.93a2 2 0 001.664-.89l.812-1.22A2 2 0 0110.07 4h3.86a2 2 0 011.664.89l.812 1.22A2 2 0 0018.07 7H19a2 2 0 012 2v9a2 2 0 01-2 2H5a2 2 0 01-2-2V9z" /><path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M15 13a3 3 0 11-6 0 3 3 0 016 0z" /></svg>,
                x: <svg fill="none" viewBox="0 0 24 24" stroke="currentColor"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M6 18L18 6M6 6l18 18" /></svg>
            };
            return <div className={className}>{icons[name] || '•'}</div>;
        };

        const INITIAL_STUDENTS = [
            { id: 1, name: 'Ana Silva', serie: '1ª Série', turma: 'A' },
            { id: 2, name: 'Bruno Santos', serie: '1ª Série', turma: 'B' },
            { id: 3, name: 'Carla Oliveira', serie: '2ª Série', turma: 'A' },
        ];

        function App() {
            const [activeTab, setActiveTab] = useState('dashboard');
            const [selectedSerie, setSelectedSerie] = useState('1ª Série');
            const [selectedTurma, setSelectedTurma] = useState('A');
            const [selectedBimestre, setSelectedBimestre] = useState('1');
            const [students, setStudents] = useState(INITIAL_STUDENTS);
            const [profileImage, setProfileImage] = useState(null);
            const [showAddModal, setShowAddModal] = useState(false);
            const [newStudentName, setNewStudentName] = useState('');
            const fileInputRef = useRef(null);

            const filteredStudents = useMemo(() => {
                return students.filter(s => s.serie === selectedSerie && s.turma === selectedTurma);
            }, [students, selectedSerie, selectedTurma]);

            const handleImageChange = (e) => {
                const file = e.target.files[0];
                if (!file) return;
                if (file.size > 2 * 1024 * 1024) {
                    alert('Arquivo muito grande. Máximo 2MB');
                    return;
                }
                const reader = new FileReader();
                reader.onloadend = () => setProfileImage(reader.result);
                reader.readAsDataURL(file);
            };

            const handleAddStudent = (e) => {
                e.preventDefault();
                const name = newStudentName.trim();
                if (name.length < 3) {
                    alert('Nome muito curto');
                    return;
                }
                const newStudent = { 
                    id: Date.now(), 
                    name, 
                    serie: selectedSerie, 
                    turma: selectedTurma 
                };
                setStudents([...students, newStudent]);
                setNewStudentName('');
                setShowAddModal(false);
            };

            return (
                <div className="flex h-screen bg-emerald-50 text-slate-900 overflow-hidden">
                    {/* Sidebar */}
                    <aside className="hidden md:flex w-64 bg-green-950 text-white flex-col p-5 space-y-6 shadow-2xl">
                        <div className="flex flex-col items-center space-y-4 pt-2">
                            <div className="relative group cursor-pointer" onClick={() => fileInputRef.current?.click()}>
                                <div className="w-20 h-20 bg-green-900 rounded-3xl border-2 border-green-800 flex items-center justify-center overflow-hidden hover:border-green-400 transition-all">
                                    {profileImage ? (
                                        <img src={profileImage} alt="Perfil" className="w-full h-full object-cover" />
                                    ) : (
                                        <Icon name="camera" className="w-10 h-10 text-green-700" />
                                    )}
                                </div>
                                <input type="file" ref={fileInputRef} onChange={handleImageChange} className="hidden" accept="image/*" />
                            </div>
                            <div className="text-center">
                                <span className="text-2xl font-black text-green-400">EDF J7</span>
                                <p className="text-[8px] uppercase font-bold text-green-600 tracking-widest">Pro Version</p>
                            </div>
                        </div>

                        <nav className="flex-1 space-y-1">
                            {[
                                { id: 'dashboard', label: 'Painel Geral', icon: 'dashboard' },
                                { id: 'alunos', label: 'Gestão de Alunos', icon: 'users' }
                            ].map(item => (
                                <button
                                    key={item.id}
                                    onClick={() => setActiveTab(item.id)}
                                    className={`w-full flex items-center space-x-3 p-3 rounded-xl transition-all ${
                                        activeTab === item.id ? 'bg-green-600 text-white shadow-lg' : 'text-green-100/50 hover:bg-green-800'
                                    }`}
                                >
                                    <Icon name={item.icon} />
                                    <span className="text-sm font-semibold">{item.label}</span>
                                </button>
                            ))}
                        </nav>
                    </aside>

                    {/* Main Content */}
                    <main className="flex-1 flex flex-col min-w-0 overflow-hidden">
                        <header className="bg-white border-b border-green-100 px-4 md:px-8 py-4 flex items-center justify-between shadow-sm">
                            <div className="flex items-center space-x-4">
                                <div className="md:hidden w-10 h-10 bg-green-900 rounded-lg flex items-center justify-center text-green-400 font-bold">J7</div>
                                <div>
                                    <h1 className="text-lg font-bold text-green-900 hidden md:block">Sistema de Gestão</h1>
                                    <p className="text-[10px] text-slate-400 uppercase font-bold">{selectedSerie} - Turma {selectedTurma}</p>
                                </div>
                            </div>
                            <button 
                                onClick={() => setShowAddModal(true)}
                                className="bg-green-600 hover:bg-green-700 text-white p-2 md:px-6 md:py-2.5 rounded-xl font-bold text-sm shadow-md flex items-center space-x-2"
                            >
                                <Icon name="plus" className="w-4 h-4" />
                                <span className="hidden md:inline">Novo Aluno</span>
                            </button>
                        </header>

                        <div className="flex-1 overflow-y-auto p-4 md:p-8">
                            {activeTab === 'dashboard' ? (
                                <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
                                    <div className="md:col-span-2 bg-white p-6 rounded-2xl shadow-sm border border-green-50">
                                        <h2 className="font-bold text-slate-700 mb-4">Visão Geral</h2>
                                        <div className="h-48 bg-emerald-50 rounded-xl border-2 border-dashed border-emerald-100 flex items-center justify-center text-emerald-300 italic">
                                            Gráficos Ativos
                                        </div>
                                    </div>
                                    <div className="bg-green-900 p-6 rounded-2xl text-white shadow-xl">
                                        <p className="text-xs font-bold text-green-400 uppercase">Total de Alunos</p>
                                        <p className="text-5xl font-black mt-2">{filteredStudents.length}</p>
                                        <div className="mt-6 pt-6 border-t border-green-800">
                                            <p className="text-xs font-bold text-green-400 uppercase">Frequência Média</p>
                                            <p className="text-3xl font-bold">94.2%</p>
                                        </div>
                                    </div>
                                </div>
                            ) : (
                                <div className="bg-white rounded-2xl shadow-sm border border-green-100 overflow-hidden">
                                    <table className="w-full text-left border-collapse">
                                        <thead className="bg-green-50 text-green-900 text-xs font-bold uppercase">
                                            <tr>
                                                <th className="px-6 py-4">Aluno</th>
                                                <th className="px-6 py-4 hidden md:table-cell">Série/Turma</th>
                                                <th className="px-6 py-4 text-center">Ações</th>
                                            </tr>
                                        </thead>
                                        <tbody className="divide-y divide-green-50">
                                            {filteredStudents.map(student => (
                                                <tr key={student.id} className="hover:bg-green-50/50 transition-colors">
                                                    <td className="px-6 py-4 font-semibold text-slate-700">{student.name}</td>
                                                    <td className="px-6 py-4 hidden md:table-cell text-slate-500">{student.serie} {student.turma}</td>
                                                    <td className="px-6 py-4 text-center">
                                                        <button 
                                                            onClick={() => setStudents(students.filter(s => s.id !== student.id))}
                                                            className="text-red-400 hover:text-red-600 p-2"
                                                        >
                                                            <Icon name="trash" className="w-5 h-5" />
                                                        </button>
                                                    </td>
                                                </tr>
                                            ))}
                                        </tbody>
                                    </table>
                                </div>
                            )}
                        </div>
                    </main>

                    {/* Modal Adicionar */}
                    {showAddModal && (
                        <div className="fixed inset-0 bg-green-950/90 backdrop-blur-sm flex items-center justify-center p-4 z-50">
                            <div className="bg-white w-full max-w-md rounded-3xl p-6 shadow-2xl scale-in">
                                <div className="flex justify-between items-center mb-6">
                                    <h3 className="text-xl font-black text-green-900 uppercase">Nova Matrícula</h3>
                                    <button onClick={() => setShowAddModal(false)} className="text-slate-400 hover:text-slate-600">
                                        <Icon name="x" className="w-6 h-6" />
                                    </button>
                                </div>
                                <form onSubmit={handleAddStudent} className="space-y-4">
                                    <div>
                                        <label className="text-[10px] font-bold text-slate-400 uppercase mb-1 block">Nome do Aluno</label>
                                        <input 
                                            autoFocus
                                            type="text" 
                                            value={newStudentName}
                                            onChange={(e) => setNewStudentName(e.target.value)}
                                            className="w-full bg-slate-50 border-2 border-slate-100 rounded-xl p-3 outline-none focus:border-green-500 transition-all font-semibold"
                                            placeholder="Ex: Carlos Andrade"
                                        />
                                    </div>
                                    <div className="grid grid-cols-2 gap-4">
                                        <div className="bg-slate-50 p-3 rounded-xl border border-slate-100">
                                            <p className="text-[8px] font-bold text-slate-400 uppercase">Série Selecionada</p>
                                            <p className="text-sm font-bold">{selectedSerie}</p>
                                        </div>
                                        <div className="bg-slate-50 p-3 rounded-xl border border-slate-100">
                                            <p className="text-[8px] font-bold text-slate-400 uppercase">Turma</p>
                                            <p className="text-sm font-bold">{selectedTurma}</p>
                                        </div>
                                    </div>
                                    <button type="submit" className="w-full bg-green-600 hover:bg-green-700 text-white font-bold py-4 rounded-xl shadow-lg mt-4">
                                        Confirmar Cadastro
                                    </button>
                                </form>
                            </div>
                        </div>
                    )}
                </div>
            );
        }

        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<App />);
    </script>
</body>
</html>
