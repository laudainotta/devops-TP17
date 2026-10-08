## TP17 - Detección de Secretos con Gitleaks (Secret Scanning)

Gitleaks audita el grafo completo de commits de Git para detectar credenciales expuestas (regex por firma + entropía de Shannon). Se aplica en dos niveles de defensa en profundidad:

- **Estación local:** hook `.git/hooks/pre-commit` con `gitleaks protect --staged`, que bloquea el commit antes de que el secreto entre al historial.
- **CI/CD:** jobs `gitleaks-andon-cord` y `gitleaks-audit-report` en la Fase 2 del pipeline, con `fetch-depth: 0`.

### Matriz de Control

| Job en Pipeline | Tipo de Evaluación | Exit Code | Acción ante Hallazgos | Evidencia Generada |
|---|---|---|---|---|
| `gitleaks-andon-cord` | Secret scanning en historial Git | 1 | Andon Cord: bloquea el pipeline y el merge del PR | Resumen en `$GITHUB_STEP_SUMMARY` + comentario en el PR |
| `gitleaks-audit-report` | Reporte de inspección JSON | 0 | Informativo: genera evidencia | Artifact JSON descargable (`--redact`) |

### Configuración

- `.gitleaks.toml`: hereda las reglas por defecto (`[extend] useDefault = true`) y declara una allowlist con los valores de prueba conocidos del template (`devops123`, `dev-password`).
- `scripts/verificar-gitleaks.sh`: verificación local de instalación, workflow y escaneo preventivo.

### Remediación ante una fuga

1. Rotar o revocar la credencial en el proveedor (paso obligatorio: GitHub conserva commits huérfanos).
2. Purgar el historial: `git filter-repo --force --replace-text <archivo-de-reemplazos>` con un placeholder que no coincida con las reglas (`***REMOVED***`).
3. Volver a agregar `origin` y sincronizar: `git push origin --force --all`.
