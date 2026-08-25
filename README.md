# Debate Diario

## 1. Subir a GitHub

```bash
cd debate-diario
git init
git add .
git commit -m "Primera versión"
```

Crea un repositorio nuevo en GitHub (vacío, sin README) y luego:

```bash
git remote add origin https://github.com/TU-USUARIO/debate-diario.git
git branch -M main
git push -u origin main
```

## 2. Desplegar en Vercel

1. Entra en vercel.com → "Add New Project".
2. Importa el repositorio `debate-diario` que acabas de subir.
3. Vercel detectará que es un proyecto Vite automáticamente (gracias al `vercel.json` ya incluido). No hace falta tocar nada en "Build & Output Settings".
4. **Antes de darle a Deploy**, ve a "Environment Variables" y añade:
   - Nombre: `ANTHROPIC_API_KEY`
   - Valor: tu clave de API de Anthropic (la consigues en console.anthropic.com → API Keys)
5. Deploy.

Si más adelante cambias algo y haces `git push`, Vercel vuelve a publicar solo.

## Por qué esto no debería dar 404

El error 404 en Vercel casi siempre pasa por una de estas razones, y todas están cubiertas aquí:
- Faltaba `index.html` en la raíz → ya está.
- Vercel no sabía qué carpeta servir tras el build → `vercel.json` le dice explícitamente `dist`.
- Se subía la carpeta equivocada a GitHub (por ejemplo, una carpeta padre en vez de esta) → asegúrate de que `package.json` queda en la raíz del repositorio.

## Nota sobre los datos

De momento los temas, votos y tu racha se guardan en el propio navegador (`localStorage`), no en una base de datos compartida. Eso significa que si entras desde el móvil no verás lo mismo que desde el ordenador, y otras personas no verán tus votos. Ese es justamente el siguiente paso: conectar Supabase para que los datos sean compartidos y reales.
