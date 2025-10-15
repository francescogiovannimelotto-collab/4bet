import React, { useState, useReducer } from 'react';
import { Lock, Plus, User, LogOut, Users, Coins, Trophy, Clock, ChevronDown } from 'lucide-react';

const ADMIN_PASSWORD = 'azzurra69';

// Initial data stored in state, not localStorage
const initialState = {
  users: {},
  events: [],
  bets: []
};

const dataReducer = (state, action) => {
  switch (action.type) {
    case 'ADD_USER':
      return {
        ...state,
        users: {
          ...state.users,
          [action.payload.username]: { password: action.payload.password, credits: 0 }
        }
      };
    case 'UPDATE_CREDITS':
      return {
        ...state,
        users: {
          ...state.users,
          [action.payload.username]: {
            ...state.users[action.payload.username],
            credits: action.payload.credits
          }
        }
      };
    case 'ADD_EVENT':
      return {
        ...state,
        events: [...state.events, action.payload]
      };
    case 'RESOLVE_EVENT':
      const updatedUsers = { ...state.users };
      const eventBets = state.bets.filter(b => b.eventId === action.payload.eventId);
      const event = state.events.find(e => e.id === action.payload.eventId);
      
      eventBets.forEach(bet => {
        if (bet.optionIndex === action.payload.winningIndex) {
          const winnings = bet.amount * event.options[action.payload.winningIndex].multiplier;
          updatedUsers[bet.username].credits += winnings;
        }
      });
      
      return {
        ...state,
        users: updatedUsers,
        events: state.events.map(e => 
          e.id === action.payload.eventId 
            ? { ...e, status: 'resolved', winner: action.payload.winningIndex }
            : e
        )
      };
    case 'ADD_BET':
      return {
        ...state,
        bets: [...state.bets, action.payload],
        users: {
          ...state.users,
          [action.payload.username]: {
            ...state.users[action.payload.username],
            credits: state.users[action.payload.username].credits - action.payload.amount
          }
        }
      };
    default:
      return state;
  }
};

export default function BettingApp() {
  const [data, dispatch] = useReducer(dataReducer, initialState);
  const [currentUser, setCurrentUser] = useState(null);
  const [showAuth, setShowAuth] = useState(true);
  const [isRegister, setIsRegister] = useState(false);
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');
  
  const [showAdminPanel, setShowAdminPanel] = useState(false);
  const [adminPassword, setAdminPassword] = useState('');
  const [showCreateEvent, setShowCreateEvent] = useState(false);
  const [isAdminAuthenticated, setIsAdminAuthenticated] = useState(false);
  
  const [newEvent, setNewEvent] = useState({
    title: '',
    type: 'yesno',
    options: []
  });

  const handleAuth = () => {
    if (isRegister) {
      if (data.users[username]) {
        alert('Username già esistente');
        return;
      }
      if (username.length < 3) {
        alert('Username deve essere lungo almeno 3 caratteri');
        return;
      }
      if (password.length < 3) {
        alert('Password deve essere lunga almeno 3 caratteri');
        return;
      }
      dispatch({
        type: 'ADD_USER',
        payload: { username, password }
      });
      setCurrentUser(username);
      setShowAuth(false);
      setUsername('');
      setPassword('');
    } else {
      if (!username || !password) {
        alert('Compila username e password');
        return;
      }
      if (data.users[username] && data.users[username].password === password) {
        setCurrentUser(username);
        setShowAuth(false);
        setUsername('');
        setPassword('');
      } else {
        alert('Username o password errati');
      }
    }
  };

  const handleLogout = () => {
    setCurrentUser(null);
    setShowAuth(true);
    setShowAdminPanel(false);
    setIsAdminAuthenticated(false);
  };

  const handleAdminAccess = () => {
    if (adminPassword === ADMIN_PASSWORD) {
      setIsAdminAuthenticated(true);
      setAdminPassword('');
    } else {
      alert('Password errata');
      setAdminPassword('');
    }
  };

  const createEvent = () => {
    if (!newEvent.title || newEvent.options.length === 0) {
      alert('Compila tutti i campi');
      return;
    }
    dispatch({
      type: 'ADD_EVENT',
      payload: {
        id: Date.now(),
        title: newEvent.title,
        type: newEvent.type,
        options: newEvent.options,
        status: 'open',
        createdAt: new Date().toISOString()
      }
    });
    setNewEvent({ title: '', type: 'yesno', options: [] });
    setShowCreateEvent(false);
    alert('Evento creato!');
  };

  const placeBet = (eventId, optionIndex, amount) => {
    if (data.users[currentUser].credits < amount) {
      alert('Crediti insufficienti');
      return;
    }
    
    dispatch({
      type: 'ADD_BET',
      payload: {
        id: Date.now(),
        eventId,
        username: currentUser,
        optionIndex,
        amount,
        timestamp: new Date().toISOString()
      }
    });
    alert('Scommessa piazzata!');
  };

  const updateOption = (index, field, value) => {
    const newOptions = [...newEvent.options];
    newOptions[index][field] = value;
    setNewEvent({ ...newEvent, options: newOptions });
  };

  const addEventOption = () => {
    if (newEvent.type === 'yesno') {
      setNewEvent({
        ...newEvent,
        options: [
          { label: 'Sì', multiplier: 1.5 },
          { label: 'No', multiplier: 2.0 }
        ]
      });
    } else {
      setNewEvent({
        ...newEvent,
        options: [...newEvent.options, { label: '', multiplier: 1.0 }]
      });
    }
  };

  if (showAuth) {
    return (
      <div className="min-h-screen bg-gradient-to-br from-slate-900 via-purple-900 to-slate-900 flex items-center justify-center p-4">
        <div className="w-full max-w-md">
          <div className="bg-white/10 backdrop-blur-xl rounded-2xl shadow-2xl p-8 border border-white/20">
            <div className="flex items-center justify-center mb-2">
              <div className="relative">
                <div className="absolute inset-0 bg-gradient-to-r from-yellow-400 to-orange-400 rounded-full blur opacity-75"></div>
                <div className="relative bg-slate-900 rounded-full p-3">
                  <Trophy className="w-8 h-8 text-yellow-400" />
                </div>
              </div>
            </div>
            <h1 className="text-4xl font-black text-center bg-gradient-to-r from-yellow-400 via-orange-400 to-yellow-300 bg-clip-text text-transparent mb-2">
              4Bet
            </h1>
            <p className="text-center text-white/60 text-sm mb-8 tracking-wide">Scommesse Scolastiche</p>
            
            <h2 className="text-xl font-bold text-center mb-8 text-white">
              {isRegister ? 'Crea un Account' : 'Accedi'}
            </h2>
            
            <div className="space-y-4 mb-6">
              <div className="relative">
                <User className="absolute left-3 top-3.5 w-5 h-5 text-white/40" />
                <input
                  type="text"
                  placeholder="Username"
                  value={username}
                  onChange={(e) => setUsername(e.target.value)}
                  onKeyPress={(e) => e.key === 'Enter' && handleAuth()}
                  className="w-full pl-10 pr-4 py-3 bg-white/10 border border-white/20 rounded-lg text-white placeholder-white/50 focus:outline-none focus:ring-2 focus:ring-yellow-400/50 focus:border-transparent backdrop-blur transition"
                />
              </div>
              
              <div className="relative">
                <Lock className="absolute left-3 top-3.5 w-5 h-5 text-white/40" />
                <input
                  type="password"
                  placeholder="Password"
                  value={password}
                  onChange={(e) => setPassword(e.target.value)}
                  onKeyPress={(e) => e.key === 'Enter' && handleAuth()}
                  className="w-full pl-10 pr-4 py-3 bg-white/10 border border-white/20 rounded-lg text-white placeholder-white/50 focus:outline-none focus:ring-2 focus:ring-yellow-400/50 focus:border-transparent backdrop-blur transition"
                />
              </div>
            </div>
            
            <button
              onClick={handleAuth}
              className="w-full bg-gradient-to-r from-yellow-400 to-orange-400 text-slate-900 py-3 rounded-lg hover:shadow-lg hover:shadow-yellow-400/50 transition font-bold mb-4 active:scale-95"
            >
              {isRegister ? 'Registrati' : 'Accedi'}
            </button>
            
            <button
              onClick={() => {
                setIsRegister(!isRegister);
                setUsername('');
                setPassword('');
              }}
              className="w-full text-yellow-400 hover:text-yellow-300 transition text-sm font-medium"
            >
              {isRegister ? 'Hai già un account? Accedi' : 'Non hai un account? Registrati'}
            </button>
          </div>
        </div>
      </div>
    );
  }

  return (
    <div className="min-h-screen bg-gradient-to-br from-slate-900 via-slate-800 to-slate-900">
      {/* Navigation */}
      <nav className="bg-slate-800/50 backdrop-blur-xl border-b border-white/10 sticky top-0 z-40">
        <div className="max-w-7xl mx-auto px-6 py-4 flex justify-between items-center">
          <div className="flex items-center gap-3">
            <div className="bg-gradient-to-r from-yellow-400 to-orange-400 p-2 rounded-lg">
              <Trophy className="w-6 h-6 text-slate-900" />
            </div>
            <h1 className="text-2xl font-black bg-gradient-to-r from-yellow-400 to-orange-400 bg-clip-text text-transparent">4Bet</h1>
          </div>
          
          <div className="flex items-center gap-4">
            <div className="bg-gradient-to-r from-yellow-500/20 to-orange-500/20 border border-yellow-400/30 px-4 py-2 rounded-lg flex items-center gap-2 backdrop-blur">
              <Coins className="w-5 h-5 text-yellow-400" />
              <span className="font-bold text-yellow-400">{data.users[currentUser]?.credits || 0}</span>
            </div>
            
            <div className="flex items-center gap-2 text-white/80">
              <User className="w-4 h-4" />
              <span className="font-semibold">{currentUser}</span>
            </div>
            
            <button
              onClick={() => setShowAdminPanel(!showAdminPanel)}
              className="p-2 bg-purple-600/30 hover:bg-purple-600/50 border border-purple-400/30 text-purple-300 rounded-lg transition backdrop-blur"
              title="Pannello Admin"
            >
              <Lock className="w-5 h-5" />
            </button>
            
            <button
              onClick={handleLogout}
              className="p-2 bg-red-600/30 hover:bg-red-600/50 border border-red-400/30 text-red-300 rounded-lg transition backdrop-blur"
              title="Logout"
            >
              <LogOut className="w-5 h-5" />
            </button>
          </div>
        </div>
      </nav>

      <div className="max-w-7xl mx-auto px-6 py-8">
        {/* Admin Panel */}
        {showAdminPanel && (
          <div className="bg-slate-800/50 backdrop-blur-xl rounded-2xl border border-white/10 p-8 mb-8">
            <h2 className="text-2xl font-bold flex items-center gap-3 mb-6 text-white">
              <Lock className="w-6 h-6 text-purple-400" />
              Pannello Admin
            </h2>
            
            {!isAdminAuthenticated ? (
              <div className="max-w-md">
                <label className="block text-sm font-semibold mb-3 text-white/80">Accedi al Pannello</label>
                <div className="flex gap-2">
                  <input
                    type="password"
                    value={adminPassword}
                    onChange={(e) => setAdminPassword(e.target.value)}
                    onKeyPress={(e) => e.key === 'Enter' && handleAdminAccess()}
                    className="flex-1 px-4 py-2 bg-white/10 border border-white/20 rounded-lg text-white placeholder-white/50 focus:outline-none focus:ring-2 focus:ring-purple-400/50 backdrop-blur"
                    placeholder="Password admin"
                    autoFocus
                  />
                  <button
                    onClick={handleAdminAccess}
                    className="px-6 py-2 bg-purple-600 hover:bg-purple-700 text-white rounded-lg transition font-semibold"
                  >
                    Accedi
                  </button>
                </div>
              </div>
            ) : !showCreateEvent ? (
              <div className="space-y-8">
                <div className="flex justify-between items-center">
                  <h3 className="text-lg font-bold text-white">Controlli Admin</h3>
                  <div className="flex gap-2">
                    <button
                      onClick={() => setShowCreateEvent(true)}
                      className="px-4 py-2 bg-gradient-to-r from-purple-600 to-purple-700 hover:from-purple-700 hover:to-purple-800 text-white rounded-lg transition flex items-center gap-2 font-semibold"
                    >
                      <Plus className="w-4 h-4" />
                      Crea Evento
                    </button>
                    <button
                      onClick={() => setIsAdminAuthenticated(false)}
                      className="px-4 py-2 bg-slate-700/50 hover:bg-slate-700 text-white rounded-lg transition"
                    >
                      Logout
                    </button>
                  </div>
                </div>
                
                <div>
                  <h3 className="font-bold text-lg mb-4 flex items-center gap-2 text-white">
                    <Users className="w-5 h-5 text-blue-400" />
                    Gestisci Crediti
                  </h3>
                  <div className="bg-slate-700/30 rounded-xl p-4 border border-white/10">
                    {Object.keys(data.users).length === 0 ? (
                      <p className="text-white/60">Nessun utente registrato</p>
                    ) : (
                      Object.keys(data.users).map(u => (
                        <div key={u} className="flex items-center gap-3 mb-3 p-3 bg-slate-600/20 rounded-lg border border-white/5 last:mb-0">
                          <span className="w-40 font-semibold text-white">{u}</span>
                          <input
                            type="number"
                            value={data.users[u].credits}
                            onChange={(e) => dispatch({
                              type: 'UPDATE_CREDITS',
                              payload: { username: u, credits: parseFloat(e.target.value) || 0 }
                            })}
                            className="px-3 py-1 bg-white/10 border border-white/20 rounded-lg text-white w-32 focus:outline-none focus:ring-2 focus:ring-blue-400/50"
                          />
                          <span className="text-sm text-white/60">crediti</span>
                        </div>
                      ))
                    )}
                  </div>
                </div>
                
                <div>
                  <h3 className="font-bold text-lg mb-4 text-white">Risolvi Eventi</h3>
                  {data.events.filter(e => e.status === 'open').length === 0 ? (
                    <p className="text-white/60">Nessun evento aperto</p>
                  ) : (
                    data.events.filter(e => e.status === 'open').map(event => (
                      <div key={event.id} className="bg-slate-700/30 p-4 rounded-xl mb-3 border border-white/10 last:mb-0">
                        <p className="font-semibold text-white mb-3">{event.title}</p>
                        <div className="flex flex-wrap gap-2">
                          {event.options.map((option, idx) => (
                            <button
                              key={idx}
                              onClick={() => dispatch({
                                type: 'RESOLVE_EVENT',
                                payload: { eventId: event.id, winningIndex: idx }
                              })}
                              className="px-4 py-2 bg-gradient-to-r from-green-600 to-green-700 hover:from-green-700 hover:to-green-800 text-white rounded-lg transition text-sm font-semibold"
                            >
                              {option.label} vince
                            </button>
                          ))}
                        </div>
                      </div>
                    ))
                  )}
                </div>
              </div>
            ) : (
              <div className="max-w-2xl space-y-4">
                <input
                  type="text"
                  placeholder="Titolo evento"
                  value={newEvent.title}
                  onChange={(e) => setNewEvent({ ...newEvent, title: e.target.value })}
                  className="w-full px-4 py-3 bg-white/10 border border-white/20 rounded-lg text-white placeholder-white/50 focus:outline-none focus:ring-2 focus:ring-purple-400/50 backdrop-blur"
                />
                
                <div>
                  <label className="block text-sm font-semibold mb-2 text-white/80">Tipo di evento</label>
                  <select
                    value={newEvent.type}
                    onChange={(e) => setNewEvent({ ...newEvent, type: e.target.value, options: [] })}
                    className="w-full px-4 py-3 bg-white/10 border border-white/20 rounded-lg text-white focus:outline-none focus:ring-2 focus:ring-purple-400/50 backdrop-blur"
                  >
                    <option value="yesno" className="bg-slate-900">Sì/No</option>
                    <option value="time" className="bg-slate-900">Intervalli di tempo</option>
                    <option value="custom" className="bg-slate-900">Personalizzato</option>
                  </select>
                </div>
                
                {newEvent.type === 'yesno' && newEvent.options.length === 0 && (
                  <button
                    onClick={addEventOption}
                    className="w-full px-4 py-3 bg-gradient-to-r from-blue-600 to-blue-700 hover:from-blue-700 hover:to-blue-800 text-white rounded-lg transition font-semibold"
                  >
                    Genera opzioni Sì/No
                  </button>
                )}
                
                {(newEvent.type !== 'yesno' || newEvent.options.length > 0) && (
                  <div>
                    <div className="flex justify-between items-center mb-3">
                      <label className="text-sm font-semibold text-white/80">Opzioni</label>
                      {newEvent.type !== 'yesno' && (
                        <button
                          onClick={addEventOption}
                          className="px-3 py-1 bg-blue-600 hover:bg-blue-700 text-white rounded text-sm transition flex items-center gap-1 font-semibold"
                        >
                          <Plus className="w-4 h-4" />
                          Aggiungi
                        </button>
                      )}
                    </div>
                    
                    {newEvent.options.map((option, idx) => (
                      <div key={idx} className="flex gap-2 mb-2">
                        <input
                          type="text"
                          placeholder="Descrizione"
                          value={option.label}
                          onChange={(e) => updateOption(idx, 'label', e.target.value)}
                          className="flex-1 px-3 py-2 bg-white/10 border border-white/20 rounded-lg text-white placeholder-white/50 focus:outline-none focus:ring-2 focus:ring-purple-400/50 backdrop-blur"
                          disabled={newEvent.type === 'yesno'}
                        />
                        <input
                          type="number"
                          step="0.1"
                          placeholder="Molt."
                          value={option.multiplier}
                          onChange={(e) => updateOption(idx, 'multiplier', parseFloat(e.target.value))}
                          className="w-24 px-3 py-2 bg-white/10 border border-white/20 rounded-lg text-white placeholder-white/50 focus:outline-none focus:ring-2 focus:ring-purple-400/50 backdrop-blur"
                        />
                      </div>
                    ))}
                  </div>
                )}
                
                <div className="flex gap-2 pt-2">
                  <button
                    onClick={createEvent}
                    className="flex-1 px-4 py-3 bg-gradient-to-r from-green-600 to-green-700 hover:from-green-700 hover:to-green-800 text-white rounded-lg transition font-semibold"
                  >
                    Crea Evento
                  </button>
                  <button
                    onClick={() => {
                      setShowCreateEvent(false);
                      setNewEvent({ title: '', type: 'yesno', options: [] });
                    }}
                    className="px-4 py-3 bg-slate-700/50 hover:bg-slate-700 text-white rounded-lg transition"
                  >
                    Annulla
                  </button>
                </div>
              </div>
            )}
          </div>
        )}

        {/* Main Content */}
        <div className="grid lg:grid-cols-2 gap-8">
          {/* Open Events */}
          <div>
            <h2 className="text-2xl font-bold flex items-center gap-3 mb-6 text-white">
              <Clock className="w-6 h-6 text-blue-400" />
              Eventi Aperti
            </h2>
            {data.events.filter(e => e.status === 'open').length === 0 ? (
              <div className="bg-slate-800/50 backdrop-blur-xl rounded-xl border border-white/10 p-8 text-center">
                <p className="text-white/60">Nessun evento disponibile</p>
              </div>
            ) : (
              data.events.filter(e => e.status === 'open').map(event => (
                <div key={event.id} className="bg-slate-800/50 backdrop-blur-xl rounded-xl border border-white/10 p-6 mb-4 hover:border-white/20 transition">
                  <h3 className="text-xl font-bold mb-4 text-white">{event.title}</h3>
                  <div className="space-y-2">
                    {event.options.map((option, idx) => (
                      <div key={idx} className="flex items-center gap-3 p-3 bg-slate-700/30 rounded-lg border border-white/5 hover:border-white/10 transition">
                        <div className="flex-1">
                          <span className="font-semibold text-white">{option.label}</span>
                          <span className="text-green-400 font-bold ml-2">{option.multiplier}x</span>
                        </div>
                        <input
                          type="number"
                          placeholder="Importo"
                          className="w-20 px-3 py-2 bg-white/10 border border-white/20 rounded-lg text-white placeholder-white/50 focus:outline-none focus:ring-2 focus:ring-blue-400/50 text-sm"
                          id={`bet-${event.id}-${idx}`}
                        />
                        <button
                          onClick={() => {
                            const amount = parseFloat(document.getElementById(`bet-${event.id}-${idx}`).value);
                            if (amount > 0) {
                              placeBet(event.id, idx, amount);
                              document.getElementById(`bet-${event.id}-${idx}`).value = '';
                            }
                          }}
                          className="px-4 py-2 bg-gradient-to-r from-blue-600 to-blue-700 hover:from-blue-700 hover:to-blue-800 text-white rounded-lg transition text-sm font-semibold whitespace-nowrap"
                        >
                          Scommetti
                        </button>
                      </div>
                    ))}
                  </div>
                </div>
              ))
            )}
          </div>

          {/* My Bets */}
          <div>
            <h2 className="text-2xl font-bold flex items-center gap-3 mb-6 text-white">
              <Trophy className="w-6 h-6 text-yellow-400" />
              Le Mie Scommesse
            </h2>
            {data.bets.filter(b => b.username === currentUser).length === 0 ? (
              <div className="bg-slate-800/50 backdrop-blur-xl rounded-xl border border-white/10 p-8 text-center">
                <p className="text-white/60">Non hai ancora piazzato scommesse</p>
              </div>
            ) : (
              data.bets.filter(b => b.username === currentUser).map(bet => {
                const event = data.events.find(e => e.id === bet.eventId);
                if (!event) return null;
                const option = event.options[bet.optionIndex];
                const isWon = event.status === 'resolved' && event.winner === bet.optionIndex;
                
                return (
                  <div key={bet.id} className={`bg-slate-800/50 backdrop-blur-xl rounded-xl border transition p-4 mb-3 ${
                    event.status === 'resolved' 
                      ? isWon ? 'border-green-400/30' : 'border-red-400/30'
                      : 'border-white/10 hover:border-white/20'
                  }`}>
                    <div className="flex justify-between items-start mb-2">
                      <div>
                        <p className="font-bold text-white">{event.title}</p>
                        <p className="text-sm text-white/60">Opzione: {option.label}</p>
                      </div>
                      {event.status === 'resolved' && (
                        <span className={`text-xs font-bold px-3 py-1 rounded-full ${
                          isWon 
                            ? 'bg-green-500/30 text-green-300'
                            : 'bg-red-500/30 text-red-300'
                        }`}>
                          {isWon ? '✓ VINTA' : '✗ PERSA'}
                        </span>
                      )}
                    </div>
                    <div className="grid grid-cols-2 gap-2 text-sm">
                      <div>
                        <span className="text-white/60">Importo:</span>
                        <p className="font-bold text-blue-400">{bet.amount}</p>
                      </div>
                      <div>
                        <span className="text-white/60">Vincita:</span>
                        <p className="font-bold text-green-400">{(bet.amount * option.multiplier).toFixed(2)}</p>
                      </div>
                    </div>
                  </div>
                );
              })
            )}
          </div>
        </div>
      </div>
    </div>
  );
}
