# KasazNeta – kod źródłowy do przeniesienia

To pełny kod strony **KasazNeta** (React + TypeScript + Vite + Tailwind CSS 4,
framework TanStack Start). Zdjęcia, logo banków i ikona aplikacji są w środku
jako zwykłe pliki, więc po wypakowaniu nic nie "wisia" z Lovable.

## 1. Uruchomienie na własnym komputerze

```bash
npm install
cp .env.example .env     # uzupełnij adresy własnego backendu
npm run dev              # strona pod http://localhost:8080
npm run build            # wersja produkcyjna do wgrania na serwer
```

Wymagany Node.js w wersji 20 lub nowszej. Zamiast `npm` można użyć `bun`.

## 2. To jest najważniejsze: backend (logowanie i dane)

Strona sama w sobie nie przechowuje kont ani statusów ofert – robi to osobna
baza danych (Supabase). Po przeniesieniu musisz założyć własną, inaczej
logowanie i "Moje osiągnięcia" nie zadziałają.

1. Załóż darmowe konto na supabase.com i nową bazę (albo użyj własnego
   Postgresa + Supabase, jeśli już masz).
2. W edytorze SQL (SQL Editor) wklej i uruchom zawartość pliku
   `backend-schema.sql` z tego folderu. Tworzy to:
   - tabelę `user_offers` (oferty i ich statusy: `rozpoczety`, `w_trakcie`,
     `wyplacony`),
   - tabelę `user_roles` (kto jest administratorem) i funkcję `has_role`,
   - reguły dostępu (RLS): zwykły użytkownik widzi tylko swoje oferty i może
     tylko dodać status `rozpoczety`, a statusy zmienia wyłącznie administrator,
   - automat: nadaje uprawnienia administratora adresowi `kamisplich@gmail.com`.
3. Jeśli administratorem ma być inny adres – w pliku `backend-schema.sql`
   zamień `kamisplich@gmail.com` na swój (występuje w 3 miejscach) i uruchom
   ponownie te fragmenty.
4. W ustawieniach bazy (Authentication → Providers) włącz:
   - **Email** (rejestracja i logowanie mailem),
   - **Google** (podaj klient OAuth z Google Cloud Console).
5. Adres bazy i klucz **publishable** wklej do pliku `.env`
   (patrz `.env.example`). Klucz publishable jest publiczny – może być w kodzie.
   Nigdzie nie umieszczaj klucza `secret` / `service_role`.

## 3. Gdzie to można wgrać

To zwykła aplikacja React, więc potrzebujesz hostingu, który przyjmuje własny
kod i serwuje Node/Edge:

- **Cloudflare Pages / Workers** – build tworzy już gotowe pliki dla Cloudflare
  (`dist/`, `dist/nitro.json`, `wrangler.json`), więc to najprostsza droga:
  `npx wrangler deploy` lub wgranie przez panel Cloudflare.
- **Vercel, Netlify** – działają, ale trzeba ustawić podgląd dla SSR
  (build: `npm run build`, katalog: `dist/client`, funkcja SSR z `dist/server`).
- **Własny serwer** – `npm run build`, potem `npx vite preview` albo Node/Edge
  runtime z `dist/`.
- **Nie zadziała** w builderach bez własnego kodu (Wix, WordPress, Shopify,
  Hostinger kreator, Lovable-替代 etc.) – one nie przyjmują aplikacji React.

## 4. Co jest w środku

| Folder / plik | Co zawiera |
| --- | --- |
| `src/routes/` | strony: główna (`index.tsx`), logowanie (`auth.tsx`), "Moje osiągnięcia" (`_authenticated/osiagniecia.tsx`) |
| `src/components/` | nagłówek i stopka strony |
| `src/lib/offers.ts` | **lista ofert, kwoty, warunki i linki afiliacyjne** – to tu zmieniasz oferty |
| `src/assets/` | zdjęcie z strony głównej i logo banków (prawdziwe pliki PNG) |
| `public/` | ikona aplikacji, manifest (instalacja na telefonie), `robots.txt` |
| `backend-schema.sql` | cała struktura bazy w jednym pliku |
| `drizzle/migrations/` | te same zapisy bazy, ale jako migawki (dla narzędzia drizzle-kit) |
| `supabase/config.toml` | ustawienia lokalnej instancji Supabase |

## 5. Czego ten eksport NIE zawiera

- **Danych użytkowników** (konta, e-maile, statusy ofert) – są w bazie
  Lovable Cloud. Jeśli chcesz je przenieść, poproś o wyeksportowanie tabel
  `user_offers` i `user_roles` do pliku SQL, a wgrasz do nowej bazy.
- **Hasła administratora** – w nowej bazie każdy użytkownik ustawia hasło sam
  (rejestracja), ewentualnie przez "Nie pamiętam hasła".
- **Darmowej domeny** `kasazneta.lovable.app` – po przeniesieniu potrzebujesz
  własnej domeny (albo subdomeny nowego hostingu).
- **Sklepu Play** – strona jest instalowalna na telefonie (manifest + ikony),
  ale pliku dla Androida jeszcze nie ma.

## 6. Szybka zmiana treści

- Oferty i linki: `src/lib/offers.ts`
- Teksty na stronie głównej i opinie: `src/routes/index.tsx`
- Kolorystyka: `src/styles.css` (sekcja `:root`, wartości `oklch`)
- Link do kontaktu (Facebook): `src/components/SiteHeader.tsx`
