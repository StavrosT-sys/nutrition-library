# Best Practices for Nutrition API Integration

This document outlines proven patterns and best practices for implementing nutrition tracking features in your fitness app.

## Table of Contents

1. [Performance Optimization](#performance-optimization)
2. [Caching Strategies](#caching-strategies)
3. [User Experience](#user-experience)
4. [Data Accuracy](#data-accuracy)
5. [Error Handling](#error-handling)
6. [Compliance and Attribution](#compliance-and-attribution)

---

## Performance Optimization

### API Call Reduction

**Problem:** Each API call adds latency and consumes quota.

**Solution:** Implement smart caching and batch requests.

```javascript
// ❌ Bad: Multiple individual calls
async function getUserMeal(foods) {
  const nutrition = [];
  for (const food of foods) {
    nutrition.push(await api.getNutrition(food));
  }
  return nutrition;
}

// ✅ Good: Batch request
async function getUserMeal(foods) {
  return await api.getNutritionBatch(foods);
}
```

### Request Debouncing

For search-as-you-type features, debounce API calls:

```javascript
class DebouncedSearch {
  constructor(api, delayMs = 300) {
    this.api = api;
    this.delayMs = delayMs;
    this.timeout = null;
  }

  search(query) {
    return new Promise((resolve, reject) => {
      clearTimeout(this.timeout);
      this.timeout = setTimeout(async () => {
        try {
          const results = await this.api.search(query);
          resolve(results);
        } catch (error) {
          reject(error);
        }
      }, this.delayMs);
    });
  }
}

// Usage
const search = new DebouncedSearch(api);
inputElement.addEventListener('input', (e) => {
  search.search(e.target.value)
    .then(results => displayResults(results));
});
```

### Connection Pooling

Reuse HTTP connections for better performance:

```javascript
const axios = require('axios');
const http = require('http');
const https = require('https');

const httpAgent = new http.Agent({ keepAlive: true, maxSockets: 50 });
const httpsAgent = new https.Agent({ keepAlive: true, maxSockets: 50 });

const api = axios.create({
  httpAgent: httpAgent,
  httpsAgent: httpsAgent,
  timeout: 5000
});
```

---

## Caching Strategies

### Three-Tier Caching

Implement caching at multiple levels for optimal performance:

```javascript
class CachingNutritionAPI {
  constructor(api) {
    this.api = api;
    this.memoryCache = new Map();        // L1: In-memory
    this.localStorageCache = {};         // L2: Browser storage
    this.indexedDBCache = null;          // L3: IndexedDB (large datasets)
  }

  async getNutrition(query) {
    const key = this.getCacheKey(query);

    // L1: Check memory cache
    if (this.memoryCache.has(key)) {
      console.log('L1 Cache hit');
      return this.memoryCache.get(key);
    }

    // L2: Check localStorage
    const stored = localStorage.getItem(key);
    if (stored) {
      console.log('L2 Cache hit');
      const data = JSON.parse(stored);
      this.memoryCache.set(key, data);
      return data;
    }

    // L3: Check IndexedDB
    const dbData = await this.getFromIndexedDB(key);
    if (dbData) {
      console.log('L3 Cache hit');
      this.memoryCache.set(key, dbData);
      return dbData;
    }

    // API call
    console.log('Cache miss - calling API');
    const data = await this.api.getNutrition(query);

    // Store in all caches
    this.memoryCache.set(key, data);
    localStorage.setItem(key, JSON.stringify(data));
    await this.saveToIndexedDB(key, data);

    return data;
  }

  getCacheKey(query) {
    return query.toLowerCase().trim();
  }

  async getFromIndexedDB(key) {
    // Implementation depends on your IndexedDB setup
    return null;
  }

  async saveToIndexedDB(key, data) {
    // Implementation depends on your IndexedDB setup
  }
}
```

### Cache Invalidation

Implement smart cache expiration:

```javascript
class ExpiringCache {
  constructor(ttlMinutes = 24 * 60) {
    this.cache = new Map();
    this.ttlMs = ttlMinutes * 60 * 1000;
  }

  set(key, value) {
    this.cache.set(key, {
      value,
      expiresAt: Date.now() + this.ttlMs
    });
  }

  get(key) {
    const item = this.cache.get(key);
    
    if (!item) return null;
    
    if (Date.now() > item.expiresAt) {
      this.cache.delete(key);
      return null;
    }
    
    return item.value;
  }

  clear() {
    this.cache.clear();
  }
}
```

### Offline Support

Cache popular foods for offline access:

```javascript
class OfflineNutritionAPI {
  constructor(api) {
    this.api = api;
    this.offlineDB = new Map();
    this.isOnline = navigator.onLine;

    window.addEventListener('online', () => this.isOnline = true);
    window.addEventListener('offline', () => this.isOnline = false);
  }

  async searchFood(query) {
    // Try online first
    if (this.isOnline) {
      try {
        const results = await this.api.searchFood(query);
        // Cache results for offline
        this.cacheForOffline(query, results);
        return results;
      } catch (error) {
        console.warn('Online search failed, trying offline cache');
      }
    }

    // Fall back to offline cache
    const cached = this.offlineDB.get(query.toLowerCase());
    if (cached) {
      return cached;
    }

    throw new Error('No internet connection and no cached data');
  }

  cacheForOffline(query, results) {
    const key = query.toLowerCase();
    this.offlineDB.set(key, results);
  }
}
```

---

## User Experience

### Search Autocomplete

Implement autocomplete with local suggestions:

```javascript
class AutocompleteSearch {
  constructor(api) {
    this.api = api;
    this.suggestions = [];
    this.frequentFoods = this.loadFrequentFoods();
  }

  async getSuggestions(query) {
    if (!query || query.length < 2) {
      // Show frequent foods
      return this.frequentFoods.slice(0, 5);
    }

    // Combine API results with frequent foods
    const apiResults = await this.api.searchFood(query);
    const combined = [
      ...this.frequentFoods.filter(f => 
        f.name.toLowerCase().includes(query.toLowerCase())
      ),
      ...apiResults
    ];

    // Remove duplicates and limit
    return Array.from(new Map(
      combined.map(item => [item.id, item])
    ).values()).slice(0, 10);
  }

  recordFoodSelection(food) {
    // Track user selections for frequent foods
    const index = this.frequentFoods.findIndex(f => f.id === food.id);
    
    if (index > -1) {
      this.frequentFoods[index].count++;
    } else {
      this.frequentFoods.push({ ...food, count: 1 });
    }

    // Sort by frequency
    this.frequentFoods.sort((a, b) => b.count - a.count);
    
    // Save to storage
    this.saveFrequentFoods();
  }

  loadFrequentFoods() {
    const stored = localStorage.getItem('frequentFoods');
    return stored ? JSON.parse(stored) : [];
  }

  saveFrequentFoods() {
    localStorage.setItem('frequentFoods', JSON.stringify(this.frequentFoods));
  }
}
```

### Barcode Scanning

Implement barcode scanning for quick food entry:

```javascript
class BarcodeScanner {
  constructor(api) {
    this.api = api;
    this.scanner = null;
  }

  async initScanner() {
    // Using QuaggaJS for barcode detection
    Quagga.init({
      inputStream: {
        name: 'Live',
        type: 'LiveStream',
        target: document.querySelector('#scanner')
      },
      decoder: {
        readers: ['ean_reader', 'upc_reader']
      }
    }, (err) => {
      if (err) {
        console.error('Scanner init error:', err);
        return;
      }
      Quagga.start();
    });

    Quagga.onDetected((result) => {
      this.handleBarcodeScan(result.codeResult.code);
    });
  }

  async handleBarcodeScan(barcode) {
    try {
      const food = await this.api.getFoodByBarcode(barcode);
      return food;
    } catch (error) {
      console.error('Barcode lookup failed:', error);
      return null;
    }
  }

  stopScanner() {
    Quagga.stop();
  }
}
```

### Portion Size Estimation

Help users estimate portions accurately:

```javascript
class PortionEstimator {
  constructor() {
    this.commonPortions = {
      'chicken breast': {
        small: { grams: 100, description: 'Small (palm size)' },
        medium: { grams: 150, description: 'Medium (deck of cards)' },
        large: { grams: 200, description: 'Large (fist size)' }
      },
      'rice': {
        small: { grams: 75, description: 'Small (½ cup cooked)' },
        medium: { grams: 150, description: 'Medium (1 cup cooked)' },
        large: { grams: 225, description: 'Large (1.5 cups cooked)' }
      }
      // ... more foods
    };
  }

  getPortionOptions(foodName) {
    const normalized = foodName.toLowerCase();
    return this.commonPortions[normalized] || this.getDefaultPortions();
  }

  getDefaultPortions() {
    return {
      small: { grams: 100, description: 'Small' },
      medium: { grams: 150, description: 'Medium' },
      large: { grams: 200, description: 'Large' }
    };
  }

  calculateNutrition(baseNutrition, grams) {
    const multiplier = grams / 100; // Assuming base is per 100g
    return {
      calories: Math.round(baseNutrition.calories * multiplier),
      protein: Math.round(baseNutrition.protein * multiplier),
      carbs: Math.round(baseNutrition.carbs * multiplier),
      fat: Math.round(baseNutrition.fat * multiplier)
    };
  }
}
```

---

## Data Accuracy

### Validation and Verification

Validate nutrition data before storing:

```javascript
class NutritionValidator {
  validate(nutrition) {
    const errors = [];

    // Check required fields
    if (!nutrition.calories || nutrition.calories < 0) {
      errors.push('Invalid calories value');
    }

    // Check macros sum reasonably
    const macroCalories = (
      (nutrition.protein || 0) * 4 +
      (nutrition.carbs || 0) * 4 +
      (nutrition.fat || 0) * 9
    );

    if (Math.abs(macroCalories - nutrition.calories) > nutrition.calories * 0.2) {
      errors.push('Macros do not align with calories');
    }

    // Check for reasonable ranges
    if (nutrition.calories > 10000) {
      errors.push('Calories suspiciously high');
    }

    return {
      isValid: errors.length === 0,
      errors
    };
  }

  sanitize(nutrition) {
    return {
      calories: Math.max(0, Math.round(nutrition.calories || 0)),
      protein: Math.max(0, Math.round(nutrition.protein || 0)),
      carbs: Math.max(0, Math.round(nutrition.carbs || 0)),
      fat: Math.max(0, Math.round(nutrition.fat || 0)),
      fiber: Math.max(0, Math.round(nutrition.fiber || 0))
    };
  }
}
```

### Multiple Source Verification

Compare data from multiple sources:

```javascript
class MultiSourceVerification {
  constructor(apis) {
    this.apis = apis;
  }

  async verifyNutrition(query) {
    const results = [];

    for (const api of this.apis) {
      try {
        const data = await api.searchFood(query);
        if (data && data.length > 0) {
          results.push({
            source: api.name,
            nutrition: data[0]
          });
        }
      } catch (error) {
        console.warn(`${api.name} failed:`, error);
      }
    }

    return this.compareResults(results);
  }

  compareResults(results) {
    if (results.length === 0) {
      throw new Error('No data from any source');
    }

    if (results.length === 1) {
      return results[0];
    }

    // Calculate average and identify outliers
    const avgCalories = results.reduce((sum, r) => 
      sum + r.nutrition.calories, 0) / results.length;

    const verified = results.filter(r => 
      Math.abs(r.nutrition.calories - avgCalories) < avgCalories * 0.15
    );

    return {
      sources: verified.map(r => r.source),
      nutrition: verified[0].nutrition,
      confidence: verified.length / results.length
    };
  }
}
```

---

## Error Handling

### Graceful Degradation

Handle API failures gracefully:

```javascript
class ResilientNutritionAPI {
  constructor(primaryApi, fallbackApis = []) {
    this.primaryApi = primaryApi;
    this.fallbackApis = fallbackApis;
  }

  async searchFood(query, options = {}) {
    const maxRetries = options.maxRetries || 3;
    const retryDelay = options.retryDelay || 1000;

    // Try primary API with retries
    for (let i = 0; i < maxRetries; i++) {
      try {
        return await this.primaryApi.searchFood(query);
      } catch (error) {
        console.warn(`Attempt ${i + 1} failed:`, error);
        if (i < maxRetries - 1) {
          await this.delay(retryDelay * Math.pow(2, i)); // Exponential backoff
        }
      }
    }

    // Try fallback APIs
    for (const fallbackApi of this.fallbackApis) {
      try {
        console.log(`Trying fallback: ${fallbackApi.name}`);
        return await fallbackApi.searchFood(query);
      } catch (error) {
        console.warn(`Fallback ${fallbackApi.name} failed:`, error);
      }
    }

    // Return cached or default data
    return this.getDefaultData(query);
  }

  delay(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }

  getDefaultData(query) {
    return [{
      id: 'unknown',
      name: query,
      calories: 0,
      source: 'user_input',
      note: 'Could not retrieve nutrition data. Please verify manually.'
    }];
  }
}
```

### Rate Limit Handling

Implement rate limit detection and handling:

```javascript
class RateLimitHandler {
  constructor() {
    this.requestCounts = new Map();
    this.limits = new Map();
  }

  setLimit(apiName, requestsPerMinute) {
    this.limits.set(apiName, requestsPerMinute);
  }

  async checkRateLimit(apiName) {
    const now = Date.now();
    const key = `${apiName}_${Math.floor(now / 60000)}`; // Per minute
    
    const count = (this.requestCounts.get(key) || 0) + 1;
    this.requestCounts.set(key, count);

    const limit = this.limits.get(apiName) || 100;

    if (count > limit) {
      const waitTime = 60000 - (now % 60000);
      throw new Error(`Rate limit exceeded. Wait ${waitTime}ms`);
    }
  }

  async executeWithRateLimit(apiName, fn) {
    await this.checkRateLimit(apiName);
    return await fn();
  }
}
```

---

## Compliance and Attribution

### Data Attribution

Always attribute data to its source:

```javascript
class AttributedNutrition {
  constructor(nutrition, source) {
    this.nutrition = nutrition;
    this.source = source;
    this.timestamp = new Date();
  }

  getAttributionText() {
    const attributions = {
      'USDA': 'Data from USDA FoodData Central',
      'OpenFoodFacts': 'Data from Open Food Facts',
      'Edamam': 'Data from Edamam Food Database',
      'user_input': 'User-entered data'
    };

    return attributions[this.source] || `Data from ${this.source}`;
  }

  toJSON() {
    return {
      ...this.nutrition,
      _source: this.source,
      _attribution: this.getAttributionText(),
      _timestamp: this.timestamp.toISOString()
    };
  }
}
```

### Privacy Compliance

Implement GDPR/CCPA compliance:

```javascript
class PrivacyCompliantAPI {
  constructor(api) {
    this.api = api;
    this.userConsent = this.loadConsent();
  }

  async searchFood(query) {
    if (!this.userConsent.analytics) {
      // Don't track search queries
      return await this.api.searchFood(query);
    }

    // Track with consent
    this.trackSearch(query);
    return await this.api.searchFood(query);
  }

  loadConsent() {
    const stored = localStorage.getItem('userConsent');
    return stored ? JSON.parse(stored) : {
      analytics: false,
      personalData: false
    };
  }

  setConsent(consent) {
    this.userConsent = consent;
    localStorage.setItem('userConsent', JSON.stringify(consent));
  }

  trackSearch(query) {
    // Only if user consented
    if (this.userConsent.analytics) {
      // Send anonymized analytics
      console.log('Tracking search:', query);
    }
  }

  deleteUserData() {
    // GDPR right to be forgotten
    localStorage.removeItem('frequentFoods');
    localStorage.removeItem('userConsent');
    // ... delete other user data
  }
}
```

---

## Summary of Best Practices

| Category | Best Practice |
|----------|---|
| **Performance** | Use caching, batch requests, debounce searches |
| **Reliability** | Implement fallbacks, retry logic, error handling |
| **UX** | Provide autocomplete, barcode scanning, portion guides |
| **Accuracy** | Validate data, compare multiple sources |
| **Compliance** | Attribute data, respect privacy, handle consent |

---

**Last Updated:** January 2026  
**Maintained By:** Nutrition Library Team
