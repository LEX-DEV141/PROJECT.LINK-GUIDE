# 📝 Coding Standards - LEX DEV

## 🎯 Tujuan Pembelajaran
- Memahami pentingnya consistent coding standards
- Menguasai naming conventions dan code formatting
- Mampu menulis clean, readable, dan maintainable code
- Memahami code review process dan quality metrics

## 📚 Prinsip Dasar Clean Code

### Why Coding Standards Matter?
- **Readability**: Code dibaca lebih sering daripada ditulis
- **Maintainability**: Mudah di-update dan di-debug
- **Collaboration**: Tim dapat memahami code satu sama lain
- **Quality**: Mengurangi bugs dan technical debt

### Clean Code Principles
1. **Meaningful Names** - Variable, function, dan class names yang descriptive
2. **Small Functions** - One function, one responsibility
3. **Clear Comments** - Explain WHY, not WHAT
4. **Consistent Formatting** - Follow team standards
5. **Error Handling** - Graceful error handling

## 🔧 General Coding Standards

### Naming Conventions

#### Variables & Functions
```javascript
// ✅ Good
const userAccountBalance = 1500;
const isUserActive = true;
function calculateTotalPrice(items) { ... }
function getUserById(userId) { ... }

// ❌ Bad
const uab = 1500;
const flag = true;
function calc(items) { ... }
function get(id) { ... }
```

#### Constants
```javascript
// ✅ Good
const MAX_RETRY_ATTEMPTS = 3;
const API_BASE_URL = 'https://api.lexdev.com';
const ERROR_MESSAGES = {
  INVALID_EMAIL: 'Email format is invalid',
  USER_NOT_FOUND: 'User not found'
};

// ❌ Bad
const max = 3;
const url = 'https://api.lexdev.com';
```

#### Classes & Components
```javascript
// ✅ Good (PascalCase)
class UserAccountManager { ... }
class EmailValidator { ... }
const UserProfile = () => { ... }

// ❌ Bad
class userAccountManager { ... }
class email_validator { ... }
```

### File & Folder Structure
```
src/
├── components/          # Reusable UI components
│   ├── common/         # Generic components
│   └── specific/       # Feature-specific components
├── pages/              # Page components
├── services/           # API calls & business logic
├── utils/              # Helper functions
├── hooks/              # Custom React hooks
├── constants/          # App constants
├── types/              # TypeScript type definitions
└── tests/              # Test files
```

### Function Design
```javascript
// ✅ Good - Single responsibility
function validateEmail(email) {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
}

function formatCurrency(amount, currency = 'IDR') {
  return new Intl.NumberFormat('id-ID', {
    style: 'currency',
    currency: currency
  }).format(amount);
}

// ❌ Bad - Multiple responsibilities
function processUser(userData) {
  // Validate email
  if (!userData.email.includes('@')) return false;
  
  // Save to database
  database.save(userData);
  
  // Send welcome email
  emailService.send(userData.email, 'Welcome!');
  
  // Log activity
  console.log('User processed');
}
```

## 🎨 Language-Specific Standards

### JavaScript/TypeScript
```typescript
// Interface naming
interface User {
  id: string;
  email: string;
  createdAt: Date;
}

// Enum naming
enum OrderStatus {
  PENDING = 'pending',
  PROCESSING = 'processing',
  COMPLETED = 'completed',
  CANCELLED = 'cancelled'
}

// Function with proper typing
async function fetchUserOrders(
  userId: string,
  status?: OrderStatus
): Promise<Order[]> {
  try {
    const response = await api.get(`/users/${userId}/orders`, {
      params: { status }
    });
    return response.data;
  } catch (error) {
    console.error('Failed to fetch user orders:', error);
    throw new Error('Unable to fetch orders');
  }
}
```

### React Components
```tsx
// ✅ Good Component Structure
interface UserCardProps {
  user: User;
  onEdit?: (user: User) => void;
  className?: string;
}

export const UserCard: React.FC<UserCardProps> = ({
  user,
  onEdit,
  className = ''
}) => {
  const handleEditClick = useCallback(() => {
    onEdit?.(user);
  }, [user, onEdit]);

  return (
    <div className={`user-card ${className}`}>
      <h3>{user.name}</h3>
      <p>{user.email}</p>
      {onEdit && (
        <button onClick={handleEditClick}>
          Edit User
        </button>
      )}
    </div>
  );
};
```

### CSS/SCSS Standards
```scss
// BEM Methodology
.user-card {
  padding: 1rem;
  border: 1px solid #ddd;
  border-radius: 0.5rem;

  &__header {
    display: flex;
    justify-content: space-between;
    margin-bottom: 1rem;
  }

  &__title {
    font-size: 1.25rem;
    font-weight: 600;
  }

  &--featured {
    border-color: #007bff;
    background-color: #f8f9ff;
  }
}
```

## ✅ Best Practices

### Code Organization
```javascript
// ✅ Good - Organized imports
// External libraries
import React, { useState, useEffect } from 'react';
import axios from 'axios';

// Internal utilities
import { formatDate, validateEmail } from '../utils';
import { API_ENDPOINTS } from '../constants';

// Types
import type { User, ApiResponse } from '../types';

// Components
import { Button, Input } from '../components/common';
```

### Error Handling
```javascript
// ✅ Good - Comprehensive error handling
async function createUser(userData) {
  try {
    // Validate input
    if (!userData.email || !validateEmail(userData.email)) {
      throw new ValidationError('Invalid email address');
    }

    // API call
    const response = await api.post('/users', userData);
    
    return {
      success: true,
      data: response.data,
      message: 'User created successfully'
    };
  } catch (error) {
    // Log error for debugging
    console.error('User creation failed:', error);
    
    // Return user-friendly error
    if (error instanceof ValidationError) {
      return { success: false, error: error.message };
    }
    
    return { 
      success: false, 
      error: 'Failed to create user. Please try again.' 
    };
  }
}
```

### Comments & Documentation
```javascript
/**
 * Calculates the discounted price based on user membership level
 * @param {number} originalPrice - The original price before discount
 * @param {string} membershipLevel - User's membership level (bronze, silver, gold)
 * @returns {number} The final price after applying discount
 */
function calculateDiscountedPrice(originalPrice, membershipLevel) {
  const discountRates = {
    bronze: 0.05,   // 5% discount
    silver: 0.10,   // 10% discount
    gold: 0.15      // 15% discount
  };

  const discountRate = discountRates[membershipLevel] || 0;
  return originalPrice * (1 - discountRate);
}

// TODO: Add support for premium membership level
// FIXME: Handle negative prices
// NOTE: This function assumes valid membershipLevel input
```

## 🔍 Code Review Checklist

### Before Submitting PR
- [ ] Code follows naming conventions
- [ ] Functions are small and focused
- [ ] Error handling is implemented
- [ ] No console.log in production code
- [ ] TypeScript types are properly defined
- [ ] Tests are written and passing
- [ ] Documentation is updated

### Reviewing Code
- [ ] Code is readable and understandable
- [ ] Logic is correct and efficient
- [ ] Edge cases are handled
- [ ] No code duplication
- [ ] Security considerations are met
- [ ] Performance implications are considered

## 🧪 Latihan Praktis

### Exercise 1: Code Refactoring
Refactor kode berikut agar mengikuti standards:
```javascript
// Bad code
function proc(d) {
  let r = '';
  for(let i = 0; i < d.length; i++) {
    if(d[i].active) {
      r += d[i].name + ', ';
    }
  }
  return r;
}
```

### Exercise 2: Error Handling
Implement proper error handling untuk function API call.

### Exercise 3: Component Structure
Create a React component yang mengikuti best practices LEX DEV.

## 🛠️ Tools & Automation

### ESLint Configuration
```json
{
  "extends": [
    "eslint:recommended",
    "@typescript-eslint/recommended",
    "prettier"
  ],
  "rules": {
    "no-console": "warn",
    "no-unused-vars": "error",
    "prefer-const": "error",
    "no-var": "error"
  }
}
```

### Prettier Configuration
```json
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 80,
  "tabWidth": 2
}
```

### Pre-commit Hooks
```json
{
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  "lint-staged": {
    "*.{js,ts,tsx}": ["eslint --fix", "prettier --write"]
  }
}
```

## 📖 Resources Tambahan
- [Clean Code by Robert Martin](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882)
- [JavaScript Style Guide - Airbnb](https://github.com/airbnb/javascript)
- [TypeScript Do's and Don'ts](https://www.typescriptlang.org/docs/handbook/declaration-files/do-s-and-don-ts.html)
- [React Best Practices](https://react.dev/learn/thinking-in-react)

## ❓ FAQ

**Q: Kapan boleh break naming convention?**
A: Hanya untuk external library integration yang membutuhkan specific naming.

**Q: Berapa panjang maksimal sebuah function?**
A: Idealnya tidak lebih dari 20 baris. Jika lebih, consider untuk dipecah.

**Q: Boleh menggunakan abbreviations?**
A: Hindari abbreviations kecuali yang sudah umum (id, url, api, etc).

## 🎯 Quality Metrics
- **Cyclomatic Complexity**: < 10 per function
- **Function Length**: < 20 lines
- **File Length**: < 300 lines
- **Test Coverage**: > 80%
- **Code Duplication**: < 5%

## 🏆 Checklist Keberhasilan
- [ ] Dapat menulis meaningful variable/function names
- [ ] Dapat mengorganisir code dengan clean structure
- [ ] Dapat implement proper error handling
- [ ] Dapat menulis clear comments dan documentation
- [ ] Dapat melakukan code review yang konstruktif
- [ ] Menggunakan automated tools (ESLint, Prettier)
