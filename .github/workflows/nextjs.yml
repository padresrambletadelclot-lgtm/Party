import React, { useState, useEffect } from 'react';
import { initializeApp } from 'firebase/app';
import { 
  getFirestore, 
  collection, 
  onSnapshot, 
  addDoc, 
  deleteDoc, 
  doc, 
  query, 
  setDoc,
  getDoc
} from 'firebase/firestore';
import { 
  getAuth, 
  signInAnonymously, 
  signInWithCustomToken,
  onAuthStateChanged 
} from 'firebase/auth';
import { 
  Plus, 
  Trash2, 
  CheckCircle2, 
  AlertCircle, 
  ShoppingBag, 
  User, 
  X 
} from 'lucide-react';

// --- CONFIGURACIÓN DE FIREBASE ---
const firebaseConfig = JSON.parse(__firebase_config);
const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getFirestore(app);
const appId = typeof __app_id !== 'undefined' ? __app_id : 'party-planner-v1';

export default function App() {
  const [user, setUser] = useState(null);
  const [items, setItems] = useState([]);
  const [hostRules, setHostRules] = useState({ wishlist: '', noList: '' });
  const [isModalOpen, setIsModalOpen] = useState(false);
  const [isHostMode, setIsHostMode] = useState(false);
  const [loading, setLoading] = useState(true);

  // Form states
  const [itemName, setItemName] = useState('');
  const [quantity, setQuantity] = useState('1');
  const [userName, setUserName] = useState('');

  // 1. FLUJO DE AUTENTICACIÓN (REGLA 3)
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

    const unsubscribeAuth = onAuthStateChanged(auth, (currentUser) => {
      setUser(currentUser);
      if (currentUser) {
        const savedName = localStorage.getItem('party_user_name');
        if (savedName) setUserName(savedName);
      }
    });

    return () => unsubscribeAuth();
  }, []);

  // 2. FETCH DE DATOS (PROTEGIDO POR USER)
  useEffect(() => {
    if (!user) return;

    // Items Listener con callback de error obligatorio
    const itemsCol = collection(db, 'artifacts', appId, 'public', 'data', 'items');
    const unsubscribeItems = onSnapshot(
      itemsCol, 
      (snapshot) => {
        const itemList = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
        // Ordenación manual en memoria (Regla 2)
        itemList.sort((a, b) => (b.timestamp || 0) - (a.timestamp || 0));
        setItems(itemList);
        setLoading(false);
      }, 
      (error) => {
        console.error("Error en items snapshot:", error);
      }
    );

    // Rules Listener con callback de error obligatorio
    const rulesDoc = doc(db, 'artifacts', appId, 'public', 'data', 'hostRules', 'global');
    const unsubscribeRules = onSnapshot(
      rulesDoc, 
      (docSnap) => {
        if (docSnap.exists()) {
          setHostRules(docSnap.data());
        }
      },
      (error) => {
        console.error("Error en rules snapshot:", error);
      }
    );

    return () => {
      unsubscribeItems();
      unsubscribeRules();
    };
  }, [user]);

  // Acciones
  const addItem = async (e) => {
    e.preventDefault();
    if (!itemName || !userName || !user) return;

    localStorage.setItem('party_user_name', userName);
    
    try {
      await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'items'), {
        name: itemName,
        quantity,
        contributor: userName,
        userId: user.uid,
        timestamp: Date.now()
      });
      setItemName('');
      setQuantity('1');
      setIsModalOpen(false);
    } catch (error) {
      console.error("Error al añadir item:", error);
    }
  };

  const deleteItem = async (id) => {
    if (!user) return;
    try {
      await deleteDoc(doc(db, 'artifacts', appId, 'public', 'data', 'items', id));
    } catch (error) {
      console.error("Error al borrar item:", error);
    }
  };

  const updateRules = async () => {
    if (!user) return;
    try {
      await setDoc(doc(db, 'artifacts', appId, 'public', 'data', 'hostRules', 'global'), hostRules);
      setIsHostMode(false);
    } catch (error) {
      console.error("Error al actualizar reglas:", error);
    }
  };

  if (loading && !user) {
    return (
      <div className="flex items-center justify-center min-h-screen bg-slate-50">
        <div className="animate-spin rounded-full h-12 w-12 border-b-2 border-blue-600"></div>
      </div>
    );
  }

  return (
    <div className="max-w-md mx-auto min-h-screen bg-slate-50 text-slate-900 pb-24 font-sans">
      {/* Header */}
      <header className="p-6 bg-white border-b sticky top-0 z-10 flex justify-between items-center">
        <div>
          <h1 className="text-xl font-bold flex items-center gap-2">
            <ShoppingBag className="text-blue-600" size={24} />
            Plan Fiesta
          </h1>
          <p className="text-xs text-slate-500">{items.length} cosas confirmadas</p>
        </div>
        <button 
          onClick={() => setIsHostMode(!isHostMode)}
          className="text-xs font-semibold bg-slate-100 px-3 py-1.5 rounded-full text-slate-600 hover:bg-slate-200"
        >
          Anfitrión
        </button>
      </header>

      <main className="p-4 space-y-6">
        {/* Vista de Reglas */}
        <section className="grid grid-cols-1 gap-4">
          <div className="bg-emerald-50 border border-emerald-100 p-4 rounded-2xl relative">
            <h3 className="text-xs font-bold text-emerald-700 uppercase mb-2 flex items-center gap-1">
              <CheckCircle2 size={14} /> Wishlist del Anfitrión
            </h3>
            <p className="text-sm text-emerald-900 italic">
              {hostRules.wishlist || "El anfitrión no ha pedido nada específico aún."}
            </p>
          </div>

          <div className="bg-rose-50 border border-rose-100 p-4 rounded-2xl">
            <h3 className="text-xs font-bold text-rose-700 uppercase mb-2 flex items-center gap-1">
              <AlertCircle size={14} /> NO TRAER (Ya tenemos)
            </h3>
            <p className="text-sm text-rose-900 italic">
              {hostRules.noList || "¡Trae lo que quieras!"}
            </p>
          </div>
        </section>

        {/* Lista de Items */}
        <section className="space-y-3">
          <h2 className="text-lg font-bold px-1">¿Qué llevamos?</h2>
          {items.length === 0 ? (
            <div className="text-center py-10 text-slate-400">
              <ShoppingBag className="mx-auto mb-2 opacity-20" size={48} />
              <p>Nadie ha apuntado nada todavía.</p>
              <p className="text-sm">¡Sé el primero!</p>
            </div>
          ) : (
            items.map((item) => (
              <div 
                key={item.id} 
                className="bg-white p-4 rounded-2xl shadow-sm border border-slate-100 flex justify-between items-center animate-in fade-in slide-in-from-bottom-2"
              >
                <div className="flex-1">
                  <div className="flex items-center gap-2">
                    <span className="font-bold text-slate-800">{item.name}</span>
                    <span className="bg-slate-100 text-slate-600 text-[10px] px-2 py-0.5 rounded-full font-black">
                      x{item.quantity}
                    </span>
                  </div>
                  <div className="text-xs text-slate-400 flex items-center gap-1 mt-1">
                    <User size={10} /> {item.contributor}
                  </div>
                </div>
                {item.userId === user?.uid && (
                  <button 
                    onClick={() => deleteItem(item.id)}
                    className="p-2 text-rose-300 hover:text-rose-500 transition-colors"
                  >
                    <Trash2 size={18} />
                  </button>
                )}
              </div>
            ))
          )}
        </section>
      </main>

      {/* Botón Flotante */}
      <button 
        onClick={() => setIsModalOpen(true)}
        className="fixed bottom-6 right-6 w-16 h-16 bg-blue-600 text-white rounded-full shadow-2xl flex items-center justify-center hover:bg-blue-700 active:scale-90 transition-all z-20"
      >
        <Plus size={32} />
      </button>

      {/* Modal: Añadir Item */}
      {isModalOpen && (
        <div className="fixed inset-0 bg-black/60 backdrop-blur-sm z-50 flex items-end sm:items-center justify-center p-0 sm:p-4">
          <div className="bg-white w-full max-w-md rounded-t-3xl sm:rounded-3xl p-6 shadow-2xl animate-in slide-in-from-bottom-full duration-300">
            <div className="flex justify-between items-center mb-6">
              <h2 className="text-xl font-bold">¿Qué vas a traer?</h2>
              <button onClick={() => setIsModalOpen(false)} className="p-2 bg-slate-100 rounded-full">
                <X size={20} />
              </button>
            </div>
            
            <form onSubmit={addItem} className="space-y-4">
              <div>
                <label className="text-xs font-bold text-slate-500 uppercase ml-1">Tu Nombre</label>
                <input 
                  required
                  type="text" 
                  value={userName}
                  onChange={(e) => setUserName(e.target.value)}
                  placeholder="Ej: Nacho"
                  className="w-full p-4 bg-slate-50 border-none rounded-2xl mt-1 focus:ring-2 focus:ring-blue-500 outline-none"
                />
              </div>

              <div className="flex gap-3">
                <div className="flex-[3]">
                  <label className="text-xs font-bold text-slate-500 uppercase ml-1">¿Qué es?</label>
                  <input 
                    required
                    type="text" 
                    value={itemName}
                    onChange={(e) => setItemName(e.target.value)}
                    placeholder="Ej: Patatas fritas"
                    className="w-full p-4 bg-slate-50 border-none rounded-2xl mt-1 focus:ring-2 focus:ring-blue-500 outline-none"
                  />
                </div>
                <div className="flex-1">
                  <label className="text-xs font-bold text-slate-500 uppercase ml-1">Cant.</label>
                  <input 
                    type="number" 
                    min="1"
                    value={quantity}
                    onChange={(e) => setQuantity(e.target.value)}
                    className="w-full p-4 bg-slate-50 border-none rounded-2xl mt-1 focus:ring-2 focus:ring-blue-500 outline-none text-center"
                  />
                </div>
              </div>

              <button 
                type="submit"
                className="w-full bg-blue-600 text-white font-bold py-4 rounded-2xl shadow-lg shadow-blue-200 hover:bg-blue-700 active:scale-[0.98] transition-all mt-4"
              >
                Añadir a la lista
              </button>
            </form>
          </div>
        </div>
      )}

      {/* Modal: Configuración Anfitrión */}
      {isHostMode && (
        <div className="fixed inset-0 bg-slate-900/90 backdrop-blur-md z-50 flex items-center justify-center p-4">
          <div className="bg-white w-full max-w-md rounded-3xl p-6 shadow-2xl">
            <h2 className="text-xl font-bold mb-1">Zona del Anfitrión</h2>
            <p className="text-xs text-slate-500 mb-6">Configura las reglas del grupo</p>
            
            <div className="space-y-4">
              <div>
                <label className="text-xs font-bold text-emerald-600 uppercase ml-1">Wishlist (Sugerencias)</label>
                <textarea 
                  rows="3"
                  value={hostRules.wishlist}
                  onChange={(e) => setHostRules({...hostRules, wishlist: e.target.value})}
                  className="w-full p-4 bg-emerald-50 border border-emerald-100 rounded-2xl mt-1 outline-none focus:ring-2 focus:ring-emerald-500"
                  placeholder="Ej: Necesitamos hielo, vasos y servilletas..."
                />
              </div>

              <div>
                <label className="text-xs font-bold text-rose-600 uppercase ml-1">No traer (Ya hay)</label>
                <textarea 
                  rows="3"
                  value={hostRules.noList}
                  onChange={(e) => setHostRules({...hostRules, noList: e.target.value})}
                  className="w-full p-4 bg-rose-50 border border-rose-100 rounded-2xl mt-1 outline-none focus:ring-2 focus:ring-rose-500"
                  placeholder="Ej: No traigáis más cerveza, la nevera está llena."
                />
              </div>

              <div className="flex gap-2 pt-4">
                <button 
                  onClick={() => setIsHostMode(false)}
                  className="flex-1 py-4 font-bold text-slate-500 hover:bg-slate-100 rounded-2xl transition-colors"
                >
                  Cancelar
                </button>
                <button 
                  onClick={updateRules}
                  className="flex-1 bg-slate-900 text-white font-bold py-4 rounded-2xl shadow-xl hover:bg-black transition-all"
                >
                  Guardar Reglas
                </button>
              </div>
            </div>
          </div>
        </div>
      )}
    </div>
  );
}
