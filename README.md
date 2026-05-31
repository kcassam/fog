# Fog

## ⚠️ Projet non maintenu / archivé

**Ce projet n'est plus développé ni supporté.** Il est conservé à titre d'archive
uniquement et **ne fonctionne plus en l'état** — ne l'utilisez pas tel quel en production.

---

## Ce qu'était Fog

Fog est un petit utilitaire **PHP** qui se connectait à une boîte **Gmail** en **IMAP**,
recherchait les messages correspondant à un critère donné (ex. les courriels restés sans
réponse depuis une certaine date) et les **réexpédiait** vers une adresse de destination
via **SMTP** (avec PHPMailer). Les messages traités étaient marqués comme « Answered »
(`\Answered`) afin de ne pas être renvoyés deux fois.

Cœur du projet :
- `fog.php` — la classe `Fog` (connexion IMAP, parsing MIME, décodage, réexpédition SMTP).
- `use.php` — exemple de configuration et d'utilisation.

## Pourquoi il est abandonné

- **Authentification Gmail obsolète** : l'authentification par mot de passe de compte
  (« applications moins sécurisées ») a été **désactivée par Google le 30 mai 2022**. Le
  code actuel ne peut donc plus se connecter.
- **Dépendance cassée** : le sous-module PHPMailer pointe vers `git://github.com/qoqa/phpmailer.git`.
  Le protocole `git://` a été **désactivé par GitHub en janvier 2022** et le dossier
  `externals/phpmailer/` est vide — la librairie n'est même plus récupérable.
- **Incompatibilité PHP moderne** : l'ancien PHPMailer 5.x ne tourne pas sur PHP 8.4, et
  l'**extension `imap` a été retirée du cœur de PHP** (déplacée vers PECL) depuis PHP 8.4.

## Pour réactiver / désobsolétiser le projet

Si quelqu'un souhaite reprendre le projet, voici les étapes **minimales** identifiées :

1. **Réparer la dépendance PHPMailer**
   - Supprimer le sous-module cassé (`externals/phpmailer`, `.gitmodules`).
   - Passer à **Composer** avec `phpmailer/phpmailer ^6.9`.
   - Adapter les appels obsolètes à l'API 6.x : classes namespacées
     (`PHPMailer\PHPMailer\PHPMailer`), `isSMTP()`, `msgHTML()`, `addAddress()`, `send()`
     (minuscules), `SMTPSecure = PHPMailer::ENCRYPTION_SMTPS`, gestion par exceptions.

2. **Installer l'extension IMAP**
   - Sur PHP ≥ 8.4, l'installer via PECL : `pecl install imap` (puis activer `extension=imap`).

3. **Moderniser l'authentification Gmail** (au choix)
   - **Mot de passe d'application** (le plus simple) : activer la validation en 2 étapes sur
     le compte Google, générer un *App Password* et l'utiliser à la place du mot de passe.
     Compatible avec le flux IMAP/SMTP existant, sans réécriture de logique.
   - **OAuth2 (XOAUTH2)** (le plus robuste) : enregistrer un client OAuth sur Google Cloud,
     obtenir un *refresh token*, puis authentifier en XOAUTH2 — côté IMAP via
     `imap_open(..., OP_XOAUTH2)` et côté SMTP via le support OAuth natif de PHPMailer 6
     (`league/oauth2-google`).

## Crédits

- Parsing du corps des messages inspiré de :
  http://www.electrictoolbox.com/php-imap-message-body-attachments/
