# Nutrition Library for Fitness Apps

A comprehensive guide and reference implementation for integrating food nutrition databases and APIs into fitness tracking applications. This project provides developers with curated options, integration patterns, and best practices for building calorie tracking features.

## Overview

This repository serves as a central hub for nutrition data integration, offering:

- **API Comparison Guide:** Detailed analysis of free and paid nutrition databases
- **Integration Examples:** Code samples for popular APIs (USDA, Open Food Facts, Edamam, etc.)
- **Data Schemas:** Standardized JSON schemas for nutrition data
- **Best Practices:** Implementation patterns for offline caching, barcode scanning, and more
- **Calculation Formulas:** BMR, MET-based calorie burn, and macro tracking

## Quick Start

### For App Developers

1. **Choose Your API:** Review the [API Comparison Guide](./docs/api-comparison.md) to select the best fit for your use case
2. **Integrate:** Use examples from the [Integration Guide](./docs/integration-guide.md)
3. **Reference Data:** Access standardized schemas in the [Data Schemas](./schemas/) directory

### For Team Members

- **Documentation:** Start with [CONTRIBUTING.md](./CONTRIBUTING.md) for contribution guidelines
- **API Keys:** See [Setup Instructions](./docs/setup.md) for configuring API credentials
- **Testing:** Run tests with `npm test` or `python -m pytest`

## Repository Structure

```
nutrition-library/
├── README.md                      # This file
├── docs/
│   ├── api-comparison.md          # Detailed API feature comparison
│   ├── integration-guide.md        # Step-by-step integration examples
│   ├── setup.md                   # Environment setup and configuration
│   ├── calculations.md            # Calorie burn and BMR formulas
│   └── best-practices.md          # Implementation patterns and tips
├── schemas/
│   ├── nutrition-data.json        # Standard nutrition data schema
│   ├── food-item.json             # Food item structure
│   └── user-metrics.json          # User profile and metrics schema
├── examples/
│   ├── usda-api-example.js        # JavaScript example for USDA API
│   ├── usda-api-example.py        # Python example for USDA API
│   ├── edamam-example.js          # JavaScript example for Edamam API
│   └── open-food-facts-example.py # Python example for Open Food Facts
├── tests/
│   ├── test-usda-integration.js
│   ├── test-edamam-integration.js
│   └── test-calculations.py
├── CONTRIBUTING.md                # Contribution guidelines
└── LICENSE                        # MIT License

```

## Key Features

### Supported APIs

| API | Type | Free Tier | Best For |
|-----|------|-----------|----------|
| USDA FoodData Central | REST | ✓ Unlimited | US/Canada, high accuracy |
| Open Food Facts | REST | ✓ No signup | Global, open-source |
| CalorieNinjas | REST | ✓ Limited | Natural language queries |
| Edamam | REST/SDK | ✓ 10K/month | Barcode scanning, recipes |
| FatSecret | REST/SDK | ✓ Limited | International, 56 countries |
| Nutritionix | REST | ✓ Limited | Restaurant data, barcodes |
| Passio Nutrition-AI | REST/SDK | Trial | Photo recognition |
| LogMeal | REST | Trial | Meal photo analysis |

### Calculation Support

- **Basal Metabolic Rate (BMR):** Harris-Benedict formula for men and women
- **Activity Calories:** MET-based calculations for 100+ activities
- **Macro Tracking:** Protein, carbohydrates, fats, and micronutrients
- **Daily Targets:** Customizable nutrition goals

## Getting Started

### Prerequisites

- Node.js 14+ or Python 3.8+
- Git
- API keys for desired services (most have free tiers)

### Installation

```bash
# Clone the repository
git clone https://github.com/yourusername/nutrition-library.git
cd nutrition-library

# Install dependencies (Node.js)
npm install

# Or install dependencies (Python)
pip install -r requirements.txt
```

### Configuration

1. Copy the example environment file:
   ```bash
   cp .env.example .env
   ```

2. Add your API keys:
   ```
   USDA_API_KEY=your_key_here
   EDAMAM_APP_ID=your_app_id
   EDAMAM_APP_KEY=your_app_key
   FATSECRET_CONSUMER_KEY=your_key
   FATSECRET_CONSUMER_SECRET=your_secret
   ```

3. See [Setup Instructions](./docs/setup.md) for detailed configuration

## Usage Examples

### JavaScript - USDA API

```javascript
const axios = require('axios');

async function searchFood(foodName) {
  const response = await axios.get('https://api.nal.usda.gov/fdc/v1/foods/search', {
    params: {
      query: foodName,
      pageSize: 10
    }
  });
  
  return response.data.foods;
}

// Usage
searchFood('chicken breast').then(foods => {
  console.log(foods);
});
```

### Python - Open Food Facts

```python
import requests

def get_product_by_barcode(barcode):
    url = f'https://world.openfoodfacts.org/api/v0/product/{barcode}.json'
    response = requests.get(url)
    return response.json()

# Usage
product = get_product_by_barcode('5000112126619')
print(product['product']['nutriments'])
```

See the [examples/](./examples/) directory for more detailed implementations.

## Documentation

- **[API Comparison Guide](./docs/api-comparison.md)** - Detailed feature and pricing comparison
- **[Integration Guide](./docs/integration-guide.md)** - Step-by-step integration instructions
- **[Calculation Formulas](./docs/calculations.md)** - BMR, MET, and macro calculations
- **[Best Practices](./docs/best-practices.md)** - Performance, caching, and UX tips
- **[Data Schemas](./schemas/)** - JSON schema definitions

## Contributing

We welcome contributions! Please see [CONTRIBUTING.md](./CONTRIBUTING.md) for guidelines on:

- Reporting issues
- Submitting pull requests
- Adding new API integrations
- Improving documentation

## Data Schemas

All nutrition data follows standardized JSON schemas for consistency:

- **[nutrition-data.json](./schemas/nutrition-data.json)** - Complete nutrition profile
- **[food-item.json](./schemas/food-item.json)** - Individual food item structure
- **[user-metrics.json](./schemas/user-metrics.json)** - User profile and daily metrics

## License

This project is licensed under the MIT License - see [LICENSE](./LICENSE) file for details.

## Support

- **Issues:** Report bugs or request features via [GitHub Issues](https://github.com/yourusername/nutrition-library/issues)
- **Discussions:** Join community discussions in [GitHub Discussions](https://github.com/yourusername/nutrition-library/discussions)
- **Documentation:** Check the [docs/](./docs/) directory for detailed guides

## Acknowledgments

- USDA FoodData Central for comprehensive US food data
- Open Food Facts for global crowdsourced nutrition data
- All API providers for excellent documentation and free tiers
- Contributors and community members

## Roadmap

- [ ] Add support for more international APIs
- [ ] Create mobile SDK wrappers (iOS/Android)
- [ ] Implement caching layer with Redis
- [ ] Add recipe analysis and meal planning
- [ ] Create web dashboard for data visualization
- [ ] Add AI-powered photo recognition integration

---

**Last Updated:** January 2026  
**Maintainer:** Your Team Name  
**Status:** Active Development
