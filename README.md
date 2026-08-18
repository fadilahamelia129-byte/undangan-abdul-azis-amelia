# undangan-abdul-azis-amelia
# Undangan Pernikahan Abdul Azis & Amelia

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
