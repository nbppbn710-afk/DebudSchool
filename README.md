# DebudSchool
Okok
import { useState, useEffect, useRef } from "react";

const mockTransactions = [
  { id: 1, date: "2026-05-28", desc: "Supermarket Alpha", amount: -320000, category: "Groceries", account: "BCA", type: "expense", receipt: null },
  { id: 2, date: "2026-05-27", desc: "Gaji Mei 2026", amount: 8500000, category: "Income", account: "BCA", type: "income", receipt: null },
  { id: 3, date: "2026-05-26", desc: "Netflix Subscription", amount: -186000, category: "Entertainment", account: "Mandiri", type: "expense", receipt: null },
  { id: 4, date: "2026-05-25", desc: "Grab Food", amount: -95000, category: "Food & Dining", account: "GoPay", type: "expense", receipt: null },
  { id: 5, date: "2026-05-24", desc: "PLN Token", amount: -250000, category: "Utilities", account: "BCA", type: "expense", receipt: null },
  { id: 6, date: "2026-05-23", desc: "Gym Membership", amount: -450000, category: "Health", account: "Mandiri", type: "expense", receipt: null },
  { id: 7, date: "2026-05-22", desc: "Shopee Transfer", amount: -175000, category: "Shopping", account: "GoPay", type: "expense", receipt: null },
  { id: 8, date: "2026-05-21", desc: "Freelance Project", amount: 2000000, category: "Income", account: "BCA", type: "income", receipt: null },
  { id: 9, date: "2026-05-20", desc: "Bensin Pertamina", amount: -120000, category: "Transport", account: "GoPay", type: "expense", receipt: null },
  { id: 10, date: "2026-05-19", desc: "Kopi Kenangan", amount: -45000, category: "Food & Dining", account: "GoPay", type: "expense", receipt: null },
];

const mockBudgets = [
  { category: "Groceries", limit: 1500000, spent: 320000, color: "#34d399" },
  { category: "Food & Dining", limit: 800000, spent: 650000, color: "#f59e0b" },
  { category: "Entertainment", limit: 400000, spent: 380000, color: "#f87171" },
  { category: "Transport", limit: 500000, spent: 120000, color: "#60a5fa" },
  { category: "Utilities", limit: 600000, spent: 250000, color: "#a78bfa" },
  { category: "Health", limit: 500000, spent: 450000, color: "#fb923c" },
  { category: "Shopping", limit: 1000000, spent: 175000, color: "#38bdf8" },
];

const mockSharedWallet = [
  { member: "Anda", avatar: "A", spent: 1145000, color: "#6366f1" },
  { member: "Pasangan", avatar: "P", spent: 836000, color: "#ec4899" },
];

const mockPaymentSchedule = [
  { id: 1, name: "Cicilan KPR", amount: 2500000, dueDay: 5, account: "BCA", category: "Housing", status: "upcoming", autoDebit: true },
  { id: 2, name: "Asuransi Jiwa", amount: 350000, dueDay: 10, account: "Mandiri", category: "Insurance", status: "upcoming", autoDebit: false },
  { id: 3, name: "Internet Indihome", amount: 375000, dueDay: 15, account: "BCA", category: "Utilities", status: "paid", autoDebit: true },
  { id: 4, name: "BPJS Kesehatan", amount: 150000, dueDay: 20, account: "BCA", category: "Health", status: "missed", autoDebit: false },
  { id: 5, name: "Spotify Premium", amount: 54990, dueDay: 25, account: "GoPay", category: "Entertainment", status: "upcoming", autoDebit: true },
  { id: 6, name: "Kartu Kredit Mandiri", amount: 1200000, dueDay: 28, account: "Mandiri", category: "Credit Card", status: "upcoming", autoDebit: false },
];

const CATEGORIES = ["Groceries", "Food & Dining", "Entertainment", "Transport", "Utilities", "Health", "Shopping", "Income", "Housing", "Insurance", "Credit Card", "Other"];
const ACCOUNTS = ["BCA", "Mandiri", "GoPay", "OVO", "Dana"];

const fmt = (n) => new Intl.NumberFormat("id-ID", { style: "currency", currency: "IDR", maximumFractionDigits: 0 }).format(n);

export default function App() {
  const [tab, setTab] = useState("dashboard");
  const [privacyMode, setPrivacyMode] = useState(false);
  const [transactions, setTransactions] = useState(mockTransactions);
  const [budgets] = useState(mockBudgets);
  const [paymentSchedule, setPaymentSchedule] = useState(mockPaymentSchedule);
  const [showAddModal, setShowAddModal] = useState(false);
  const [showOCRModal, setShowOCRModal] = useState(false);
  const [showPaymentModal, setShowPaymentModal] = useState(false);
  const [aiLoading, setAiLoading] = useState(false);
  const [aiInsight, setAiInsight] = useState("");
  const [newTx, setNewTx] = useState({ desc: "", amount: "", category: "Food & Dining", account: "BCA", type: "expense", date: "2026-06-01" });
  const [newPayment, setNewPayment] = useState({ name: "", amount: "", dueDay: 1, account: "BCA", category: "Other", autoDebit: false });
  const [ocrText, setOcrText] = useState("");
  const [ocrLoading, setOcrLoading] = useState(false);
  const [exportLoading, setExportLoading] = useState("");
  const [filterRange, setFilterRange] = useState("month");
  const [notification, setNotification] = useState(null);
  const fileRef = useRef();

  const totalBalance = 24750000;
  const totalIncome = transactions.filter(t => t.type === "income").reduce((s, t) => s + t.amount, 0);
  const totalExpense = Math.abs(transactions.filter(t => t.type === "expense").reduce((s, t) => s + t.amount, 0));

  const showNotif = (msg, type = "success") => {
    setNotification({ msg, type });
    setTimeout(() => setNotification(null), 3500);
  };

  const addTransaction = () => {
    if (!newTx.desc || !newTx.amount) return;
    const amt = newTx.type === "expense" ? -Math.abs(Number(newTx.amount)) : Math.abs(Number(newTx.amount));
    setTransactions([{ ...newTx, id: Date.now(), amount: amt }, ...transactions]);
    setShowAddModal(false);
    setNewTx({ desc: "", amount: "", category: "Food & Dining", account: "BCA", type: "expense", date: "2026-06-01" });
    showNotif("Transaksi berhasil ditambahkan!");
  };

  const addPayment = () => {
    if (!newPayment.name || !newPayment.amount) return;
    setPaymentSchedule([...paymentSchedule, { ...newPayment, id: Date.now(), amount: Number(newPayment.amount), status: "upcoming" }]);
    setShowPaymentModal(false);
    setNewPayment({ name: "", amount: "", dueDay: 1, account: "BCA", category: "Other", autoDebit: false });
    showNotif("Tagihan berhasil dijadwalkan!");
  };

  const markPayment = (id, status) => {
    setPaymentSchedule(paymentSchedule.map(p => p.id === id ? { ...p, status } : p));
    showNotif(status === "paid" ? "Pembayaran ditandai lunas!" : "Status diperbarui.");
  };

  const handleOCR = async (file) => {
    if (!file) return;
    setOcrLoading(true);
    setOcrText("");
    const reader = new FileReader();
    reader.onload = async (e) => {
      const base64 = e.target.result.split(",")[1];
      try {
        const res = await fetch("https://api.anthropic.com/v1/messages", {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({
            model: "claude-sonnet-4-20250514",
            max_tokens: 1000,
            messages: [{
              role: "user",
              content: [
                { type: "image", source: { type: "base64", media_type: file.type || "image/jpeg", data: base64 } },
                { type: "text", text: `Kamu adalah asisten OCR keuangan. Baca struk/tagihan ini dan ekstrak informasi berikut dalam format JSON:\n{\n  "merchant": "nama toko",\n  "date": "tanggal (YYYY-MM-DD)",\n  "items": [{"name": "...", "price": 0}],\n  "total": 0,\n  "category": "kategori pengeluaran (pilih dari: Groceries, Food & Dining, Entertainment, Transport, Utilities, Health, Shopping, Other)"\n}\nJika tidak ada struk, kembalikan JSON kosong dengan total 0. Hanya balas dengan JSON murni tanpa penjelasan.` }
              ]
            }]
          })
        });
        const data = await res.json();
        const raw = data.content?.map(c => c.text || "").join("") || "{}";
        const clean = raw.replace(/```json|```/g, "").trim();
        const parsed = JSON.parse(clean);
        setOcrText(JSON.stringify(parsed, null, 2));
        if (parsed.total && parsed.total > 0) {
          setNewTx(prev => ({
            ...prev,
            desc: parsed.merchant || "Struk Belanja",
            amount: String(parsed.total),
            category: parsed.category || "Groceries",
            date: parsed.date || prev.date,
            type: "expense"
          }));
          setShowOCRModal(false);
          setShowAddModal(true);
          showNotif("OCR berhasil! Data terisi otomatis.");
        }
      } catch {
        setOcrText('{"error": "Gagal membaca gambar. Pastikan gambar jelas."}');
      }
      setOcrLoading(false);
    };
    reader.readAsDataURL(file);
  };

  const fetchAIInsight = async () => {
    setAiLoading(true);
    setAiInsight("");
    const summary = {
      totalIncome, totalExpense,
      topCategories: budgets.map(b => ({ cat: b.category, spent: b.spent })),
      budgetAlerts: budgets.filter(b => b.spent / b.limit >= 0.8).map(b => b.category),
      transactions: transactions.slice(0, 10).map(t => ({ desc: t.desc, amount: t.amount, category: t.category }))
    };
    try {
      const res = await fetch("https://api.anthropic.com/v1/messages", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          model: "claude-sonnet-4-20250514",
          max_tokens: 1000,
          messages: [{
            role: "user",
            content: `Kamu adalah AI Financial Advisor. Analisis data keuangan berikut dan berikan insight singkat dalam Bahasa Indonesia:\n${JSON.stringify(summary)}\n\nBerikan:\n1. 🔍 Analisis pola pengeluaran (2-3 kalimat)\n2. 💡 3 saran penghematan spesifik\n3. 📈 Prediksi cash flow bulan depan\n4. ⚠️ Peringatan anggaran (jika ada)\n\nGunakan format yang mudah dibaca dengan emoji. Maksimal 200 kata.`
          }]
        })
      });
      const data = await res.json();
      setAiInsight(data.content?.map(c => c.text || "").join("") || "Gagal mengambil insight.");
    } catch {
      setAiInsight("❌ Tidak dapat terhubung ke AI. Periksa koneksi Anda.");
    }
    setAiLoading(false);
  };

  const exportReport = async (format) => {
    setExportLoading(format);
    try {
      const res = await fetch("https://api.anthropic.com/v1/messages", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          model: "claude-sonnet-4-20250514",
          max_tokens: 1000,
          messages: [{
            role: "user",
            content: `Buat ringkasan laporan keuangan dalam format ${format.toUpperCase()} untuk periode Mei 2026. Data: total pemasukan Rp ${fmt(totalIncome)}, total pengeluaran Rp ${fmt(totalExpense)}, saldo Rp ${fmt(totalBalance)}. Kategori terbesar: ${budgets[0].category} Rp ${fmt(budgets[0].spent)}. Berikan ringkasan naratif profesional dalam 3 paragraf singkat dalam Bahasa Indonesia. Format laporan untuk dokumen ${format === "pdf" ? "PDF visual" : format === "excel" ? "Excel spreadsheet" : "Word dokumen"}.`
          }]
        })
      });
      const data = await res.json();
      const content = data.content?.map(c => c.text || "").join("") || "";
      const blob = new Blob([`LAPORAN KEUANGAN PRIBADI - MEI 2026\n\n${content}`], { type: "text/plain" });
      const url = URL.createObjectURL(blob);
      const a = document.createElement("a");
      a.href = url;
      a.download = `laporan-keuangan-mei2026.${format === "excel" ? "csv" : format === "word" ? "txt" : "txt"}`;
      a.click();
      showNotif(`Laporan ${format.toUpperCase()} berhasil diunduh!`);
    } catch {
      showNotif("Gagal mengekspor laporan.", "error");
    }
    setExportLoading("");
  };

  const missedPayments = paymentSchedule.filter(p => p.status === "missed").length;
  const upcomingPayments = paymentSchedule.filter(p => p.status === "upcoming");

  return (
    <div style={{ fontFamily: "'DM Sans', system-ui, sans-serif", background: "#0f0f13", minHeight: "100vh", color: "#e8e8f0" }}>
      <style>{`
        @import url('https://fonts.googleapis.com/css2?family=DM+Sans:wght@300;400;500;600;700&family=Space+Mono:wght@400;700&display=swap');
        * { box-sizing: border-box; margin: 0; padding: 0; }
        ::-webkit-scrollbar { width: 4px; } ::-webkit-scrollbar-track { background: #1a1a22; } ::-webkit-scrollbar-thumb { background: #3a3a52; border-radius: 4px; }
        .card { background: #16161e; border: 1px solid #2a2a38; border-radius: 16px; padding: 20px; }
        .btn { border: none; cursor: pointer; border-radius: 10px; font-family: inherit; font-weight: 600; transition: all 0.2s; }
        .btn-primary { background: linear-gradient(135deg, #6366f1, #8b5cf6); color: white; padding: 10px 20px; }
        .btn-primary:hover { transform: translateY(-1px); box-shadow: 0 4px 20px rgba(99,102,241,0.4); }
        .btn-ghost { background: transparent; color: #a0a0c0; padding: 8px 14px; border: 1px solid #2a2a38; }
        .btn-ghost:hover { background: #1e1e2a; color: #e8e8f0; }
        .btn-danger { background: #ef4444; color: white; padding: 8px 14px; }
        .btn-success { background: #10b981; color: white; padding: 8px 14px; }
        .input { background: #1a1a24; border: 1px solid #2a2a38; border-radius: 10px; padding: 10px 14px; color: #e8e8f0; font-family: inherit; font-size: 14px; width: 100%; outline: none; transition: border 0.2s; }
        .input:focus { border-color: #6366f1; }
        .badge { display: inline-block; padding: 3px 10px; border-radius: 20px; font-size: 11px; font-weight: 700; letter-spacing: 0.5px; }
        .badge-green { background: rgba(52,211,153,0.15); color: #34d399; }
        .badge-red { background: rgba(248,113,113,0.15); color: #f87171; }
        .badge-yellow { background: rgba(251,191,36,0.15); color: #fbbf24; }
        .badge-blue { background: rgba(96,165,250,0.15); color: #60a5fa; }
        .tab { padding: 8px 16px; border-radius: 8px; font-size: 13px; font-weight: 600; cursor: pointer; transition: all 0.2s; color: #6060a0; border: none; background: transparent; font-family: inherit; }
        .tab.active { background: #1e1e2c; color: #a5b4fc; }
        .tab:hover:not(.active) { color: #c0c0e0; }
        .modal-bg { position: fixed; inset: 0; background: rgba(0,0,0,0.7); backdrop-filter: blur(4px); z-index: 100; display: flex; align-items: center; justify-content: center; padding: 20px; }
        .modal { background: #16161e; border: 1px solid #2a2a38; border-radius: 20px; padding: 24px; width: 100%; max-width: 440px; max-height: 90vh; overflow-y: auto; }
        .notif { position: fixed; top: 20px; right: 20px; z-index: 200; padding: 12px 20px; border-radius: 12px; font-weight: 600; font-size: 14px; animation: slideIn 0.3s ease; }
        .notif-success { background: #10b981; color: white; }
        .notif-error { background: #ef4444; color: white; }
        @keyframes slideIn { from { transform: translateX(100px); opacity: 0; } to { transform: translateX(0); opacity: 1; } }
        .progress-bar { height: 6px; background: #2a2a38; border-radius: 10px; overflow: hidden; }
        .progress-fill { height: 100%; border-radius: 10px; transition: width 0.5s; }
        .stat-card { background: #16161e; border: 1px solid #2a2a38; border-radius: 14px; padding: 16px; }
        .blur-mode { filter: blur(8px); user-select: none; }
        select.input { appearance: none; }
        .ai-card { background: linear-gradient(135deg, #0f0f1a, #1a1030); border: 1px solid #3a2a5a; border-radius: 16px; padding: 20px; }
        .shared-bar { height: 8px; border-radius: 10px; display: flex; overflow: hidden; }
        .tx-row { display: flex; align-items: center; gap: 12px; padding: 12px 0; border-bottom: 1px solid #1e1e2a; }
        .tx-row:last-child { border-bottom: none; }
        .avatar { width: 36px; height: 36px; border-radius: 10px; display: flex; align-items: center; justify-content: center; font-size: 14px; font-weight: 700; flex-shrink: 0; }
        .missed-badge { animation: pulse 2s infinite; }
        @keyframes pulse { 0%,100% { opacity:1; } 50% { opacity:0.6; } }
        .payment-row { display: flex; align-items: center; gap: 10px; padding: 12px; background: #1a1a24; border-radius: 12px; margin-bottom: 8px; }
        .hero-balance { font-family: 'Space Mono', monospace; font-size: 36px; font-weight: 700; letter-spacing: -1px; }
      `}</style>

      {/* Notification */}
      {notification && (
        <div className={`notif notif-${notification.type}`}>{notification.msg}</div>
      )}

      {/* Header */}
      <div style={{ background: "#12121a", borderBottom: "1px solid #1e1e2a", padding: "16px 20px", display: "flex", alignItems: "center", justifyContent: "space-between", position: "sticky", top: 0, zIndex: 50 }}>
        <div style={{ display: "flex", alignItems: "center", gap: 10 }}>
          <div style={{ width: 32, height: 32, background: "linear-gradient(135deg,#6366f1,#8b5cf6)", borderRadius: 10, display: "flex", alignItems: "center", justifyContent: "center", fontSize: 16 }}>💰</div>
          <div>
            <div style={{ fontSize: 14, fontWeight: 700, color: "#e8e8f0" }}>FinanceOS</div>
            <div style={{ fontSize: 10, color: "#6060a0" }}>Personal Finance Manager</div>
          </div>
        </div>
        <div style={{ display: "flex", gap: 8, alignItems: "center" }}>
          {missedPayments > 0 && (
            <div className="missed-badge" style={{ background: "rgba(239,68,68,0.15)", color: "#f87171", fontSize: 11, fontWeight: 700, padding: "4px 10px", borderRadius: 20, border: "1px solid rgba(239,68,68,0.3)" }}>
              ⚠️ {missedPayments} Terlewat
            </div>
          )}
          <button className="btn btn-ghost" style={{ fontSize: 12, padding: "6px 12px" }} onClick={() => setPrivacyMode(!privacyMode)}>
            {privacyMode ? "👁️ Show" : "🙈 Hide"}
          </button>
          <button className="btn btn-primary" style={{ fontSize: 12, padding: "8px 14px" }} onClick={() => setShowAddModal(true)}>+ Tambah</button>
        </div>
      </div>

      {/* Tabs */}
      <div style={{ padding: "12px 20px", display: "flex", gap: 4, overflowX: "auto", borderBottom: "1px solid #1a1a24" }}>
        {[
          { id: "dashboard", label: "📊 Dashboard" },
          { id: "transactions", label: "📋 Transaksi" },
          { id: "budget", label: "🎯 Anggaran" },
          { id: "payments", label: "📅 Tagihan" },
          { id: "shared", label: "👨‍👩‍👧 Shared" },
          { id: "insights", label: "🤖 AI Insight" },
          { id: "reports", label: "📄 Laporan" },
        ].map(t => (
          <button key={t.id} className={`tab ${tab === t.id ? "active" : ""}`} onClick={() => setTab(t.id)}>{t.label}</button>
        ))}
      </div>

      <div style={{ padding: "20px", maxWidth: 760, margin: "0 auto" }}>

        {/* ═══ DASHBOARD ═══ */}
        {tab === "dashboard" && (
          <div style={{ display: "flex", flexDirection: "column", gap: 16 }}>
            {/* Balance Card */}
            <div style={{ background: "linear-gradient(135deg, #1a1040, #0f1a30)", border: "1px solid #2a1a5a", borderRadius: 20, padding: 24, position: "relative", overflow: "hidden" }}>
              <div style={{ position: "absolute", top: -40, right: -40, width: 160, height: 160, background: "radial-gradient(circle, rgba(99,102,241,0.15), transparent)", borderRadius: "50%" }} />
              <div style={{ fontSize: 12, color: "#8080c0", marginBottom: 8, fontWeight: 600, letterSpacing: 1 }}>TOTAL SALDO</div>
              <div className={`hero-balance ${privacyMode ? "blur-mode" : ""}`}>{fmt(totalBalance)}</div>
              <div style={{ marginTop: 16, display: "flex", gap: 24 }}>
                <div>
                  <div style={{ fontSize: 11, color: "#60a060", marginBottom: 2 }}>▲ Pemasukan</div>
                  <div className={`${privacyMode ? "blur-mode" : ""}`} style={{ fontWeight: 700, color: "#34d399", fontFamily: "Space Mono", fontSize: 14 }}>{fmt(totalIncome)}</div>
                </div>
                <div>
                  <div style={{ fontSize: 11, color: "#a06060", marginBottom: 2 }}>▼ Pengeluaran</div>
                  <div className={`${privacyMode ? "blur-mode" : ""}`} style={{ fontWeight: 700, color: "#f87171", fontFamily: "Space Mono", fontSize: 14 }}>{fmt(totalExpense)}</div>
                </div>
              </div>
            </div>

            {/* Account Cards */}
            <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr 1fr", gap: 10 }}>
              {[{ name: "BCA", bal: 14250000, color: "#3b82f6" }, { name: "Mandiri", bal: 7300000, color: "#f59e0b" }, { name: "GoPay", bal: 3200000, color: "#10b981" }].map(acc => (
                <div key={acc.name} className="stat-card" style={{ textAlign: "center" }}>
                  <div style={{ fontSize: 12, fontWeight: 700, color: acc.color, marginBottom: 4 }}>{acc.name}</div>
                  <div className={`${privacyMode ? "blur-mode" : ""}`} style={{ fontSize: 13, fontWeight: 700, fontFamily: "Space Mono", color: "#e8e8f0" }}>{fmt(acc.bal)}</div>
                </div>
              ))}
            </div>

            {/* Budget Alerts */}
            {budgets.filter(b => b.spent / b.limit >= 0.8).map(b => (
              <div key={b.category} style={{ background: "rgba(251,191,36,0.08)", border: "1px solid rgba(251,191,36,0.25)", borderRadius: 12, padding: "12px 16px", display: "flex", alignItems: "center", justifyContent: "space-between" }}>
                <div style={{ fontSize: 13 }}>⚠️ <strong>{b.category}</strong>: {Math.round(b.spent / b.limit * 100)}% dari anggaran terpakai</div>
                <span className="badge badge-yellow">{fmt(b.limit - b.spent)} sisa</span>
              </div>
            ))}

            {/* Upcoming Payments Alert */}
            {upcomingPayments.slice(0, 2).map(p => (
              <div key={p.id} style={{ background: "rgba(96,165,250,0.08)", border: "1px solid rgba(96,165,250,0.2)", borderRadius: 12, padding: "12px 16px", display: "flex", alignItems: "center", justifyContent: "space-between" }}>
                <div style={{ fontSize: 13 }}>📅 <strong>{p.name}</strong> jatuh tempo tgl {p.dueDay}</div>
                <span className="badge badge-blue">{fmt(p.amount)}</span>
              </div>
            ))}

            {/* Recent Transactions */}
            <div className="card">
              <div style={{ fontSize: 14, fontWeight: 700, marginBottom: 14, color: "#a0a0c0" }}>TRANSAKSI TERBARU</div>
              {transactions.slice(0, 5).map(tx => (
                <div key={tx.id} className="tx-row">
                  <div className="avatar" style={{ background: tx.type === "income" ? "rgba(52,211,153,0.1)" : "rgba(248,113,113,0.1)" }}>
                    {tx.type === "income" ? "💵" : "🛍️"}
                  </div>
                  <div style={{ flex: 1 }}>
                    <div style={{ fontSize: 13, fontWeight: 600, color: "#e0e0f0" }}>{tx.desc}</div>
                    <div style={{ fontSize: 11, color: "#6060a0" }}>{tx.date} · {tx.category} · {tx.account}</div>
                  </div>
                  <div className={`${privacyMode ? "blur-mode" : ""}`} style={{ fontFamily: "Space Mono", fontSize: 13, fontWeight: 700, color: tx.amount > 0 ? "#34d399" : "#f87171" }}>
                    {tx.amount > 0 ? "+" : ""}{fmt(tx.amount)}
                  </div>
                </div>
              ))}
            </div>
          </div>
        )}

        {/* ═══ TRANSACTIONS ═══ */}
        {tab === "transactions" && (
          <div style={{ display: "flex", flexDirection: "column", gap: 12 }}>
            <div style={{ display: "flex", gap: 8, alignItems: "center" }}>
              <div style={{ flex: 1 }}>
                <div style={{ fontSize: 18, fontWeight: 700 }}>Semua Transaksi</div>
                <div style={{ fontSize: 12, color: "#6060a0" }}>{transactions.length} transaksi tercatat</div>
              </div>
              <button className="btn btn-ghost" style={{ fontSize: 12, padding: "8px 14px" }} onClick={() => setShowOCRModal(true)}>📷 Scan Struk</button>
            </div>
            <div style={{ display: "flex", gap: 6 }}>
              {["week", "month", "3month", "year"].map(r => (
                <button key={r} className={`btn ${filterRange === r ? "btn-primary" : "btn-ghost"}`} style={{ fontSize: 11, padding: "6px 12px" }} onClick={() => setFilterRange(r)}>
                  {{ week: "7 Hari", month: "Bulan Ini", "3month": "3 Bulan", year: "Tahun Ini" }[r]}
                </button>
              ))}
            </div>
            <div className="card" style={{ padding: 0, overflow: "hidden" }}>
              {transactions.map(tx => (
                <div key={tx.id} className="tx-row" style={{ padding: "12px 16px" }}>
                  <div className="avatar" style={{ background: tx.type === "income" ? "rgba(52,211,153,0.1)" : "rgba(248,113,113,0.1)", color: tx.type === "income" ? "#34d399" : "#f87171" }}>
                    {tx.type === "income" ? "📥" : "📤"}
                  </div>
                  <div style={{ flex: 1 }}>
                    <div style={{ fontSize: 13, fontWeight: 600 }}>{tx.desc}</div>
                    <div style={{ fontSize: 11, color: "#6060a0", marginTop: 2 }}>
                      {tx.date} · <span className="badge badge-blue" style={{ fontSize: 10 }}>{tx.category}</span> · {tx.account}
                    </div>
                  </div>
                  <div className={`${privacyMode ? "blur-mode" : ""}`} style={{ fontFamily: "Space Mono", fontSize: 13, fontWeight: 700, color: tx.amount > 0 ? "#34d399" : "#f87171", textAlign: "right" }}>
                    {tx.amount > 0 ? "+" : ""}{fmt(tx.amount)}
                  </div>
                </div>
              ))}
            </div>
          </div>
        )}

        {/* ═══ BUDGET ═══ */}
        {tab === "budget" && (
          <div style={{ display: "flex", flexDirection: "column", gap: 12 }}>
            <div>
              <div style={{ fontSize: 18, fontWeight: 700 }}>Smart Budgeting</div>
              <div style={{ fontSize: 12, color: "#6060a0" }}>Anggaran Mei 2026 · Peringatan otomatis di 80%</div>
            </div>
            {budgets.map(b => {
              const pct = Math.min(b.spent / b.limit * 100, 100);
              const over = b.spent >= b.limit;
              const warn = pct >= 80;
              return (
                <div key={b.category} className="card">
                  <div style={{ display: "flex", justifyContent: "space-between", marginBottom: 10, alignItems: "flex-start" }}>
                    <div>
                      <div style={{ fontSize: 14, fontWeight: 700, color: "#e0e0f0" }}>{b.category}</div>
                      <div style={{ fontSize: 11, color: "#6060a0", marginTop: 2 }}>
                        <span className={`${privacyMode ? "blur-mode" : ""}`}>{fmt(b.spent)}</span> / <span className={`${privacyMode ? "blur-mode" : ""}`}>{fmt(b.limit)}</span>
                      </div>
                    </div>
                    <div style={{ textAlign: "right" }}>
                      <span className={`badge ${over ? "badge-red" : warn ? "badge-yellow" : "badge-green"}`}>{pct.toFixed(0)}%</span>
                      {warn && !over && <div style={{ fontSize: 10, color: "#fbbf24", marginTop: 4 }}>⚠️ Hampir habis</div>}
                      {over && <div style={{ fontSize: 10, color: "#f87171", marginTop: 4 }}>🚨 Melewati batas</div>}
                    </div>
                  </div>
                  <div className="progress-bar">
                    <div className="progress-fill" style={{ width: `${pct}%`, background: over ? "#ef4444" : warn ? "#f59e0b" : b.color }} />
                  </div>
                  <div style={{ fontSize: 11, color: "#6060a0", marginTop: 8 }}>
                    Sisa: <span style={{ color: over ? "#f87171" : "#34d399", fontWeight: 600 }} className={`${privacyMode ? "blur-mode" : ""}`}>{fmt(Math.max(0, b.limit - b.spent))}</span>
                  </div>
                </div>
              );
            })}
          </div>
        )}

        {/* ═══ PAYMENT SCHEDULE ═══ */}
        {tab === "payments" && (
          <div style={{ display: "flex", flexDirection: "column", gap: 12 }}>
            <div style={{ display: "flex", alignItems: "center", justifyContent: "space-between" }}>
              <div>
                <div style={{ fontSize: 18, fontWeight: 700 }}>Jadwal Tagihan</div>
                <div style={{ fontSize: 12, color: "#6060a0" }}>Absen & pantau pembayaran rutin</div>
              </div>
              <button className="btn btn-primary" style={{ fontSize: 12, padding: "8px 14px" }} onClick={() => setShowPaymentModal(true)}>+ Tagihan</button>
            </div>

            {/* Summary */}
            <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr 1fr", gap: 10 }}>
              {[
                { label: "Menunggu", count: paymentSchedule.filter(p => p.status === "upcoming").length, color: "#60a5fa" },
                { label: "Lunas", count: paymentSchedule.filter(p => p.status === "paid").length, color: "#34d399" },
                { label: "Terlewat", count: paymentSchedule.filter(p => p.status === "missed").length, color: "#f87171" },
              ].map(s => (
                <div key={s.label} className="stat-card" style={{ textAlign: "center" }}>
                  <div style={{ fontSize: 22, fontWeight: 700, color: s.color }}>{s.count}</div>
                  <div style={{ fontSize: 11, color: "#6060a0" }}>{s.label}</div>
                </div>
              ))}
            </div>

            {paymentSchedule.map(p => (
              <div key={p.id} className="payment-row">
                <div style={{ width: 38, height: 38, borderRadius: 10, background: p.status === "paid" ? "rgba(52,211,153,0.1)" : p.status === "missed" ? "rgba(239,68,68,0.1)" : "rgba(99,102,241,0.1)", display: "flex", alignItems: "center", justifyContent: "center", fontSize: 16, flexShrink: 0 }}>
                  {p.status === "paid" ? "✅" : p.status === "missed" ? "❌" : "🔔"}
                </div>
                <div style={{ flex: 1 }}>
                  <div style={{ display: "flex", alignItems: "center", gap: 6, marginBottom: 2 }}>
                    <span style={{ fontSize: 13, fontWeight: 700 }}>{p.name}</span>
                    {p.autoDebit && <span style={{ fontSize: 10, background: "rgba(96,165,250,0.15)", color: "#60a5fa", padding: "2px 6px", borderRadius: 6, fontWeight: 700 }}>AUTO</span>}
                  </div>
                  <div style={{ fontSize: 11, color: "#6060a0" }}>Tgl {p.dueDay} · {p.account} · {p.category}</div>
                </div>
                <div style={{ textAlign: "right" }}>
                  <div className={`${privacyMode ? "blur-mode" : ""}`} style={{ fontFamily: "Space Mono", fontSize: 13, fontWeight: 700, color: p.status === "missed" ? "#f87171" : "#e0e0f0", marginBottom: 4 }}>{fmt(p.amount)}</div>
                  {p.status !== "paid" && (
                    <div style={{ display: "flex", gap: 4 }}>
                      <button className="btn btn-success" style={{ fontSize: 10, padding: "4px 8px" }} onClick={() => markPayment(p.id, "paid")}>Lunas</button>
                      {p.status !== "missed" && <button className="btn btn-danger" style={{ fontSize: 10, padding: "4px 8px" }} onClick={() => markPayment(p.id, "missed")}>Lewat</button>}
                    </div>
                  )}
                  {p.status === "missed" && (
                    <button className="btn btn-success" style={{ fontSize: 10, padding: "4px 8px" }} onClick={() => markPayment(p.id, "paid")}>Bayar Sekarang</button>
                  )}
                </div>
              </div>
            ))}
          </div>
        )}

        {/* ═══ SHARED WALLET ═══ */}
        {tab === "shared" && (
          <div style={{ display: "flex", flexDirection: "column", gap: 16 }}>
            <div>
              <div style={{ fontSize: 18, fontWeight: 700 }}>Shared Wallet</div>
              <div style={{ fontSize: 12, color: "#6060a0" }}>Kelola keuangan bersama pasangan/keluarga</div>
            </div>
            <div className="card">
              <div style={{ fontSize: 12, color: "#6060a0", marginBottom: 12 }}>ANGGARAN RUMAH TANGGA - MEI 2026</div>
              <div style={{ fontSize: 13, fontWeight: 600, marginBottom: 8 }}>Total Pengeluaran Keluarga</div>
              <div className={`hero-balance ${privacyMode ? "blur-mode" : ""}`} style={{ fontSize: 28, marginBottom: 16 }}>{fmt(mockSharedWallet.reduce((s, m) => s + m.spent, 0))}</div>
              <div className="shared-bar" style={{ marginBottom: 12 }}>
                {mockSharedWallet.map((m, i) => {
                  const total = mockSharedWallet.reduce((s, x) => s + x.spent, 0);
                  return <div key={m.member} style={{ width: `${m.spent / total * 100}%`, background: m.color, height: "100%" }} />;
                })}
              </div>
              {mockSharedWallet.map(m => (
                <div key={m.member} style={{ display: "flex", alignItems: "center", gap: 12, marginBottom: 12 }}>
                  <div style={{ width: 36, height: 36, borderRadius: "50%", background: m.color, display: "flex", alignItems: "center", justifyContent: "center", fontWeight: 700, fontSize: 14 }}>{m.avatar}</div>
                  <div style={{ flex: 1 }}>
                    <div style={{ fontSize: 13, fontWeight: 600 }}>{m.member}</div>
                    <div className="progress-bar" style={{ marginTop: 4, height: 4 }}>
                      <div className="progress-fill" style={{ width: `${m.spent / mockSharedWallet.reduce((s, x) => s + x.spent, 0) * 100}%`, background: m.color }} />
                    </div>
                  </div>
                  <div className={`${privacyMode ? "blur-mode" : ""}`} style={{ fontFamily: "Space Mono", fontWeight: 700, color: m.color, fontSize: 13 }}>{fmt(m.spent)}</div>
                </div>
              ))}
            </div>
            <div className="card">
              <div style={{ fontSize: 13, fontWeight: 700, marginBottom: 12 }}>Anggaran Bersama per Kategori</div>
              {[
                { cat: "Groceries", limit: 2000000, spent: 1450000 },
                { cat: "Utilities", limit: 1500000, spent: 900000 },
                { cat: "Education", limit: 800000, spent: 350000 },
              ].map(b => (
                <div key={b.cat} style={{ marginBottom: 12 }}>
                  <div style={{ display: "flex", justifyContent: "space-between", fontSize: 12, marginBottom: 4 }}>
                    <span>{b.cat}</span>
                    <span className={`${privacyMode ? "blur-mode" : ""}`} style={{ color: "#a0a0c0" }}>{fmt(b.spent)} / {fmt(b.limit)}</span>
                  </div>
                  <div className="progress-bar">
                    <div className="progress-fill" style={{ width: `${b.spent / b.limit * 100}%`, background: "#6366f1" }} />
                  </div>
                </div>
              ))}
            </div>
          </div>
        )}

        {/* ═══ AI INSIGHTS ═══ */}
        {tab === "insights" && (
          <div style={{ display: "flex", flexDirection: "column", gap: 16 }}>
            <div>
              <div style={{ fontSize: 18, fontWeight: 700 }}>AI Financial Insights</div>
              <div style={{ fontSize: 12, color: "#6060a0" }}>Analisis pola & prediksi cash flow berbasis AI</div>
            </div>
            <div className="ai-card">
              <div style={{ display: "flex", alignItems: "center", gap: 12, marginBottom: 16 }}>
                <div style={{ width: 42, height: 42, background: "linear-gradient(135deg,#6366f1,#8b5cf6)", borderRadius: 12, display: "flex", alignItems: "center", justifyContent: "center", fontSize: 20 }}>🤖</div>
                <div>
                  <div style={{ fontWeight: 700, fontSize: 15 }}>AI Financial Advisor</div>
                  <div style={{ fontSize: 12, color: "#8080c0" }}>Didukung Claude AI</div>
                </div>
              </div>
              {aiInsight ? (
                <div style={{ fontSize: 13, lineHeight: 1.7, color: "#c8c8e8", whiteSpace: "pre-line" }}>{aiInsight}</div>
              ) : (
                <div style={{ fontSize: 13, color: "#8080c0", lineHeight: 1.6 }}>
                  Klik tombol di bawah untuk mendapatkan analisis mendalam tentang pola pengeluaran, saran penghematan, dan prediksi cash flow bulan depan berbasis data transaksi Anda.
                </div>
              )}
              <button className="btn btn-primary" style={{ marginTop: 16, width: "100%", padding: "12px" }} onClick={fetchAIInsight} disabled={aiLoading}>
                {aiLoading ? "⏳ Menganalisis data..." : "✨ Generate AI Insight"}
              </button>
            </div>

            {/* Cash Flow Chart (Simple) */}
            <div className="card">
              <div style={{ fontSize: 13, fontWeight: 700, marginBottom: 12, color: "#a0a0c0" }}>TREN PENGELUARAN 6 BULAN</div>
              <div style={{ display: "flex", alignItems: "flex-end", gap: 8, height: 80 }}>
                {[1.2, 1.8, 1.4, 2.1, 1.6, 1.95].map((v, i) => (
                  <div key={i} style={{ flex: 1, display: "flex", flexDirection: "column", alignItems: "center", gap: 4 }}>
                    <div style={{ width: "100%", background: i === 5 ? "#6366f1" : "#2a2a3a", borderRadius: 4, height: `${v / 2.1 * 70}px`, transition: "height 0.5s" }} />
                    <div style={{ fontSize: 9, color: "#6060a0" }}>{["Des", "Jan", "Feb", "Mar", "Apr", "Mei"][i]}</div>
                  </div>
                ))}
              </div>
            </div>
          </div>
        )}

        {/* ═══ REPORTS ═══ */}
        {tab === "reports" && (
          <div style={{ display: "flex", flexDirection: "column", gap: 16 }}>
            <div>
              <div style={{ fontSize: 18, fontWeight: 700 }}>Laporan Keuangan</div>
              <div style={{ fontSize: 12, color: "#6060a0" }}>Ekspor laporan profesional multi-format</div>
            </div>
            <div style={{ display: "flex", gap: 6, flexWrap: "wrap" }}>
              {["week", "month", "3month", "year"].map(r => (
                <button key={r} className={`btn ${filterRange === r ? "btn-primary" : "btn-ghost"}`} style={{ fontSize: 11, padding: "6px 12px" }} onClick={() => setFilterRange(r)}>
                  {{ week: "7 Hari", month: "Bulan Ini", "3month": "3 Bulan", year: "Tahun Ini" }[r]}
                </button>
              ))}
            </div>

            {/* Summary */}
            <div className="card">
              <div style={{ fontSize: 13, fontWeight: 700, color: "#a0a0c0", marginBottom: 12 }}>RINGKASAN LAPORAN - MEI 2026</div>
              {[
                { label: "Total Pemasukan", value: totalIncome, color: "#34d399" },
                { label: "Total Pengeluaran", value: totalExpense, color: "#f87171" },
                { label: "Net Cashflow", value: totalIncome - totalExpense, color: "#60a5fa" },
                { label: "Saldo Akhir", value: totalBalance, color: "#a78bfa" },
              ].map(s => (
                <div key={s.label} style={{ display: "flex", justifyContent: "space-between", padding: "8px 0", borderBottom: "1px solid #1e1e2a" }}>
                  <span style={{ fontSize: 13, color: "#a0a0c0" }}>{s.label}</span>
                  <span className={`${privacyMode ? "blur-mode" : ""}`} style={{ fontFamily: "Space Mono", fontSize: 13, fontWeight: 700, color: s.color }}>{fmt(s.value)}</span>
                </div>
              ))}
            </div>

            {/* Export Buttons */}
            <div className="card">
              <div style={{ fontSize: 13, fontWeight: 700, color: "#a0a0c0", marginBottom: 12 }}>EKSPOR LAPORAN (AI-Generated)</div>
              <div style={{ display: "flex", flexDirection: "column", gap: 10 }}>
                {[
                  { fmt: "pdf", icon: "📄", label: "PDF Report", desc: "Ringkasan visual dengan grafik & tabel", color: "#ef4444" },
                  { fmt: "excel", icon: "📊", label: "Excel Spreadsheet", desc: "Data mentah untuk pivot table & analisis", color: "#10b981" },
                  { fmt: "word", icon: "📝", label: "Word Document", desc: "Laporan naratif profesional siap kirim", color: "#3b82f6" },
                ].map(e => (
                  <div key={e.fmt} style={{ display: "flex", alignItems: "center", gap: 12, padding: "12px", background: "#1a1a24", borderRadius: 12, border: "1px solid #2a2a38" }}>
                    <div style={{ fontSize: 24 }}>{e.icon}</div>
                    <div style={{ flex: 1 }}>
                      <div style={{ fontSize: 13, fontWeight: 700 }}>{e.label}</div>
                      <div style={{ fontSize: 11, color: "#6060a0" }}>{e.desc}</div>
                    </div>
                    <button className="btn btn-primary" style={{ fontSize: 12, padding: "8px 14px", background: `linear-gradient(135deg, ${e.color}99, ${e.color})` }} onClick={() => exportReport(e.fmt)} disabled={exportLoading === e.fmt}>
                      {exportLoading === e.fmt ? "⏳" : "⬇ Unduh"}
                    </button>
                  </div>
                ))}
              </div>
            </div>
          </div>
        )}
      </div>

      {/* ═══ MODAL: ADD TRANSACTION ═══ */}
      {showAddModal && (
        <div className="modal-bg" onClick={(e) => e.target === e.currentTarget && setShowAddModal(false)}>
          <div className="modal">
            <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 20 }}>
              <div style={{ fontSize: 16, fontWeight: 700 }}>Tambah Transaksi</div>
              <button className="btn btn-ghost" style={{ padding: "4px 10px", fontSize: 16 }} onClick={() => setShowAddModal(false)}>✕</button>
            </div>
            <div style={{ display: "flex", gap: 6, marginBottom: 14 }}>
              {["expense", "income"].map(t => (
                <button key={t} className={`btn ${newTx.type === t ? "btn-primary" : "btn-ghost"}`} style={{ flex: 1, fontSize: 13 }} onClick={() => setNewTx({ ...newTx, type: t })}>
                  {t === "expense" ? "📤 Pengeluaran" : "📥 Pemasukan"}
                </button>
              ))}
            </div>
            <div style={{ display: "flex", flexDirection: "column", gap: 10 }}>
              <input className="input" placeholder="Deskripsi (misal: Makan Siang)" value={newTx.desc} onChange={e => setNewTx({ ...newTx, desc: e.target.value })} />
              <input className="input" type="number" placeholder="Nominal (Rp)" value={newTx.amount} onChange={e => setNewTx({ ...newTx, amount: e.target.value })} />
              <select className="input" value={newTx.category} onChange={e => setNewTx({ ...newTx, category: e.target.value })}>
                {CATEGORIES.map(c => <option key={c} value={c}>{c}</option>)}
              </select>
              <select className="input" value={newTx.account} onChange={e => setNewTx({ ...newTx, account: e.target.value })}>
                {ACCOUNTS.map(a => <option key={a} value={a}>{a}</option>)}
              </select>
              <input className="input" type="date" value={newTx.date} onChange={e => setNewTx({ ...newTx, date: e.target.value })} />
              <div style={{ display: "flex", gap: 8 }}>
                <button className="btn btn-ghost" style={{ flex: 1 }} onClick={() => setShowOCRModal(true)}>📷 Scan Struk</button>
                <button className="btn btn-primary" style={{ flex: 1 }} onClick={addTransaction}>Simpan</button>
              </div>
            </div>
          </div>
        </div>
      )}

      {/* ═══ MODAL: OCR ═══ */}
      {showOCRModal && (
        <div className="modal-bg" onClick={(e) => e.target === e.currentTarget && setShowOCRModal(false)}>
          <div className="modal">
            <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 20 }}>
              <div style={{ fontSize: 16, fontWeight: 700 }}>📷 Scan Struk (OCR)</div>
              <button className="btn btn-ghost" style={{ padding: "4px 10px" }} onClick={() => setShowOCRModal(false)}>✕</button>
            </div>
            <div style={{ background: "#1a1a24", border: "2px dashed #3a3a52", borderRadius: 12, padding: 30, textAlign: "center", marginBottom: 14, cursor: "pointer" }} onClick={() => fileRef.current.click()}>
              <div style={{ fontSize: 36, marginBottom: 8 }}>📤</div>
              <div style={{ fontSize: 14, fontWeight: 600, color: "#a0a0c0" }}>Upload foto struk belanja</div>
              <div style={{ fontSize: 12, color: "#6060a0", marginTop: 4 }}>JPG, PNG, atau HEIC · Max 5MB</div>
            </div>
            <input ref={fileRef} type="file" accept="image/*" style={{ display: "none" }} onChange={e => handleOCR(e.target.files[0])} />
            {ocrLoading && (
              <div style={{ textAlign: "center", padding: 20 }}>
                <div style={{ fontSize: 24, marginBottom: 8 }}>⏳</div>
                <div style={{ fontSize: 13, color: "#a0a0c0" }}>AI sedang membaca struk...</div>
              </div>
            )}
            {ocrText && (
              <div style={{ background: "#12121a", borderRadius: 10, padding: 12, fontSize: 12, fontFamily: "Space Mono", color: "#a0d0a0", overflow: "auto", maxHeight: 200 }}>
                {ocrText}
              </div>
            )}
            <button className="btn btn-ghost" style={{ width: "100%", marginTop: 12 }} onClick={() => fileRef.current.click()}>Pilih Gambar</button>
          </div>
        </div>
      )}

      {/* ═══ MODAL: ADD PAYMENT ═══ */}
      {showPaymentModal && (
        <div className="modal-bg" onClick={(e) => e.target === e.currentTarget && setShowPaymentModal(false)}>
          <div className="modal">
            <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 20 }}>
              <div style={{ fontSize: 16, fontWeight: 700 }}>📅 Jadwalkan Tagihan</div>
              <button className="btn btn-ghost" style={{ padding: "4px 10px" }} onClick={() => setShowPaymentModal(false)}>✕</button>
            </div>
            <div style={{ display: "flex", flexDirection: "column", gap: 10 }}>
              <input className="input" placeholder="Nama tagihan (misal: Cicilan Mobil)" value={newPayment.name} onChange={e => setNewPayment({ ...newPayment, name: e.target.value })} />
              <input className="input" type="number" placeholder="Nominal (Rp)" value={newPayment.amount} onChange={e => setNewPayment({ ...newPayment, amount: e.target.value })} />
              <div style={{ display: "flex", gap: 8 }}>
                <input className="input" type="number" min={1} max={31} placeholder="Tanggal jatuh tempo (1-31)" value={newPayment.dueDay} onChange={e => setNewPayment({ ...newPayment, dueDay: Number(e.target.value) })} style={{ flex: 2 }} />
              </div>
              <select className="input" value={newPayment.account} onChange={e => setNewPayment({ ...newPayment, account: e.target.value })}>
                {ACCOUNTS.map(a => <option key={a} value={a}>{a}</option>)}
              </select>
              <select className="input" value={newPayment.category} onChange={e => setNewPayment({ ...newPayment, category: e.target.value })}>
                {CATEGORIES.map(c => <option key={c} value={c}>{c}</option>)}
              </select>
              <label style={{ display: "flex", alignItems: "center", gap: 10, fontSize: 13, cursor: "pointer" }}>
                <input type="checkbox" checked={newPayment.autoDebit} onChange={e => setNewPayment({ ...newPayment, autoDebit: e.target.checked })} />
                <span>Auto Debit (tandai sebagai otomatis)</span>
              </label>
              <button className="btn btn-primary" style={{ padding: 12 }} onClick={addPayment}>Jadwalkan Tagihan</button>
            </div>
          </div>
        </div>
      )}
    </div>
  );
}
