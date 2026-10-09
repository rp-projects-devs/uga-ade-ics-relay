# uga-ade-ics-relay

Abonne **ton calendrier** (Google Agenda, Apple Calendrier, Thunderbird…) à ton emploi du temps **ADE** de l'Université Grenoble Alpes, même quand le lien d'export n'est accessible que via le VPN de l'établissement.

Toutes les 3 h, une GitHub Action se connecte au VPN, télécharge ton export `.ics`, puis le republie sur un **Gist secret**. Ton agenda s'abonne à ce Gist et reçoit les mises à jour automatiquement.

```
ADE (derrière le VPN) ──► GitHub Actions (OpenVPN + curl) ──► Gist secret ──► ton agenda
```

## Points clés

- **Gratuit**, sans serveur à maintenir.
- **Sécurisé** : config VPN, identifiants et lien ADE sont stockés dans les Secrets GitHub, jamais dans le code ni dans les logs.
- **Fiable** : si un téléchargement échoue, le calendrier précédent est conservé.
- **Autonome** : le workflow se maintient actif tout seul, sans désactivation après 60 jours d'inactivité.

> ⚠️ Ton compte VPN est personnel : vérifie que cet usage respecte la charte informatique de ton établissement. Ce projet ne fonctionne pas si la connexion VPN exige une double authentification.

> 🗓️ Ce projet repose sur ADE. Il a été conçu pour l'année **2026/2027** : si l'université change d'outil d'emploi du temps par la suite, il risque de devenir obsolète.

---

## Installation

### 1. Récupérer ton lien d'emploi du temps

1. Connecte-toi à **Zenith**.
2. Clique sur **« Emploi du temps export ics »**.
3. Copie le lien obtenu.

Ce lien n'a pas de date limite : il reste valable toute l'année et renvoie toujours la version à jour de ton emploi du temps.

### 2. Copier ce dépôt

Clique sur **Fork** en haut de la page. Dans ton fork, ouvre l'onglet **Actions** et clique sur **I understand my workflows, go ahead and enable them**.

### 3. Créer un Gist secret

1. Va sur <https://gist.github.com>.
2. Nomme le fichier `cours.ics` (exactement ce nom).
3. Mets ce contenu provisoire :
   ```
   BEGIN:VCALENDAR
   END:VCALENDAR
   ```
4. Clique sur **Create secret gist**.

L'identifiant à la fin de l'URL du Gist (`https://gist.github.com/<pseudo>/<GIST_ID>`) est ton `GIST_ID`.

### 4. Créer un jeton d'accès au Gist

Sur <https://github.com/settings/tokens>, clique sur **Generate new token** → **Generate new token (classic)**. Coche **uniquement** la case `gist`, génère le jeton, puis copie-le (il commence par `ghp_`).

### 5. Ajouter les secrets

Dans ton fork, va dans **Settings → Secrets and variables → Actions → New repository secret**, et crée ces six secrets :

| Secret | Contenu |
|---|---|
| `VPN_CONFIG` | tout le contenu du fichier `.ovpn` fourni par ton établissement |
| `VPN_USERNAME` | ton identifiant VPN |
| `VPN_PASSWORD` | ton mot de passe VPN |
| `ICS_URL` | le lien copié depuis Zenith |
| `GIST_ID` | l'identifiant du Gist |
| `GIST_TOKEN` | le jeton `ghp_...` |

Si le serveur d'emploi du temps demande une authentification, le workflow réutilise automatiquement les identifiants VPN (même compte universitaire). S'ils sont différents, ajoute deux secrets facultatifs : `ICS_USERNAME` et `ICS_PASSWORD`.

### 6. Tester

Ouvre **Actions → Sync calendrier ADE → Run workflow**. Après environ une minute, tu dois voir :

- ✅ VPN connecté
- ✅ Calendrier récupéré
- ✅ Gist mis à jour

### 7. S'abonner dans ton agenda

Ajoute un calendrier « depuis une URL » avec :

```
https://gist.githubusercontent.com/<pseudo>/<GIST_ID>/raw/cours.ics
```

N'ajoute pas de hash de commit dans l'URL : sans hash, elle pointe toujours vers la dernière version.

---

## Dépannage

| Message | Solution |
|---|---|
| `Secret manquant` | Vérifie le nom exact des secrets (en majuscules). |
| `AUTH_FAILED` | Identifiant ou mot de passe VPN incorrect. |
| `TLS handshake failed` ou délai dépassé | `VPN_CONFIG` est incomplet (il faut tout le fichier, bloc `<ca>` compris), ou le serveur VPN bloque les IP de GitHub. |
| `nom d'hôte introuvable` / `délai dépassé` | Le lien n'est pas joignable via le VPN : vérifie `ICS_URL`. |
| `Identifiants refusés par le serveur d'emploi du temps` | Ajoute les secrets `ICS_USERNAME` / `ICS_PASSWORD` avec les bons identifiants. |
| `HTTP 4xx/5xx` | Le lien est invalide ou ADE est indisponible : recopie-le depuis Zenith. |
| `n'est pas un calendrier ICS complet` | Le lien renvoie une page d'erreur ou de connexion : recopie-le depuis Zenith. |
| `GIST_TOKEN invalide ou expiré` | Régénère le jeton et mets à jour le secret. |
| `GIST_ID introuvable` | Vérifie l'identifiant, et que le Gist appartient bien au même compte que le jeton. |
| L'agenda ne se met pas à jour | Les agendas rafraîchissent les abonnements à leur propre rythme (souvent plusieurs heures) : patiente. |

## Confidentialité

- Les logs Actions d'un dépôt public sont visibles par tous. Le workflow n'y écrit aucune donnée personnelle : les secrets sont masqués, les adresses IP filtrées, et le contenu du calendrier n'apparaît jamais.
- Le Gist secret n'est ni listé ni indexé, mais toute personne qui possède son URL peut lire l'emploi du temps.
- Les forks ne reçoivent jamais tes secrets.
- Pour tout arrêter : **Actions → Sync calendrier ADE → ⋯ → Disable workflow**, puis révoque le jeton.

## Licence

[MIT](LICENSE)
