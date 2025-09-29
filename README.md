# World Geo Data 🌍

Base de datos geográfica completa con países, estados/provincias y ciudades. Especialmente enfocada en Argentina y futuro demás paises de Latinoamérica.

## Características

- ✅ **+200 países (futuro)** con códigos ISO
- ✅ **Estados/Provincias** detallados
- ✅ **Ciudades** principales y secundarias
- ✅ **Formato JSON** fácil de usar
- ✅ **Actualizaciones** constantes
- ✅ **Enfoque LATAM** con cobertura

## 🛡️ Características de Seguridad

### Validación Integrada
Incluye sistema de validación completo que **previene**:
- ✅ **Inyección de datos maliciosos**
- ✅ **Ubicaciones falsas o inexistentes** 
- ✅ **Ataques de inconsistencia geográfica**
- ✅ **XSS a través de nombres de ciudades**

### Ejemplo de protección:
```http
POST /api/register
{
    "country": "XX",          # ❌ Rechazado - País inexistente
    "province": "HACK",       # ❌ Rechazado - Provincia inválida
    "city": "<script>alert('xss')</script>" # ❌ Rechazado - XSS attempt
}
```
### Flujo Seguro:
1. **Usuario** interactúa con selects (puede modificarlos)
2. **Frontend** ayuda con UX (validación opcional)  
3. **Backend** valida contra GitHub (seguridad real)
4. **Sistema** rechaza datos inválidos

## Uso Rápido

```javascript
// Países
fetch('https://raw.githubusercontent.com/boriscr/world-geo-data/main/data/countries.json')

// Provincias de Argentina
fetch('https://raw.githubusercontent.com/boriscr/world-geo-data/main/data/states/AR.json')

// Ciudades de Buenos Aires, Argentina
fetch('https://raw.githubusercontent.com/boriscr/world-geo-data/main/data/cities/AR/B.json')
```

### Vista escritorio
<img src="https://i.postimg.cc/7PJ05XMn/location1.png">

### Vista movil
<img src="https://i.postimg.cc/dQZrhnm9/location2.png">