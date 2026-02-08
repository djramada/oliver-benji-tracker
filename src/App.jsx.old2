import React, { useState, useEffect } from 'react';
import { Trophy, CheckCircle, Circle, RefreshCw, Filter, Cloud, CloudOff, User } from 'lucide-react';
import { initializeApp } from 'firebase/app';
import { 
  getAuth, 
  onAuthStateChanged, 
  signInAnonymously, 
  signInWithCustomToken 
} from 'firebase/auth';
import { 
  getFirestore, 
  doc, 
  setDoc, 
  onSnapshot, 
  collection 
} from 'firebase/firestore';

// --- CONFIGURACIÓN DE FIREBASE ---
const firebaseConfig = {
  apiKey: "AIzaSyCZR6KXJTdgmUM3XsfLtfZEjCJMXKSnr6w",
  authDomain: "oliver-benji-tracker-db.firebaseapp.com",
  projectId: "oliver-benji-tracker-db",
  storageBucket: "oliver-benji-tracker-db.firebasestorage.app",
  messagingSenderId: "295424223271",
  appId: "1:295424223271:web:1c1c6635049c78e24e4602"
};

// Initialize Firebase
const app = initializeApp(firebaseConfig);



const auth = getAuth(app);
const db = getFirestore(app);
const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';

export default function App() {
  const [episodes, setEpisodes] = useState(initialEpisodes);
  const [filter, setFilter] = useState('all'); // all, pending, watched
  const [user, setUser] = useState(null);
  const [isSyncing, setIsSyncing] = useState(false);
  const [isOnline, setIsOnline] = useState(false);

  // 1. Manejo de Autenticación (MANDATORIO: Primero Auth, luego DB)
  useEffect(() => {
    const initAuth = async () => {
      try {
        if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
          await signInWithCustomToken(auth, __initial_auth_token);
        } else {
          await signInAnonymously(auth);
        }
      } catch (error) {
        console.error("Error de autenticación:", error);
      }
    };

    initAuth();
    const unsubscribe = onAuthStateChanged(auth, (currentUser) => {
      setUser(currentUser);
      setIsOnline(!!currentUser);
    });
    return () => unsubscribe();
  }, []);

  // 2. Sincronización con Base de Datos
  useEffect(() => {
    if (!user) return;

    // Ruta segura para guardar datos privados del usuario
    // artifacts/{appId}/users/{userId}/data/progress
    const docRef = doc(db, 'artifacts', appId, 'users', user.uid, 'data', 'progress');

    setIsSyncing(true);
    const unsubscribe = onSnapshot(docRef, (docSnap) => {
      setIsSyncing(false);
      if (docSnap.exists()) {
        const data = docSnap.data();
        const watchedIds = new Set(data.watchedIds || []);
        
        // Actualizamos el estado local mezclando los datos de la DB con la lista estática
        setEpisodes(prev => prev.map(ep => ({
          ...ep,
          watched: watchedIds.has(ep.id)
        })));
      }
    }, (error) => {
      console.error("Error leyendo datos:", error);
      setIsSyncing(false);
    });

    return () => unsubscribe();
  }, [user]);

  // Manejar el click en un capítulo y guardar en la Nube
  const toggleEpisode = async (id) => {
    // 1. Actualización optimista (rápida en UI)
    const newEpisodes = episodes.map(ep => 
      ep.id === id ? { ...ep, watched: !ep.watched } : ep
    );
    setEpisodes(newEpisodes);

    // 2. Guardar en Base de Datos si hay usuario
    if (user) {
      const watchedIds = newEpisodes
        .filter(ep => ep.watched)
        .map(ep => ep.id);
      
      try {
        const docRef = doc(db, 'artifacts', appId, 'users', user.uid, 'data', 'progress');
        await setDoc(docRef, { watchedIds }, { merge: true });
      } catch (e) {
        console.error("Error guardando progreso:", e);
      }
    }
  };

  // Función para resetear todo (en la nube también)
  const resetAll = async () => {
    if (window.confirm('¿Estás seguro de que quieres borrar todo tu progreso guardado?')) {
      const resetEpisodes = initialEpisodes.map(ep => ({ ...ep, watched: false }));
      setEpisodes(resetEpisodes); // UI inmediata
      
      if (user) {
        try {
          const docRef = doc(db, 'artifacts', appId, 'users', user.uid, 'data', 'progress');
          await setDoc(docRef, { watchedIds: [] });
        } catch (e) {
          console.error("Error reseteando:", e);
        }
      }
    }
  };

  // Calcular progreso
  const watchedCount = episodes.filter(ep => ep.watched).length;
  const progressPercentage = Math.round((watchedCount / episodes.length) * 100);

  // Filtrar episodios para mostrar
  const displayedEpisodes = episodes.filter(ep => {
    if (filter === 'watched') return ep.watched;
    if (filter === 'pending') return !ep.watched;
    return true;
  });

  return (
    <div className="min-h-screen bg-gray-100 font-sans text-gray-800">
      
      {/* Header Estilo Campo de Fútbol */}
      <header className="bg-green-600 text-white sticky top-0 z-10 shadow-lg border-b-4 border-white">
        <div className="max-w-3xl mx-auto px-4 py-4">
          <div className="flex justify-between items-center mb-2">
            <div className="flex items-center gap-2">
              <Trophy className="h-6 w-6 text-yellow-300" />
              <h1 className="text-xl md:text-2xl font-bold tracking-tight">Oliver y Benji (1983)</h1>
            </div>
            
            {/* Indicador de Estado de Nube */}
            <div className="flex items-center gap-2 text-xs md:text-sm bg-green-700/50 px-2 py-1 rounded">
              {isOnline ? (
                <span className="flex items-center gap-1 text-green-100">
                  <Cloud size={14} /> 
                  {isSyncing ? "Guardando..." : "Guardado"}
                </span>
              ) : (
                <span className="flex items-center gap-1 text-red-200">
                  <CloudOff size={14} /> Offline
                </span>
              )}
            </div>
          </div>

          <div className="flex justify-between items-center text-sm mb-1 px-1">
            <span className="font-semibold text-green-100">Progreso total</span>
            <span className="font-bold text-white">{watchedCount} / {episodes.length}</span>
          </div>

          {/* Barra de Progreso */}
          <div className="w-full bg-green-800 rounded-full h-4 border border-green-500 overflow-hidden relative shadow-inner">
            <div 
              className="bg-yellow-400 h-full transition-all duration-500 ease-out flex items-center justify-end pr-1 shadow"
              style={{ width: `${progressPercentage}%` }}
            >
              {progressPercentage > 5 && (
                <span className="text-[10px] font-bold text-green-900 leading-none drop-shadow-sm">{progressPercentage}%</span>
              )}
            </div>
          </div>
        </div>
      </header>

      {/* Controles y Filtros */}
      <div className="max-w-3xl mx-auto px-4 py-4 sticky top-[108px] z-10 bg-gray-100/95 backdrop-blur-sm border-b border-gray-200">
        <div className="flex justify-between items-center gap-2">
          <div className="flex bg-white rounded-lg shadow-sm p-1 border border-gray-200 overflow-x-auto">
            <button 
              onClick={() => setFilter('all')}
              className={`px-3 py-1.5 text-sm rounded-md transition-colors whitespace-nowrap ${filter === 'all' ? 'bg-blue-600 text-white font-medium shadow-sm' : 'text-gray-600 hover:bg-gray-100'}`}
            >
              Todos
            </button>
            <button 
              onClick={() => setFilter('pending')}
              className={`px-3 py-1.5 text-sm rounded-md transition-colors whitespace-nowrap ${filter === 'pending' ? 'bg-blue-600 text-white font-medium shadow-sm' : 'text-gray-600 hover:bg-gray-100'}`}
            >
              Pendientes
            </button>
            <button 
              onClick={() => setFilter('watched')}
              className={`px-3 py-1.5 text-sm rounded-md transition-colors whitespace-nowrap ${filter === 'watched' ? 'bg-blue-600 text-white font-medium shadow-sm' : 'text-gray-600 hover:bg-gray-100'}`}
            >
              Vistos
            </button>
          </div>
          
          <button 
            onClick={resetAll}
            className="flex-shrink-0 text-gray-500 hover:text-red-600 p-2 rounded-full hover:bg-red-50 transition-colors border border-transparent hover:border-red-100"
            title="Reiniciar progreso"
          >
            <RefreshCw size={18} />
          </button>
        </div>
        
        {/* User ID Display (Pequeño footer técnico para identificar sesión) */}
        {user && (
          <div className="mt-2 text-[10px] text-gray-400 flex items-center justify-end gap-1">
            <User size={10} />
            ID de Usuario: {user.uid.slice(0, 8)}...
          </div>
        )}
      </div>

      {/* Lista de Episodios */}
      <main className="max-w-3xl mx-auto p-4 pb-20">
        <div className="bg-white rounded-xl shadow-sm border border-gray-200 overflow-hidden">
          {displayedEpisodes.length === 0 ? (
            <div className="p-8 text-center text-gray-500">
              <p>No hay episodios que mostrar con este filtro.</p>
              {filter !== 'all' && (
                <button 
                  onClick={() => setFilter('all')}
                  className="mt-4 text-blue-600 hover:text-blue-800 text-sm font-medium underline"
                >
                  Ver todos los episodios
                </button>
              )}
            </div>
          ) : (
            <ul className="divide-y divide-gray-100">
              {displayedEpisodes.map((ep) => (
                <li 
                  key={ep.id} 
                  onClick={() => toggleEpisode(ep.id)}
                  className={`
                    group flex items-center p-4 cursor-pointer transition-all duration-200 hover:bg-blue-50/50
                    ${ep.watched ? 'bg-gray-50' : 'bg-white'}
                  `}
                >
                  <div className="flex-shrink-0 mr-4">
                    <button 
                      className={`
                        w-6 h-6 rounded-full border-2 flex items-center justify-center transition-all duration-300
                        ${ep.watched 
                          ? 'bg-green-500 border-green-500 text-white scale-110 shadow-sm' 
                          : 'bg-white border-gray-300 text-transparent group-hover:border-blue-400 group-hover:scale-105'}
                      `}
                    >
                      <CheckCircle size={14} fill="currentColor" className={ep.watched ? 'opacity-100' : 'opacity-0'} />
                    </button>
                  </div>
                  
                  <div className="flex-1 min-w-0">
                    <div className="flex flex-col sm:flex-row sm:items-baseline gap-1 sm:gap-0">
                      <span className={`
                        text-xs sm:text-sm font-mono font-bold mr-3 transition-colors duration-200 w-12 flex-shrink-0
                        ${ep.watched ? 'text-gray-400' : 'text-blue-600'}
                      `}>
                        Cap {ep.id}
                      </span>
                      <h3 className={`
                        text-sm sm:text-base font-medium leading-tight truncate-multiline transition-all duration-200
                        ${ep.watched ? 'text-gray-400 line-through decoration-gray-400 decoration-1' : 'text-gray-900'}
                      `}>
                        {ep.title}
                      </h3>
                    </div>
                  </div>
                </li>
              ))}
            </ul>
          )}
        </div>
      </main>
    </div>
  );
}

// Datos: Lista completa de episodios en Español
const initialEpisodes = [
  { id: 1, title: "El gran desafío", watched: false },
  { id: 2, title: "Las dos escuelas rivales", watched: false },
  { id: 3, title: "El nuevo equipo", watched: false },
  { id: 4, title: "Mi mejor amigo", watched: false },
  { id: 5, title: "Empieza el torneo", watched: false },
  { id: 6, title: "El regreso de papá", watched: false },
  { id: 7, title: "El espectáculo debe continuar", watched: false },
  { id: 8, title: "Un dúo perfecto", watched: false },
  { id: 9, title: "La última oportunidad", watched: false },
  { id: 10, title: "Nos llevaremos a Oliver", watched: false },
  { id: 11, title: "El rival inesperado", watched: false },
  { id: 12, title: "Dirigidos hacia el triunfo", watched: false },
  { id: 13, title: "El partido en el barro", watched: false },
  { id: 14, title: "La joven promesa", watched: false },
  { id: 15, title: "Una prueba de carácter", watched: false },
  { id: 16, title: "El nuevo capitán", watched: false },
  { id: 17, title: "Empieza el campeonato nacional", watched: false },
  { id: 18, title: "Duelo de capitanes", watched: false },
  { id: 19, title: "La mejor defensa es un buen ataque", watched: false },
  { id: 20, title: "Una dura batalla", watched: false },
  { id: 21, title: "El gol de la victoria", watched: false },
  { id: 22, title: "Los gemelos maravillosos", watched: false },
  { id: 23, title: "El gol en propia meta", watched: false },
  { id: 24, title: "Amigo y rival", watched: false },
  { id: 25, title: "El mejor portero del campeonato", watched: false },
  { id: 26, title: "Un frágil campeón", watched: false },
  { id: 27, title: "Pugna en las semifinales", watched: false },
  { id: 28, title: "Los bravos jugadores del norte", watched: false },
  { id: 29, title: "Una feroz confrontación", watched: false },
  { id: 30, title: "El príncipe herido", watched: false },
  { id: 31, title: "Una pelea brillante", watched: false },
  { id: 32, title: "Oliver contra la trampa del fuera de juego", watched: false },
  { id: 33, title: "El final del viaje", watched: false },
  { id: 34, title: "Una difícil decisión", watched: false },
  { id: 35, title: "Mi corazón late, todavía te quiero", watched: false },
  { id: 36, title: "La última esperanza", watched: false },
  { id: 37, title: "El tiro largo secreto", watched: false },
  { id: 38, title: "El tigre dormido", watched: false },
  { id: 39, title: "El despertar del tigre", watched: false },
  { id: 40, title: "El tiro gemelo", watched: false },
  { id: 41, title: "El duelo continua", watched: false },
  { id: 42, title: "Saber perder", watched: false },
  { id: 43, title: "El águila catalana", watched: false },
  { id: 44, title: "La promesa de Mark", watched: false },
  { id: 45, title: "El equipo en crisis", watched: false },
  { id: 46, title: "Todos para uno", watched: false },
  { id: 47, title: "Un mar de pánico", watched: false },
  { id: 48, title: "Una victoria sin emociones", watched: false },
  { id: 49, title: "Días calurosos", watched: false },
  { id: 50, title: "El partido contra el séptimo", watched: false },
  { id: 51, title: "Nuevos rivales", watched: false },
  { id: 52, title: "Empieza el contraataque", watched: false },
  { id: 53, title: "El dúo de oro ataca de nuevo", watched: false },
  { id: 54, title: "El duelo final", watched: false },
  { id: 55, title: "La despedida", watched: false },
  { id: 56, title: "Nuevos viajes", watched: false },
  { id: 57, title: "Empezando el tercer año", watched: false },
  { id: 58, title: "El tiro del halcón", watched: false },
  { id: 59, title: "Oliver contra Patrick Everett", watched: false },
  { id: 60, title: "El halcón pierde sus plumas", watched: false },
  { id: 61, title: "El tiro del águila", watched: false },
  { id: 62, title: "En plena lucha", watched: false },
  { id: 63, title: "El retorno de Julian Ross", watched: false },
  { id: 64, title: "Julian Ross vuelve al campo", watched: false },
  { id: 65, title: "Benji y el grande de Europa", watched: false },
  { id: 66, title: "Duelo en Europa", watched: false },
  { id: 67, title: "Un noble y su estilo de juego", watched: false },
  { id: 68, title: "La carta de Benji", watched: false },
  { id: 69, title: "El tigre afila sus garras", watched: false },
  { id: 70, title: "Una defensa de hierro", watched: false },
  { id: 71, title: "Oliver lesionado", watched: false },
  { id: 72, title: "El gran ausente", watched: false },
  { id: 73, title: "Vuelve el capitán", watched: false },
  { id: 74, title: "Camino a la final", watched: false },
  { id: 75, title: "El salto acrobático", watched: false },
  { id: 76, title: "El tiro de la navaja", watched: false },
  { id: 77, title: "Una nueva técnica", watched: false },
  { id: 78, title: "Los cuartos de final", watched: false },
  { id: 79, title: "Lo que hace el esfuerzo", watched: false },
  { id: 80, title: "El gigante de la montaña", watched: false },
  { id: 81, title: "Duelo de halcones", watched: false },
  { id: 82, title: "El tiro con efecto", watched: false },
  { id: 83, title: "Gol a toda costa", watched: false },
  { id: 84, title: "Trabajo de equipo", watched: false },
  { id: 85, title: "Los cuatro mejores", watched: false },
  { id: 86, title: "El fracaso de Julian Ross", watched: false },
  { id: 87, title: "El tigre enjaulado", watched: false },
  { id: 88, title: "La brigada del tigre", watched: false },
  { id: 89, title: "La carta de Italia", watched: false },
  { id: 90, title: "La mejor selección", watched: false },
  { id: 91, title: "Los nuevos reyes del campo", watched: false },
  { id: 92, title: "¡Rueda balón!", watched: false },
  { id: 93, title: "Un ataque valiente", watched: false },
  { id: 94, title: "Ataque excesivo", watched: false },
  { id: 95, title: "La quinta victoria consecutiva", watched: false },
  { id: 96, title: "Adiós Roberto", watched: false },
  { id: 97, title: "El desafío", watched: false },
  { id: 98, title: "Recuerdos", watched: false },
  { id: 99, title: "Oliver contra el tanque", watched: false },
  { id: 100, title: "Sorpresa en el campo", watched: false },
  { id: 101, title: "Oliver contra Pierre", watched: false },
  { id: 102, title: "El emperador de Europa", watched: false },
  { id: 103, title: "A por el tercer título", watched: false },
  { id: 104, title: "La última recuperación", watched: false },
  { id: 105, title: "Mark vuelve al campo", watched: false },
  { id: 106, title: "El último partido", watched: false },
  { id: 107, title: "Todos a la ofensiva", watched: false },
  { id: 108, title: "El tiro con efecto contraataca", watched: false },
  { id: 109, title: "El tiro del tigre", watched: false },
  { id: 110, title: "El triunfo de Mark Lenders", watched: false },
  { id: 111, title: "A pesar de las heridas", watched: false },
  { id: 112, title: "El gol de la suerte", watched: false },
  { id: 113, title: "Acrobacias", watched: false },
  { id: 114, title: "Defensa compartida", watched: false },
  { id: 115, title: "Fiebre en el campo", watched: false },
  { id: 116, title: "El gol de la victoria", watched: false },
  { id: 117, title: "El cuarto gol", watched: false },
  { id: 118, title: "A por el empate", watched: false },
  { id: 119, title: "La batalla de los capitanes", watched: false },
  { id: 120, title: "El capitán valiente", watched: false },
  { id: 121, title: "La camilla de los heridos", watched: false },
  { id: 122, title: "Guerra de fantasía", watched: false },
  { id: 123, title: "¡A por el último gol!", watched: false },
  { id: 124, title: "El cañonazo", watched: false },
  { id: 125, title: "La victoria compartida", watched: false },
  { id: 126, title: "Mi amigo Benji", watched: false },
  { id: 127, title: "Mi amigo Tom", watched: false },
  { id: 128, title: "Un ejército de campeones", watched: false }
];

