# Medizano-Docs

Documentación técnica y operativa de [MediZano Botica](https://github.com/rbn-69-cod/MediZano_microservicios), publicada con MkDocs Material.

## Vista local

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
mkdocs serve
```

Abra `http://127.0.0.1:8000`.

## Compilar el sitio

```bash
mkdocs build --strict
```

Los diagramas están escritos en Mermaid dentro de los archivos Markdown para que puedan mantenerse junto con la arquitectura.
