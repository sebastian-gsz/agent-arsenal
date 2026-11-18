# Sistema de diseño unificado para data apps

Toda aplicación de datos construida con esta skill DEBE adherirse a estos principios visuales y funcionales para asegurar una experiencia consistente y profesional de "grado SaaS".

## Principios de diseño
1. **Chrome mínimo**: oculta la UI por defecto del framework (encabezados, pies, toolbars) para que la app se sienta independiente.
2. **Layout basado en tarjetas**: cada pieza lógica de contenido (KPIs, gráficos, tablas) DEBE vivir en una tarjeta con borde y sombras sutiles.
3. **Paleta de colores sobria**: 
   - Usa una paleta inspirada en "Zinc" (grises/Zinc para texto y bordes).
   - Acento primario: azul (`#2563eb`).
4. **Codificación semántica**:
   - Éxito: verde
   - Error: rojo
   - Advertencia: ámbar
5. **Tipografía**:
   - Cuerpo/encabezados: `DM Sans`
   - Datos/código: `JetBrains Mono`
6. **Espaciado**: usa padding generoso (al menos `1.25rem` dentro de las tarjetas) y gaps consistentes entre elementos.

## Paleta de colores (estilo Zinc/shadcn)
- **Fondo**: blanco (claro) / `#09090b` (oscuro)
- **Tarjeta**: blanco (claro) / `#0c0c0f` (oscuro)
- **Borde**: `#e4e4e7` (claro) / `#1e1e24` (oscuro)
- **Texto**: `#09090b` (claro) / `#fafafa` (oscuro)
- **Texto atenuado**: `#71717a` (estándar) / `#52525b` (oscuro)
