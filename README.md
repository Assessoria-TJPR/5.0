<div align="center">
<img width="1200" height="475" alt="GHBanner" src="https://github.com/user-attachments/assets/0aa67016-6eaf-458a-adb2-6e31a0763ed6" />
</div>

# Run and deploy your AI Studio app

This contains everything you need to run your app locally.

View your app in AI Studio: https://ai.studio/apps/drive/1RjGL6xAfQtkkJD-zhr57V1MYrOxLqjBC

## Run Locally

**Prerequisites:**  Node.js


1. Install dependencies:
   `npm install`
2. Set the `GEMINI_API_KEY` in [.env.local](.env.local) to your Gemini API key
3. Run the app:
   `npm run dev`

## Autenticação (Firebase)

1. Crie um projeto no Firebase e ative **Auth → Email/Password**.
2. Crie um banco no **Firestore**.
3. Em **Auth → Settings → Authorized domains**, adicione seu domínio do GitHub Pages.
4. Crie um `.env.local` com as variáveis abaixo:

```bash
VITE_FIREBASE_API_KEY=...
VITE_FIREBASE_AUTH_DOMAIN=...
VITE_FIREBASE_PROJECT_ID=...
VITE_FIREBASE_STORAGE_BUCKET=...
VITE_FIREBASE_MESSAGING_SENDER_ID=...
VITE_FIREBASE_APP_ID=...
VITE_ADMIN_EMAILS=admin@empresa.com
VITE_ALLOWED_EMAIL_DOMAIN=tjpr.jus.br
VITE_AUTH_ACTION_URL=https://seu-dominio.com
VITE_AUTH_LINK_DOMAIN=auth.seu-dominio.com
VITE_AUTH_DYNAMIC_LINK_DOMAIN=seu-dominio.page.link
```

Notas:
- `VITE_ADMIN_EMAILS` aceita lista separada por vírgula.
- O admin envia **reset de senha por e-mail** para usuários.
- Cadastro é aberto; admins podem promover/rebaixar perfis no painel.
- Cada triagem registrada incrementa `triageCount` no perfil do usuário.
- Cadastro permitido apenas para contas `@tjpr.jus.br` (pode ajustar via `VITE_ALLOWED_EMAIL_DOMAIN`).
- No cadastro, o Firebase envia um aviso por e-mail (use o template de verificação do Auth).
- Login só é liberado após confirmação do e-mail.
- Perfil permite atualizar nome, foto (URL) e preferência de tema.
- `VITE_AUTH_ACTION_URL` deve apontar para um domínio autorizado no Firebase Auth.
- Para melhorar a entrega dos e-mails, configure um domínio personalizado de link de ação no Firebase Auth e use `VITE_AUTH_LINK_DOMAIN`.
- `VITE_AUTH_DYNAMIC_LINK_DOMAIN` é opcional (legado para mobile/dynamic links).

Regras sugeridas (Firestore):

```txt
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    function isSignedIn() {
      return request.auth != null;
    }
    function currentUser() {
      return get(/databases/$(database)/documents/users/$(request.auth.uid));
    }
    function isActiveAdmin() {
      return isSignedIn() &&
        currentUser().data.role == 'admin' &&
        currentUser().data.active == true;
    }
    function isOwner(userId) {
      return isSignedIn() && request.auth.uid == userId;
    }
    function userKeysValid() {
      return request.resource.data.keys().hasOnly([
        'uid',
        'email',
        'name',
        'photoURL',
        'role',
        'active',
        'triageCount',
        'theme',
        'createdAt',
        'updatedAt',
        'deletedAt'
      ]);
    }
    function selfCreateAllowed(userId) {
      return isOwner(userId) &&
        request.resource.data.uid == userId &&
        request.resource.data.email == request.auth.token.email &&
        request.resource.data.role == 'user' &&
        request.resource.data.active == true &&
        userKeysValid();
    }
    function adminCreateAllowed(userId) {
      return isActiveAdmin() &&
        request.resource.data.uid == userId &&
        userKeysValid();
    }
    function selfUpdateAllowed(userId) {
      return isOwner(userId) &&
        request.resource.data.uid == userId &&
        request.resource.data.email == request.auth.token.email &&
        request.resource.data.role == resource.data.role &&
        request.resource.data.active == resource.data.active &&
        request.resource.data.createdAt == resource.data.createdAt &&
        request.resource.data.deletedAt == resource.data.deletedAt &&
        userKeysValid() &&
        request.resource.data.diff(resource.data).changedKeys().hasOnly([
          'name',
          'photoURL',
          'theme',
          'updatedAt',
          'triageCount',
          'email'
        ]) &&
        (
          !request.resource.data.diff(resource.data).changedKeys().hasAny(['triageCount']) ||
          request.resource.data.triageCount == resource.data.triageCount + 1
        );
    }
    match /users/{userId} {
      allow get: if isOwner(userId) || isActiveAdmin();
      allow list: if isActiveAdmin();
      allow create: if adminCreateAllowed(userId) || selfCreateAllowed(userId);
      allow update: if isActiveAdmin() || selfUpdateAllowed(userId);
      allow delete: if isOwner(userId) || isActiveAdmin();
    }
    function adminRequestKeysValid() {
      return request.resource.data.keys().hasOnly([
        'uid',
        'email',
        'name',
        'status',
        'createdAt',
        'updatedAt'
      ]);
    }
    function selfAdminRequestAllowed(requestId) {
      return isOwner(requestId) &&
        request.resource.data.uid == request.auth.uid &&
        request.resource.data.status == 'pending' &&
        adminRequestKeysValid();
    }
    function adminRequestUpdateAllowed(requestId) {
      return isOwner(requestId) &&
        request.resource.data.uid == request.auth.uid &&
        request.resource.data.status == 'pending' &&
        adminRequestKeysValid() &&
        request.resource.data.diff(resource.data).changedKeys().hasOnly([
          'email',
          'name',
          'status',
          'updatedAt'
        ]);
    }
    match /adminRequests/{requestId} {
      allow get: if isOwner(requestId) || isActiveAdmin();
      allow list: if isActiveAdmin();
      allow create: if selfAdminRequestAllowed(requestId);
      allow update: if isActiveAdmin() || adminRequestUpdateAllowed(requestId);
      allow delete: if isActiveAdmin();
    }
  }
}
```

## Deploy to GitHub Pages

Build vai para `docs/` (Vite `base: './'`):

1. Build: `npm run build`
2. Publicar:
   - `npm run deploy:gh` (envia `docs/` para branch `gh-pages`)
   - ou `git subtree push --prefix docs origin gh-pages`
   - ou configure Pages para Source: `main` + folder `/docs`
   - ou habilite “GitHub Actions” (workflow `deploy.yml` já envia `docs/`)

Notes:
- `npm run build` gera `docs/404.html` (SPA fallback).
- `base: './'` garante assets em subpath `/repo-name/`.

### Variáveis no GitHub Actions

Para o deploy funcionar no GitHub Pages, defina **Secrets** no repositório com as chaves abaixo
(Settings → Secrets and variables → Actions):

- `VITE_FIREBASE_API_KEY`
- `VITE_FIREBASE_AUTH_DOMAIN`
- `VITE_FIREBASE_PROJECT_ID`
- `VITE_FIREBASE_STORAGE_BUCKET`
- `VITE_FIREBASE_MESSAGING_SENDER_ID`
- `VITE_FIREBASE_APP_ID`
- `VITE_ADMIN_EMAILS`
- `VITE_ALLOWED_EMAIL_DOMAIN`

Também adicione o domínio `USERNAME.github.io` em **Firebase → Auth → Settings → Authorized domains**.
