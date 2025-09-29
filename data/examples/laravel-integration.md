# Integración con Laravel 🌍🚀

Guía completa para integrar world-geo-data en tu aplicación Laravel.

## 📋 Requisitos

- Laravel 9+ 
- PHP 8.0+
- Conexión a internet (para fetch de GitHub)

## 🚀 Instalación Rápida

### 1. Crear el Controlador

```bash
php artisan make:controller Api/LocationController
```
### 2. Configurar el Controlador

app/Http/Controllers/Api/LocationController.php:
```
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use Illuminate\Http\JsonResponse;
use Illuminate\Support\Facades\Cache;
use Illuminate\Support\Facades\Log;

class LocationController extends Controller
{
    private $githubBaseUrl = 'https://raw.githubusercontent.com/boriscr/world-geo-data/main/data';
    private $cacheTime = 86400; // 24 horas

    public function getCountries(): JsonResponse
    {
        $data = $this->cachedResponse('countries.json', []);
        return response()->json($data);
    }

    public function getStates(string $countryCode): JsonResponse
    {
        $data = $this->cachedResponse("states/{$countryCode}.json", []);
        return response()->json($data);
    }

    public function getCities(string $countryCode, string $stateCode): JsonResponse
    {
        $cities = $this->fetchFromGitHub("cities/{$countryCode}/{$stateCode}.json") ?? [];
        
        $formattedCities = array_map(function($city) {
            return ['name' => $city];
        }, $cities);
        
        return response()->json($formattedCities);
    }

    private function cachedResponse(string $path, $default = [])
    {
        $cacheKey = "geo_data_" . md5($path);
        
        return Cache::remember($cacheKey, $this->cacheTime, function () use ($path, $default) {
            return $this->fetchFromGitHub($path) ?? $default;
        });
    }

    private function fetchFromGitHub(string $path)
    {
        $url = "{$this->githubBaseUrl}/{$path}";
        
        try {
            $context = stream_context_create([
                'http' => ['timeout' => 5, 'user_agent' => 'Laravel-App/1.0']
            ]);
            
            $content = file_get_contents($url, false, $context);
            return $content ? json_decode($content, true) : null;
            
        } catch (\Exception $e) {
            Log::warning("GitHub fetch failed [{$path}]: " . $e->getMessage());
            return $this->getFallbackData($path);
        }
    }

    private function getFallbackData(string $path)
    {
        $localPath = resource_path("geo/{$path}");
        return file_exists($localPath) ? json_decode(file_get_contents($localPath), true) : null;
    }
}
```
### 3. Configurar Rutas
routes/web.php:
```
// Ubicación geográfica
Route::get('/location/countries', [App\Http\Controllers\Api\LocationController::class, 'getCountries']);
Route::get('/location/countries/{countryCode}/states', [App\Http\Controllers\Api\LocationController::class, 'getStates']);
Route::get('/location/countries/{countryCode}/states/{stateCode}/cities', [App\Http\Controllers\Api\LocationController::class, 'getCities']);
```
### 4. Frontend
#### Blade:
##### Opcion 1 sin componentes:
```
<div class="grid grid-cols-1 md:grid-cols-3 gap-4">
    <!-- País -->
    <div class="form-group">
        <label for="country">País *</label>
        <select name="country" id="country" required onchange="loadStates(this.value)">
            <option value="">Seleccionar país</option>
        </select>
    </div>

    <!-- Provincia -->
    <div class="form-group">
        <label for="province">Provincia *</label>
        <select name="province" id="province" required onchange="loadCities(this.value)" disabled>
            <option value="">Primero selecciona un país</option>
        </select>
    </div>

    <!-- Ciudad -->
    <div class="form-group">
        <label for="city">Ciudad *</label>
        <select name="city" id="city" required disabled>
            <option value="">Primero selecciona una provincia</option>
        </select>
    </div>
</div>
```
##### Opcion 2 uso de componentes:
###### 1. Crear el componente:
```
php artisan make:component select
```
###### 2. Configurar el componente
resources/views/components/select.blade.php
```
@props([
    'name', 
    'label', 
    'icon' => null, 
    'required' => false,
    'disabled' => false,
    'onchange' => null,
])

<div class="item">
    <x-input-label :for="$name" :icon="$icon" :value="$label" :required="$required" />
    <select 
        name="{{ $name }}" 
        id="{{ $name }}" 
        class="w-full rounded"
        @if($required) required @endif
        @if($disabled) disabled @endif
        @if($onchange) onchange="{{ $onchange }}" @endif
        {{ $attributes }}
    >
        {{ $slot }}
    </select>
    <x-input-error :messages="$errors->get($name)" class="mt-2" />
</div>
```
###### 3. Configurar el componente input-label
resources/views/components/input-label.blade.php
```
@props(['value', 'required' => false, 'icon' => ''])

<label {{ $attributes->merge(['class' => 'block font-medium text-sm']) }}>
    @if ($required)
    <span aria-hidden="true" style="color: red;">*</span>
    @endif
    <i class="bi bi-{{ $icon??'person' }}"></i>
    {{ $value ?? $slot }}
</label>
```
###### 4. Vista front usando componentes

```
{{-- Sección de ubicación --}}
{{-- País --}}
<div class="grid grid-cols-1 md:grid-cols-3 gap-4">
    {{-- País --}}
    <x-select icon="globe" name="country" :label="__('contact.country')" :required="true"
        onchange="loadStates(this.value)">
        <option value="">Seleccionar país</option>
    </x-select>

    {{-- Provincia --}}
    <x-select icon="geo-alt" name="province" :label="__('contact.province')" :required="true"
        onchange="loadCities(this.value)" disabled>
        <option value="">Primero selecciona un país</option>
    </x-select>

    {{-- Ciudad --}}
    <x-select icon="building" name="city" :label="__('contact.city')" :required="true" disabled>
        <option value="">Primero selecciona una provincia</option>
    </x-select>
</div>
```

#### JavaScript:
```
<script>
// Location Manager - Gestor de ubicaciones
class LocationManager {
    constructor() {
        this.baseUrl = window.location.origin;
        this.initialized = false;
    }

    initialize() {
        if (this.initialized) return;

        const countrySelect = document.getElementById('country');
        const provinceSelect = document.getElementById('province');
        const citySelect = document.getElementById('city');

        if (!countrySelect || !provinceSelect || !citySelect) {
            console.warn('Location selects not found, retrying...');
            setTimeout(() => this.initialize(), 100);
            return;
        }

        console.log('Location manager initialized');
        this.initialized = true;
        this.loadCountries();
    }

    async loadCountries() {
        try {
            const response = await fetch(`${this.baseUrl}/location/countries`);
            if (!response.ok) throw new Error('Error HTTP: ' + response.status);

            const countries = await response.json();
            const countrySelect = document.getElementById('country');

            countrySelect.innerHTML = '<option value="">Seleccionar país</option>';
            countries.forEach(country => {
                const option = document.createElement('option');
                option.value = country.code;
                option.textContent = `${country.name} ${country.flag || ''}`;
                countrySelect.appendChild(option);
            });

            // Agregar event listener
            countrySelect.onchange = (e) => this.loadStates(e.target.value);

        } catch (error) {
            console.error('Error loading countries:', error);
            this.loadManualCountries();
        }
    }

    async loadStates(countryCode) {
        const stateSelect = document.getElementById('province');
        const citySelect = document.getElementById('city');

        if (!countryCode) {
            this.resetLocationSelects();
            return;
        }

        try {
            stateSelect.disabled = true;
            stateSelect.innerHTML = '<option value="">Cargando provincias...</option>';

            const response = await fetch(`${this.baseUrl}/location/countries/${countryCode}/states`);
            if (!response.ok) throw new Error('Error HTTP: ' + response.status);

            const states = await response.json();
            stateSelect.innerHTML = '<option value="">Seleccionar provincia</option>';

            states.forEach(state => {
                const option = document.createElement('option');
                option.value = state.code;
                option.textContent = state.name;
                stateSelect.appendChild(option);
            });

            stateSelect.disabled = false;
            stateSelect.onchange = (e) => this.loadCities(e.target.value);
            this.resetCityField();

        } catch (error) {
            console.error('Error loading states:', error);
            stateSelect.innerHTML = '<option value="">Error al cargar</option>';
        }
    }

    async loadCities(stateCode) {
        const countryCode = document.getElementById('country').value;
        const citySelect = document.getElementById('city');

        if (!countryCode || !stateCode) {
            this.resetCityField();
            return;
        }

        try {
            citySelect.disabled = true;
            citySelect.innerHTML = '<option value="">Cargando ciudades...</option>';

            const response = await fetch(`${this.baseUrl}/location/countries/${countryCode}/states/${stateCode}/cities`);
            if (!response.ok) throw new Error('Error HTTP: ' + response.status);

            const cities = await response.json();
            citySelect.innerHTML = '<option value="">Seleccionar ciudad</option>';

            if (cities.length > 0) {
                cities.forEach(city => {
                    const option = document.createElement('option');
                    option.value = city.name;
                    option.textContent = city.name;
                    citySelect.appendChild(option);
                });
                citySelect.disabled = false;
            } else {
                citySelect.innerHTML = '<option value="">No hay ciudades disponibles</option>';
            }

        } catch (error) {
            console.error('Error loading cities:', error);
            citySelect.innerHTML = '<option value="">Error al cargar ciudades</option>';
        }
    }

    resetCityField() {
        const citySelect = document.getElementById('city');
        citySelect.disabled = true;
        citySelect.innerHTML = '<option value="">Selecciona provincia primero</option>';
    }

    resetLocationSelects() {
        document.getElementById('province').disabled = true;
        document.getElementById('province').innerHTML = '<option value="">Primero selecciona un país</option>';
        this.resetCityField();
    }

    loadManualCountries() {
        const countries = [
            { code: 'AR', name: 'Argentina' }, { code: 'BO', name: 'Bolivia' },
            { code: 'UY', name: 'Uruguay' }, { code: 'CL', name: 'Chile' }
        ];

        const countrySelect = document.getElementById('country');
        countrySelect.innerHTML = '<option value="">Seleccionar país</option>';
        countries.forEach(country => {
            const option = document.createElement('option');
            option.value = country.code;
            option.textContent = country.name;
            countrySelect.appendChild(option);
        });
    }
    async validateSelections() {
        const country = document.getElementById('country').value;
        const province = document.getElementById('province').value;
        const city = document.getElementById('city').value;

        // Validación UX (no de seguridad)
        if (!country || !province || !city) {
            this.showValidationError('Por favor, completa todas las ubicaciones');
            return false;
        }

        // Verificar que las selecciones sean consistentes (UX)
        const isValid = await this.areSelectionsConsistent(country, province, city);

        if (!isValid) {
            this.showValidationError('Las ubicaciones seleccionadas no son consistentes');
            return false;
        }

        this.clearValidationError();
        return true;
    }

    async areSelectionsConsistent(country, province, city) {
        try {
            // Validar que la provincia pertenece al país
            const provincesResponse = await fetch(`${this.baseUrl}/location/countries/${country}/states`);
            const provinces = await provincesResponse.json();
            const validProvince = provinces.some(p => p.code === province);
            if (!validProvince) return false;

            // Validar que la ciudad pertenece a la provincia
            const citiesResponse = await fetch(`${this.baseUrl}/location/countries/${country}/states/${province}/cities`);
            const cities = await citiesResponse.json();
            const validCity = cities.some(c => c.name === city);

            return validCity;

        } catch (error) {
            console.error('Validation error:', error);
            return true; // ✅ Si falla, dejamos que el backend decida
        }
    }

    showValidationError(message) {
        this.clearValidationError();

        const errorDiv = document.createElement('div');
        errorDiv.className = 'validation-error mt-2 p-2 bg-red-100 border border-red-400 text-red-700 rounded';
        errorDiv.id = 'location-validation-error';
        errorDiv.textContent = message;

        const citySelect = document.getElementById('city');
        citySelect.parentNode.appendChild(errorDiv);
    }

    clearValidationError() {
        const existingError = document.getElementById('location-validation-error');
        if (existingError) existingError.remove();
    }

}

// Crear instancia global
window.locationManager = new LocationManager();

// Inicializar cuando el DOM esté listo
document.addEventListener('DOMContentLoaded', function () {
    window.locationManager.initialize();
});

// También inicializar si el DOM ya está listo
if (document.readyState === 'interactive' || document.readyState === 'complete') {
    window.locationManager.initialize();
}
</script>
```
### 5. Validación en FormRequest
app/Http/Requests/RegisteredStoreRequest.php:
```
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Support\Facades\Cache;

class RegisteredStoreRequest extends FormRequest
{
    private $cacheTime = 3600;
    private $githubBaseUrl = 'https://raw.githubusercontent.com/boriscr/world-geo-data/main/data';

    public function rules(): array
    {
        return [
            // ... otros campos ...
            
            'country' => [
                'required', 
                'string', 
                'max:2',
                function ($attribute, $value, $fail) {
                    if (!$this->isValidCountry($value)) {
                        $fail('El país seleccionado no es válido.');
                    }
                }
            ],
            'province' => [
                'required',
                'string', 
                'max:10',
                function ($attribute, $value, $fail) {
                    $country = $this->input('country');
                    if (!$this->isValidProvince($country, $value)) {
                        $fail('La provincia seleccionada no es válida para el país.');
                    }
                }
            ],
            'city' => [
                'required',
                'string', 
                'max:100',
                function ($attribute, $value, $fail) {
                    $country = $this->input('country');
                    $province = $this->input('province');
                    if (!$this->isValidCity($country, $province, $value)) {
                        $fail('La ciudad seleccionada no es válida para la provincia.');
                    }
                }
            ],
        ];
    }

    private function isValidCountry(string $countryCode): bool
    {
        if (empty($countryCode)) return false;

        $countries = Cache::remember('valid_countries', $this->cacheTime, function () {
            return $this->fetchFromGitHub('countries.json') ?? [];
        });

        return collect($countries)->pluck('code')->contains($countryCode);
    }

    private function isValidProvince(string $countryCode, string $provinceCode): bool
    {
        if (empty($countryCode) || empty($provinceCode)) return false;

        $cacheKey = "provinces_{$countryCode}";
        
        $provinces = Cache::remember($cacheKey, $this->cacheTime, function () use ($countryCode) {
            return $this->fetchFromGitHub("states/{$countryCode}.json") ?? [];
        });

        return collect($provinces)->contains('code', $provinceCode);
    }

    private function isValidCity(string $countryCode, string $provinceCode, string $cityName): bool
    {
        if (empty($countryCode) || empty($provinceCode) || empty($cityName)) return false;

        $cacheKey = "cities_{$countryCode}_{$provinceCode}";
        
        $cities = Cache::remember($cacheKey, $this->cacheTime, function () use ($countryCode, $provinceCode) {
            return $this->fetchFromGitHub("cities/{$countryCode}/{$provinceCode}.json") ?? [];
        });

        return in_array($cityName, $cities);
    }

    private function fetchFromGitHub(string $path)
    {
        $url = "{$this->githubBaseUrl}/{$path}";
        
        $ch = curl_init();
        
        curl_setopt_array($ch, [
            CURLOPT_URL => $url,
            CURLOPT_RETURNTRANSFER => true,
            CURLOPT_TIMEOUT => 10,
            CURLOPT_FOLLOWLOCATION => true,
            CURLOPT_SSL_VERIFYPEER => app()->environment('production'),
            CURLOPT_SSL_VERIFYHOST => app()->environment('production') ? 2 : false,
            CURLOPT_USERAGENT => 'Laravel-App/1.0',
        ]);
        
        $response = curl_exec($ch);
        $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
        
        curl_close($ch);
        
        if ($httpCode === 200 && $response !== false) {
            return json_decode($response, true);
        }
        
        return null;
    }

    public function messages(): array
    {
        return [
            'country.*' => 'País inválido.',
            'province.*' => 'Provincia inválida para el país seleccionado.',
            'city.*' => 'Ciudad inválida para la provincia seleccionada.',
        ];
    }
}
```