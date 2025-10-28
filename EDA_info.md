# 1.  Verificación de integridad

- Se analizaron las carpetas `train/images` y `train/masks`.
- Todas las imágenes tienen su correspondiente máscara.
- Las dimensiones y resoluciones son consistentes (800×800 píxeles).
- Las imágenes son RGB (3 canales) y las máscaras son binarias (0/1).
- No se detectaron máscaras vacías ni completamente llenas.

**Conclusión:** el dataset se encuentra limpio y listo para el análisis exploratorio y la preparación de datos.
