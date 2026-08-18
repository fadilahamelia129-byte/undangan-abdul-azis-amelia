# undangan-abdul-azis-amelia
from pathlib import Path
import shutil, zipfile

src = Path("/mnt/data/undangan-nextjs")
if not src.exists():
    raise FileNotFoundError("Starter project tidak ditemukan.")

# Upgrade the project with a more complete wedding experience.
page = src / "components/WeddingPage.tsx"
page.write_text(r''' "use client";

import { FormEvent, useEffect, useMemo, useState } from "react";
import { motion } from "framer-motion";
import { CalendarDays, Clock3, ExternalLink, Heart, MapPin, Music2, Pause, Play, Sparkles } from "lucide-react";
import { wedding } from "@/lib/events";

type Wish = { id: string; name: string; message: string; created_at: string };

const fadeUp = {
  initial: { opacity: 0, y: 24 },
  whileInView: { opacity: 1, y: 0 },
  viewport: { once: true, amount: 0.18 },
  transition: { duration: 0.72, ease: "easeOut" as const }
};

function Countdown() {
  const target = useMemo(() => new Date(wedding.dateISO).getTime(), []);
  const [left, setLeft] = useState(Math.max(0, target - Date.now()));

  useEffect(() => {
    const id = window.setInterval(() => setLeft(Math.max(0, target - Date.now())), 1000);
    return () => window.clearInterval(id);
  }, [target]);

  const units = [
    ["Hari", Math.floor(left / 86400000)],
    ["Jam", Math.floor((left % 86400000) / 3600000)],
    ["Menit", Math.floor((left % 3600000) / 60000)],
    ["Detik", Math.floor((left % 60000) / 1000)]
  ];

  return (
    <div className="grid grid-cols-4 gap-2 sm:gap-3 max-w-xl mx-auto">
      {units.map(([label, value]) => (
        <div key={String(label)} className="glass rounded-2xl p-3 sm:p-5 text-center">
          <div className="text-2xl sm:text-4xl font-semibold gold-text tabular-nums">{String(value).padStart(2, "0")}</div>
          <div className="mt-1 text-[9px] sm:text-[10px] uppercase tracking-[.22em] text-stone-500">{label}</div>
        </div>
      ))}
    </div>
  );
}

function EventCard({ title, time, venue, map }: { title: string; time: string; venue: string; map: string }) {
  return (
    <div className="rounded-3xl bg-white/75 p-6 sm:p-8 shadow-[0_20px_70px_rgba(73,61,49,.08)] gold-border">
      <p className="text-xs uppercase tracking-[.28em] gold-text">{title}</p>
      <h3 className="mt-3 text-2xl font-display">{venue}</h3>
      <div className="mt-6 space-y-3 text-sm text-stone-600">
        <p className="flex gap-3"><CalendarDays size={18} /> {wedding.dateLabel}</p>
        <p className="flex gap-3"><Clock3 size={18} /> {time}</p>
        <p className="flex gap-3"><MapPin size={18} /> {venue}</p>
      </div>
      <a href={map} target="_blank" rel="noreferrer" className="mt-7 inline-flex items-center gap-2 rounded-full bg-stone-800 px-5 py-3 text-sm text-white transition hover:-translate-y-0.5 hover:bg-stone-700">
        Buka Maps <ExternalLink size={15} />
      </a>
    </div>
  );
}

export default function WeddingPage() {
  const [opened, setOpened] = useState(false);
  const [music, setMusic] = useState(false);
  const [rsvp, setRsvp] = useState({ name: "", attendance: "Hadir", guests: "1" });
  const [wish, setWish] = useState({ name: "", message: "" });
  const [wishes, setWishes] = useState<Wish[]>([]);
  const [busy, setBusy] = useState(false);
  const [polishing, setPolishing] = useState(false);
  const [notice, setNotice] = useState("");

  async function loadWishes() {
    const res = await fetch("/api/wishes", { cache: "no-store" });
    if (res.ok) setWishes(await res.json());
  }

  useEffect(() => { loadWishes(); }, []);

  function openInvitation() {
    setOpened(true);
    window.setTimeout(() => document.getElementById("detail")?.scrollIntoView({ behavior: "smooth" }), 120);
  }

  async function submitRsvp(e: FormEvent) {
    e.preventDefault(); setBusy(true); setNotice("");
    const res = await fetch("/api/rsvp", {
      method: "POST", headers: { "Content-Type": "application/json" }, body: JSON.stringify(rsvp)
    });
    setBusy(false);
    setNotice(res.ok ? "Konfirmasi kehadiran berhasil dikirim. Terima kasih!" : "Maaf, RSVP belum berhasil dikirim.");
    if (res.ok) setRsvp({ name: "", attendance: "Hadir", guests: "1" });
  }

  async function polishWish() {
    if (!wish.message.trim()) return;
    setPolishing(true); setNotice("");
    const res = await fetch("/api/polish", {
      method: "POST", headers: { "Content-Type": "application/json" }, body: JSON.stringify({ message: wish.message })
    });
    const data = await res.json();
    setPolishing(false);
    if (res.ok) setWish(v => ({ ...v, message: data.message }));
    else setNotice(data.error ?? "Ucapan belum dapat diperindah.");
  }

  async function submitWish(e: FormEvent) {
    e.preventDefault(); setBusy(true); setNotice("");
    const res = await fetch("/api/wishes", {
      method: "POST", headers: { "Content-Type": "application/json" }, body: JSON.stringify(wish)
    });
    const data = await res.json();
    setBusy(false);
    if (res.ok) {
      setWish({ name: "", message: "" }); setNotice("Ucapan berhasil ditambahkan. ❤️"); await loadWishes();
    } else setNotice(data.error ?? "Ucapan belum berhasil dikirim.");
  }

  if (!opened) {
    return (
      <main className="cover-screen relative overflow-hidden">
        <div className="absolute inset-5 sm:inset-10 border border-[#b88732]/35" />
        <div className="absolute inset-8 sm:inset-16 border border-[#b88732]/15" />
        <div className="botanical botanical-left">❦</div>
        <div className="botanical botanical-right">❧</div>
        <motion.div initial={{ opacity: 0, y: 25 }} animate={{ opacity: 1, y: 0 }} transition={{ duration: 1 }} className="relative z-10 max-w-xl px-7 text-center">
          <p className="text-xs uppercase tracking-[.35em] gold-text">The Wedding Of</p>
          <p className="mt-9 text-sm text-stone-500">Kepada Yth.</p>
          <h1 className="mt-2 text-3xl sm:text-4xl font-display">Bapak/Ibu/Saudara/i</h1>
          <p className="mt-5 text-sm leading-7 text-stone-600">Dengan penuh kebahagiaan, kami mengundang Anda untuk hadir di hari istimewa kami.</p>
          <h2 className="mt-9 text-5xl sm:text-7xl font-display gold-text">Azis <span className="text-stone-400">&</span> Amelia</h2>
          <p className="mt-5 text-sm tracking-[.18em] text-stone-500">24 AGUSTUS 2026</p>
          <button onClick={openInvitation} className="mt-10 rounded-full bg-stone-800 px-8 py-3.5 text-sm text-white shadow-lg transition hover:-translate-y-0.5 hover:bg-stone-700">Buka Undangan</button>
          <p className="mt-4 text-[11px] text-stone-400">Mohon maaf apabila terdapat kesalahan penulisan nama/alamat.</p>
        </motion.div>
      </main>
    );
  }

  return (
    <main className="paper min-h-screen overflow-hidden">
      <nav className="fixed bottom-4 left-1/2 z-50 -translate-x-1/2 rounded-full glass shadow-xl px-3 py-2 flex gap-1">
        {[
          ["#detail", "Detail"], ["#acara", "Acara"], ["#rsvp", "RSVP"], ["#wishes", "Ucapan"]
        ].map(([href, label]) => <a key={href} href={href} className="rounded-full px-3 py-2 text-[11px] sm:text-xs hover:bg-white">{label}</a>)}
        <button onClick={() => setMusic(v => !v)} aria-label="Musik" className="rounded-full p-2 hover:bg-white">{music ? <Pause size={15}/> : <Music2 size={15}/>}</button>
      </nav>

      <section className="relative min-h-screen flex items-center justify-center px-5 py-20">
        <div className="absolute inset-5 sm:inset-10 border border-[#b88732]/30 pointer-events-none" />
        <div className="absolute inset-8 sm:inset-16 border border-[#b88732]/15 pointer-events-none" />
        <div className="botanical botanical-left">❦</div>
        <div className="botanical botanical-right">❧</div>
        <motion.div initial={{ opacity: 0, scale: .96 }} animate={{ opacity: 1, scale: 1 }} transition={{ duration: 1.15 }} className="relative z-10 text-center max-w-3xl">
          <p className="uppercase tracking-[.35em] text-xs gold-text">The Wedding Of</p>
          <h1 className="mt-7 text-6xl sm:text-8xl font-display font-light gold-text">Azis <span className="text-stone-400">&</span> Amelia</h1>
          <p className="mt-7 text-stone-600 text-sm sm:text-base leading-7">Dua hati, satu janji, dan sebuah perjalanan yang akan kami mulai bersama.</p>
          <div className="mt-10"><Countdown /></div>
          <a href="#detail" className="mt-10 inline-block rounded-full border border-[#b88732]/50 px-6 py-3 text-sm hover:bg-white/60 transition">Lihat Detail</a>
        </motion.div>
      </section>

      <section id="detail" className="max-w-6xl mx-auto px-5 py-20 sm:py-28">
        <motion.div {...fadeUp} className="text-center">
          <p className="text-xs uppercase tracking-[.3em] gold-text">A Promise of Forever</p>
          <h2 className="mt-4 text-4xl sm:text-5xl font-display">Abdul Azis & Ameliafadillah</h2>
          <p className="mt-5 text-sm text-stone-600 leading-7 max-w-2xl mx-auto">
            Abdul Azis (Azis), putra ke-7 Bapak Djaya Susanto & Ibu Rohati, dengan Ameliafadillah (Amelia), putri ke-4 Bapak Abas & Ibu Saridah.
          </p>
        </motion.div>
      </section>

      <section id="acara" className="px-5 pb-20 sm:pb-28">
        <div className="max-w-6xl mx-auto">
          <motion.div {...fadeUp} className="grid md:grid-cols-2 gap-6">
            <EventCard {...wedding.akad} />
            <EventCard {...wedding.reception} />
          </motion.div>
        </div>
      </section>

      <section id="rsvp" className="bg-[#eee7da]/60 px-5 py-20 sm:py-28">
        <div className="max-w-3xl mx-auto">
          <motion.div {...fadeUp} className="text-center">
            <p className="text-xs uppercase tracking-[.3em] gold-text">RSVP</p>
            <h2 className="mt-4 text-4xl font-display">Konfirmasi Kehadiran</h2>
          </motion.div>
          <form onSubmit={submitRsvp} className="mt-10 space-y-4 rounded-3xl bg-white/75 p-6 sm:p-8 gold-border">
            <input required value={rsvp.name} onChange={e => setRsvp(v => ({...v, name: e.target.value}))} placeholder="Nama lengkap" className="w-full rounded-2xl border border-stone-200 bg-white px-4 py-3 outline-none focus:border-[#b88732]" />
            <div className="grid sm:grid-cols-2 gap-4">
              <select value={rsvp.attendance} onChange={e => setRsvp(v => ({...v, attendance: e.target.value}))} className="rounded-2xl border border-stone-200 bg-white px-4 py-3"><option>Hadir</option><option>Tidak Hadir</option><option>Masih Tentatif</option></select>
              <input type="number" min="1" max="10" value={rsvp.guests} onChange={e => setRsvp(v => ({...v, guests: e.target.value}))} className="rounded-2xl border border-stone-200 bg-white px-4 py-3" placeholder="Jumlah tamu" />
            </div>
            <button disabled={busy} className="w-full rounded-2xl bg-stone-800 px-5 py-3 text-white disabled:opacity-50">{busy ? "Mengirim..." : "Kirim RSVP"}</button>
          </form>
        </div>
      </section>

      <section id="wishes" className="max-w-5xl mx-auto px-5 py-20 sm:py-28">
        <motion.div {...fadeUp} className="text-center">
          <p className="text-xs uppercase tracking-[.3em] gold-text">Love Notes</p>
          <h2 className="mt-4 text-4xl font-display">Ucapan & Doa</h2>
          <p className="mt-4 text-sm text-stone-600">Tuliskan doa terbaik. Gunakan AI untuk membuatnya lebih hangat dan puitis sebelum ditampilkan.</p>
        </motion.div>

        <form onSubmit={submitWish} className="mt-10 rounded-3xl bg-white/75 p-6 sm:p-8 gold-border">
          <input required value={wish.name} onChange={e => setWish(v => ({...v, name: e.target.value}))} placeholder="Nama Anda" className="w-full rounded-2xl border border-stone-200 bg-white px-4 py-3 outline-none focus:border-[#b88732]" />
          <textarea required maxLength={500} value={wish.message} onChange={e => setWish(v => ({...v, message: e.target.value}))} placeholder="Tulis ucapan dan doa..." rows={5} className="mt-4 w-full rounded-2xl border border-stone-200 bg-white px-4 py-3 outline-none focus:border-[#b88732]" />
          <div className="mt-4 flex flex-col sm:flex-row gap-3">
            <button type="button" onClick={polishWish} disabled={polishing || !wish.message.trim()} className="inline-flex flex-1 justify-center items-center gap-2 rounded-2xl border border-[#b88732]/50 px-5 py-3 text-sm hover:bg-[#fbf5e9] disabled:opacity-50"><Sparkles size={17} /> {polishing ? "AI sedang memperindah..." : "Perindah dengan AI"}</button>
            <button disabled={busy} className="flex-1 rounded-2xl bg-stone-800 px-5 py-3 text-white disabled:opacity-50">{busy ? "Mengirim..." : "Kirim Ucapan"}</button>
          </div>
          {notice && <p className="mt-4 text-center text-sm text-stone-600">{notice}</p>}
        </form>

        <div className="mt-10 grid gap-4">
          {wishes.length === 0 ? <p className="text-center text-sm text-stone-500">Belum ada ucapan. Jadilah yang pertama. ❤️</p> : wishes.map(item => (
            <motion.article initial={{opacity:0,y:10}} animate={{opacity:1,y:0}} key={item.id} className="rounded-3xl bg-white/70 p-6 gold-border">
              <div className="flex items-center gap-2 text-sm font-semibold"><Heart size={15} className="gold-text" /> {item.name}</div>
              <p className="mt-3 text-sm leading-7 text-stone-600">{item.message}</p>
            </motion.article>
          ))}
        </div>
      </section>

      <footer className="px-5 py-20 text-center border-t border-[#b88732]/15">
        <p className="font-display text-4xl gold-text">Azis & Amelia</p>
        <p className="mt-3 text-xs uppercase tracking-[.25em] text-stone-500">24 Agustus 2026</p>
        <p className="mt-6 text-sm text-stone-500">Terima kasih atas doa dan kehadiran Anda.</p>
      </footer>
    </main>
  );
}''', encoding="utf-8")

css = src / "app/globals.css"
css.write_text(r'''@import "tailwindcss";

:root {
  --cream: #f8f3ea;
  --paper: #fffdf8;
  --gold: #b88732;
  --ink: #493d31;
}

* { box-sizing: border-box; }
html { scroll-behavior: smooth; }
body { margin: 0; background: var(--cream); color: var(--ink); font-family: Georgia, "Times New Roman", serif; }
::selection { background: #d9c29a; color: #33291f; }
.font-display { font-family: "Times New Roman", Georgia, serif; letter-spacing: .04em; }
.gold-text { color: var(--gold); }
.paper {
  background:
    radial-gradient(circle at 12% 12%, rgba(184,135,50,.08), transparent 25%),
    radial-gradient(circle at 88% 82%, rgba(120,136,121,.09), transparent 26%),
    var(--paper);
}
.cover-screen {
  min-height: 100svh;
  display: flex;
  align-items: center;
  justify-content: center;
  background:
    radial-gradient(circle at 18% 20%, rgba(184,135,50,.12), transparent 28%),
    radial-gradient(circle at 82% 78%, rgba(120,136,121,.13), transparent 30%),
    #f8f3ea;
}
.gold-border { border: 1px solid rgba(184,135,50,.45); }
.glass { background: rgba(255,253,248,.76); backdrop-filter: blur(12px); border: 1px solid rgba(184,135,50,.18); }
.botanical {
  position: absolute;
  color: rgba(120,136,121,.35);
  font-size: clamp(110px, 18vw, 240px);
  line-height: 1;
  pointer-events: none;
  filter: blur(.2px);
}
.botanical-left { left: -30px; top: 12%; transform: rotate(-28deg); }
.botanical-right { right: -30px; bottom: 10%; transform: rotate(28deg); }
@media (prefers-reduced-motion: reduce) {
  html { scroll-behavior: auto; }
}
''', encoding="utf-8")

readme = src / "README.md"
readme.write_text(r'''# Undangan Pernikahan Abdul Azis & Amelia

Website undangan digital premium berbasis Next.js App Router, Tailwind CSS, Framer Motion, Supabase, dan OpenAI.

## Fitur
- Cover "Buka Undangan"
- Hero dengan animasi halus
- Countdown 24 Agustus 2026
- Profil kedua mempelai & orang tua
- Akad dan resepsi + tombol Maps
- Navigasi floating yang nyaman di mobile
- RSVP
- Wishlist / ucapan & doa
- Tombol AI "Perindah dengan AI"
- Responsive mobile-first
- Struktur API server-side untuk Supabase & OpenAI

## Jalankan
```bash
npm install
cp .env.example .env.local
npm run dev
```

## Database
Jalankan `supabase/schema.sql` pada Supabase SQL Editor.

## Environment
```env
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=
OPENAI_API_KEY=
OPENAI_MODEL=gpt-5.6
```

## Produksi
Tambahkan rate limiting/CAPTCHA dan moderasi ucapan sebelum membagikan URL publik.

Untuk deployment Vercel, import repository GitHub lalu masukkan environment variables di Project Settings > Environment Variables.
''', encoding="utf-8")

# Add a simple robots/sitemap-friendly metadata route.
(root := src / "app").mkdir(exist_ok=True)
(root / "robots.ts").write_text(r'''import type { MetadataRoute } from "next";

export default function robots(): MetadataRoute.Robots {
  return { rules: { userAgent: "*", allow: "/" } };
}''', encoding="utf-8")

# Rebuild ZIP
zip_path = Path("/mnt/data/undangan-abdul-azis-amelia-final.zip")
if zip_path.exists():
    zip_path.unlink()
with zipfile.ZipFile(zip_path, "w", zipfile.ZIP_DEFLATED) as z:
    for p in src.rglob("*"):
        if p.is_file():
            z.write(p, p.relative_to(src))

print(zip_path)
