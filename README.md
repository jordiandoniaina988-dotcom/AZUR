# Azur – Mes finances (PWA)

Soldes, enveloppes, dettes, actifs et assistant IA (Google Gemini).
Fonctionne en ligne et hors ligne, installable sur téléphone et ordinateur.

## Contenu du dossier

```
index.html        l'application complète
sw.js             service worker (mode hors ligne)
manifest.json     informations d'installation PWA
icons/            icônes (192, 512, maskable, iOS, favicon)
.nojekyll         évite le traitement Jekyll sur GitHub Pages
```

## 1. Héberger sur GitHub Pages

1. Crée un dépôt GitHub (par ex. `azur`).
2. Envoie **tout le contenu de ce dossier à la racine** du dépôt (fichier `.nojekyll` compris, c'est un fichier caché).
3. Dépôt → **Settings → Pages** → *Build and deploy* : **Deploy from a branch** → branche `main`, dossier `/ (root)` → **Save**.
4. Après environ une minute, l'app est disponible sur `https://TON-PSEUDO.github.io/azur/`.

HTTPS est fourni par GitHub : c'est obligatoire pour l'installation PWA et le mode hors ligne.
Tous les chemins sont relatifs, l'app marche donc dans un sous-dossier (`/azur/`) comme sur un domaine perso.

## 2. Installer l'application

- **Android / Chrome / Edge / ordinateur** : bouton « Installer l'application » dans Réglages → Aperçu, ou icône d'installation dans la barre d'adresse.
- **iPhone (Safari)** : bouton Partager → « Sur l'écran d'accueil ».

Après la première visite en ligne, l'app s'ouvre sans connexion.

## 3. Configurer Supabase (synchronisation)

Dans Supabase → SQL Editor :

```sql
create table finance_state (
  user_id uuid primary key references auth.users on delete cascade,
  data jsonb not null,
  updated_at timestamptz default now()
);
alter table finance_state enable row level security;
create policy "mes données" on finance_state
  for all using (auth.uid() = user_id) with check (auth.uid() = user_id);
```

Puis dans l'app : ⚙️ → Réglages → renseigne **URL du projet** et **clé anon** (Supabase → Project Settings → API), enregistre, et crée ton compte.
Dans Supabase → Authentication → URL Configuration, mets l'adresse de ton site GitHub Pages comme *Site URL*.

La clé `anon` est faite pour être publique : la sécurité repose sur la règle RLS ci-dessus.

## 4. Clé Gemini

Crée une clé sur Google AI Studio et colle-la dans Réglages. Elle reste dans ton navigateur, sur ton appareil.
Dans Google AI Studio / Cloud, **limite la clé à ton domaine** (restriction « HTTP referrers »).

## 5. Mettre à jour l'app

1. Modifie les fichiers et envoie-les sur GitHub.
2. **Change `VERSION` en haut de `sw.js`** (par ex. `azur-v4`). Sans cela, les appareils gardent l'ancienne copie en cache.
3. Au prochain lancement, un message propose de recharger.

## Raccourcis d'écran d'accueil

Un appui long sur l'icône (Android / Chrome) propose : Ajouter un revenu, Nouvelle dette, Nouvel actif.

## Données

Tout est enregistré d'abord dans le navigateur (`localStorage`), puis synchronisé avec Supabase quand tu es connecté et en ligne (la version la plus récente gagne).
Vider les données du site efface la copie locale : connecte-toi à Supabase pour la récupérer.
