# ============================================
# STAGE 1: Build stage
# ============================================
# Koristimo Node.js 20 na Alpine Linuxu kao baznu sliku.
# Alpine je minimalna Linux distribucija (~5MB) što rezultira manjom
# i sigurnijom kontejnerskom slikom jer ima manje paketa = manja napadna površina.
FROM node:20-alpine AS build

# Postavljamo radni direktorij unutar kontejnera.
# Sve naredbe nakon ovoga izvršavat će se u /app direktoriju.
WORKDIR /app

# Kopiramo SAMO package.json i package-lock.json prije ostatka koda.
# Ovo je optimizacija — Docker kešira svaki sloj (layer), pa ako se
# ovisnosti nisu promijenile, npm install se neće ponovno pokretati
# pri svakom rebuildu, što drastično ubrzava buildove.
COPY package.json package-lock.json* ./

# Instaliramo SAMO produkcijske ovisnosti (bez devDependencies poput nodemon-a).
# --omit=dev znači da preskačemo devDependencies jer nam u produkciji ne trebaju.
RUN npm ci --omit=dev

# ============================================
# STAGE 2: Runtime stage
# ============================================
# Počinjemo novi stage s čistom Alpine slikom.
# Ovo je "multi-stage build" — iz prvog stagea kopiramo samo ono što trebamo,
# a sve ostalo (npm cache, build alate) ostavljamo iza sebe.
# Rezultat je puno manja finalna slika.
FROM node:20-alpine

# Kreiramo neprivilegiranog korisnika (non-root).
# Po defaultu kontejneri rade kao root, što je sigurnosni rizik —
# ako napadač kompromitira aplikaciju, ima root pristup unutar kontejnera.
# S non-root korisnikom ograničavamo štetu koju napadač može napraviti.
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /app

# Kopiramo instalirane node_modules iz build stagea.
# --from=build znači "uzmi iz prvog stagea koji smo nazvali 'build'".
COPY --from=build /app/node_modules ./node_modules

# Kopiramo source kod i package.json u kontejner.
COPY src/ ./src/
COPY package.json ./

# Prebacujemo se na non-root korisnika.
# Od ove točke nadalje, sve se izvršava kao 'appuser', ne kao root.
USER appuser

# Dokumentiramo da ovaj kontejner sluša na portu 8080.
# EXPOSE sam po sebi ne otvara port — to je samo metadata/dokumentacija.
EXPOSE 8080

# Naredba koja se izvršava kad se kontejner pokrene.
CMD ["node", "src/server.js"]