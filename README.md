# World Geo Data 🌍
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)

Base de datos geográfica completa con países, estados/provincias y ciudades. Especialmente enfocada en Argentina y futuro demás paises de Latinoamérica.

## 🚀 ¿Por qué usar World Geo Data?

### Para Desarrolladores que odian configurar bases de datos geográficas:

| Problema común                               | Solución                                     |
| -------------------------------------------- | -------------------------------------------- |
| "Configurar países/provincias es un dolor"   | ✅ **JSON listo para usar**                   |
| "Los datos se desactualizan"                 | ✅ **Actualizaciones automáticas vía GitHub** |
| "No encuentro ciudades de Latinoamérica"     | ✅ **Enfoque LATAM con cobertura profunda**   |
| "La validación de ubicaciones es complicada" | ✅ **Sistema de validación integrado**        |

### ⚡ Empezar en 2 minutos

**1. Obtén países:**
```javascript
// ¡Solo una línea!
const países = await fetch('https://raw.githubusercontent.com/boriscr/world-geo-data/main/data/countries.json').then(r => r.json());
```
**2. Integra en tu formulario:**
```
<select id="país">
    <option value="">Seleccionar país</option>
</select>

<script>
fetch('https://raw.githubusercontent.com/boriscr/world-geo-data/main/data/countries.json')
    .then(response => response.json())
    .then(países => {
        países.forEach(país => {
            document.getElementById('país').innerHTML += 
                `<option value="${país.code}">${país.name} ${país.flag}</option>`;
        });
    });
</script>
```
### 📊 Datos Disponibles
**🌎 Países (Cobertura actual)**
```
[
  {"code": "AR", "name": "Argentina", "flag": "🇦🇷"},
  {"code": "BO", "name": "Bolivia", "flag": "🇧🇴"},
  {"code": "UY", "name": "Uruguay", "flag": "🇺🇾"},
  {"code": "CL", "name": "Chile", "flag": "🇨🇱"},
  {"code": "BR", "name": "Brasil", "flag": "🇧🇷"},
  {"code": "PY", "name": "Paraguay", "flag": "🇵🇾"}
]
```
### 🛡️ Seguridad Incorporada
```
// Backend validation - Sin esfuerzo extra
if (!isValidCountry($request->país)) {
    return response()->json(['error' => 'País inválido'], 422);
}
```
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

### 🔧 Integración Rápida
**Para Laravel (¡Más fácil!)**
<a href="https://github.com/boriscr/world-geo-data/blob/main/data/examples/laravel-integration.md">Ver guía completa</a>
```
# 1. Controlador de ubicaciones
php artisan make:controller Api/LocationController

# 2. Copiar nuestro código probado
# 3. ¡Listo! Validaciones automáticas incluidas
```
## Uso Rápido

```javascript
// Países
fetch('https://raw.githubusercontent.com/boriscr/world-geo-data/main/data/countries.json')

// Provincias de Argentina
fetch('https://raw.githubusercontent.com/boriscr/world-geo-data/main/data/states/AR.json')

// Ciudades de Buenos Aires, Argentina
fetch('https://raw.githubusercontent.com/boriscr/world-geo-data/main/data/cities/AR/B.json')
```
**Para cualquier stack:**
```
// React/Vue/Angular
const [países, setPaíses] = useState([]);

useEffect(() => {
    fetch('https://raw.githubusercontent.com/boriscr/world-geo-data/main/data/countries.json')
        .then(res => res.json())
        .then(setPaíses);
}, []);
```

### Vista escritorio
<a href="https://postimages.org/" target="_blank"><img src="https://i.postimg.cc/7PJ05XMn/location1.png" alt="location1"/></a>

### Vista movil
<a href="https://postimages.org/" target="_blank"><img src="https://i.postimg.cc/dQZrhnm9/location2.png" alt="location2"/></a>


## ❓ Preguntas Frecuentes
**¿Es gratis?**
¡Sí! Totalmente gratis y open source.

**¿Los datos se actualizan?**
Sí, actualizamos constantemente y aceptamos contribuciones.

**¿Puedo usarlo en producción?**
¡Absolutamente! Diseñado para uso en producción.