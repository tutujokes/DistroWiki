# 🔧 Solução: Erro 404 no Vercel

## Problema

```
Error: 404: NOT_FOUND
```

Ao acessar `https://distrowiki.vercel.app/`, o Vercel retornava 404.

## Causa Raiz

O `vercel.json` estava configurado incorretamente:
- Não estava especificando `@vercel/static-build` para compilar o frontend
- O Vercel não sabia que precisa rodar `npm run build` antes de fazer deploy

## Solução Implementada

### 1. Atualizar `vercel.json`

```json
{
  "version": 2,
  "outputDirectory": "dist",
  "builds": [
    {
      "src": "package.json",
      "use": "@vercel/static-build",
      "config": {
        "distDir": "dist"
      }
    },
    {
      "src": "api/main.py",
      "use": "@vercel/python"
    }
  ],
  "routes": [
    {
      "src": "/api/(.*)",
      "dest": "api/main.py"
    },
    {
      "src": "/assets/(.*)",
      "dest": "dist/assets/$1"
    },
    {
      "src": "/(.*)",
      "dest": "dist/index.html"
    }
  ]
}
```

**Mudanças:**
- ✅ Adicionado `@vercel/static-build` para compilar Vite
- ✅ Definido `outputDirectory: "dist"`
- ✅ Adicionadas rotas específicas para `/assets/` (cache buster)
- ✅ Mantida rota SPA com fallback para `index.html`

### 2. Criar `build.sh`

Script auxiliar para garantir que o build funciona em qualquer ambiente:

```bash
#!/bin/bash
npm install
npm run build
```

### 3. Criar `index.js`

Arquivo vazio para marcar o projeto como Node.js:

```javascript
module.exports = {};
```

## Flow de Build Agora

```
1. Vercel detecta package.json
2. Roda: npm install
3. Roda: npm run build (conforme package.json)
4. Gera: dist/
5. Rota / → dist/index.html
6. Rota /api/* → api/main.py
7. Deploy ✅
```

## Como Fazer o Redeploy

### Opção 1: Push para GitHub (Recomendado)

```bash
git add .
git commit -m "fix: update vercel.json for proper static build"
git push origin main
```

Vercel fará redeploy automaticamente!

### Opção 2: Redeploy Manual no Dashboard

1. Acesse [vercel.com/dashboard](https://vercel.com/dashboard)
2. Selecione projeto `distrowiki`
3. Clique em **"Deployments"**
4. Clique em **"..."** do deploy mais recente
5. Selecione **"Redeploy"**

### Opção 3: CLI do Vercel

```bash
vercel --prod
```

## Verificar o Fix

Após o redeploy, teste:

```bash
# Frontend
curl https://distrowiki.vercel.app/

# API
curl https://distrowiki.vercel.app/api/distros

# SPA Routing
curl https://distrowiki.vercel.app/distro/123
```

## Checklist de Verificação

- [ ] Build `dist/` está sendo gerado no Vercel
- [ ] Página inicial carrega em `/`
- [ ] Página de distros carrega em `/distros`
- [ ] Página de detalhe carrega em `/distro/123`
- [ ] API funciona em `/api/distros`
- [ ] Sem erros de CORS no console
- [ ] Sem erros 404 desnecessários

## Estrutura Final no Vercel

```
Vercel Deployment
├── Static Files (dist/)
│   ├── index.html
│   ├── assets/
│   │   ├── main.*.js
│   │   └── main.*.css
│   └── robots.txt
│
└── Function (api/main.py)
    └── Handles /api/* routes
```

## Próximas Observações

Se ainda receber 404:
1. Verifique Vercel → Deployments → Logs
2. Procure por erros no build
3. Confira se `npm run build` está em `package.json`
4. Verifique se `vite.config.ts` tem `outDir: 'dist'`

---

**Status**: ✅ Corrigido  
**Última atualização**: Novembro 2025
