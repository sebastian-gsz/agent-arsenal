# Redención de setup token

Lee este archivo cuando haya un setup token presente y la máquina no esté autenticada.

Un setup token es una credencial de un solo uso y vida corta suministrada por una superficie de Tavily autenticada (p. ej. el Playground). **No es una API key normal de Tavily.**

## Contrato

1. Haz `POST` del token a `https://api.tavily.com/v1/setup-tickets/redeem` con JSON `{ "ticket": "..." }`. Si Tavily despliega otro endpoint, usa `TAVILY_SETUP_REDEEM_URL`.
2. Espera la API key existente del usuario en la respuesta como `api_key`.
3. Persiste la key en `~/.tavily/config.json` con permisos solo-owner y en el formato de almacenamiento actual del Tavily CLI. Nunca pongas la key en el historial de shell ni la imprimas. **Si no puedes verificar el formato de config actual del CLI, no escribas el archivo a mano — cae a OAuth por navegador.**
4. Descarta el setup token y la key en memoria en cuanto dejen de hacer falta.
5. Verifica con `tvly auth --json`.

## Redención por curl (token solo como variable de entorno, sin pasar por argv)

```bash
printf '{"ticket": "%s"}' "$TAVILY_SETUP_TOKEN" | curl -sS -X POST \
  "${TAVILY_SETUP_REDEEM_URL:-https://api.tavily.com/v1/setup-tickets/redeem}" \
  -H 'Content-Type: application/json' \
  --data @-
```

El helper empaquetado, si esta skill lo incluye, implementa este flujo:

```bash
python scripts/tavily_auth_bootstrap.py [--setup-token "$TAVILY_SETUP_TOKEN"]
```

Si `scripts/tavily_auth_bootstrap.py` no existe en la carpeta de la skill, usa la redención por curl de arriba o cae directamente a OAuth por navegador.

## Reglas

- Nunca imprimir, hacer echo, loguear, commitear ni incluir el setup token o la API key resultante en la salida del chat.
- No afirmes que el `tvly` público tiene una opción `--setup-token` salvo que esa opción haya salido de verdad.
- Si la redención falla porque el token expiró, ya se usó o el endpoint no está disponible, cae automáticamente a OAuth por navegador.
