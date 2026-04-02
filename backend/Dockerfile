FROM dhi.io/python:3.12-debian12-dev AS build

WORKDIR /app
COPY --from=ghcr.io/astral-sh/uv:0.10.0 /uv /uvx /bin/

# On copie le code et les fichiers de configuration
COPY . .

# On installe les paquets "en dur" dans un dossier spécifique /app/deps
# Cela contourne totalement le problème des liens symboliques cassés
RUN uv pip install --system --target /app/deps .

FROM dhi.io/python:3.12-debian12 AS prod

WORKDIR /app
# On force Python à regarder dans notre dossier /app/deps pour trouver Gunicorn et Django
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PYTHONPATH="/app/deps:$PYTHONPATH" \
    PATH="/app/deps/bin:$PATH"

# On récupère uniquement notre dossier de dépendances physiques
COPY --from=build /app/deps /app/deps
# On copie le reste du code
COPY . .

EXPOSE 8000
CMD ["python", "-m", "gunicorn", "config.wsgi:application", "--bind", "0.0.0.0:8000", "--access-logfile", "-", "--error-logfile", "-", "--log-level", "info"]
