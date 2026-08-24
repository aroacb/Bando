import React, { useState, useEffect, useRef } from "react";

const TOKENS = {
  ink: "#141A22",
  paper: "#E7E0D0",
  paperDim: "#DCD4C0",
  favor: "#3F6E52",
  favorSoft: "#DDE8DE",
  contra: "#A8432E",
  contraSoft: "#F1DED7",
  gold: "#B9902E",
  goldSoft: "#EFE3C4",
  ink70: "rgba(20,26,34,0.7)",
  ink45: "rgba(20,26,34,0.45)",
};

const MODEL = "claude-sonnet-4-6";

const CATEGORY_BY_WEEKDAY = ["Comunidad", "Actualidad", "Dilemas morales", "Ciencia", "Relaciones y sociedad", "Cultura", "Debates absurdos"];

const ACCENT_BY_CATEGORY = {
  Comunidad: "#6B6A63",
  Actualidad: "#2F6FA6",
  "Dilemas morales": "#6A4E8C",
  Ciencia: "#1F7A6C",
  "Relaciones y sociedad": "#B14C74",
  Cultura: "#C08A2E",
  "Debates absurdos": "#C1562B",
};

const SEED_TOPICS = [
  { id: "t1", text: "¿Deberían prohibirse los móviles en los institutos?", category: "Actualidad", votes: 3, usedDates: [] },
  { id: "t2", text: "¿Es peor mentir para proteger a alguien o decir la verdad aunque le destruya?", category: "Dilemas morales", votes: 2, usedDates: [] },
  { id: "t3", text: "Si pudieras eliminar un recuerdo doloroso, ¿deberías hacerlo?", category: "Dilemas morales", votes: 1, usedDates: [] },
  { id: "t4", text: "¿Es ético editar genéticamente a los futuros bebés para eliminar enfermedades?", category: "Ciencia", votes: 2, usedDates: [] },
  { id: "t5", text: "¿Debería existir una edad mínima para usar redes sociales?", category: "Relaciones y sociedad", votes: 4, usedDates: [] },
  { id: "t6", text: "¿Es ético utilizar IA para hacer arte?", category: "Cultura", votes: 3, usedDates: [] },
  { id: "t7", text: "¿Es peor pinchar la pizza con piña o poner ketchup a la tortilla de patatas?", category: "Debates absurdos", votes: 1, usedDates: [] },
];

function todayKey() {
  return new Date().toISOString().slice(0, 10);
}
function yesterdayKey() {
  const d = new Date();
  d.setDate(d.getDate() - 1);
  return d.toISOString().slice(0, 10);
}

async function askClaude(prompt) {
  const res = await fetch("https://api.anthropic.com/v1/messages", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ model: MODEL, max_tokens: 1000, messages: [{ role: "user", content: prompt }] }),
  });
  const data = await res.json();
  const text = (data.content || []).map((b) => (b.type === "text" ? b.text : "")).join("\n");
  return JSON.parse(text.replace(/```json|```/g, "").trim());
}

async function loadTopicData() {
  try {
    const r = await window.storage.get("topics-data", true);
    return JSON.parse(r.value);
  } catch (e) {
    const initial = { topics: SEED_TOPICS, history: {} };
    await window.storage.set("topics-data", JSON.stringify(initial), true);
    return initial;
  }
}
async function saveTopicData(data) {
  await window.storage.set("topics-data", JSON.stringify(data), true);
}

async function loadStreak() {
  try {
    const r = await window.storage.get("streak-data", false);
    return JSON.parse(r.value);
  } catch (e) {
    return { count: 0, lastDate: null };
  }
}
async function bumpStreak() {
  const s = await loadStreak();
  const key = todayKey();
  if (s.lastDate === key) return s;
  const next = s.lastDate === yesterdayKey() ? { count: s.count + 1, lastDate: key } : { count: 1, lastDate: key };
  await window.storage.set("streak-data", JSON.stringify(next), false);
  return next;
}

function pickTodayTopic(data) {
  const key = todayKey();
  if (data.history[key]) {
    const t = data.topics.find((x) => x.id === data.history[key].topicId);
    if (t) return { topic: t, reason: data.history[key].reason, data };
  }
  const category = CATEGORY_BY_WEEKDAY[new Date().getDay()];
  const pool = data.topics.filter((t) => t.category === category);
  const unused = pool.filter((t) => t.usedDates.length === 0);
  let chosen, reason;
  if (unused.length > 0) {
    chosen = unused.reduce((a, b) => (b.votes > a.votes ? b : a));
    reason = "votado";
  } else if (pool.length > 0) {
    chosen = pool[Math.floor(Math.random() * pool.length)];
    reason = "aleatorio";
  } else {
    return { topic: null, reason: "vacio", data };
  }
  chosen.usedDates.push(key);
  data.history[key] = { topicId: chosen.id, reason };
  return { topic: chosen, reason, data };
}

function AnimatedBar({ label, value, color }) {
  const [w, setW] = useState(0);
  useEffect(() => {
    const t = setTimeout(() => setW(value), 60);
    return () => clearTimeout(t);
  }, [value]);
  return (
    <div style={{ marginBottom: 14 }}>
      <div style={{ display: "flex", justifyContent: "space-between", fontFamily: "'IBM Plex Mono', monospace", fontSize: 12, color: TOKENS.ink70, marginBottom: 4, letterSpacing: 0.4 }}>
        <span>{label.toUpperCase()}</span>
        <span>{value}/100</span>
      </div>
      <div style={{ height: 8, background: TOKENS.paperDim, borderRadius: 999 }}>
        <div style={{ width: `${w}%`, height: "100%", background: color, borderRadius: 999, transition: "width 0.9s cubic-bezier(.2,.8,.2,1)" }} />
      </div>
    </div>
  );
}

function Rope({ percentFavor }) {
  return (
    <div style={{ margin: "22px 0 30px" }}>
      <div style={{ display: "flex", justifyContent: "space-between", fontFamily: "'IBM Plex Mono', monospace", fontSize: 11, letterSpacing: 1, color: TOKENS.ink70, marginBottom: 6 }}>
        <span style={{ color: TOKENS.favor }}>A FAVOR</span>
        <span style={{ color: TOKENS.contra }}>EN CONTRA</span>
      </div>
      <div style={{ position: "relative", height: 3, background: `linear-gradient(to right, ${TOKENS.favor}, ${TOKENS.contra})`, borderRadius: 2 }}>
        <div style={{ position: "absolute", left: `${percentFavor}%`, top: "50%", transform: "translate(-50%, -50%)", width: 16, height: 16, borderRadius: "50%", background: TOKENS.paper, border: `3px solid ${TOKENS.ink}`, transition: "left 0.7s cubic-bezier(.2,.8,.2,1)" }} />
      </div>
    </div>
  );
}

function Loading({ text }) {
  return <div style={{ fontFamily: "'IBM Plex Mono', monospace", fontSize: 13, color: TOKENS.ink45, padding: "18px 0" }}>{text}<span className="dots" /></div>;
}

function SuggestPanel({ data, setData, category, accent, onClose }) {
  const [text, setText] = useState("");
  const [busy, setBusy] = useState(false);
  const pool = data.topics.filter((t) => t.category === category).sort((a, b) => b.votes - a.votes);

  async function vote(id) {
    setBusy(true);
    const next = { ...data, topics: data.topics.map((t) => (t.id === id ? { ...t, votes: t.votes + 1 } : t)) };
    await saveTopicData(next);
    setData(next);
    setBusy(false);
  }
  async function addTopic() {
    if (!text.trim()) return;
    setBusy(true);
    const newTopic = { id: "t" + Date.now(), text: text.trim(), category, votes: 0, usedDates: [] };
    const next = { ...data, topics: [...data.topics, newTopic] };
    await saveTopicData(next);
    setData(next);
    setText("");
    setBusy(false);
  }

  return (
    <div style={{ background: TOKENS.paperDim, borderRadius: 8, padding: 16, marginTop: 20 }}>
      <p style={{ fontFamily: "'IBM Plex Mono', monospace", fontSize: 11, letterSpacing: 0.6, color: TOKENS.ink70, marginBottom: 10 }}>
        TEMAS PROPUESTOS · {category.toUpperCase()} · visibles para todo el mundo
      </p>
      {pool.length === 0 && <p style={{ fontSize: 13, color: TOKENS.ink70 }}>Aún no hay temas en esta categoría.</p>}
      {pool.map((t) => (
        <div key={t.id} style={{ display: "flex", justifyContent: "space-between", alignItems: "center", gap: 10, padding: "8px 0", borderBottom: `1px solid ${TOKENS.ink45}` }}>
          <span style={{ fontSize: 13.5 }}>{t.text}</span>
          <button className="df-vote-btn" onClick={() => vote(t.id)} disabled={busy} style={{ flexShrink: 0, fontFamily: "'IBM Plex Mono', monospace", fontSize: 12, border: `1.5px solid ${accent}`, background: "transparent", color: accent, borderRadius: 999, padding: "4px 10px", cursor: "pointer" }}>
            ▲ {t.votes}
          </button>
        </div>
      ))}
      <div style={{ display: "flex", gap: 8, marginTop: 12 }}>
        <input value={text} onChange={(e) => setText(e.target.value)} placeholder="Propón un tema de debate..." style={{ flex: 1, border: `1.5px solid ${TOKENS.ink45}`, borderRadius: 6, padding: "8px 10px", fontSize: 13.5, fontFamily: "inherit" }} />
        <button className="df-submit-btn" onClick={addTopic} disabled={busy || !text.trim()} style={{ background: TOKENS.ink, color: TOKENS.paper, border: "none", borderRadius: 6, padding: "8px 14px", fontWeight: 600, cursor: "pointer", opacity: busy || !text.trim() ? 0.55 : 1, fontSize: 13 }}>
          Proponer
        </button>
      </div>
      <button onClick={onClose} style={{ marginTop: 12, background: "none", border: "none", color: TOKENS.ink70, fontSize: 12.5, cursor: "pointer", textDecoration: "underline" }}>
        Cerrar
      </button>
    </div>
  );
}

export default function DebateDiario() {
  const [data, setData] = useState(null);
  const [todayTopic, setTodayTopic] = useState(null);
  const [reason, setReason] = useState(null);
  const [showSuggest, setShowSuggest] = useState(false);
  const [streak, setStreak] = useState(null);
  const [mounted, setMounted] = useState(false);

  const [stance, setStance] = useState(null);
  const [round, setRound] = useState(1);
  const [arg1, setArg1] = useState("");
  const [analysis1, setAnalysis1] = useState(null);
  const [loading1, setLoading1] = useState(false);
  const [opponent, setOpponent] = useState(null);
  const [loadingOpp, setLoadingOpp] = useState(false);
  const [arg2, setArg2] = useState("");
  const [analysis2, setAnalysis2] = useState(null);
  const [loading2, setLoading2] = useState(false);
  const [changedMind, setChangedMind] = useState(null);
  const [error, setError] = useState(null);
  const streakBumped = useRef(false);

  useEffect(() => {
    (async () => {
      const [d, s] = await Promise.all([loadTopicData(), loadStreak()]);
      const { topic, reason, data: updated } = pickTodayTopic(d);
      await saveTopicData(updated);
      setData(updated);
      setTodayTopic(topic);
      setReason(reason);
      setStreak(s);
      requestAnimationFrame(() => setMounted(true));
    })();
  }, []);

  useEffect(() => {
    if (changedMind && !streakBumped.current) {
      streakBumped.current = true;
      bumpStreak().then(setStreak);
    }
  }, [changedMind]);

  const stanceLabel = { favor: "A favor", contra: "En contra", duda: "No estoy segura" };
  const stanceColor = { favor: TOKENS.favor, contra: TOKENS.contra, duda: TOKENS.gold };
  const category = CATEGORY_BY_WEEKDAY[new Date().getDay()];
  const accent = ACCENT_BY_CATEGORY[category];

  async function submitRound1() {
    if (!arg1.trim()) return;
    setLoading1(true);
    setError(null);
    try {
      const prompt = `Eres un juez de debates riguroso pero constructivo. Tema: "${todayTopic.text}". Postura del participante: "${stanceLabel[stance]}". Su argumento: "${arg1}".
Responde SOLO con JSON válido: {"solidez": <0-100>, "evidencia": <0-100>, "falacias": ["..."], "puntos_faltantes": ["..."], "mejor_contraargumento": "..."}. Textos breves (máx 20 palabras).`;
      const result = await askClaude(prompt);
      setAnalysis1(result);
      setLoadingOpp(true);
      const oppPrompt = `Tema: "${todayTopic.text}". El participante defendió "${stanceLabel[stance]}" con: "${arg1}". Escribe en primera persona un argumento breve (máx 45 palabras) de alguien del bando contrario, basado en: "${result.mejor_contraargumento}". Responde SOLO el texto, sin comillas.`;
      const res = await fetch("https://api.anthropic.com/v1/messages", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ model: MODEL, max_tokens: 300, messages: [{ role: "user", content: oppPrompt }] }),
      });
      const d = await res.json();
      setOpponent((d.content || []).map((b) => b.text || "").join("\n").trim());
      setRound(2);
    } catch (e) {
      setError("No se pudo analizar el argumento. Inténtalo de nuevo.");
    } finally {
      setLoading1(false);
      setLoadingOpp(false);
    }
  }

  async function submitRound2() {
    if (!arg2.trim()) return;
    setLoading2(true);
    setError(null);
    try {
      const prompt = `Tema: "${todayTopic.text}". Postura "${stanceLabel[stance]}" responde a: "${opponent}". Su respuesta: "${arg2}".
Responde SOLO con JSON: {"capacidad_respuesta": <0-100>, "comprension_contrario": <0-100>, "comentario": "...", "probabilidad_persuasion": <0-100>}. Comentario máx 25 palabras.`;
      const result = await askClaude(prompt);
      setAnalysis2(result);
      setRound(3);
    } catch (e) {
      setError("No se pudo analizar la respuesta. Inténtalo de nuevo.");
    } finally {
      setLoading2(false);
    }
  }

  const nivel = analysis1 && analysis2 ? ((analysis1.solidez + analysis1.evidencia + analysis2.capacidad_respuesta + analysis2.comprension_contrario) / 40).toFixed(1) : null;

  return (
    <div style={{ minHeight: "100%", background: TOKENS.ink, padding: "28px 16px", fontFamily: "'Source Sans 3', 'Inter', sans-serif", color: TOKENS.ink, lineHeight: 1.55, WebkitFontSmoothing: "antialiased" }}>
      <style>{`
        @import url('https://fonts.googleapis.com/css2?family=Fraunces:opsz,wght@9..144,500;9..144,700&family=Source+Sans+3:wght@400;600&family=IBM+Plex+Mono:wght@400;500&display=swap');
        .dots::after { content: ''; animation: dots 1.4s steps(4,end) infinite; }
        @keyframes dots { 0%{content:''} 25%{content:'.'} 50%{content:'..'} 75%{content:'...'} 100%{content:''} }
        button:focus-visible, textarea:focus-visible, input:focus-visible { outline: 3px solid ${TOKENS.gold}; outline-offset: 2px; }
        .df-card { opacity: 0; transform: translateY(14px); transition: opacity 0.5s ease, transform 0.5s ease; }
        .df-card.on { opacity: 1; transform: translateY(0); }
        .df-stance-btn { transition: transform 0.15s ease, background 0.15s ease; }
        .df-stance-btn:hover { transform: translateY(-2px); background: rgba(20,26,34,0.04); }
        .df-submit-btn { transition: opacity 0.15s ease, transform 0.15s ease; }
        .df-submit-btn:hover:not(:disabled) { opacity: 0.88; transform: translateY(-1px); }
        .df-vote-btn { transition: background 0.15s ease, transform 0.15s ease; }
        .df-vote-btn:hover:not(:disabled) { transform: translateY(-1px); }
        @media (prefers-reduced-motion: reduce) { * { transition: none !important; animation: none !important; } .df-card { opacity: 1 !important; transform: none !important; } }
      `}</style>

      <div style={{ maxWidth: 640, margin: "0 auto" }}>
        <div style={{ display: "flex", justifyContent: "space-between", alignItems: "baseline", fontFamily: "'IBM Plex Mono', monospace", fontSize: 12, letterSpacing: 1, color: TOKENS.paper, opacity: 0.65, marginBottom: 10 }}>
          <span>DEBATE DIARIO</span>
          <span style={{ display: "flex", gap: 14 }}>
            {streak && streak.count > 0 && <span style={{ color: TOKENS.gold, opacity: 1 }}>RACHA · {streak.count} {streak.count === 1 ? "DÍA" : "DÍAS"}</span>}
            <span>RONDA {round}/3</span>
          </span>
        </div>

        <div className={`df-card${mounted ? " on" : ""}`} style={{ background: TOKENS.paper, borderRadius: 10, overflow: "hidden", boxShadow: "0 20px 50px rgba(0,0,0,0.35)" }}>
          <div style={{ height: 5, background: accent }} />
          <div style={{ padding: "26px 26px 28px" }}>
            {!data && <Loading text="Cargando el tema de hoy" />}

            {data && !todayTopic && (
              <div>
                <p style={{ fontFamily: "'Fraunces', serif", fontSize: 20, marginBottom: 10 }}>Aún no hay temas para hoy ({category}).</p>
                <p style={{ fontSize: 14, color: TOKENS.ink70, marginBottom: 16 }}>Sé la primera en proponer uno.</p>
                <SuggestPanel data={data} setData={setData} category={category} accent={accent} onClose={() => {}} />
              </div>
            )}

            {data && todayTopic && (
              <>
                <span style={{ display: "inline-block", fontFamily: "'IBM Plex Mono', monospace", fontSize: 11, letterSpacing: 0.8, color: accent, background: `${accent}1A`, padding: "3px 9px", borderRadius: 999, marginBottom: 12 }}>
                  {category.toUpperCase()}
                </span>
                <h1 style={{ fontFamily: "'Fraunces', serif", fontWeight: 700, fontSize: 30, letterSpacing: "-0.01em", lineHeight: 1.18, margin: "0 0 8px" }}>{todayTopic.text}</h1>
                <p style={{ fontFamily: "'IBM Plex Mono', monospace", fontSize: 11, letterSpacing: 0.4, color: TOKENS.ink45, marginBottom: 4 }}>
                  {reason === "votado" ? "elegido por votos" : "aleatorio, ya había salido antes"}
                </p>

                <Rope percentFavor={analysis2 ? Math.min(85, Math.max(15, analysis2.probabilidad_persuasion)) : 50} />

                {!stance && (
                  <div>
                    <p style={{ fontFamily: "'IBM Plex Mono', monospace", fontSize: 12, color: TOKENS.ink70, marginBottom: 12 }}>ELIGE TU POSTURA</p>
                    <div style={{ display: "flex", gap: 10, flexWrap: "wrap" }}>
                      {["favor", "contra", "duda"].map((s) => (
                        <button key={s} className="df-stance-btn" onClick={() => setStance(s)} style={{ flex: "1 1 140px", padding: "12px 14px", border: `2px solid ${stanceColor[s]}`, background: "transparent", color: stanceColor[s], fontWeight: 600, borderRadius: 8, cursor: "pointer", fontSize: 14 }}>
                          {stanceLabel[s]}
                        </button>
                      ))}
                    </div>
                    <button onClick={() => setShowSuggest((v) => !v)} style={{ marginTop: 16, background: "none", border: "none", color: TOKENS.ink70, fontSize: 12.5, cursor: "pointer", textDecoration: "underline" }}>
                      {showSuggest ? "Ocultar temas propuestos" : "Ver o proponer otros temas"}
                    </button>
                    {showSuggest && <SuggestPanel data={data} setData={setData} category={category} accent={accent} onClose={() => setShowSuggest(false)} />}
                  </div>
                )}

                {stance && round === 1 && (
                  <div>
                    <p style={{ fontFamily: "'IBM Plex Mono', monospace", fontSize: 12, color: TOKENS.ink70, marginBottom: 10 }}>
                      POSTURA: <span style={{ color: stanceColor[stance] }}>{stanceLabel[stance].toUpperCase()}</span> · ARGUMENTA TU POSICIÓN
                    </p>
                    <textarea value={arg1} onChange={(e) => setArg1(e.target.value)} placeholder="No vale “porque sí”. Explica tu razonamiento..." rows={4} style={{ width: "100%", border: `1.5px solid ${TOKENS.ink45}`, borderRadius: 8, padding: 12, fontFamily: "inherit", fontSize: 15, lineHeight: 1.5, resize: "vertical", background: "#fff", boxSizing: "border-box" }} />
                    <button className="df-submit-btn" onClick={submitRound1} disabled={loading1 || !arg1.trim()} style={{ marginTop: 12, padding: "11px 20px", background: TOKENS.ink, color: TOKENS.paper, border: "none", borderRadius: 8, fontWeight: 600, cursor: loading1 ? "default" : "pointer", opacity: loading1 || !arg1.trim() ? 0.55 : 1 }}>
                      {loading1 ? "Analizando..." : "Enviar argumento"}
                    </button>
                    {loading1 && <Loading text="Evaluando solidez y evidencia" />}
                    {loadingOpp && !loading1 && <Loading text="Preparando la réplica del otro bando" />}
                  </div>
                )}

                {round === 2 && analysis1 && (
                  <div>
                    <div style={{ background: TOKENS.paperDim, borderRadius: 8, padding: 16, marginBottom: 18 }}>
                      <AnimatedBar label="Solidez" value={analysis1.solidez} color={accent} />
                      <AnimatedBar label="Evidencia" value={analysis1.evidencia} color={accent} />
                      {analysis1.falacias?.length > 0 && <p style={{ fontSize: 13, color: TOKENS.contra, margin: "8px 0 4px" }}>⚠ {analysis1.falacias.join(" · ")}</p>}
                      {analysis1.puntos_faltantes?.length > 0 && <p style={{ fontSize: 13, color: TOKENS.ink70, margin: "4px 0 0" }}>Te falta: {analysis1.puntos_faltantes.join(" · ")}</p>}
                    </div>
                    <div style={{ background: stance === "favor" ? TOKENS.contraSoft : TOKENS.favorSoft, borderLeft: `4px solid ${stance === "favor" ? TOKENS.contra : TOKENS.favor}`, padding: 14, borderRadius: 8, marginBottom: 16, fontSize: 15, lineHeight: 1.5, fontFamily: "'Fraunces', serif", fontStyle: "italic" }}>
                      “{opponent}”
                    </div>
                    <p style={{ fontFamily: "'IBM Plex Mono', monospace", fontSize: 12, letterSpacing: 0.4, color: TOKENS.ink70, marginBottom: 10 }}>RONDA 2 · RESPONDE</p>
                    <textarea value={arg2} onChange={(e) => setArg2(e.target.value)} placeholder="Responde al argumento contrario..." rows={4} style={{ width: "100%", border: `1.5px solid ${TOKENS.ink45}`, borderRadius: 8, padding: 12, fontFamily: "inherit", fontSize: 15, lineHeight: 1.5, resize: "vertical", background: "#fff", boxSizing: "border-box" }} />
                    <button className="df-submit-btn" onClick={submitRound2} disabled={loading2 || !arg2.trim()} style={{ marginTop: 12, padding: "11px 20px", background: TOKENS.ink, color: TOKENS.paper, border: "none", borderRadius: 8, fontWeight: 600, cursor: loading2 ? "default" : "pointer", opacity: loading2 || !arg2.trim() ? 0.55 : 1 }}>
                      {loading2 ? "Analizando..." : "Enviar respuesta"}
                    </button>
                    {loading2 && <Loading text="Midiendo capacidad de réplica" />}
                  </div>
                )}

                {round === 3 && analysis2 && !changedMind && (
                  <div>
                    <p style={{ fontSize: 14, marginBottom: 14 }}>{analysis2.comentario}</p>
                    <p style={{ fontFamily: "'IBM Plex Mono', monospace", fontSize: 12, color: TOKENS.ink70, marginBottom: 10 }}>¿HAS CAMBIADO DE OPINIÓN?</p>
                    <div style={{ display: "flex", gap: 10, flexWrap: "wrap" }}>
                      {[{ k: "no", l: "No", c: TOKENS.contra }, { k: "poco", l: "Un poco", c: TOKENS.gold }, { k: "si", l: "Sí", c: TOKENS.favor }].map((o) => (
                        <button key={o.k} className="df-stance-btn" onClick={() => setChangedMind(o.k)} style={{ flex: "1 1 100px", padding: "11px 14px", border: `2px solid ${o.c}`, background: "transparent", color: o.c, fontWeight: 600, borderRadius: 8, cursor: "pointer" }}>
                          {o.l}
                        </button>
                      ))}
                    </div>
                  </div>
                )}

                {changedMind && analysis1 && analysis2 && (
                  <div>
                    <p style={{ fontFamily: "'IBM Plex Mono', monospace", fontSize: 12, letterSpacing: 1, color: TOKENS.ink70, marginBottom: 14 }}>TU RESULTADO DE HOY</p>
                    <AnimatedBar label="Argumentación" value={analysis1.solidez} color={TOKENS.favor} />
                    <AnimatedBar label="Evidencia" value={analysis1.evidencia} color={TOKENS.favor} />
                    <AnimatedBar label="Capacidad de responder" value={analysis2.capacidad_respuesta} color={TOKENS.favor} />
                    <AnimatedBar label="Comprensión del contrario" value={analysis2.comprension_contrario} color={TOKENS.favor} />
                    <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginTop: 18, paddingTop: 16, borderTop: `1.5px solid ${TOKENS.ink45}` }}>
                      <span style={{ fontFamily: "'Fraunces', serif", fontSize: 16 }}>Nivel de debatiente</span>
                      <span style={{ fontFamily: "'IBM Plex Mono', monospace", fontSize: 28, fontWeight: 600, letterSpacing: "-0.02em", color: accent }}>{nivel}/10</span>
                    </div>
                    <div style={{ background: `${accent}1A`, borderRadius: 8, padding: 14, marginTop: 16 }}>
                      <p style={{ fontFamily: "'IBM Plex Mono', monospace", fontSize: 11, letterSpacing: 0.6, marginBottom: 4, color: accent }}>ÍNDICE DE PERSUASIÓN (simulado)</p>
                      <p style={{ fontSize: 13, margin: 0 }}>
                        Tu respuesta tendría un {analysis2.probabilidad_persuasion}% de probabilidad estimada de convencer a alguien del bando contrario. En la versión con backend, este número vendría de votos reales de otras personas.
                      </p>
                    </div>
                    {streak && (
                      <p style={{ fontSize: 13, color: TOKENS.ink70, marginTop: 14 }}>
                        Llevas <span style={{ color: accent, fontWeight: 600 }}>{streak.count} {streak.count === 1 ? "día" : "días"}</span> seguidos debatiendo. Vuelve mañana para no perder la racha.
                      </p>
                    )}
                  </div>
                )}
              </>
            )}

            {error && <p style={{ color: TOKENS.contra, fontSize: 13, marginTop: 10 }}>{error}</p>}
          </div>
        </div>
      </div>
    </div>
  );
}
