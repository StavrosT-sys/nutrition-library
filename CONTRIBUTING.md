# Contributing to Nutrition Library

Thank you for your interest in contributing to the Nutrition Library! This document provides guidelines and instructions for contributing.

## Code of Conduct

- Be respectful and inclusive
- Provide constructive feedback
- Focus on the code, not the person
- Help others learn and grow

## Getting Started

### Prerequisites

- Node.js 14+ or Python 3.8+
- Git
- Familiarity with the [API Comparison Guide](./docs/api-comparison.md)

### Setup Development Environment

```bash
# Clone the repository
git clone https://github.com/yourusername/nutrition-library.git
cd nutrition-library

# Install dependencies
npm install
# or
pip install -r requirements.txt

# Create a feature branch
git checkout -b feature/your-feature-name
```

## Types of Contributions

### 1. Adding New API Integrations

**Steps:**
1. Create a new file in `examples/` directory
2. Implement the API wrapper class
3. Add documentation in `docs/integration-guide.md`
4. Add tests in `tests/` directory
5. Update `docs/api-comparison.md` with API details

**Example:**
```javascript
// examples/new-api-example.js
class NewNutritionAPI {
  constructor(apiKey) {
    this.apiKey = apiKey;
    this.baseURL = 'https://api.example.com/v1';
  }

  async searchFood(query) {
    // Implementation
  }
}

module.exports = NewNutritionAPI;
```

### 2. Improving Documentation

**Areas for improvement:**
- Clarifying existing documentation
- Adding code examples
- Fixing typos or outdated information
- Adding new guides or tutorials

**Steps:**
1. Edit the relevant markdown file
2. Test that links and examples work
3. Submit a pull request with clear description

### 3. Adding Calculation Formulas

**Steps:**
1. Add formula to `docs/calculations.md`
2. Implement in JavaScript and Python
3. Add tests
4. Include usage examples

### 4. Improving Data Schemas

**Steps:**
1. Update JSON schema in `schemas/`
2. Add examples
3. Document changes in PR description
4. Ensure backward compatibility

### 5. Bug Fixes

**Steps:**
1. Create an issue describing the bug
2. Create a branch: `fix/issue-description`
3. Fix the bug with tests
4. Submit PR with reference to issue

## Development Workflow

### Creating a Pull Request

1. **Fork the repository** (if you don't have write access)

2. **Create a feature branch:**
   ```bash
   git checkout -b feature/descriptive-name
   ```

3. **Make your changes:**
   - Write clear, commented code
   - Follow existing code style
   - Add tests for new functionality

4. **Test your changes:**
   ```bash
   npm test
   # or
   python -m pytest
   ```

5. **Commit with clear messages:**
   ```bash
   git commit -m "Add: Brief description of changes"
   ```

   **Commit message format:**
   - `Add:` for new features
   - `Fix:` for bug fixes
   - `Docs:` for documentation
   - `Refactor:` for code improvements
   - `Test:` for test additions

6. **Push to your branch:**
   ```bash
   git push origin feature/descriptive-name
   ```

7. **Create a Pull Request:**
   - Clear title describing the change
   - Detailed description of what and why
   - Reference any related issues
   - Include screenshots/examples if applicable

### PR Checklist

Before submitting, ensure:
- [ ] Code follows project style guidelines
- [ ] Tests pass locally (`npm test` or `pytest`)
- [ ] New code includes tests
- [ ] Documentation is updated
- [ ] Commit messages are clear
- [ ] No breaking changes (or clearly documented)
- [ ] No sensitive information (API keys, etc.)

## Code Style Guidelines

### JavaScript

```javascript
// Use const/let, not var
const api = new NutritionAPI();

// Use arrow functions
const result = foods.map(f => f.calories);

// Use async/await
async function getData() {
  try {
    const data = await api.search('chicken');
    return data;
  } catch (error) {
    console.error('Error:', error);
  }
}

// Comment complex logic
// Calculate total macros from food array
const totalMacros = foods.reduce((sum, food) => ({
  protein: sum.protein + food.macros.protein_g,
  carbs: sum.carbs + food.macros.carbs_g,
  fat: sum.fat + food.macros.fat_g
}), { protein: 0, carbs: 0, fat: 0 });
```

### Python

```python
# Follow PEP 8
import requests
from typing import Dict, List

class NutritionAPI:
    """Nutrition API wrapper."""
    
    def __init__(self, api_key: str):
        self.api_key = api_key
        self.base_url = 'https://api.example.com/v1'
    
    async def search_food(self, query: str) -> List[Dict]:
        """Search for foods by name.
        
        Args:
            query: Food name to search
            
        Returns:
            List of food items with nutrition data
        """
        try:
            response = requests.get(
                f'{self.base_url}/search',
                params={'q': query}
            )
            response.raise_for_status()
            return response.json()
        except requests.RequestException as e:
            raise ValueError(f'API request failed: {e}')
```

## Testing

### JavaScript Testing

```javascript
// tests/example.test.js
const assert = require('assert');
const API = require('../examples/api-example');

describe('API Tests', () => {
  let api;

  beforeEach(() => {
    api = new API('test-key');
  });

  it('should search for foods', async () => {
    const results = await api.searchFood('chicken');
    assert(Array.isArray(results));
    assert(results.length > 0);
  });

  it('should extract nutrition data', () => {
    const food = { calories: 100, protein: 20 };
    const nutrition = api.extractNutrition(food);
    assert.equal(nutrition.calories, 100);
  });
});
```

### Python Testing

```python
# tests/test_example.py
import unittest
from nutrition_api import NutritionAPI

class TestNutritionAPI(unittest.TestCase):
    def setUp(self):
        self.api = NutritionAPI('test-key')

    def test_search_food(self):
        results = self.api.search_food('chicken')
        self.assertIsInstance(results, list)
        self.assertGreater(len(results), 0)

    def test_extract_nutrition(self):
        food = {'calories': 100, 'protein': 20}
        nutrition = self.api.extract_nutrition(food)
        self.assertEqual(nutrition['calories'], 100)

if __name__ == '__main__':
    unittest.main()
```

## Documentation Standards

### Markdown Format

```markdown
# Section Title

Brief introduction paragraph explaining the section.

## Subsection

More detailed explanation with examples.

### Code Example

```javascript
// Code here
```

### Important Notes

> Use blockquotes for important information or warnings

| Column 1 | Column 2 |
|----------|----------|
| Data 1   | Data 2   |

### Lists

- Item 1
- Item 2
  - Nested item
  - Another nested item
```

### API Documentation Template

```markdown
## API Name

**Overview:** Brief description

**Key Features:**
- Feature 1
- Feature 2

**Pricing:** Free tier details and paid options

**Coverage:** Geographic and product coverage

**Integration:**
```
Base URL: https://...
Endpoints:
  - GET /endpoint - Description
```

**Pros:**
- Pro 1
- Pro 2

**Cons:**
- Con 1
- Con 2

**Best For:** Use cases

**Example:**
```javascript
// Code example
```
```

## Reporting Issues

### Bug Report Template

```markdown
**Description:** Clear description of the bug

**Steps to Reproduce:**
1. Step 1
2. Step 2
3. Step 3

**Expected Behavior:** What should happen

**Actual Behavior:** What actually happens

**Environment:**
- OS: 
- Node/Python version:
- Library version:

**Attachments:** Screenshots, logs, etc.
```

### Feature Request Template

```markdown
**Description:** Clear description of the feature

**Use Case:** Why this feature is needed

**Proposed Solution:** How it could be implemented

**Alternatives:** Other approaches considered

**Additional Context:** Any other relevant information
```

## Review Process

1. **Automated Checks:**
   - Tests must pass
   - Code style must be valid
   - No security issues

2. **Code Review:**
   - At least one maintainer review
   - Feedback on code quality
   - Suggestions for improvement

3. **Approval:**
   - Changes approved by maintainer
   - All feedback addressed
   - Ready to merge

## Release Process

Releases follow semantic versioning (MAJOR.MINOR.PATCH):

- **MAJOR:** Breaking changes
- **MINOR:** New features (backward compatible)
- **PATCH:** Bug fixes

## Questions?

- **Documentation:** Check [README.md](./README.md) and [docs/](./docs/)
- **Issues:** Search existing [GitHub Issues](https://github.com/yourusername/nutrition-library/issues)
- **Discussions:** Join [GitHub Discussions](https://github.com/yourusername/nutrition-library/discussions)

## License

By contributing, you agree that your contributions will be licensed under the MIT License.

---

Thank you for contributing to the Nutrition Library! 🎉
