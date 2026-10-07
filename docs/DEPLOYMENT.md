# Deployment Guide

**MATH** is a static single-page application. Because it has zero build dependencies and no backend requirements, it can be deployed to any static web hosting platform in under two minutes.

---

## 1. GitHub Pages (Recommended)

GitHub Pages hosts the site directly from your repository branch:

1. Push your repository to GitHub:
   ```bash
   git push origin main
   ```
2. Navigate to your repository on GitHub:
   - Go to **Settings** → **Pages** (in the left-hand navigation sidebar).
3. Under **Build and deployment**:
   - **Source**: Select `Deploy from a branch`.
   - **Branch**: Select `main` and folder `/(root)`.
   - Click **Save**.
4. Your application will be live at:
   ```
   https://<your-username>.github.io/Math/
   ```

---

## 2. Vercel

### Option A: Via GitHub Integration
1. Go to [vercel.com](https://vercel.com) and log in.
2. Click **Add New Project** → **Import Git Repository**.
3. Select `Math`.
4. Keep the default settings (Framework Preset: *Other*, Root Directory: `./`).
5. Click **Deploy**.

### Option B: Via Vercel CLI
```bash
npm install -g vercel
vercel deploy --prod
```

---

## 3. Netlify

### Option A: Drag-and-Drop (Netlify Drop)
1. Open [app.netlify.com/drop](https://app.netlify.com/drop).
2. Drag and drop the repository folder directly into your browser window.
3. Your site deploys immediately with a shareable URL.

### Option B: Git Continuous Deployment
1. Log in to [Netlify](https://www.netlify.com/).
2. Select **Add new site** → **Import an existing project**.
3. Select GitHub and choose your `Math` repository.
4. Leave build command blank and publish directory as `.`.
5. Click **Deploy site**.

---

## 4. Cloudflare Pages

1. Log in to the [Cloudflare Dashboard](https://dash.cloudflare.com/) and navigate to **Workers & Pages**.
2. Click **Create Application** → **Pages** → **Connect to Git**.
3. Select the `Math` repository.
4. Set **Build command** to empty and **Build output directory** to `.`.
5. Click **Save and Deploy**.

---

## 5. Local Development & Self-Hosting

You can serve the application locally using any standard HTTP server:

### Using Node.js / npx
```bash
# Using npx serve (configured in package.json)
npm start

# Or directly
npx serve -l 3000 .
```

### Using Python 3
```bash
python -m http.server 3000
```

### Using Docker & Nginx
If running inside a containerized environment:

```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

Build and run:
```bash
docker build -t math-trainer .
docker run -d -p 8080:80 math-trainer
```
Access at `http://localhost:8080`.
